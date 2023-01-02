This project is about moving data from transaction database (MySQL RDS) in AWS into a Snowflake datawarehouse.
The key snowflake concepts covered here are:

Storage Integration Object,
File format Object,
Virtual Warehouses,
Stages,
Table types,
Copy command,
Snowpipe,
Streams,
Tasks & Task hierarchy,
Time Travel,
Data Masking and
Materialized views

The key AWS concepts covered here are:

AWS Data Pipeline,
S3, S3 Event notification,
IAM role and Trust relationship between AWS and 3rd party,
Glue Crawler and
Athena

There are 3 flows created in this project. A diagrammatic representation of the 3 flows is available in the "Architecture Diagram" PPT.
Before going into the details of the 3 flows, we need to setup the data in AWS MySQL RDS.

1. RDS.txt talks about how to provision a MySQL RDS in AWS and the create table statements for the orders table and the customers table.
2. RDS data - customers.sql has the insert statements for the customers table.
3. RDS data - orders.sql has the insert statements for the orders table.

*********************************************** Flow 1 ***********************************************

Moving historical data from transaction database (MySQL RDS) in AWS into the Snowflake datawarehouse.

4. AWS Data Pipeline - historical load.txt talks about how to setup the AWS Data Pipeline that will bring historical data from MySQL RDS into S3.
5. AWS Glue Crawler and Athena.txt talks about how to setup the AWS Glue Crawler and Athena. Use Athena to check if the data in S3 is the same as MySQL data.
6. Snowflake setup.txt is basically about all the setup that needs to be done in Snowflake in order to be able connect to and process data from RDS. 
   This talks about creating the virtual warehouse, storage integration object, file format object, stage, databases and tables in Snowflake.
7. Snowflake one-time historical load.txt shows how the copy command is run to load the historical data into the Snowflake warehouse, using all the objects created in the previous step.

*********************************************** Flow 2 ***********************************************

Moving recent data from transaction database (MySQL RDS) in AWS into the Snowflake datawarehouse.
This is a hypothetical case where majority of the historical data has been moved into Snowflake but due to some delays in enabling the movement of current data, we have some delta to be moved from MySQL RDS into the Snowflake datawarehouse. The difference here compared to moving historical data is that the data movement from AWS into Snowflake is automatic, through the use of a Snowpipe. Manual execution of copy command is not needed here.
This flow can also be used to move any specific adhoc data from AWS into the Snowflake datawarehouse.

8. Snowpipe - recent orders load.txt talks about how to setup the Snowpipe, so that any data coming into the 'recent' folder of the S3 bucket gets processed automatically into Snowflake.
   Since the 'recent' data is configured to be loaded into a different folder than the 'historical' data, another stage is setup to read from this folder.
9. AWS Data Pipeline - recent load.txt talks about how to setup the AWS data pipeline that will move the 'recent' or any adhoc data from MySQL RDS into S3. 
   As soon as data is available in S3, the Snowpipe configure in the previous step gets activated and the data is processed into the Snowflake warehouse.
10. Recent data validation.txt talks about validating the data in Snowflake against the data in MySQL RDS.
   Also, this talks about how the Crawler needs to be modified to process the new 'recent' folder and how to validate Snowflake data with Athena data.

*********************************************** Flow 3 ***********************************************

Moving current data from transaction database (MySQL RDS) in AWS into the Snowflake datawarehouse.
This is the special case where we have a staging layer before the final table and we perform CDC by making use of Tasks and Streams.

11. CDC Setup - Staging layer and Stream.txt talks about the creation of Staging table and a Stream on the Staging table.
12. Snowpipe - current orders load.txt talks about setting up the Stage and Snowpipe for processing current data. 
We need to use a different S3 bucket here, because S3 bucket does not allow 2 SQS event notifications for the same bucket. Event notification setup is needed for the Snowpipe setup. 
So we create another S3 bucket for current data, setup an event notification on this bucket based on the current data snowpipe. 
Note that the recent data snowpipe loads into the final 'orders' table in Snowflake but the current data snowpipe loads into the 'orders_staging' table in Snowflake. 
The Storage integration object needs to be replaced to accommodate the new bucket and the IAM role in AWS needs to be updated accordingly. 
Only then, the current orders snowpipe will work.
13. Changes in source RDS MySQL database.txt talks about the changes in MySQL RDS, so that we can verify if CDC is working in Snowflake.
14. AWS Data Pipeline - current load.txt shows how to setup the AWS data Pipeline to process 'current' data into S3. Make sure the changes done in previous step are related to current data so that these get captured in Snowflake.
15. Validate the data in staging table and stream.txt talks about verifying if all the current data is available in Staging layer of Snowflake and in the Stream.
16. CDC Setup - Task.txt talks about how to setup the task that uses the Stream and the final 'orders' table in Snowflake to implement CDC. This task is scheduled to run once in every hour.
17. Validate the data in final table.txt is about validating the data in the final 'orders' table - check if all the new records and all the updates done in MySQL RDS are available in Snowflake.
18. Staging table cleanup task.txt is about setting up another task with a dependency on the CDC task, to truncate the staging table after the processing into final table is done.

19. Time Travel.docx demonstrates Time Travel in Snowflake.
20. Data Masking.docx shows how to create masking policy and apply on a table / column.
21. Materialized Views.docx shows how to create materialized views and their benefits / use-cases / limitations.

22. Task and Stream - an alternate method.txt demonstrates a different approach for using / setting up tasks and streams in Snowflake.
23. Cleanup resources.txt -> drop all Snowflake / AWS resources created for the project.
