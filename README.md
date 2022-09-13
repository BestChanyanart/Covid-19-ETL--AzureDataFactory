# Data Engineering by Azure Data Factory
It is a part of Udemy course: [Azure Data Factory For Data Engineers - Project on Covid19](https://www.udemy.com/course/learn-azure-data-factory-from-scratch/)
I enrolled this course because I want to practice how to use Azure Service, mainly Azure Data Factory - Data Flow / Pipeline / Trigger. 

This project is about COVID-19 data, focusing on the ETL process with Azure Data Factory, but also use other tools in the Pipeline, following:  

The required tool: 
- Azure Data Factory (ADF)
- Storage Account 
- Azure Data Lake Gen2 (ADLS)
- Azure SQL database 
- Azure Databrick 
- Azure HD Insight
- Power BI


## WorkFlow
![workflow](https://user-images.githubusercontent.com/63108802/189586397-e0ab64c0-c0f0-456b-80d9-a53f02a78587.PNG)


## Ingest/Extract data from Sorce

We ingested data from 2 sources by using Azure Data Factory, create Link Service to each sources, then create the dataset for Source and Sink. 

1. ECDC Covid data - Ingest these file directly to ADLS
 - [Covid-19 Cases and Deaths data](https://github.com/cloudboxacademy/covid19/raw/main/ecdc_data/cases_deaths.csv)
 - [Covid-19 Hospital Admission data](https://github.com/cloudboxacademy/covid19/raw/main/ecdc_data/hospital_admissions.csv)
 - [Covid-19 Testing data](https://github.com/cloudboxacademy/covid19/raw/main/ecdc_data/testing.csv)
 
 ![Copy activity 3](https://user-images.githubusercontent.com/63108802/189801820-28f39bac-41cd-48c7-a59a-42fe4ff3750b.PNG)


2. Population data
  - Population data 
  The data file is zipped. Put the file into Blob Storage on Storage Account, then ingest the file to ADLS through Data Factory. 
![Copy activity 2](https://user-images.githubusercontent.com/63108802/189594590-77d1ffc3-229a-41d7-9aeb-a3c505d78c23.PNG)



## Tranform data
Transform data through 
1. Data Flow in Data Factory
2. Azure HD Insight
3. Azure Databrick 

---

1. Data Flow 

- <B>Cases and Deaths data</B>
![Transform Cases and Deaths](https://user-images.githubusercontent.com/63108802/189596367-dd1403c6-0043-45ba-93cc-b83f55220c83.PNG)

![Data Flow1](https://user-images.githubusercontent.com/63108802/189597861-9205aee2-003d-43d2-b62f-44d6f45dd3e6.PNG)

--Action-- 
1. Source Transformation: Add Souce File "Cases and Deaths" dataset, and edit Schema 
2. Filter Transformation: Filter to get the data only Europe country 
````
         continent == 'Europe' && not(isNull(country_code))
````      
3. Select Transformation: Select only relevant columns and remove irrelavant columns
4. Pivot Transformation: To Transform data format from column "indicator" to separate into 2 columns (confirmed cases / deaths)

    4.1 Group By: To groupby columns
    
    4.2 Pivot Key: To identify column that we want to transform which is "indicator" columns and value "confirmed cases" , "deaths"
    
    4.3 Pivot Columns: To count number of each "confirmed cases" and "deaths" 
````    
         sum(daily_count) = count 
````         
5. LookUp Transformation: To Join 2 dataset with Primary key ( Join with "CountryLookup" dataset)
6. Select Transformation: To Select only Relevant columns and reorder columns. Last recheck before Sink file
7. Sink Transformation: To identify the destination of dataset to Sink. 

--Create Pipeline--
1. Pipeline to Run the dataflow
2. Pipeline to identify the sink destination, which is "Azure SQL databases", and need to identify linkservice

---

- <B>Hospital Admission (split to Daily and Weekly)</B>

We split the file into 2 dataset which are 'Daily' and 'Weekly' because the original file put the number of cases both weekly and daily into one file. It's really difficult to analyze data when we use PowerBI tool. 

<I>Daily</I>

This is the columns that we are going to adjust, and what column that we expected to be. 
![Transform Hospital Admission daily](https://user-images.githubusercontent.com/63108802/189596624-cdf5a77b-e7a0-4be4-ac01-c9fde8d446af.PNG)

----

<I>Weekly</I>

This is for weekly dataset that we need it to be. 
![Transform Hospital Admission weekly](https://user-images.githubusercontent.com/63108802/189596652-dd358aa0-e3da-4cfa-a0e5-b6e42e5d4691.PNG)

This is the "Data Flow" which created on Data Factory, and then create Pipeline to run the Data Flow. 
![Data Flow2](https://user-images.githubusercontent.com/63108802/189597891-6e829bb3-8295-4bb8-8100-c62c0b1ee998.PNG)


--Action-- 
1. Source Transformation: Add Souce File "Hospital Admission" dataset, and edit Schema 
2. Select Transformation: Select only relevant columns and remove irrelavant columns
3. Lookup Transformation: To Join 2 dataset with Primary key "country" name ( Join with "SourceLookup" dataset) to get the 2and3 digit of Country code columns
4. Select Transformation: Select only relevant columns and remove irrelavant columns
5. Conditional-Split Transformation: To identify the Condition for Split "Weekly" and "Daily" 
````
            Weekly - indicator == 'Weekly new ICU admissions per 100k' || indicator == 'Weekly new hospital admissions per 100k'
            Daily - (default: Rows that do not meet any condition will use this output stream) 
````            
6. (Only Weekly) Join Transformation: To do Inner Join with another dataset "DimDate" to get start date and end date of week. 

    6.1 Derived-columns: To transform format of data (To be like this -  2016-W02  )
````    
            ecdc_year_week = year + '-W' + lpad(week_of_year, 2, '0')   
````

     6.2 Aggragate: To find the start date and end date of week 
    
````    
            week_start_date = min(date)            
            week_end_date = max(date)
````      

7. (Weekly) Pivot Transformation: To Transform data format from column "indicator" to separate into 2 columns 

    7.1 Group By: To groupby columns
    
    7.2 Pivot Key: To identify column that we want to transform which is "indicator" columns and value 
````    
            "Weekly new ICU admissions per 100k" , "Weekly new hospital admissions per 100k"
````   

     7.3 Pivot Columns: To count number of each
    
````    
            sum(indicator) = count 
````         
7. (Daily) Pivot Transformation: To Transform data format from column "indicator" to separate into 2 columns 

    7.1 Group By: To groupby columns
    
    7.2 Pivot Key: To identify column that we want to transform which is "indicator" columns and value 
````    
             "Daily hospital occupancy" , "Daily ICU occupancy"
````    

     7.3 Pivot Columns: To count number of each
    
````
             sum(indicator) = count 
````
8. Select Transformation: To Select only Relevant columns and reorder columns. Last recheck before Sink file
7. Sink Transformation: To identify the destination of dataset to Sink. 

--Create Pipeline--
1. Pipeline to Run the dataflow of each Daily and Weekly
2. Pipeline to identify the sink destination, which is "Azure SQL databases", and need to identify linkservice

---
2. Azure HD Insight

Transform "Testing data" with Azure HD Insight, Firstly, To create HDInsight Cluster, upload the Hive script and Transform it through Data Factory. 
![testing data](https://user-images.githubusercontent.com/63108802/189602900-5e1e4f5b-78f2-4a42-9e13-65b9b5621e6f.PNG)


----
3. Azure Databrick 

To create Databrick Cluster, Mount storage to ADLS, and Transform the Population data with Notebook. 
![transform Population data](https://user-images.githubusercontent.com/63108802/189602941-acda3cca-3357-437f-83a6-9b4d3f297444.PNG)

- [Mount_storage](https://github.com/BestChanyanart/Covid-19-ETL--AzureDataFactory/raw/main/databrick_notebook/mount_storage.py)
- [Transform with PySpark](https://github.com/BestChanyanart/Covid-19-ETL--AzureDataFactory/raw/main/databrick_notebook/transform_population_data.py)

## Load data to Azure SQL databases 

Create the Notebook and define Schema, upload into Azure SQL. Then load the data through Azure Data Factory.

## Summary services for this dataset that I created.

![all service](https://user-images.githubusercontent.com/63108802/189931516-a3d858e8-ce07-4409-846c-9f6c2016c0d8.jpg)




