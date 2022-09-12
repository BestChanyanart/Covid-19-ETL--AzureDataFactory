# Data Engineering by Azure Data Factory
It is a part of Udemy course: [Azure Data Factory For Data Engineers - Project on Covid19](https://www.udemy.com/course/learn-azure-data-factory-from-scratch/)
I enrolled this course the practice how to use Azure Service, mainly Azure Data Factory. 

This project is about COVID-19 data, focusing on the ETL process with Azure Data Factory, but I also use other tools to do the Pipeline, following:  

The required tool: 
- Azure Data Factory (ADF)
- Storage Account 
- Azure Data Lake Gen2 (ADLS)
- Azure SQL database 
- Azure Databrick 
- Azure HD Insight
- Power BI

![workflow](https://user-images.githubusercontent.com/63108802/189586397-e0ab64c0-c0f0-456b-80d9-a53f02a78587.PNG)

## WorkFlow

<B>Ingest/Extarct data from Sorce</B>

We ingested data from 2 sources by using Azure Data Factory, create Link Service to each sources, then create the dataset for Source and Sink. 

1. ECDC Covid data - Ingest these file directly to ADLS
 - [Covid-19 Cases and Deaths data](https://github.com/cloudboxacademy/covid19/raw/main/ecdc_data/cases_deaths.csv)
 - [Covid-19 Hospital Admission data](https://github.com/cloudboxacademy/covid19/raw/main/ecdc_data/hospital_admissions.csv)
 - [Covid-19 Testing data](https://github.com/cloudboxacademy/covid19/raw/main/ecdc_data/testing.csv)
 
![Copy activity 2](https://user-images.githubusercontent.com/63108802/189594590-77d1ffc3-229a-41d7-9aeb-a3c505d78c23.PNG)


2. Population data
  - Population data 
  The data file is zipped. Put the file into Blob Storage on Storage Account, then ingest the file to ADLS through Data Factory. 
![Copy activity](https://user-images.githubusercontent.com/63108802/189594063-b219e7b2-e1b4-4698-be30-74d0ec2de029.PNG)


<B>Tranform data</B>
Transform data through 
1. Data Flow in Data Factory
- Focus on <B>Cases and Deaths data</B>
![Transform Cases and Deaths](https://user-images.githubusercontent.com/63108802/189596367-dd1403c6-0043-45ba-93cc-b83f55220c83.PNG)

![Data Flow1](https://user-images.githubusercontent.com/63108802/189597861-9205aee2-003d-43d2-b62f-44d6f45dd3e6.PNG)


- Focus <B>Hospital Admission (divided by Daily and Weekly)</B>
<B>Daily</B>
![Transform Hospital Admission daily](https://user-images.githubusercontent.com/63108802/189596624-cdf5a77b-e7a0-4be4-ac01-c9fde8d446af.PNG)


<B>Weekly</B>
![Transform Hospital Admission weekly](https://user-images.githubusercontent.com/63108802/189596652-dd358aa0-e3da-4cfa-a0e5-b6e42e5d4691.PNG)
![Data Flow2](https://user-images.githubusercontent.com/63108802/189597891-6e829bb3-8295-4bb8-8100-c62c0b1ee998.PNG)


3. Azure HD Insight
Transform data with Azure HD Insight, Firstly, To create HDInsight Cluster, upload the Hive script. 
![testing data](https://user-images.githubusercontent.com/63108802/189602900-5e1e4f5b-78f2-4a42-9e13-65b9b5621e6f.PNG)


4. Azure Databrick 
![transform Population data](https://user-images.githubusercontent.com/63108802/189602941-acda3cca-3357-437f-83a6-9b4d3f297444.PNG)



<B>Load data</B> to Azure SQL databases 
Create the Notebook and define Schema, upload into Azure SQL. Then load the data through Azure Data Factory.



