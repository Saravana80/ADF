# Azure Data Factory Training

- Create a new Azure Data Factory resource in Azure and specify details of Github URL specified above while creating your Data Factory

## 1-azure-data-factory-mapping-data-flows-basics (basics branch)
1. Create blank SQL Database - (dbatinaug21)
```
Server Name: sqlsvraug21.database.windows.net
```

- Note: make sure to enable firewall settings in Networking section to Public Endpoint and Allow Client IP and Azure Services
	
1. Create cars table in the database
```
CREATE TABLE Cars (
	Make nvarchar(100),
	Model nvarchar(200),
	Type nvarchar(100),
	Origin nvarchar(100),
	DriveTrain nvarchar(100),
	Length decimal(18,0)
)
```

1. Create a new Data Factory (adfatinaug21)
1. Create storage account (sadfaug21), create container (input) and upload cars.csv to it
    - https://raw.githubusercontent.com/atingupta2005/adf-apr-21/main/2-azure-data-factory-parametrization/cars.csv
3. Create linked service to connect to storage account - (lsStorage)
4. Create linked service to connect to sql server - (lsSQL)
5. Add dataset to connect to sql table - (outTable)
6. Create dblob dataset to connect to cars.csv	- (inputCSV)
7. Create pipeline
8. Add copy data activity to the pipeline
9. Configure copy data activity
10. Run pipeline
11. Verify data is there in the cars table
```
select * from [dbo].[Cars]
```

## 2-azure-data-factory-parametrization (parametrization branch)
1. Create planes tables in SQL database
```
CREATE TABLE Planes (
	ICAO nvarchar(200),
	IATA nvarchar(200),
	MAKER nvarchar(200),
	DESCRIPTION nvarchar(200)
)
```

1. Add parameter to dataset - inputCSV
```
fileName
```
1. Use parameter in the connection tab
1. Add parameter to dataset - outputTable
```
tableName
```
1. Upload planes.csv to storage account\input container
    - https://raw.githubusercontent.com/atingupta2005/adf-apr-21/main/2-azure-data-factory-parametrization/planes.csv
3. Use parameter in the connection tab
4. Now add parameters to the pipeline itself
5. Use those parameters in copy data activity
6. Now while running pipeline, it will ask the inputs
7. Add trigger to pipeline
8. Publish pipeline
9. Run pipeline using trigger
10. Monitor pipeline execution in Monitor tab


## 3-azure-data-factory-mapping-data-flows (data-flows branch)
1. Create another container in storage account - output
1. Upload movies.csv to input
   - https://raw.githubusercontent.com/atingupta2005/adf-apr-21/main/3-azure-data-factory-mapping-data-flows/movies.csv
3. Create/update linked services
4. Create dataset to input\movies.csv (dsInMovies)
5. Create dataset to output\ (dsOutMovies)
	- Select first row as header
	- Disable import schema
6. Create a Data Flow resource
7. Add Source -> dsInMovies dataset
8. Enable Data flow debug
9. Wait for 5-10 min
10. Select step and open tab Data Preview
11. Add new step - Derived Column. Name it - YearTitleExtraction
12. Enter column Name (Year) and Add expression to it
```
toInteger(trim(right(title, 6), '()'))
```
1. Preview data
1. Add new column - Title and add expression
```
toString(left(title, length(title)-6))
```
1. Preview the data on each step to verify changes
1. Add new Step - Sink
1. Specify dataset - dsOutMovies
1. Add a new Pipeline (pipeline2) and add Dataflow to it
1. Run Pipeline and Refer to output container and open file - part*
1. To create a single file open dataflow
1. Open sink step and specify "Output to Single File" in Settings tab


## 4-azure-data-factory-self-hosted-integration-runtime - (ir branch)
1. Create a data factory
1. Open Manage Tab\connections\intergration runtimes and create new
1. Create Self Hosted Integration runtime (irSelfHosted)
1. Download and install on VM
```
https://www.microsoft.com/en-us/download/details.aspx?id=39717
Setup File Name: IntegrationRuntime_5.8.7872.1.msi
```
1. Register by specifying the key
1. Verify the connection status in Portal
1. Create a folder on the VM:
```
C:\ir
```
1. Create a linked service to read file (lsirFile)
```
Integration Runtime: Self Hosted
Host: C:\ir
Username: vmwinadf.centralus.cloudapp.azure.com\atingupta2005
Password: <Password of VM>
```
1. Put a cars.csv in C:\ir
1. Create a dataset of type Fileservice and connect to that file (dsirInput)
1. Create a dataset of type Fileservice and connect to the folder and specify a new file name - carscopy.csv (dsirOut)
1. Create a pipeline (pipeline2) and Copy Data activity and configure it
1. Notice that we can copy content on the VM


## 5-Get Metadata Activity (branch getMetaData)
1. Create/Update the Linked Service
1. Create a new Pipeline (pipeline3)
1. Add activity - Get Metadata
1. Connect to the File Linked Service created using Self Hosted Runtime
1. Create a dataset to represent files in folder (dsFolderIn)
1. This dataset should refer to the folder - c:\ir
1. Connect activity with dataset and add new Field in the activity - Child items
1. Publish and Run pipeline
1.  Inspect the output JSON
1. Add a foreach activity named - For Each Table
1. In Setting\Items Add dynamic content and put below expression:
```
@activity('Get Metadata1').output.childitems
```
1. Create a variable in pipeline level - (fileName). It will be used in ForEach
1. Open For each activity
1. Add Set Variable activity
1. Use @item().name to refer to the current item in For each loop
1. Debug pipeline and verify the output

## 6-Storage Events Trigger (6-storage-events-trigger branch)
- Create Azure Data Lake named - adldatafactoryaug21
- Create Linked Service to connect to Azure Data Lake - lsInput_adls
- Create Linked Service to connect to Azure SQL Server - lsout_sql
- Create container - input
- Upload CSV files (cars.csv, movies.csv, planes.csv) to container - input
- Create Dataset inputCSV
- Add parameters to dataset
- Create Dataset outSQL
- Add parameters to dataset
- Add parameters to pipeline and use those in pipeline
- Add copy activity in the pipeline
- Add pipeline parameters in Copy Activity
- For sink tab in Copy activity, use below formula
- Register Event Grid in Subscription\Resource Providers (If needed). Its needed as we are now using Storage Events to trigger our pipeline
```
@replace(toUpper(pipeline().parameters.fileName), '.CSV', '')
```
- Create an event trigger
```
Type: Storage events
Name: triggerEvents
```
- Formula to capture file name
```
@triggerBody().fileName
```
- Create Merge request to merge with branch - main_publish
- Switch to branch - main_publish
- Click Publish
- Upload files to the container
- Notice the pipeline is executed on file upload event

## 7-Filter Activity (Filter_if_append branch):
1. Create/Update Linked Service
```
Username: vmwinadf.centralus.cloudapp.azure.com\atingupta2005
```
1. Create a new dataset to refer to the output folder in storage account (dsOutFolderSA)
1. In previous pipeline add a filter activity after Get Metadata and remove for each
1. Add dynamic content and put below expression:
	@activity('Get Metadata1').output.childitems
1. Add a file on the VM where Self Hosted IR is setup. File name should start with - cars
1. Add a filter condition
	@startswith(item().name, 'c')
1. Debug pipeline and see the JSON output of the activities
1. Add For each loop and configure
	@activity('Filter1').output.Value
1. Now go into For Each activity and create Copy Data activity
	Source: dsFolderIn, Wildcard Path, @item().name
	Sink: dsOutFolderSA
	

- Refer: pipeline3

## 8-Copy multiple tables in Bulk with Lookup & ForEach
- Create Storage account (If required)
```
Name: sadfaug21
```
- Create Container (If required)
```
Name: exported-data
```
- Create SQL Database (If required)
```
Name: dbatinapr21
```
- Login to SQL Database
- Create table using the script (If required)
   - https://raw.githubusercontent.com/atingupta2005/adf-apr-21/main/8-Copy-multiple-tables-in-Bulk-with-Lookup-ForEach/00-create-sample-database.sql.txt
- These tables will be created
   - dbo.Cars
   - dbo.Countries
   - dbo.Movies
- Steps to perform to create data factory pipeline
   - List Tables with Lookup
      - Create SQL Linked Service
      - Create Dataset to List Tables named - TableList
   - Create Pipeline
   - Create activity in Pipeline - Lookup. Name it - ListTables
      - Settings Tab
         - Select DataSource
         - Change to Query radio button
         - Paste below query
		```
		SELECT * FROM
		<database_name>.INFORMATION_SCHEMA.TABLES
		WHERE table_type = 'BASE TABLE'
		```

	 - Deselect Checkbox - First Row Only
	 - Click - Preview Data
    - Create activity in Pipeline - ForEach and connect with previous activity
    	- Settings
    	    - Items: Add Dynamic Content
		```
		@activity('List Tables').output.value
		```
    - Create New LinkedService to Azure Blob Storage to export output
    - Create Dataset for input - SQLTable
	    ```
	    Type: Azure SQL
	    Name: SQLTable
	    ```
    - Create Parameters of Dataset - SQLTable
	    ```
	    TableName
	    SchemaName
	    ```

    - User TableName parameter inside tab - Connection
       - Enable Edit Checkbox
       - Specify Dynamic Schema Name and Table Name
    
	     
    - Create Dataset for output
    ```
    Type: Azure Blob Storage - DelimittedText
    Name: CSVTable
    File path: exported-data
    First Row as header: Checked
    Import Schema: None
    ```
    
   - Specify Parameters of Dataset - CSVTable
   ```
   FileName
   ```
   - Use parameter in Connection Tab - File path - file
   - Add Copy activity to Pipeline inside Foreach
   ```
   Name: Export Table
   ```
   - Configure Source of Copy Activity
      - Select Source Dataset - SQLTable
      - TableName
      ```
      @item().table_name
      ```
      - SchemaName
      ```
      @item().table_schema
      ```
   - Configure Sink of Copy Activity
      - Select Sink Dataset - CSVTable
      - FileName
   ```
   @concat(item().table_schema, '_', item().table_name, '.csv')
   ```
   
## 9-Custom Email Notifications
- Create Logic App of HTTP Trigger
- Generate Schema using below Sample Payload
```
{
    "title": "",
    "message": "",
    "color": "",
    "dataFactoryName": "",
    "pipelineName": "",
    "pipelineRunId": "",
    "time": ""
}
```
- Add New Step - Outlook.com
- Action: Send an email (V2)
- Sign In to Outlook.com using atin.intellipaat@outlook.com
- Specify To: <YourEmailID> - emample: atin.intellipaat@gmail.com
- Specify Subject - Dynamic Content
```
title
```
- Create a Variable\\Initialize named - "Email Body"
- Put below content (Replace placeholder with dynamic content)
```
<hr/>
<h2 style='color:____color_here____'>____title_here____</h2>
<hr/>
Data Factory Name: <b>____name_here____</b><br/>
Pipeline Name: <b>____name_here____</b><br/>
Pipeline Run Id: <b>____id_here____</b><br/>
Time: <b>____time_here____</b><br/>
<hr/>
Information<br/>
<p style='color:____color_here____'>____message_here____</p>
<hr/>
<p style='color:gray;'>This email was generated automatically. Please do not respond to it. Contact team at: atin@atin.com </p>
```
- Specify Body - Dynamic Content and specify variable - "Email Body" we created earlier
- Save Logic App
- Copy HTTP URL of Logic APP
- Create a Pipeline - MASTER
- Add activity - Execute Pipeline
- Settings
```
Invoked Pipeline: DEMO-PIPELINE
```
- Add another activity - Web - Name: Send Success Email
- Connect it with Success output from previous activity - Execute Pipeline
- Customize Activity - Send Success Email
- URL: Logic APP URL
- Method: Post
- Body
```
{
    "title": "@{activity('Execute Pipeline1').output.pipelineName} SUCCEEDED!",
    "message": "Pipeline run finished successfully!",
    "color": "Green",
    "dataFactoryName": "@{pipeline().DataFactory}",
    "pipelineName": "@{activity('Execute Pipeline1').output.pipelineName}",
    "pipelineRunId": "@{activity('Execute Pipeline1').output.pipelineRunId}",
    "time": "@{utcnow()}"
}
```
- Add another activity - Web - Name: Send Failure Email
- Add Failure output to activity - Execute Pipeline
- Connect it with Failure output from previous activity - Execute Pipeline
- Customize Activity - Send Failure Email
- URL: Logic APP URL
- Body
```
{
    "title": "@{activity('Execute Pipeline1').output.pipelineName} FAILED!",
    "message": "Error: @{activity('Execute Pipeline1').error.message}",
    "color": "Red",
    "dataFactoryName": "@{pipeline().DataFactory}",
    "pipelineName": "@{activity('Execute Pipeline1').output.pipelineName}",
    "pipelineRunId": "@{activity('Execute Pipeline1').output.pipelineRunId}",
    "time": "@{utcnow()}"
 }
```
- Delete cars.csv from input containter and run the pipeline again. It will send failure email


## 10-Trigger Data Factory Pipeline from Azure Logic App
1. Open ADF
1. Select Branch -> 2-parametrization branch
1. Publish it by creating a Pull Request and then Publish the branch - main_publish
1. Create a new Logic App
1. Trigger - Schedule or HTTP
1. Add action - Azure Data Factory \\ Create a Pipeline Run
1. Specify details of pipeline to execute
1. Specify Parameter:
```
{"fileName": "cars.csv", "tableName": "Cars"}
```
1. Trigger logic app and confirm if data factory pipeline is executed


## 11-Incremental Load of Data from SourceDB to TargetDB (10-Incrementally-Load-Data)
- Create Cars table in Source and Target databases
```
CREATE TABLE Cars (
	Make nvarchar(100),
	Model nvarchar(200),
	Type nvarchar(100),
	Origin nvarchar(100),
	DriveTrain nvarchar(100),
	Length decimal(18,0)
)
```
```
ALTER TABLE [dbo].[Cars] ADD InsertDateTime datetime
```
```
update [dbo].[Cars] 
set InsertDateTime = GETDATE()
```
- Create watermark table in target database
```
Create Table dbo.watermarktable
(
tableName varchar(255),
WaterMarkValue datetime
);
Insert into watermarktable
VALUES ('Cars', '1900-01-01');
GO
CREATE PROCEDURE spSetWatermark @TableName varchar(50), @NewTime datetime
AS
BEGIN
   UPDATE watermarktable
   SET [WaterMarkValue] = @NewTime
WHERE [TableName] = @TableName
END
```
- Create Linked Service to Source Database - lsSource
- Create Linked Service to Target Database - lsTarget
- Create dataset for Source
```
Name: ds_CarsSource
TableName: Cars
```
- Create dataset for Target watermark
```
Name: ds_watermarktable
TableName: watermarktable
```
- Create dataset for Target
```
Name: ds_CarsTarget
TableName: Cars
```
- Create Pipeline
- Add Lookup activity named - lkpOldWaterMarkValue
   - Configure Settings
    ```
    Source dataset: ds_watermarktable
    User query: Query
    ```
    ```
    Select TableName, WaterMarkValue From dbo.watermarktable WHERE TableName = 'Cars'
    ```
- Add Lookup activity named - lkpNewWaterMarkValue
  - Configure Settings
   ```
   Source dataset: ds_CarsSource
   User query: Query
   ```
   ```
   SELECT max(InsertDateTime) AS NewWaterMarkValue FROM dbo.Cars
   ```
- Debug the pipeline and inspect the output of each activity
- Add Copy Data activity and send output of both lookup activities to Copy Data activity
- Configure Source
```
Source dataset: ds_CarsSource
Use query: Query
```
- Add dynamic content for query
```
SELECT * From dbo.Cars Where InsertDateTime > '@{activity('lkpOldWaterMarkValue').output.firstRow.WaterMarkValue}' AND 
InsertDateTime <= '@{activity('lkpNewWaterMarkValue').output.firstRow.NewWaterMarkValue}'
```
- Configure Sink
```
Sink dataset: ds_CarsTarget
```
- Import Schema by just providing some dummy data
- Notice mapping is done successfully
- Add activity - Stored Procedure
- Configure Settings
```
LinkedService: lsTarget
Stored Procedure Name: dbo.spSetWatermark
```
- Click Import to import the input parameters of the stored procedure
- Add dynamic content to NewTime value
```
@activity('lkpNewWaterMarkValue').output.firstRow.NewWatermarkValue
```
- Add dynamic content to TableName value
```
@activity('lkpOldWaterMarkValue').output.firstRow.TableName
```
- Debug pipeline
- Notice data has been shifted to target table
- Insert few records to the Source table Cars
```
Insert Into [dbo].[Cars] 
Values ('Acura', 'MDX', 'SUV', 'Asia', 'All', 4451, GETDATE() )
```
- Run pipeline again and notice only increamental changes are added to the target


## 12-ComosDB Integration (11-ComosDB-Integration)
- Create Azure Cosmos DB -> SQL API
- Create container in Database - Cars
- Create Linked Service - lsCosmos
- Create Linked Service - lsStorage
- Create Dataset - dsCosmos
- Create Dataset - inputCSV
- Create Pipeline and add Copy Data Activity to it
- Configure Source and target
- Debug Pipeline
- Refresh Cosmos DB Data to inspect the updates

## 13-Databricks Integration
- Create Databricks Python Notebook - myFirstNotebook
- Use below code:
```
# Creating widgets for leveraging parameters, and printing the parameters

dbutils.widgets.text("input", "","")
y = dbutils.widgets.get("input")
print ("Param -\'input':")
print (y)
```
- Generate an access token in Azure Databricks
- Create Pipeline and add activity - Notebook
- Configure activity to connect to the notebook created in Databricks
- Create a linked service to - 'Azure Databricks'
- Specify Notebook Path
- Specify Parameters to pass to notebook
```
Name: input
Value: Atin
```
- Debug pipeline
- Inspect the activity output and click URL to inspect output of Notebook Job execution

## 14-SSIS integration runtime
- Download and install SQL Server 2017 x86
- Download and install Visual Studio with "Data storage and processing"
- In Visual Studio, install extention - "SQL Server Integration Services Projects"
- Install "SSIS Azure Feature Pack for SQL Server 2017" - SsisAzureFeaturePack_2017_x86.msi
   - https://www.microsoft.com/en-us/download/details.aspx?id=54798
- Create project in Visual Studio using template - "Integration Services Project"
   - It extracts a text file named contacts.txt from the blob source and loads it into destination blog storage.
- Create SSIS Integration Runtime in Azure Data Factory
   - Select Create SSIS Catalog option to deploy packages in SSISDB, provide Azure SQL Database server endpoint, and the admin credentials to connect
- Connect VS project to Azure
- Deploy VS project to Azure
- Create a new pipeline and add activity - "Execute SSIS Package"
- Select SSIS Package in the settings of activity
- Debug Pipeline

## 15-Dynamic Pipelines-Incremental Copy of Multiple Tables
- Refer Git Repo: 13-Dynamic-Pipelines-Incremental-Copy-of-Multiple-Tables
- Open Azure SQL Database for Source - dbatinapr21 
- Open Azure SQL Database for Target - dbTarget
- Create 2 more tables (Planes and Movies) in Source and Target databases
```
CREATE TABLE [dbo].[Planes] (
	ICAO nvarchar(200),
	IATA nvarchar(200),
	MAKER nvarchar(200),
	DESCRIPTION nvarchar(200)
)
```
```
CREATE TABLE [dbo].[Movies] (
	movieId nvarchar(200),
	title nvarchar(200),
	genres nvarchar(200)
)
```

- Add InsertDateTime column in the tables in Source and Target databases
```
ALTER TABLE [dbo].[Planes] ADD InsertDateTime datetime

```
```
ALTER TABLE [dbo].[Movies] ADD InsertDateTime datetime

```

- Set default value to the column - InsertDateTime
```
Update [dbo].[Planes]
set InsertDateTime = GETDATE()
```

```
Update [dbo].[Movies]
set InsertDateTime = GETDATE()
```

- Open Target Database
- Insert records in water marktable (as needed)
```
Insert into watermarktable
VALUES ('Movies', '1900-01-01');
```
```
Insert into watermarktable
VALUES ('Planes', '1900-01-01');
```
```
Insert into watermarktable
VALUES ('Cars', '1900-01-01');
```
- Open Pipeline in ADF
- Create 2 pipeline parameters
```
TableName
WaterMarkColumn
```
- Change query of lkpOldWaterMarkValue
```
Select TableName, WaterMarkValue From dbo.watermarktable WHERE TableName = '@{pipeline().parameters.TableName}'
```

- Change query of lkpNewWaterMarkValue
```
SELECT max(@{pipeline().parameters.WaterMarkColumn}) AS NewWaterMarkValue FROM @{pipeline().parameters.TableName}
```

- Change query of activity - Copy Data
```
SELECT * From @{pipeline().parameters.TableName} Where @{pipeline().parameters.WaterMarkColumn} > '@{activity('lkpOldWaterMarkValue').output.firstRow.WaterMarkValue}' AND 
@{pipeline().parameters.WaterMarkColumn} <= '@{activity('lkpNewWaterMarkValue').output.firstRow.NewWaterMarkValue}'
```

- Change the Sink section to get table name dynamically
   - Add Parameters using Parameters tab
   ```
   TableName
   ```
   - Open tab - Connection
   - Enable checkbox - Edit
   - Add dynamic content of table name
   ```
   @pipeline().parameters.TableName
   ```
- Insert one record in each table and test the pipeline
- In target database, create configuration table to make our pipeline dynamic
```
CREATE TABLE [dbo].[ConfigurationTable] (
	TableName nvarchar(100) NULL,
	WaterMarkColumn nvarchar(100) NULL
) ON [PRIMARY]
```
- Insert records in ConfigurationTable
```
Insert INTO ConfigurationTable VALUES ('Cars', 'InsertDateTime')
```
```
Insert INTO ConfigurationTable VALUES ('Movies', 'InsertDateTime')
```
```
Insert INTO ConfigurationTable VALUES ('Planes', 'InsertDateTime')
```

- Create new pipeline - Master
- Create a new Dataset to the table ConfigurationTable - dsConfigurationTable
- Add activity - Lookup
```
Name: Lookup Config
Source dataset: ConfigurationTable
```
- Uncheck - First Row Only
- Preview Data to confirm connection
- Add activity - ForEach
```
Name: ForEachConfigEntry
Settings->Item: @activity('Lookup Config').output.value
```
- Go inside ForEach activities
   - Add activity - Execute Pipeline
   ```
   Name: ExecuteIncrementalPipeline
   Settings->Invoked Pipeline: <Chose Pipeline which is doing increamental copy>
   ```   
   - Specify parameters of the child pipeline
   ```
   TableName: @{item().TableName}
   WaterMarkColumn: @{item().WaterMarkColumn}
   ```
- Debug Pipeline
- Check records in the table of source and target database
```
Select * from Cars
```
```
Select * from Planes
```
```
Select * from Movies
```
- Run pipeline
- Again check records in the table of source and target database
- Insert new records and run the pipeline in multiple iterations
   - Insert records
   ```
   Insert Into [dbo].[Cars] 
   Values ('Acura', 'MDX', 'SUV', 'Asia', 'All', 4451, GETDATE() )
   ```
   ```
   Insert Into [dbo].[Planes]
   Values ('A318','318','Airbus','A318', GETDATE() )
   ```
   ```
   Insert Into [dbo].[Movies]
   Values ('1','Toy Story (1995)', 'Adventure|Animation|Children|Comedy|Fantasy', GETDATE() )
   ```
   - Run pipeline
   - Check records in the table of source and target database
   - Repeat above step again

	
## 16-Data Flow More examples (14-data-flows-more-examples branch)
 - Refer to these data flow in Data Factory
```
1-dfJoin
2-df-exists
3-df-conditional-split-employees
4-df-employees-lookup-with-departments
5-df-employees-group-by-lookup-select
```
