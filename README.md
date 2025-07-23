# NYC Taxi Data & BI Engineering Project - Microsoft Fabric

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


|Solution architecture |
| ----------- |
 <img width="15160" height="6450" alt="Solution architecture" src="https://github.com/user-attachments/assets/63f8f86f-be32-46f7-9641-520dd9fbb9a0" />

# Project Implementation Stages 

## Setting Up the Development Environment & Loading Initial Data

Created a workspace in Microsoft Fabric, set up a Lakehouse and a Data Warehouse, uploaded NYC taxi trip Parquet files, and used a lookup pipeline to transfer reference data into a staging table.

| Created a Lakehouse to store raw NYC taxi trip data in Parquet format. This serves as the initial storage layer for further processing. |
| ----------- |
![image](https://github.com/user-attachments/assets/e62db76c-001e-44f8-9972-8b65b8245619)
![image](https://github.com/user-attachments/assets/ab7bf7a2-2e62-4154-83d5-3a107b06c014)


|Transferred lookup zone data from Lakehouse using a pipeline into a staging table for enriched processing. |
| ----------- |
![image](https://github.com/user-attachments/assets/94ce3c0b-2dd5-4a6e-8cc4-dcb32972f700)
![image](https://github.com/user-attachments/assets/cd4e5b0e-d58b-44da-9175-18d592c0a6ea)
![image](https://github.com/user-attachments/assets/050945b1-daa5-4900-a7bd-1abf65b4c2ef)

## Data Cleaning & Filtering

Identified and removed invalid records (e.g., incorrect dates) by implementing a stored procedure to ensure data quality before further processing.

|Identifying and filtering incorrect records|
| ----------- |
![image](https://github.com/user-attachments/assets/f3f3df97-05ea-4e47-baa9-f4b6f9e7dae8)

|Implementing a stored procedure to remove invalid data |
| ----------- |
![image](https://github.com/user-attachments/assets/04b73801-5963-4d3b-8360-ea368abe2896)

# Pipeline from Landing to Staging
In this step, I moved raw data from the Landing zone to the Staging area. The data was loaded into staging tables in the Data Warehouse, where I applied basic transformations like cleaning, renaming columns, and validating formats. I also configured a Data Factory pipeline to automate this process, ensuring smooth data flow and preparing it for further processing.

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
|Set up a Data Warehouse and loaded raw data into staging tables. Built a Data Factory pipeline with dynamic variables to automate file ingestion and streamline data loading.|
| ----------- |
![image](https://github.com/user-attachments/assets/d8bd3c77-bd00-4ec2-a2d3-6b656e7b1ac7)
![image](https://github.com/user-attachments/assets/5cd41263-1b00-436a-a7e3-ab671bfb478a)
![image](https://github.com/user-attachments/assets/9bb2a6de-b836-464e-928e-9818f35dfd3b)


## v_end_date
Pipeline expression for v_end_date Set Variable activity

```
@addToTime(concat(variables('v_date'),'-01'), 1, 'Month')
```

## SP Removing Outlier Dates
Pipeline expression for v_end_date Set Variable activity

```
create procedure stg.data_cleaning_stg
@end_date datetime2,
@start_date datetime2
as
delete from stg.nyctaxi_yellow where tpep_pickup_datetime < @start_date or tpep_pickup_datetime > @end_date;
```
| |
| ----------- |
![image](https://github.com/user-attachments/assets/c420857a-7e56-40cd-bb13-a1c5edf88c40)


## SP Loading Staging Metadata
Pipeline expression for v_end_date Set Variable activity

For the Stored Procedure Activity “SP Loading Staging Metadata”.

Code to create the metadata.processing_log table.
```
CREATE SCHEMA metadata;

CREATE table metadata.processing_log
(
    pipeline_run_id VARCHAR(255),
    table_processed VARCHAR(255),
    rows_processed INT,
    latest_processed_pickup datetime2(6),
    processed_datetime datetime2(6)
);
```

Created the Stored Procedure metadata.insert_staging_metadata in the Data Warehouse using the code below.


```
CREATE PROCEDURE metadata.insert_staging_metadata
    @pipeline_run_id VARCHAR(255),
    @table_name VARCHAR(255),
    @processed_date DATETIME2
AS
    INSERT INTO metadata.processing_log (pipeline_run_id, table_processed, rows_processed, latest_processed_pickup, processed_datetime)
    SELECT
        @pipeline_run_id AS pipeline_id,
        @table_name AS table_processed,
        COUNT(*) AS rows_processed,
        MAX(tpep_pickup_datetime) AS latest_processed_pickup,
        @processed_date AS processed_datetime
    FROM stg.NYC_Taxi_yellow;
);
```
| |
| ----------- |
![image](https://github.com/user-attachments/assets/f578979b-8f41-4bab-9b42-ca6a90513850)



# Dataflow from Staging to Presentation

This step involves transforming and enriching data from the Staging area and loading it into the Presentation layer. Using a Dataflow Gen2, unnecessary columns are removed, values are standardized, and lookup tables are joined to enhance the dataset. The result is a clean, analytics-ready table used for reporting in Power BI.


| |
| ----------- |
![image](https://github.com/user-attachments/assets/25504c74-b838-4f12-a766-4d815809b2ad)



## Create the dbo.nyctaxi_yellow table

This is the initial empty table so we can load the data from the Dataflow/Stored Procedure acivities

```
CREATE TABLE dbo.nyctaxi_yellow
(   vendor varchar(50),
	tpep_pickup_datetime date,
	tpep_dropoff_datetime date,
	PU_Borough varchar(100),
	PU_Zone varchar(100),
	DO_Borough varchar(100),
	DO_Zone varchar(100),
	Payment_method varchar(50),
	passenger_count int,
	trip_distance FLOAT,
	total_amount FLOAT
); 
```



## SP Processing Presentation
For the Stored Procedure Activity “SP Processing Presentation ”.

Create the Stored Procedure dbo.process_presentation in the Data Warehouse using the code below.

```
CREATE PROCEDURE dbo.process_presentation
AS
INSERT INTO dbo.nyctaxi_yellow
    SELECT
    CASE 
        WHEN nty.VendorID = 1 THEN 'Creative Mobile Technologies'
        WHEN nty.VendorID = 2 THEN 'Curb Mobility, LLC'
 	WHEN nty.VendorID = 6 THEN 'Myle Technologies Inc'
 	WHEN nty.VendorID = 7 THEN 'Helix'
        else 'Unknown'
    end as vendor,
    format(nty.tpep_pickup_datetime,'yyyy-MM-dd') as tpep_pickup_datetime,
    format(nty.tpep_dropoff_datetime,'yyyy-MM-dd') as tpep_dropoff_datetime,
    lu1.Borough as pu_borough,
    lu1.Zone as pu_zone,
    lu2.Borough as pu_borough,
    lu2.Zone as pu_zone,
    CASE 
  	WHEN nty.payment_type = 0 THEN 'Flex Fare Trip'
        WHEN nty.payment_type = 1 THEN 'Credit Card'
        WHEN nty.payment_type = 2 THEN 'Cash'
        WHEN nty.payment_type = 3 THEN 'No Charge'
        WHEN nty.payment_type = 4 THEN 'Dispute'
        WHEN nty.payment_type = 5 THEN 'Unknown'
        WHEN nty.payment_type = 6 THEN 'Voided Trip'
        else 'Unknown'
    end as payment_method,
    nty.passenger_count as passenger_count,
    nty.trip_distance as trip_distance,
    nty.total_amount as total_amount
    from  stg.NYC_Taxi_yellow nty
    left join stg.NYC_Taxi_zone_lookup lu1
    on nty.PULocationID = lu1.LocationID
    left join stg.NYC_Taxi_zone_lookup lu2
    on nty.DOLocationID = lu2.LocationID;
```
| The same transformations can be performed visually using Dataflow Gen2, offering a low-code alternative to stored procedures. |
| ----------- |
![image](https://github.com/user-attachments/assets/0b06116f-ccf4-4235-b0d8-2e8cedea92da)



## SP Loading Presentation Metadata
For the Stored Procedure Activity “SP Loading Staging Metadata”.

Create the Stored Procedure metadata.insert_staging_metadata in the Data Warehouse using the code below.

```
CREATE PROCEDURE metadata.insert_presentation_metadata
    @pipeline_run_id VARCHAR(255),
    @table_name VARCHAR(255),
    @processed_date DATETIME2
AS
    INSERT INTO metadata.processing_log (pipeline_run_id, table_processed, rows_processed, latest_processed_pickup, processed_datetime)
    SELECT
        @pipeline_run_id AS pipeline_id,
        @table_name AS table_processed,
        COUNT(*) AS rows_processed,
        MAX(tpep_pickup_datetime) AS latest_processed_pickup,
        @processed_date AS processed_datetime
    FROM dbo.nyctaxi_yellow;
```

| |
| ----------- |
![image](https://github.com/user-attachments/assets/afe808e5-1104-4888-bb87-318d359d10fa)

# Orchestration Pipeline & Validation
Built an orchestration pipeline to automate monthly data processing between staging and presentation layers. Validated the flow by checking logs and processed record counts.

| |
| ----------- |
![image](https://github.com/user-attachments/assets/0050904a-6e5a-467e-b731-f6e18791b45f)
![image](https://github.com/user-attachments/assets/951599e6-4fae-4895-a24f-6d5e6115347d)

| |
| ----------- |



# Semantic Modelling and Power BI
Built a semantic model and created a Power BI report with filters
| |
| ----------- |
![image](https://github.com/user-attachments/assets/4f105d92-52c3-4c62-b4c2-698ae5973f34)


# Conclusion
This end-to-end data project showcased the full capabilities of Microsoft Fabric for building modern data analytics solutions. Starting with raw NYC Yellow Taxi trip data in Parquet format, I designed and implemented a complete pipeline that included:

- Efficient data ingestion using Data Factory pipelines

- Lakehouse architecture for scalable storage and processing

- Automated data cleaning, transformation, and validation via Dataflows Gen2 and stored procedures

- Creation of a metadata logging system to track pipeline executions and data freshness

- Building a dynamic and interactive Power BI report for business insights

By combining key components of the Microsoft Fabric ecosystem — Lakehouse, Data Factory, Dataflows, and Power BI — the project successfully transformed over 3.7 million raw records into a reliable and insightful reporting layer.

This solution not only demonstrates strong technical implementation but also highlights best practices in data orchestration, modular architecture, and data quality control, making it an ideal framework for real-world business intelligence scenarios.

