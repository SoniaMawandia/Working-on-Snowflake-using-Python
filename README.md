# Working-on-Snowflake-using-Python
Working on Snowflake using python

In this GIT I am going to create a framework for importing data from excel to Snowflake using Python. 

Prerequisites
---------------
1. A Snowflake subscription with login, password, warehouse for the user
2. Snowsql installed on the machine where code will be executed
3. python library : pip install snowflake-connector-python

Requirement
--------------
Lets say we have data in an excel file in following format:

OwnerID   | BuildingID   | DeviceID
----------|--------------|-----------
Owner_001 |	100	         | sensor123


To upload the data into snowflake we need to perform following steps:
---------------------------------------------------------------------
1. Create a Database, Schema, Table in Snowflake (If does not exist)

```sql
   CREATE DATABASE demo_db;
```
```sql
   CREATE SCHEMA alarmdeliverysys;
```
```sql
   CREATE TABLE devices(
    
   )
```
