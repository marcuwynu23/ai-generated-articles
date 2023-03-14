# All .env Database Configuration

The .env file is a configuration file used in Laravel and other PHP applications to define various environment variables. In this file, there is a section that is dedicated to database configuration. Here we will explain the different types of database configuration and what each of them does.

### MySQL

```
  DB_CONNECTION=mysql
  DB_HOST=127.0.0.1
  DB_PORT=3306
  DB_DATABASE=laravel
  DB_USERNAME=root
  DB_PASSWORD=
```

eb applications. In this configuration, the DB_CONNECTION variable is set to mysql to indicate that we are using MySQL. The DB_HOST variable specifies the IP address of the MySQL server, which in this case is 127.0.0.1, the default IP address for the local machine. The DB_PORT variable specifies the port number used by MySQL, which is 3306 by default. The DB_DATABASE variable specifies the name of the database we want to connect to. In this case, the name of the database is laravel. The DB_USERNAME variable specifies the username used to connect to the database, which in this case is root. Finally, the DB_PASSWORD variable specifies the password used to connect to the database, which is an empty string in this case.

### Microsoft SQL Server

```
DB_CONNECTION=sqlsrv
DB_HOST=127.0.0.1
DB_PORT=1433
DB_DATABASE=testDB
DB_USERNAME=marcu
DB_PASSWORD=admin
```

Microsoft SQL Server is a popular database management system used in many enterprise-level applications. In this configuration, the DB_CONNECTION variable is set to sqlsrv to indicate that we are using Microsoft SQL Server. The DB_HOST variable specifies the IP address of the SQL Server, which is 127.0.0.1 in this case. The DB_PORT variable specifies the port number used by SQL Server, which is 1433 by default. The DB_DATABASE variable specifies the name of the database we want to connect to, which in this case is testDB. The DB_USERNAME variable specifies the username used to connect to the database, which is marcu. Finally, the DB_PASSWORD variable specifies the password used to connect to the database, which is admin.

### PostgreSQL

```
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=test_laravel
DB_USERNAME=marcu
DB_PASSWORD=admin
```

PostgreSQL is an open-source relational database management system used in many web applications. In this configuration, the DB_CONNECTION variable is set to pgsql to indicate that we are using PostgreSQL. The DB_HOST variable specifies the IP address of the PostgreSQL server, which is 127.0.0.1 in this case. The DB_PORT variable specifies the port number used by PostgreSQL, which is 5432 by default. The DB_DATABASE variable specifies the name of the database we want to connect to, which in this case is test_laravel. The DB_USERNAME variable specifies the username used to connect to the database, which is marcu. Finally, the DB_PASSWORD variable specifies the password used to connect to the database, which is admin.

### Sqlite

```
DB_CONNECTION=sqlite
DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=
```

SQLite is a self-contained, serverless, zero-configuration, transactional
