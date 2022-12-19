# AWS Data Wrangler (awswrangler) interact with S3, Glue, Athena

## Description
Walkthrough on how to interact with AWS services like S3, Glue and Athena using AWS Data Wrangler (awswrangler).

## Overview
AWS Data Wrangler is an AWS Professional Service open-source python initiative that extends the power of Pandas library to AWS connecting DataFrames and AWS data-related services.

## How to Get Data From S3 as Pandas Data Frames
First, we assume that you have already set the AWS credentials in your local machine. Let’s see how we can get data from the S3 to Python as Pandas Data Frames. I have an S3 bucket that contains the `customers.csv` data in `input` folder.

By running the following command, I will be able to get the data as a pandas data frame:

``` py
import awswrangler as wr
df = wr.s3.read_csv(path="s3://data-3652-us-east1/data/customers.csv")
df
```



## How to Write Data to S3
As, I have an S3 bucket called `data-3652-us-east1`, where I will store some data there


``` py
import awswrangler as wr
import pandas as pd
df1 = pd.DataFrame({"id": [1, 2], "name": ["foo", "boo"]})
df2 = pd.DataFrame({"id": [3], "name": ["bar"]})
bucket = 'data-3652-us-east1'
path1 = f"s3://{bucket}/input/file1.csv"
path2 = f"s3://{bucket}/input/file2.csv"
wr.s3.to_csv(df1, path1, index=False)
wr.s3.to_csv(df2, path2, index=False)
```


As we can see, both files `file1.csv` and `file2.csv` have been written to S3.



In order to make sure that we have uploaded the data correctly, let’s download them from S3 as

``` py
wr.s3.read_csv([path1])
```



We can read multiple CSV files as follows:

``` py
wr.s3.read_csv([path1, path2])
```



You can either read the data by prefix.

``` py
wr.s3.read_csv(f"s3://{bucket}/input/")
```



## How to Create AWS Glue Catalog database
The data catalog features of [AWS Glue](https://github.com/masood2iq/AWS-Athena-Glue-S3-CloudFormation-Deployment-AWSConsole) and the inbuilt integration to Amazon S3 simplify the process of identifying data and deriving the schema definition out of the discovered data. Using AWS Glue crawlers within your data catalog, you can traverse your data stored in Amazon S3 and build out the metadata tables that are defined in your data catalog. Let’s create a new database called `wrangler_db`

``` py
wr.catalog.create_database(name='wrangler_db', exist_ok=True)
```



If I go to the `AWS Glue console`, under the `Data Catalog` and in `Databases`, I will see the `wrangler_db`.



I could get all the databases programmatically using wrangler as follows:

``` py
dbs = wr.catalog.get_databases()
for db in dbs:
    print("Database name: " + db['Name'])
```



## How to Register Data with AWS Glue Catalog
Let’s see how we can register the data.

``` py
wr.catalog.create_csv_table(database='wrangler_db', path='s3://{}/input/'.format(bucket), table="wrangler_db_table", columns_types={'id': 'int', 'name': 'string'}, mode='overwrite', skip_header_line_count=1, sep=',')
```



If you go to AWS Glue, under the `wrangler_db` database there is the `wrangler_db_table` table.



## How to Review that Table Shape
We can get the metadata information of the `wrangler_db_table` table as follows:

``` py
table = wr.catalog.table(database='wrangler_db', table='wrangler_db_table')
table
```



## Run Query in AWS Athena
We will see how we can query the data in Athena from our database. Note, that in the case where you do not have a bucket for the Athena, you need to create one as follows:

``` py
wr.athena.create_athena_bucket()
```



Now, we are ready to query our database. The query will be the `select * from wrangler_db_table` and print the results as

``` py
my_query_results = wr.athena.read_sql_query(sql="""select * from wrangler_db_table""", database='wrangler_db')
print(my_query_results)
```





As we can see, the query returned the expected results. Note that our bucket contains two csv files, however, the catalog was able to merge both of them without adding an extra row for the column name of the second file. Also you can see that it’s created the Athena query output results in output bucket.





