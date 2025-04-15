# Data Engineering Pipeline: Yellow Taxi Trips NYC 2024 — Microsoft Fabric Project

This project is a comprehensive solution for creating an end-to-end pipeline for data processing, transformation and visualization using Microsoft Fabric, including Lakehouse, Data Factory, Dataflows Gen2, SQL Stored Procedures and Power BI.

Thanks to [Mr.Malvik Vaghadia](udemy.com/course/microsoft-fabric-the-ultimate-guide) for the project inspiration.

The project demonstrates the construction of a dynamic process for loading and preparing data with automatic parameterization and updating, including working with metadata and building a final report in Power BI.



## Technology Stack
-  Microsoft Fabric

-  Data Factory Pipelines

-  Lakehouse & OneLake

-  Dataflows Gen2

-  T-SQL & Stored Procedures

-  Parquet Files

-  Dynamic Content / Parameters

-  Power BI

## Solution architecture

![image](https://github.com/user-attachments/assets/e1ea2f37-13cd-4bd7-b556-c93c96e9a73c)

# Project Implementation Stages 

## Setting Up the Development Environment & Loading Initial Data

- Created a workspace in Microsoft Fabric, set up a Lakehouse and a Data Warehouse, uploaded NYC taxi trip Parquet files, and used a lookup pipeline to transfer reference data into a staging table.

| Created a Lakehouse to store raw NYC taxi trip data in Parquet format. This serves as the initial storage layer for further processing. |
| ----------- |
![image](https://github.com/user-attachments/assets/e62db76c-001e-44f8-9972-8b65b8245619)
![image](https://github.com/user-attachments/assets/ab7bf7a2-2e62-4154-83d5-3a107b06c014)


|Transferred lookup zone data from Lakehouse using a pipeline into a staging table for enriched processing. |
| ----------- |
![image](https://github.com/user-attachments/assets/94ce3c0b-2dd5-4a6e-8cc4-dcb32972f700)
![image](https://github.com/user-attachments/assets/cd4e5b0e-d58b-44da-9175-18d592c0a6ea)
![image](https://github.com/user-attachments/assets/050945b1-daa5-4900-a7bd-1abf65b4c2ef)

- Set up a Data Warehouse and loaded raw data into staging tables. Built a Data Factory pipeline with dynamic variables to automate file ingestion and streamline data loading.
![image](https://github.com/user-attachments/assets/d8bd3c77-bd00-4ec2-a2d3-6b656e7b1ac7)
![image](https://github.com/user-attachments/assets/5cd41263-1b00-436a-a7e3-ab671bfb478a)
![image](https://github.com/user-attachments/assets/9bb2a6de-b836-464e-928e-9818f35dfd3b)

## Data Cleaning & Filtering

- Identified and removed invalid records (e.g., incorrect dates) by implementing a stored procedure to ensure data quality before further processing.

|Identifying and filtering incorrect records|
| ----------- |
![image](https://github.com/user-attachments/assets/f3f3df97-05ea-4e47-baa9-f4b6f9e7dae8)

|Implementing a stored procedure to remove invalid data |
| ----------- |
![image](https://github.com/user-attachments/assets/04b73801-5963-4d3b-8360-ea368abe2896)

## Pipeline from Landing to Staging
- In this step, I moved raw data from the Landing zone to the Staging area. The data was loaded into staging tables in the Data Warehouse, where I applied basic transformations like cleaning, renaming columns, and validating formats. I also configured a Data Factory pipeline to automate this process, ensuring smooth data flow and preparing it for further processing.

|Landing to Staging Pipeline|
| ----------- |
![image](https://github.com/user-attachments/assets/6f6e2946-d610-4164-ae6c-b0876f7b7d0e)



## Latest Processed Data
For the Script Activity “Latest Processed Data”

```
select top 1 
latest_processed_pickup
from metadata.processing_log
where table_processed = 'staging_nyctaxi_yellow'
order by latest_processed_pickup desc;
```


## v_date
Pipeline expression for v_date Set Variable activity

```
@formatDateTime(addToTime(activity('Latest Processed Data').output.resultSets[0].rows[0].latest_processed_pickup,1, 'Month'), 'yyyy-MM')
```

## Copy to Staging
Pre Copy Script

```
```

## v_end_date
Pipeline expression for v_end_date Set Variable activity

```
@formatDateTime(addToTime(activity('Latest Processed Data').output.resultSets[0].rows[0].latest_processed_pickup,1, 'Month'), 'yyyy-MM')
```



