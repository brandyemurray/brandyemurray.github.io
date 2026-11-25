---
title: "Postmortem: the powder-day outage that didn't have to happen"
kind: writing
summary: >-
  A busy day and a slow payment provider were each harmless. Together, 100 payments
  waited for 8 connections and checkout failed for 19 minutes. A blameless
  postmortem: timeline, capacity math and what I'd fix first.
image: "/img/posts/writing/postmortem-cover.png"
tags: [Observability, SRE, Snowflake, OpenTelemetry]
---

On a simulated powder day, checkout at a fictional ski resort failed for about 19 minutes, and two settings that were each harmless on their own caused it.

This was a game day: a failure I triggered on purpose in my [observability lab](/2026/11/24/observability-on-snowflake.html), to see whether the platform would catch it and explain it. Summit Peak Resort, its guests and their card numbers are all synthetic. The failure pattern is not: it shows up in real payment, booking and ticketing systems every busy season, so it's worth writing up the way a real team would: a blameless postmortem, focused on how the system behaved, not on who to blame.

## Summary

At 1:19 p.m. Mountain time, traffic jumped to powder-day levels (about 30 checkouts a second, roughly 8 times normal) and the outside payment provider slowed from about 120 ms to about 600 ms per charge. The payments service talks to that provider through a pool of 8 connections. The pool ran out, requests queued behind it, and checkout for lift tickets and rentals failed almost completely.

Snowflake opened incidents for three services about 2 minutes after the first failures. Errors stopped within a minute of the provider recovering at 1:39 p.m., while traffic stayed high. The incidents closed themselves at 1:44 p.m. after five healthy minutes.

## Impact

For about 19 minutes, most guests couldn't buy a lift ticket or rent gear.

- **About 34,000 of 44,800 checkout requests failed** (76%) across lift tickets and rentals. In the worst five-minute window, lift tickets failed 94% of the time and rentals 91%, against an objective of 2%.
- **Checkout was slow even when it worked.** The 95th-percentile response time went from about 240 ms to 3 to 4.6 seconds.
- **About 14,800 charges went through after the guest had already been told checkout failed.** Payments succeeded on more requests than its callers did, because the callers stopped waiting first. In a real system, that means duplicate purchases, refunds and support calls.
- **No guest data was exposed.** Emails are hashed and card numbers masked before telemetry leaves the cluster, and a check of 6,217 log records in Snowflake found neither.

## Timeline (Mountain time, November 24)

- **1:19 p.m.** Traffic jumps to powder-day levels and the payment provider slows to about 600 ms per charge.
- **1:20 p.m.** Payment requests waiting for a connection jump from about 1 to about 100. Lift ticket and rental error rates pass 90%.
- **1:21 p.m.** Snowflake's alert opens incidents for payments, lift tickets and rentals, about 2 minutes after the first failures.
- **1:28 p.m.** The AI summary of the payments incident is first requested. It misses the cause (see below).
- **1:39 p.m.** The provider recovers. Errors fall to near zero within a minute; traffic stays at powder-day levels.
- **1:44 p.m.** All three incidents resolve on their own after five healthy minutes.

## Root cause

Neither change would have caused an outage alone. Together they asked the connection pool for more than twice what it could give.

**The capacity math.** A pool of 8 connections can complete about 8 ÷ (time per charge) charges a second:

- **Normal provider, about 120 ms:** room for about 67 charges a second. A powder day's 30 a second fits easily.
- **Slow provider, about 600 ms:** room for about 13 a second. A normal day's 4 a second still fits.
- **Slow provider on a powder day:** 30 requests a second arrive and about 13 can be served. The other 17 join a queue, every second.

**Why almost everyone failed, not just the extra 17.** The queue was first in, first out, and it grew every second. Within moments, the wait at the back was longer than the 2 seconds lift tickets and rentals were willing to wait. From then on, nearly every request timed out while it waited, including the ones that would eventually have been served. That's why checkout went from working to 94% failures instead of slowing down gradually.

**Why guests were charged after being told no.** Lift tickets and rentals give up after 2.0 seconds. Payments waits up to 2.5 seconds for a free connection, then makes the charge. So a request could time out for the guest and still succeed at payments a moment later. The service doing the work was more patient than the service waiting for it.

**The trigger was outside our control; the mechanism was ours.** A provider slowing down on a busy day is normal. Turning that into a full outage, and charging guests who saw an error, came from our pool size, our queue and our timeouts.

## What went well

- **Detection took about 2 minutes, with no one watching.** A Snowflake alert checks each service against its error-rate objective every minute and opened incidents on its own.
- **The cause was visible on one chart.** Saturation (payment requests competing for the 8 connections) jumped from about 1 to about 100 at the exact moment errors did. Measuring how full a resource is, not just how often requests fail, is what pointed straight at the pool.
- **The error rates were accurate.** Only failed and slow traces are all kept; healthy ones are sampled at 10%. Counting errors from those stored traces would have overstated a normal day's error rate nearly 10 times (1.91% instead of 0.2%). The alert counted every request before sampling, so it fired on real numbers.
- **Recovery confirmed the diagnosis.** When the provider sped up, errors dropped within a minute while traffic stayed high. The surge alone was never the problem.
- **Incidents closed themselves** once services had been healthy for five minutes, so nothing sat open by mistake.

## What didn't go well

- **Nothing warned us before guests felt it.** The alert watches errors, so it fired after checkout was already failing. Pool saturation is the earlier signal, and nothing alerted on it.
- **Payments failed slowly instead of quickly.** When the pool was full, payments kept every request waiting up to 2.5 seconds, long after there was any chance of serving it. Failing fast would have freed guests and callers sooner and stopped the late charges.
- **The timeouts disagreed** (2.0 seconds for callers, 2.5 for payments), which turned a slowdown into charges guests were told had failed.
- **The first AI summary was confidently wrong.** I asked Cortex AI to draft the incident summary from the raw evidence. It never mentioned the connection pool, put the start time 12 minutes early, and invented a failure count. After I computed the key facts in SQL and gave it a service catalog with each service's dependencies and capacity, the second version named the exhausted pool and found the log line that marked the exact moment the provider changed. It still blamed the provider alone and missed the eightfold traffic surge, which is half the cause. A fast first draft is useful; a person who knows the system still has to check it.

## Action items

1. **Alert on saturation, not just errors.** Page when payment requests waiting for a connection stay above the pool size for a minute. (Owner: Payments)
2. **Fail fast when the pool is full.** Cap the queue at about the pool size and return an immediate, friendly "try again" instead of making everyone wait. (Owner: Payments)
3. **Add a circuit breaker on the provider.** When its response time passes a threshold, stop sending it new charges for a short time and show guests a clear message. (Owner: Payments)
4. **Make the timeouts agree.** Each service should give up before the one calling it does, and every charge should carry an idempotency key so a late or retried charge can never bill a guest twice. (Owners: Payments and Guest Commerce)
5. **Plan capacity for a slow provider on a busy day.** Size the pool, or scale payments out, for peak traffic at degraded provider speed, within the provider's own connection limits. (Owner: Payments)
6. **Run this game day every season**, before the first big storm, and after any change to payments. (Owner: SRE)
7. **Keep the AI summary on a short leash.** Keep feeding it computed facts, add a rule to compare traffic before and during, and have the on-call engineer review it before it's shared. (Owner: SRE)

## The lesson

Most outages aren't one broken thing. They're two normal things, a busy day and a slow partner, meeting a limit nobody had written down. Observability's job is to make that limit visible before the busy day does: measure how full things are, not only how often they fail, and practice the bad day on purpose.

The full build, including the lab, the Snowflake scripts, the runbook and the decision records, is in the [case study](/2026/11/24/observability-on-snowflake.html) and on [GitHub](https://github.com/brandyemurray/observability-on-snowflake).
