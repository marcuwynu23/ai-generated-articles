# SQLMAP Simple Workflow

SQLmap is an open-source penetration testing tool used to detect and exploit SQL injection vulnerabilities in web applications. Here is an example of how to use SQLmap:

Install SQLmap:
SQLmap is a Python-based tool, and it can be installed on various operating systems. The easiest way to install SQLmap is by using pip, a package manager for Python. Open a command prompt and type the following command:

```sh
pip install sqlmap
```

Identify the target:
The first step is to identify the target web application. You can use various tools like Burp Suite, OWASP ZAP, or simply open the website in a browser and inspect the HTML code. Once you have identified the target URL, copy it to your clipboard.

Scan for vulnerabilities:
Open a command prompt and type the following command, replacing "target URL" with the URL you copied earlier:

```sh
sqlmap -u "target URL" --dbs
```

This command tells SQLmap to scan the target URL for SQL injection vulnerabilities and retrieve the names of the available databases. If SQLmap detects any vulnerabilities, it will display a list of the databases that it found.

Retrieve database information:
Once SQLmap has detected a vulnerable target and identified the available databases, you can use it to retrieve information about the databases. For example, to retrieve the tables in a database, use the following command:

```sh
sqlmap -u "target URL" -D "database name" --tables
```

This command tells SQLmap to use the identified database and retrieve the names of the tables in it. You can replace "database name" with the name of the database that you want to retrieve the tables from.

Dump data from a table:
Once you have identified the tables in a database, you can use SQLmap to dump the data from a table. For example, to dump the data from a table called "users", use the following command:

```sh
sqlmap -u "target URL" -D "database name" -T "table name" --dump
```

This command tells SQLmap to use the identified database and table and retrieve the data from it. You can replace "table name" with the name of the table that you want to dump data from.

Note: It's essential to use SQLmap only on web applications that you have permission to test. Otherwise, you may be breaking the law by accessing someone else's data without their consent.
