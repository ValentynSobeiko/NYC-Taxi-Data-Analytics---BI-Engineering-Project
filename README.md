##Data Engineering Pipeline: Yellow Taxi Trips NYC 2024 — Microsoft Fabric Project

This project is a comprehensive solution for creating an end-to-end pipeline for data processing, transformation and visualization using Microsoft Fabric, including Lakehouse, Data Factory, Dataflows Gen2, SQL Stored Procedures and Power BI.

Thanks to [Mr.Malvik Vaghadia](udemy.com/course/microsoft-fabric-the-ultimate-guide) for the project inspiration.

The project demonstrates the construction of a dynamic process for loading and preparing data with automatic parameterization and updating, including working with metadata and building a final report in Power BI.



# Technology Stack
-  Microsoft Fabric

-  Data Factory Pipelines

-  Lakehouse & OneLake

-  Dataflows Gen2

-  T-SQL & Stored Procedures

-  Parquet Files

-  Dynamic Content / Parameters

-  Power BI

# Solution architecture

![image](https://github.com/user-attachments/assets/e1ea2f37-13cd-4bd7-b556-c93c96e9a73c)

# Project Implementation Stages 

# Setting Up the Development Environment

-Initialized a new workspace in Microsoft Fabric, created a Lakehouse to store raw data, and uploaded Parquet files for NYC taxi trips.

![image](https://github.com/user-attachments/assets/94ce3c0b-2dd5-4a6e-8cc4-dcb32972f700)
![image](https://github.com/user-attachments/assets/cd4e5b0e-d58b-44da-9175-18d592c0a6ea)
![image](https://github.com/user-attachments/assets/050945b1-daa5-4900-a7bd-1abf65b4c2ef)





# Creating Data Warehouse & Loading Data
- Set up a Data Warehouse and loaded raw data into staging tables. Built a Data Factory pipeline with dynamic variables to automate file ingestion and streamline data loading.
![image](https://github.com/user-attachments/assets/d8bd3c77-bd00-4ec2-a2d3-6b656e7b1ac7)
![image](https://github.com/user-attachments/assets/5cd41263-1b00-436a-a7e3-ab671bfb478a)
![image](https://github.com/user-attachments/assets/9bb2a6de-b836-464e-928e-9818f35dfd3b)




