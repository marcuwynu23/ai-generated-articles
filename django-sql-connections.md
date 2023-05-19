# SQL connections for PostgreSQL, MS SQL Server, Oracle DB, and MySQL in Django.

### PostgreSQL:

First, install psycopg2 driver: pip install psycopg2
In your Django settings file, set the database configuration:

```py
DATABASES = {
'default': {
'ENGINE': 'django.db.backends.postgresql',
'NAME': '<database name>',
'USER': '<database username>',
'PASSWORD': '<database password>',
'HOST': '<database host>',
'PORT': '<database port>',
}
}
```

### MS SQL Server:

First, install pyodbc driver: pip install pyodbc
In your Django settings file, set the database configuration:

```py
DATABASES = {
'default': {
'ENGINE': 'sql_server.pyodbc',
'NAME': '<database name>',
'USER': '<database username>',
'PASSWORD': '<database password>',
'HOST': '<database host>',
'PORT': '<database port>',
'OPTIONS': {
'driver': 'ODBC Driver 17 for SQL Server', # Replace with your ODBC driver name
},
}
}
```

### Oracle DB:

First, install cx_Oracle driver: pip install cx_Oracle
In your Django settings file, set the database configuration:

```py

DATABASES = {
'default': {
'ENGINE': 'django.db.backends.oracle',
'NAME': '<database name>',
'USER': '<database username>',
'PASSWORD': '<database password>',
'HOST': '<database host>',
'PORT': '<database port>',
}
}
```

MySQL:
First, install mysqlclient driver: pip install mysqlclient
In your Django settings file, set the database configuration:

```py
DATABASES = {
'default': {
'ENGINE': 'django.db.backends.mysql',
'NAME': '<database name>',
'USER': '<database username>',
'PASSWORD': '<database password>',
'HOST': '<database host>',
'PORT': '<database port>',
}
}
```

Remember to replace <database name>, <database username>, <database password>, <database host>, and <database port> with your own database information.
