# Working-on-Snowflake-using-Python
Working on Snowflake using python

In this GIT I am going to create a framework for importing data from excel to Snowflake using Python. 

Prerequisites
---------------
1. A Snowflake subscription with login, password, warehouse for the user
2. Snowsql installed on the machine where code will be executed
3. Python library : pip install snowflake-connector-python
4. Python library : pip install pandas
5. Python library : pip install logging

Requirement
--------------
Lets say we have data in an excel file in following format:

OwnerID   | BuildingID   | DeviceID
----------|--------------|-----------
Owner_001 |	100	         | sensor123


To upload the data into snowflake we need to perform following steps:
---------------------------------------------------------------------
1. Create a Database, Schema, Table, internal stage in Snowflake (If does not exist)

```sql
   CREATE DATABASE demo_db;
```
```sql
   CREATE SCHEMA alarmdeliverysys;
```
```sql
   CREATE TABLE devices(
   OwnerID VARCHAR(20),
   BuildingID  NUMBER,
   DeviceID VARCHAR(20)
   )
```
```sql
   CREATE STAGE pythonstage;
```

2. Make sure that the user/role has access to the database,schema,table, stage objects ***

3. Replace the variables with your variables in following python script and execute:

```python
   
import snowflake.connector as sf
from snowflake.connector import DictCursor
import pandas as pd
import operator
import os
import logging
from datetime import datetimd

LOG_FILENAME = datetime.now().strftime('logfile_%d_%m_%Y.log')
format_str = '%(asctime)s\t%(levelname)s -- %(processName)s %(filename)s:%(lineno)s -- %(message)s'

#create a custome logger
logger1 = logging.getLogger(__name__)
#create handlers
handler = logging.FileHandle(filename=LOG_FILENAME)
#create handler and add it to handler
handler.setFormatter(logging.Formatter(format_str))
#add handlers to the logger
logger1.addHandler(handler)
Logger1.setLevel(logging.DEBUG)

for logger_name in ['snowflake','botocore']:
   logger = logging.getLogger(logger_name)
   logger.setLevel(logging.WARNING)
   ch = logging.FilleHandler('python_connector.log')
   ch.setLevel(logging.warning)
   ch.setFormatter(logging.Formatter('%(asctime)s - %(threadname)s %(filename)s:%(lineno)d - %(funcname)s - %(levelname)s - %(message)s'))

#update your username, password, snowflake account, database, virtual warehouse
username = 'sfuser1'
password = 'password123'
account = 'hn85515.west-us-2.azure'
warehouse = 'compute_WH'
database = 'azuredatabase' 
schema = ''
tablename = ''
conn = sf.connect(user=username, password=password, account=account)

def execute_query(connection,query):
    cursor=connection.cursor()
    cursor.execute(query)
    cursor.close()


try:
    #Select database to use            
    sql='use {}'.format(database)
    execute_query(conn,sql)

    #select Virtual VM       
    sql = 'use warehouse {}'.format(warehouse)
    execute_query(conn,sql)

    #Awaken the virtual VM
    try:
        sql='alter warehouse {} resume'.format(warehouse)
        execute_query(conn,sql)
    except:
        pass

    #Select Query               
    sql='select * from devicestable'
    cursor=conn.cursor(DictCursor)
    cursor.execute(sql)
    for c in cursor:
        print(c)
    cursor.close()

    # Copy data to a stage in snowflake from local
    # Make sure that the snawflake stage named PYTHONSTAGE is already created
    csv_file = "C:\\Users\\DELL\\Desktop\\HCL\\snowflake\\DevicesTable.csv"
    
    sql = 'put file://{0} @{1} auto_compress=true'.format(csv_file, "PYTHONSTAGE")
    execute_query(conn,sql)

    # Load data into DEVICESTABLE (make sure that the table exists)
    sql ="COPY into {0} from @{1}/DevicesTable.csv.gz FILE_FORMAT=(TYPE=csv field_delimiter=',' skip_header=1) ON_ERROR = 'ABORT_STATEMENT' ".format('DEVICESTABLE', "PYTHONSTAGE")
    execute_query(conn,sql)
except Exception as e:
    print(e)

```

***applicable only if the user who is uploading the files is different from the user who is working on Snowflake GUI. In some organizations a separate application account is created for access to snowflake from external application while GUI access is through Single-Sign ON.
