# Incremental Data Loading with Azure Data Factory

## Project Overview

This mini-project demonstrates how to use **Azure Data Factory (ADF)** to create a pipeline that transfers **incremental data** from an **Azure SQL Database** table to **Azure Data Lake Storage**. The key objective is to transfer only new or updated records by utilizing a **watermark** value for tracking changes.

### Key Components:

- **Azure SQL Database**: The source data.
- **Azure Data Lake Storage**: The destination for the incremental data.
- **Azure Data Factory**: The tool used to create the pipeline and manage data transfer.
- **Watermark Table**: Used to track the last data load timestamp.

---

## Project Structure

### 1. Setting up Azure Data Lake Storage

- A Data Lake Storage account is created to store both the **incremental data** and the **watermark** used to track changes.

### 2. Azure Data Factory Setup

- An **Azure Data Factory** instance is created to manage the data pipeline.
- **Linked services** are configured to connect Azure Data Factory to:
    - **Azure SQL Database** (source)
    - **Azure Data Lake Storage** (sink)

### 3. Datasets

- **Source Dataset**: Represents the Azure SQL Database table containing the data.
- **Sink Dataset**: Represents the Azure Data Lake destination with a unique file path:
```
@CONCAT('Incremental-', pipeline().RunId, '.txt')
```
- **Watermark Dataset**: Tracks the last loaded timestamp for incremental loading.

### 4. Building the Pipeline

The pipeline consists of several activities to perform incremental data loading:

1. **Lookup Activities**:
    - **LookUpOldWatermarkActivity**: Retrieves the last loaded watermark value by running this query:
```
SELECT * FROM [dbo].[WatermarkTable];
``` 
- **LookupNewWatermarkActivity**:Retrieves the new watermark value by running this query:
```
SELECT MAX(LastModifyTime) AS NewWatermarkValue FROM [dbo].[product_data];
```
2. **Incremental Copy Activity**:
    - Transfers only new or updated records from Azure SQL Database to Azure Data Lake based on the retrieved watermark values. The query for the source of the Copy Activity is:
```
SELECT * FROM [dbo].[product_data] 
WHERE LastModifytime > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' 
AND LastModifytime <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'
```
4. **StoredProcedure Activity**:
    - Executes a stored procedure to update the **watermark table** with the new load timestamp, ensuring future loads only get new data.

**Stored Procedure Details**:

The stored procedure usp_update_watermark is designed to update the watermark value in a database table, used to track the most recent modification times of various tables.

**Parameters**:
@LastModifiedtime datetime: The timestamp for the new watermark.
@TableName varchar(50): The name of the table for which the watermark is being updated.
The parameter code for the stored procedure is:
```
@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}
@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}
```

### 5. Executing the Pipeline

- The pipeline is triggered manually or automatically (depending on configuration).

### 6. Monitoring and Tracking

- **Monitoring Tools** in Azure Data Factory are used to check the pipeline’s performance and output, ensuring data loads as expected.

---

## How It Works

1. **Watermark Column**: The source data has a column (e.g., `last_modify_time` or `ID`) that tracks changes. This column is used to filter new or updated rows.
2. **Watermark Table**: The **watermark table** stores the last timestamp of data loaded into Azure Data Lake.
3. **Pipeline Workflow**:
    - Lookup Activities retrieve the old and new watermark values.
    - The Copy Activity transfers the incremental data based on the delta between the two watermark values.
    - The Stored Procedure updates the watermark table with the latest timestamp for future runs.

---

## Deployment Instructions

To replicate this project, follow these steps:

1. **Set up Azure SQL Database** and **Azure Data Lake Storage** in your Azure environment.
   
2. **Create the Azure Data Factory (ADF) instance**:
    - Set up **Linked Services** for the Azure SQL Database and Azure Data Lake Storage.
    - **Use the provided JSON templates** to quickly configure:
      - **ADF Pipeline**
      - **Datasets** (source, sink, and watermark datasets)
      - **Linked Services**

   You can find these JSON configuration files in the `/arm-template/` directory of this repository. They include pre-configured settings for the pipeline, datasets, and linked services to make setup faster and more accurate.

3. **Build the pipeline**:
    - Add **Lookup Activities** to retrieve the watermark values.
    - Use a **Copy Activity** to transfer incremental data.
    - Add a **StoredProcedure Activity** to update the watermark in the SQL database.

4. **Trigger the pipeline** and **monitor** the execution using the Azure Data Factory monitoring tools.

---

## Screenshots

- Screenshots of my **Azure Data Factory pipeline**, the **Lookup Activities**, the **Copy Activity**, and the **StoredProcedure Activity** to provide a visual representation of the project workflow are stored in the screenshots folder.

---

## Technologies Used

- **Azure SQL Database**
- **Azure Data Lake Storage**
- **Azure Data Factory**
- **SQL Stored Procedures**

---

## Conclusion

This project demonstrates the basic steps involved in building an incremental data load pipeline using Azure Data Factory. The watermark table ensures that only new or updated records are transferred during each run, making the process efficient and scalable.

---

## Future Enhancements

- Automate the pipeline execution using triggers.
- Implement error handling to manage potential pipeline failures.

---

### References

- [Azure Data Factory Documentation](https://docs.microsoft.com/en-us/azure/data-factory/)
