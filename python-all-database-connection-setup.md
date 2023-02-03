/*
 *   Copyright (c) 2023 
 *   All rights reserved.
 */
# MYSQL CONNECTION
import mysql.connector
cnx = mysql.connector.connect(user='root', password='',
                              host='localhost',
                              database='test')

cursor = cnx.cursor()
cursor.execute("""""")


cnx.close()



# MSSQL CONNECTION 

import pyodbc 
cnxn = pyodbc.connect("Driver={SQL Server Native Client 11.0};"
                      "Server=localhost;"
                      "Database=testDB;"
                      "Trusted_Connection=yes;")
cursor = cnxn.cursor()

cursor.execute("""
SELECT * FROM Users;
""")

print(cursor.fetchall())

# SQLITE CONNECTION

import sqlite3

conn = sqlite3.connect("test.sqlite3")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE Users(
id int,
username varchar(25),
password varchar(25)
)
""")






# MONGODB CONNECTION
import pymongo


client = pymongo.MongoClient("mongodb://localhost/tryDB")
db = client["tryDB"]
print(db.list_collection_names())
comments = db["products"]
first = comments.find_one()
for i,(key,value) in enumerate(first.items()):
	print(key,value)



