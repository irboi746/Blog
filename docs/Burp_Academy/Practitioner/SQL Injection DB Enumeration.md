# SQL Injection DB Enumeration
- For this lab reference to this database version [cheatsheet](https://portswigger.net/web-security/sql-injection/cheat-sheet#database-version) would be the most useful. We will craft our payload based on the cheatsheet given.

## SQL injection attack, querying the database type and version on Oracle
- To solve the lab, display the database version string. 

### Enumeration
- We are given a web app like below, only has "Home" and "Filter" function where "Filter" function is the only one that accepts parameter in the GET request.

#### Mapping Application Content
![[Pasted image 20220825132354.png]]
![[Pasted image 20220825132432.png]]
![[Pasted image 20220825132415.png]]

#### Analysing Attack Surface
- From the above we can see that the above web application has only one attack surface which is the "Filter" function.

### Vulnerability Assessment / Exploitation
- Therefore we will test if the "Filter" function is vulnerable to SQL injection. We will use the payload list used in the previous exercise [[SQL Injection UNION Attack#SQL injection UNION attack, retrieving multiple values in a single column]].

- We set the position of the payload in Burp Intruder as below : 

![[Pasted image 20220825134032.png]]

- It seems like simply using `NULL` value does not trigger a `200 OK` response

![[Pasted image 20220825133014.png]]

- All other payloads also gives `error 500` which means that their database type does not understand the query.

![[Pasted image 20220825133346.png]]

- But when `null, banner FROM v$version` payload is used in place of `NULL`, a `200 OK` response is triggered and the version of the database is displayed.
![[Pasted image 20220825133310.png]]

- With the above information, we can ascertain that the database used is Oracle DB as the payload is a query in the Oracle DB Query Language.

## SQL injection attack, querying the database type and version on MySQL and Microsoft
### Enumeration
- The web application given is similar to [[#SQL injection attack querying the database type and version on Oracle]] where only two functions "Home" and "Filter" are available. Hencew we will not do any futher enumeration but focus on the exploitation.

### Vulnerability Assessment / Exploitation
- Likewise, we will use the same payload list used above as well as burp intruder to enumerate the database version.

- We set the position of the payload in Burp Intruder as below : 

![[Pasted image 20220825134032.png]]

- It seems like none of the payloads work.

![[Pasted image 20220825134232.png]]

- Based on [this](https://portswigger.net/web-security/sql-injection/cheat-sheet#comments) we attempt to enumerate with an additional payload as below : 

```
' UNION SELECT @@version #     
' UNION SELECT null,@@version # 
' UNION SELECT @@version,null #
' UNION SELECT null,null,@@version #
' UNION SELECT null,@@version,null #
' UNION SELECT @@version,null,null #
```

![[Pasted image 20220825134645.png]]

- True enough, when we changed `--` to `#` as the comment line, `200 OK` response is received and the database version is enumerated.

![[Pasted image 20220825134802.png]]

- With this we can ascertain that the database used is MySQL as the comment parameter used is `#`.

## SQL injection attack, listing the database contents on non-Oracle databases
- The application has a login function, and the database contains a table that holds usernames and passwords. You need to determine the name of this table and the columns it contains, then retrieve the contents of the table to obtain the username and password of all users.
- To solve the lab, log in as the administrator user. 

### Enumeration
#### Mapping Application Content
- As we analyse the sitemap after browsing the application manually and filling the forms, we notice that there is only 2 functions in this application - Login and Filter
 
![[Pasted image 20220825135339.png]]
![[Pasted image 20220825135648.png]]

- The Login function takes a POST request and the Login and Password parameters are passed in the Body of the request. 

![[Pasted image 20220825140134.png]]

- An Invalid Usernane or password is returned when invalid username or password is entered.

![[Pasted image 20220825135622.png]]

- The Filter function uses a GET request and has the parameter is passed in the URL itself.

![[Pasted image 20220825140305.png]]
![[Pasted image 20220825140321.png]]

- Seems like URL encoding is used.

![[Pasted image 20220825140516.png]]

#### Analysing Attack Surface
- There is a "Login" function where we can test for SQL injection login bypass.
- There is a "Filter" function where we can use the payload used above to check if it is vulnerable to SQL injection and the database used.

### Vulnerability Assessment
- Hence we will seek to test the Login and Filter function.

#### Testing for Login Bypass in Login Function
- For testing Login Bypass we will use the payload [Auth_Bypass.txt](https://github.com/payloadbox/sql-injection-payload-list/blob/master/Intruder/exploit/Auth_Bypass.txt).

![[Pasted image 20220825140644.png]]

- We will test out the username parameter first then the password parameter with Burp Intruder.

![[Pasted image 20220825141022.png]]

- We noticed that the RESPONSE length for incorrect login is `3515` and based on that, after iterating through the entire payload, "username" and "password" parameter does not give us any successful bypass.
- Therefore we can conclude that Login function is not vulneable to login bypass SQLi.

#### Testing Filter Function for UNION SQLi
- Next we will follow with attacking the "Filter" function. Payload used be the one used in exercises above for UNION injection.

![[Pasted image 20220825141430.png]]

- As can be seen below, `null, null` gives a `200 OK` response hence we know we have 2 columns to play around with.

![[Pasted image 20220825141450.png]]

##### Database Version Enumeration
- Next we also found out that, the database used is PostgreSQL. On top of that, both columns are able to store strings.

![[Pasted image 20220825141615.png]]
![[Pasted image 20220825141710.png]]

##### Database Tables Enumeration
- Hence we continue with enumerating the tables in the databse with the query below.

```
' UNION SELECT table_name,table_type FROM information_schema.tables --
```

- We then come across a unusual table name `users_dcgeuu`

![[Pasted image 20220825142114.png]]

##### Column Enumeration of Table
- We then attempt to enumerate for the columns in the table with the following command : 

```
' UNION SELECT column_name,data_type FROM information_schema.columns WHERE table_name = 'users_dgceuu' --
```

- It seems this table only has two columns, `password_zviyrv` and `username_bsdnky`

![[Pasted image 20220825142301.png]]
![[Pasted image 20220825142312.png]]

### Exploitation
- We can now move on leaking the usernames and passwords with the following command:  

```
' UNION SELECT username_bsdnky,password_zviyrv FROM users_dgceuu --
```
![[Pasted image 20220825142734.png]]

- The administrator username and password is found and we login to the adminsitrator page to complete the challenge.

![[Pasted image 20220825142746.png]]
![[Pasted image 20220825142812.png]]

## SQL injection attack, listing the database contents on Oracle
### Enumeration
- We will skip right to the UNION injection phase of vulnerability assessment as the application given here is similar to [[#SQL injection attack listing the database contents on non-Oracle databases]] and that the difference is only in the type and version of database used.

### Vulnerablity Assessment
- It seems like the database is not reactive to 'NULL' values.
![[Pasted image 20220825151150.png]]

#### Database Version Enumeration
- Running the `banner FROM v$version`  query gives us `200 OK`. This signifies that Oracle DB is in use. 
- From the payload, banner is displayed in both columns. Hence we know that there are two columns and both can display strings. 

![[Pasted image 20220825151205.png]]

#### Database Table Enumeration
Since we know it is Oracle DB, we will use payloads unique to Oracle DB based on its [documentation](https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_2105.htm#REFRN20286) : 

```
' UNION SELECT table_name,tablespace_name FROM all_tables--
```

- The payload above, will list all the tables that are in the database.

![[Pasted image 20220825152325.png]]

- As can be seen below, there is a non-default table `USERS_VDYLFP`.

![[Pasted image 20220825154548.png]]

#### Column Enumeration of Table
We then follow this [all_tab_column documentation](https://docs.oracle.com/cd/B19306_01/server.102/b14237/statviews_2094.htm) to enumerate columns in the table we found.

```
' UNION SELECT column_name,data_type FROM all_tab_columns WHERE table_name = 'USERS_VDYLFP' --
```

- As can be seen there are two interesting columns `USERNAME_CNBWLE` and `PASSWORD_PQFZP`

![[Pasted image 20220825154659.png]]![[Pasted image 20220825154713.png]]

### Exploitation
- Now it is time to leak the rows in the columns which we will use the following query : 

```
' UNION SELECT USERNAME_CNBWLE,PASSWORD_PVQFZP FROM USERS_VDYLFP--
```

- Administrator username and password is found and we log in to the account to complete the challenge.

![[Pasted image 20220825155103.png]]
![[Pasted image 20220825155201.png]]