
Databricks sandbox - implementation of a lightweight medalion architecture
Task Definition:
The purpose of this task is to get familiar with the databricks environment and some of the feature available. Databricks offers a Jupyter notebook style of development with the possibility of using SQL, Python, Scala and R. In order to complete this task, use of both SQL and Python will be required. It's main intention is to use one of the data sources provided within the Databricks community edition and to process that data through the layers defined by the Medalion architecture.

Prerequisites:
Before getting started with the task, a Databricks community account needs to be created for each person. It can be done via this link: Databricks: Getting Started

Click the Try Databricks hyperlink, fill in the required information (use your personal email so you can access Databricks community anytime in the future in order to do some learning there) and upon the step which requires cloud provider selection, click "Try Databricks community edition".

After that you should be able to access your very own Databricks workspace and proceed to creating a cluster which will execute the code you run. Go to to 'Compute' on the left menu pane and click the 'Create compute' button on the top right. There is no need to configure anything there since we only have the possibility to create a 15GB, 2 core cluster with the community edition. The only thing you need to worry about is thinking of a nice name for your cluster. Also note that these clusters in the community edition are terminated after some time of inactivity so in that case you just need to create a new cluster in the same way again.

After the cluster is up and running you can get started with the task.

Task Objectives:
1. Create a raw to bronze notebook:

1.1. For this task we will be using NYC green trip data. All the datasets are located in the dbfs (Databricks File System) and can be accessed using this location: dbfs:/databricks-datasets/nyctaxi/tripdata/green. For step one, you should select 5 files from this directory starting with 30th file and ending with 35th file in the directory and put their paths in a list. The files in dbfs directories can be listed using this command: dbutils.fs.ls(path). I recommend browsing through other functionalities of dbutils calling the dbutils.help() method.

1.2. Once you have the paths in a list, next step is to process the files in a format that we can use for data manipulation. For that we will be using Spark dataframes. So for step two, process all of the selected files into dataframes and return a list of dataframes. With spark we can read a file to a dataframe using this syntax: spark.read.format(file_format).load(). There are more functionalities available that will be required reading the csv files so try to look up the needed information in the documentation to get more familiar with them.

1.3. When the list of dataframes is already present, next step is to do some standardization: create a function that accepts an input string and removes invaid characters from it, also turns all characters of it to lower case and then return the transformed string. This function needs to be used on each column of each dataframe. The result of this should be a new list of dataframes with renamed columns. The pattern of the ivalid characters can be defined like this: pattern = r"[ ,;{}()\n\t=]". Use re (Regular Expression) library to process the string in this function and replace any occurence of invalid characters with an empty string.

1.4. Once you have the list of dataframes with standardized column names, next step is to join all of these dataframes into a single dataframe. After creating a single dataframe create a temprorary view out of it so it can be queried using SQL. A temprorary view can be created using the following method: df.createOrReplaceTempView(name).

1.5. Finally, once you have the temp view created from the unified dataframe, it is time to create a Delta talbe using a simple CTAS statement. To maintain a standard, let's name this table bronze_green_trip_data.

2. Create a bronze to silver notebook:

2.1. Silver layer should contained data that is already cleaned in comparisson to bronze layer. Let's say you have received the following requirements from your stakeholder for the silver table:

all columns in the table that contain more than 40% NULL values should be dropped;

data in the silver layer should not have duplicate records.

While working on these requirements, you can create a dataframe containing bronze layer data by reading the table using the following command: spark.table(table_name).

2.2. When you have a dataframe with the required columns dropped and duplicate records removed, create a temporary view so it can be queried with SQL.
2.3. Repeat the same step as in the bronze notebook and create the silver table. Use the following name: silver_green_trip_data.
3. Create multiple gold layer notebooks as per requirements:

3.1. This is the layer that will be visible by your stakeholders and any other downstream users. You have received a request to create a gold table called gold_green_trip_data. The requirements for this table are the following:
all column names should follow this naming convention: snake_case;
columns representing pickup time and dropoff time should be named 'pickup_datetime' and 'dropoff_datetime'. Also they should be in this format: 'yyyy-MM-dd HH:mm:ss';
a new column following the dropoff_datetime column should be created with trip duration in minutes, rounded to 1 decimal;
NULL values should be filtered out in all columns, inspect the data to see if there are any cases where we have NULLs.
Use SQL and store the code in a separate notebook.

3.2. Another request came in to create a gold table that represents aggregated monthly revenue grouped by vendor and payment type. Use the previously created gold table as the source for this table. This table should contain the following columns:
vendor_id
month ('yyyy-MM-01)
payment_type
monthly_total_amount
Use SQL and store the code in a separate notebook.

3.4. Once all the layers are created, to make the end to end process callable from one place, create a notebook that executes all of these notebooks in the correct order. There are a few ways to call a notebook in Databricks, but since we are using the community edition, we can only do that with the '%run /path' command.
Some tips:

since we have a small cluster, it is wise to use just a single file for development work as it will speed up execution and include all of the files when you have a working version of code;
if you have a python notebook and want to write some SQL in a new cell, you can use the magic command %sql at the top of the cell and vice versa;
if you're stuck - don't hesitate to ask for help :)
Bonus Tasks:
After completing the initial task, you can improve it by making it more dynamic so it can process files based on given parameters. It should meet the following requirements:

bronze notebook should accept four parameters:

table_name - the name which will be used when writing the table;
path_to_files - the dbfs location where files that require processing are stored;
path_slicer_from - starting point of slicer for picking files from the directory;
path_slicer_to - ending point of slicer for picking files from the directory.
silver notebook should accept three parameters:

bronze_table_name - bronze table name which will be used as the source;
null_threshold - the percentage which will be used as a threshold to drop column containing more nulls than it;
silver_table_name - name that will be used while writting the silver table.
Once bronze and silver have been parametrized, read 5 selected yellow trip data files and run them through the process. There should be one notebook that calls the bronze and silver notebooks passing the required parameters for yellow trip data.

Parametrize the green trip data processing in the same way in a separate notebook.

Create an extra gold layer table which combines the data of green and yellow that shows the aggregated monthly revenue grouped by vendor, payment type and taxi type. It should contain the following columns:

vendor_id
month ('yyyy-MM-01')
payment_type
monthly_total_amount
taxi_type (green or yellow)
Create a notebook that calls the finalized green data and yellow data notebooks, also it should call the gold layer tables afer processing yellow taxi and green taxi data through bronze and silver layers. A visualization of how this should look is attached in the repo as a picture file: 'end_to_end_process.png'
