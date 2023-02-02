---
layout: post
title: Creating a Power BI Dashboard
image: "img/posts/AW_Dashboard_Exec_Summary.jpg"
tags: [PowerBI, Data Analysis]
---

This post is a ***deep dive*** into the steps of creating a dashboard using Microsoft Power BI. From pulling in the data, preparing it, creating a data model, to finalizing the reports each process is a fundamental step to making a complete dashboard that gives useful and most importantly correct information. This specific dataset is about Adventure Work Cycles that was provided by [Maven Analytics](https://www.mavenanalytics.io/).  

# Table of contents

- [00. Project Overview](#overview-main)
    - [The Story](#overview-story)
    - [The Objective](#overview-objective)
- [01. Connecting & Shaping Data](#connecting-overview)
- [02. Creating a Data Model](#model-overview)
- [03. Calculated Fields & Measures using DAX](#using-DAX)
- [04. Visualizing Data with Reports](#visualizing-reports)
    - [Executive Summary](#executive-summary)
    - [Product Detail](#product-detail)
    - [Customer Detail](#customer-detail)
    - [Q&A](#q&a)


___

# Project Overview  <a name="overview-main"></a>

### The Story <a name="overview-story"></a>

For this project, I was hired by Adventure Works Cycles, a global manufacturing company, to design and deliver an end-to-end business intelligence solution. Adventure Works Cycles needs a way to track KPIs (sales, revenue, profit, and returns), and identify high-value customers. I was provided a folder containing raw csv files and need to make sure the dashboard is able to refreah and update when the new Sales and Return files are uploaded to the folder.  

<br>

### The Objective <a name="overview-objective"></a>

- Connect and transform the raw data
- Build a relational data model
- Create new calculated columns and DAX measures
- Design and interactive report to analyze and visulaize the data


___

# Connecting & Shaping Data  <a name="connecting-overview"></a>

Connecting data to Power BI can happen many different ways. You can use the 'Get Data' button in the Home tab, you can use the 'File > Get Data' option, and you can even get data through the Transform Data window.

In this project I was given 10 csv files: 
- Lookup Tables
    1. AdventureWorks_Calendar
    2. AdventureWorks_Customers
    3. AdventureWorks_Product_Categories
    4. AdventureWorks_Product_Subcategories
    5. AdventureWorks_Products
    6. AdventureWorks_Territories
- Fact Tables
    1. AdventureWorks_Returns
    2. AdventureWorks_Sales_2015
    3. AdventureWorks_Sales_2016
    4. AdventureWorks_Sales_2017

<br>

### Connecting the Data

The first 6 files are considered Lookup Tables and will not be updated on a regular basis. Therefore, they can be uploaded using one of the many 'Get Data' ways, browse to find the csv file, and open it. Since I want to update the Sales and Returns files, a separate folder was created for each titled, AW_Sales and AW_Returns. The three Sales files were put into the AW_Sales folder and the Returns file was put into the AW_Returns folder. Then again use 'Get Data' but this time find the Folder option. A pop-up box will appear asking for the Folder Path. 

Getting data into Power BI is very easy even when directly using a database. 

<br>

### Shaping the Data

After opening each one of the Lookup Table csv files, a preview of the data will appear. This is where I can begin to shape the data. Generally, from this point I will select the 'Transform Data' button to shape the date. There are times when the data was cleaned using other sources such as Python or Excel and this step is not neccessary but that is rare and I almost always select 'Transform Data'. 

The Power Query Editor will open and there is so much that can be done to transform the data. The most important things to note are the ability to change the name of the table, change the types of data each column is selected as, unneeded columns can be deleted, calculated columns can be created, and so, so much more. There are 2 very important points to mention here: 
   1. At the top there are different tabs you can select. Next to Home there is Transform and Add Columns. In each of these tabs many of the options that can be done to make changes  to the data are the same. Why would there be two tabs with the same actions?
       - The Transform Tab will take the current column and change the data within that column. 
           - Ex. If a column is filled with the months and I want to abbreviate the months with the first three characters, if I was to use the Transform tab, I would change the column from the long form to the short form. That column would go from looking like this:
           
                | Long Month | Column B |
                | :---       |     ---: | 
                | January    | some     |
                | February   | data     |
                | March      | here     |
                
           - to looking like this:
           
                | Short Month | Column B |
                | :---        |     ---: | 
                | Jan         | some     |
                | Feb         | data     |
                | Mar         | here     |
       - The Add Column will leave the current column as is and will add a column to the end of the table with the new information looking like this:
       
            | Long Month | Column B | Short Month |
            | :---       |  :---:   |        ---: |
            | January    | some     | Jan         |
            | February   | data     | Feb         |
            | March      | here     | Mar         |
   
   2. On the right hand of the screen there is a list of Applied Steps. If at anytime I make a change I don't want to keep, I can just delete the step from this box and the data will go back to the way it was before that step. In the case of the Returns and Sales files, the changes that I make to these files will be saved and applied to the new files that are uploaded from the respective folders. I can also just click on the first step and walk through the changes that were made deleting any unneccessary steps. 
        
Shaping the data from the folders is the same except when I select the folder path, instead of seeing a preview of the data in the file, there is a list of the files from the folder. In this case I always chose to the 'Transform Data' button. This will again lead to the Power Query Editor with this same list but there is a button with two arrows pointing downwards in the first column. After pushing that double arrow button the pop up with the preview will come up, select OK, and now the files will be loaded and the data can be manipulated.      
               
___

<br>

# Creating a Data Model  <a name="model-overview"></a>


___

<br>

# Calculated Fields & Measures using DAX <a name="using-DAX"></a>


___

<br>

# Visualizing Data with Reports <a name="visualizing-reports"></a>
