# Query Data in Amazon Athena

When you work on big data solutions, you often need to access data from multiple sources. For example, you might pull data from various sources—such as a transactional database, a live data stream, or from unstructured web content—into Amazon Simple Storage Service (Amazon S3). With Amazon Athena, you can aggregate the files in Amazon S3 and restructure the data for further analysis. Athena is serverless, so you don't have to set up or manage any infrastructure. For more information about Athena, see  <a href="https://docs.aws.amazon.com/athena/latest/ug/what-is.html" target="_blank">What is Amazon Athena</a>.

In this lab, you will practice using the AWS Management Console to create an Athena application, define a database in Athena, create a table, define columns and data types, and run both simple and complex queries.

# Objectives

After completing this lab, you will be able to:

- Access Athena in the AWS Management Console
- Create tables and define data types
- Query data in Amazon S3 from Athena
- Optimize queries with partitioning

**Prerequisites**

This lab requires:

- Access to a notebook computer with Wi-Fi and Microsoft Windows, macOS, or Linux (Ubuntu, SuSE, or Red Hat)
- For Microsoft Windows users: Administrator access to the computer
- An internet browser such as Chrome, Firefox, or IE9 (previous versions of Internet Explorer are not supported)

**Duration**

This lab requires **60** minutes to complete. The lab will remain active for **120** minutes to allow extra time to complete the challenge questions.

## Accessing the AWS Management Console

1. At the top of these instructions, choose <span id="ssb_voc_grey">Start Lab</span> to launch your lab.

    A **Start Lab** panel opens, which displays the lab status.

2. Wait until you see the message *Lab status: ready*, then close the **Start Lab** panel by choosing the **X**.

3. At the top of these instructions, choose <span id="ssb_voc_grey">AWS</span>

  This will open the AWS Management Console in a new browser tab. The system will automatically log you in.

  **Tip**: If a new browser tab does not open, there will typically be a banner or icon at the top of your browser that indicates that your browser is preventing the website from opening pop-up windows. Choose the banner or icon and then choose **Allow pop ups**.

4. Arrange the **AWS Management Console** browser tab so that it displays next to these instructions. Ideally, you should be able to see both browser tabs at the same time, which can make it easier to follow the lab steps.

## Scenario summary ##

In this lab, you are a data analyst who works for the AnyCompany Cab company in New York City. Your manager asked you to look for patterns of customer usage. She also asked you to minimize the costs for conducting your research. You decided to start by producing some descriptive statistics for a single month of data. The data for the month of January 2017 is already stored in Amazon S3.

<img src="images/lab-2-overview.png" alt="Overview of Lab 2" width="800"> 

## Task 1: Create and query an Athena database

Amazon Athena is an interactive query service that you can use to query data that is stored in Amazon S3. Athena stores data about the data sources that you must query in a database. You can store your queries for re-use, or you can share them with other users.

For more information about Athena, see <a href="https://docs.aws.amazon.com/athena/index.html" target="_blank">Amazon Athena documentation</a>.

Your first task in this lab is to create the database by writing structured query language (SQL) statements to define the schema.

5. On the AWS Management Console, on the **Services** menu, choose **Athena**.

6. Choose **Get Started**.

      Athena provides a step-by-step tutorial for creating a database and table. You will not follow the tutorial in this lab so that you can see the steps the tutorial performs on your behalf.

7. Close the tutorial window.

You see the following message at the top of the console:

<img src="images/athenas3message.png" alt="Message for adding an S3 bucket to hold query results" width="800">

You must specify an Amazon Simple Storage Service (Amazon S3) bucket to hold the results from any queries that you run.

8. On the AWS Management Console, on the **Services** menu, choose **S3**.

Amazon S3 opens and you see the bucket that was created for you in the lab environment. 

9. Select the bucket name.

10. In the bucket properties window, choose **Copy bucket ARN**.

11. Paste the bucket Amazon Resource Name (ARN) into a text editor.

12. To return to Athena, go back the AWS Management Console and on the **Services** menu, choose **Athena**.

13. Choose **setup a query result location in Amazon S3**.

14. In the **Query Result Location** box, enter the name of the bucket. The name of the bucket is the long string of characters at the end of the bucket ARN. Before the name of the bucket, make sure to specify `s3://` and terminate the bucket name with a forward slash (`/`), as shown in the example.

<img src="images/athenaqueryresultlocation.png" alt="Dialog box for adding the S3 bucket for storing query results" width="800">

15. Choose **Save**.

16. In the Athena query editor, enter the following SQL command:

```
CREATE DATABASE taxidata;
```

17. Choose **Run**.

You see the message *Query successful*.


18. In the Athena console, choose **Create table** and then choose **from S3 bucket data**.

19. Select *taxi* from the drop-down list of databases.

20. In the **Table Name** box, enter `yellow`.

21. In the **Location of Input Data Set** box, enter the following URL:

  ```
  s3://aws-tc-largeobjects/CUR-TF-200-ACBDFO-1/Lab2/yellow/
  ```

22. Choose **Next**.

23. For data format, select **CSV**.

24. Choose **Next**.

25. Choose **Bulk add columns**.

26. To add column names and types, copy the following script and paste it into the **Bulk add columns** window.

  ```
  vendor string,
  pickup timestamp,
  dropoff timestamp,
  count int,
  distance int,
  ratecode string,
  storeflag string,
  pulocid string,
  dolocid string,
  paytype string,
  fare decimal,
  extra decimal,
  mta_tax decimal,
  tip decimal,
  tolls decimal,
  surcharge decimal,
  total decimal
  ```

27. Choose **Add**.

28. Choose **Next**.

29. On the Configure Partitions step, choose **Create table**.

    Athena creates a table in the Athena database. You will see the following query in the Athena command window. This is the query that Athena ran to create the table.

  ```

  	CREATE EXTERNAL TABLE IF NOT EXISTS taxidata.yellow (
  	  `vendor` string,
  	  `pickup` timestamp,
  	  `dropoff` timestamp,
  	  `count` int,
  	  `distance` int,
  	  `ratecode` string,
  	  `storeflag` string,
  	  `pulocid` string,
  	  `dolocid` string,
  	  `paytype` string,
  	  `fare` decimal,
  	  `extra` decimal,
  	  `mta_tax` decimal,
  	  `tip` decimal,
  	  `tolls` decimal,
  	  `surcharge` decimal,
  	  `total` decimal
  	)
  	ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
  	WITH SERDEPROPERTIES (
  	  'serialization.format' = ',',
  	  'field.delim' = ','
  	) LOCATION 's3://aws-tc-largeobjects/CUR-TF-200-ACBDFO-1/Lab2/yellow/'
  	TBLPROPERTIES ('has_encrypted_data'='false');

  ```

The table property indicates that the data is not encrypted. Serde is a data serialization format, and the Serde property specifies that the data must be comma-delimited.

Now that you have a table for querying the AnyCompany Cab data, you can write queries to retrieve data from the data source in Amazon S3. You can also get a preview of the data in the table by choosing the vertical ellipsis (three dots) on the right of the Athena database window.

30. From the list of databases, choose the database that you created.

31. Choose the vertical ellipsis (three dots) next to the *taxidata* table, and from the list of options, choose **Preview table**.

<img src="images/athena_preview.png" alt="Athena Preview Database" width="400">

In the **Results** window, you will see the first 10 records of the table.


## Task 2: Optimize the database

The Athena query engine is based on an open source tool called <a href="https://prestodb.github.io/docs/0.172/overview/use-cases.html" target="_blank">Presto</a>. To read more about the SQL functions that you can use with Athena, see <a href="https://docs.aws.amazon.com/athena/latest/ug/functions-operators-reference-section.html" target="_blank">SQL Queries, Functions, and Operators</a>.

In this task, you will compare the query time and the amount of data that is scanned for two different ways of loading the taxi trip data into Athena. First, you will create a table that contains all of the taxi trip data for 2017. Next, you will create a table that contains the data for a single month. By comparing the query processing time for the two tables, you will see which approach is faster and less expensive.

When you work with large datasets, optimizing performance and minimizing costs are two major goals. You determine your costs for running Athena based on usage. Usage is based on the amount of data that is scanned. Prices vary based on your Region. For more information about Athena pricing, see <a href="https://aws.amazon.com/athena/pricing/" target="_blank">Amazon Athena Pricing</a>.

Partitioning and compressing data are two steps that you can take to minimize your costs. You can compress your data by using one of the open standards for file compression (such as gzip or tar). Partitioning reduces the amount of data that is scanned by a query. This can reduce costs and improve performance. You can also improve performance when you query with Athena by storing data in distinct buckets. You can read more about the options for choosing to partition or to use a bucket in the <a href="https://docs.aws.amazon.com/athena/latest/ug/bucketing-vs-partitioning.html" target="_blank">Athena documentation</a>.

### Task 2.1: Create a table for the January 2017 data

In this step, you will create a table to store only the data for January 2017.

To create a table for the January data:

32. Next to the **New query** tab, choose the plus sign (+) icon.

33. Copy the following query and paste it into the Athena command window and then choose **Run query**:

  ```
  CREATE EXTERNAL TABLE IF NOT EXISTS jan (
    `vendor` string,
    `pickup` timestamp,
    `dropoff` timestamp,
    `count` int,
    `distance` int,
    `ratecode` string,
    `storeflag` string,
    `pulocid` string,
    `dolocid` string,
    `paytype` string,
    `fare` decimal,
    `extra` decimal,
    `mta_tax` decimal,
    `tip` decimal,
    `tolls` decimal,
    `surcharge` decimal,
    `total` decimal
  )
  ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe'
  WITH SERDEPROPERTIES (
    'serialization.format' = ',',
    'field.delim' = ','
  ) LOCATION 's3://aws-tc-largeobjects/CUR-TF-200-ACBDFO-1/Lab2/January2017/'
  TBLPROPERTIES ('has_encrypted_data'='false');
  ```

Athena creates a new table named *jan* that contains the taxi data for the month of January 2017.


### Task 2.2: Run a query using the data that is not divided into buckets ###

Now, run a query by using the data that was not divided into monthly buckets.

34. In the Athena command window, copy and run the following query:

  ```
  	SELECT count (count) AS "Number of trips" ,
  	         sum (total) AS "Total fares" ,
  	         pickup AS "Trip date"
  	FROM yellow WHERE pickup

  	between TIMESTAMP '2018-01-01 00:00:00'
  	   and TIMESTAMP '2018-02-01 00:00:01'
  	GROUP BY pickup;
  ```

35. Note the amount of data that was scanned.


### Task 2.3: Run a query using the data that is divided into buckets for each month ###

36. In the Athena command window, copy and run the following query:

  ```
  	SELECT count (count) AS "Number of trips" ,
  	         sum (total) AS "Total fares" ,
  	         pickup AS "Trip date"
  	FROM jan
  	GROUP BY pickup;
  ```

37. Note the amount of data that was scanned.

You should find that the following results:

- No buckets –
	- Total data scanned: 9.32 GB
- Buckets –
	- Total data scanned: 815 MB


### Task 2.4: Query partitioned data ###

Putting data in separate buckets works when you have data that has a high degree of *cardinality*. Cardinality refers to the number of distinct records in a database. In this example, the **pickup** field has a high degree of cardinality because every trip has a specific date and time. If you are interested in a column with low cardinality, you would partition the data instead of using distinct buckets. In some cases, your data will already be partitioned by another process. In this case, you will partition the data by using the **paytype** field.

The **paytype** field stores the type of payment using the following codes:

- 1 = Credit card
- 2 = Cash
- 3 = No charge
- 4 = Dispute
- 5 = Unknown
- 6 = Voided trip

Because there are only a limited number of possible values, **paytype** is an excellent column to use for creating partitions. In this task, you will use a `CREATE TABLE AS` function to partition the data. You will also specify a columnar storage format. You can store data in Athena with the Apache Parquet or Optimized Row Columnar (ORC) formats. Columnar storage formats compress the data, which will further reduce costs for your queries. To read more about columnar storage formats, see <a href="https://docs.aws.amazon.com/athena/latest/ug/columnar-storage.html" target="_blank">Columnar Storage Formats</a>.

#### Task 2.4.1: Partition the data ####

You can create a partition by running a query to select the data you want to use for a partition and by providing a storage format.

38. In the Athena command window, create a partition for *paytype = 1* by running the following query:

	```
	CREATE TABLE taxidata.creditcard
	WITH (
	  format = 'PARQUET'
	  )AS
	  SELECT * from "yellow"
	  WHERE paytype = '1';
	```

	Now, you will compare the performance of running the following queries with the partitioned data in the *creditcard* table with the non-partitioned data in the *yellow* table.

39. Run the following two queries:

	```
	  SELECT sum (total), paytype FROM yellow
	   WHERE paytype = '1' GROUP BY paytype;
	```

	```
	SELECT sum (total), paytype FROM creditcard
	   WHERE paytype = '1' GROUP BY paytype;
	```

You should see the following results:

- Yellow table –
	- Run time: 7.19 seconds
	- Data scanned: 9.32 GB
- Creditcard table –
	- Run time: 3.32 seconds
	- Data scanned: 71.8 MB


## Task 3: Create and query views ###

You can hide some of the complexity of queries from users by creating views. Also, because Athena only supports running one SQL statement at a time, you can use views to combine data from various tables. You can also use views to optimize query performance by experimenting with different ways to retrieve data, and then saving the best query as a view. To read more about views, see <a href="https://docs.aws.amazon.com/athena/latest/ug/views.html" target="_blank">Views</a> in the Athena documentation.

In this task, you will create two views to calculate the total for the taxi fares that were paid with credit cards, and the total number of fares that were paid with cash.

40. In the Athena command window, create views for the total fares paid for with credit cards and with cash by entering the following two queries:

	```
	CREATE VIEW cctrips AS
	SELECT "sum"("fare") "CreditCardFares"
	FROM yellow
	WHERE ("paytype"='1');
	```

	```
	CREATE VIEW cashtrips AS
	SELECT "sum"("fare") "CashFares"
	FROM yellow
	WHERE ("paytype"='2');
	```

	You should now have two views listed in the console: *CreditCardFares* and *CashFares*.

41. in the Athena command window, see the result of the calculation by entering the following query:

	```
	Select * from cctrips;
	```

	or

	```
	Select * from cashtrips;
	
	```


You can also use a view to join data from two tables. For example, the following query will join the data from two views. The joined data shows the credit card and cash fares for two different taxi vendors.

  ```
  CREATE VIEW comparepay AS
  WITH
     cc AS
      (SELECT sum(fare) AS cctotal,
           vendor
      FROM yellow
      WHERE paytype = '1'
      GROUP BY paytype, vendor),
      cs AS
      (SELECT sum(fare) AS cashtotal,
              vendor, paytype
       FROM yellow
       WHERE paytype = '2'
       GROUP BY paytype, vendor)

  SELECT cc.cctotal, cs.cashtotal
  FROM cc
  JOIN cs
  ON cc.vendor = cs.vendor;
  ```
You can then preview the results from the join. If you preview the *comparepay* view, you should see the following results:

- Vendor 1: `cctotal 584502884 cashtotal 	250849783`
- Vendor 2: `cctotal 460097126 cashtotal     199181978`


## Challenge questions ##

Now that you know how to create tables in Athena, try the following exercises to test your knowledge. Your lab instructor has model solutions. However, there is more than one way to solve some of the exercises.

### Challenge one ###

Write a query that identifies the most common pickup location in January 2017.

### Challenge two ###

Write a query to compare the average distance for trips that were paid with credit cards and the average distance for trips that were paid with cash in January 2017.


## Lab complete

<i class="icon-flag-checkered"></i> Congratulations! You have completed the lab.

42. At the top of this page, choose <span id="ssb_voc_grey">End Lab</span> and then confirm that you want to end the lab by choosing <span id="ssb_blue">Yes</span>

    A panel will appear, and show the following message: *DELETE has been initiated... You may close this message box now.*

43. To close the panel, go to the top-right corner and choose the **X**.

