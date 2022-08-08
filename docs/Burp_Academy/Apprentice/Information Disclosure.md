# Information Disclosure
- Accidental information disclosure by developers provide a wealth of information for pentesters. As such, there are certain enumeration steps we can take to try and find if there is such information disclosed.
- More on information disclosure can be found [here](https://portswigger.net/web-security/information-disclosure/exploiting).
## Information disclosure in error messages
- This lab's verbose error messages reveal that it is using a vulnerable version of a third-party framework. To solve the lab, obtain and submit the version number of this framework. 

### Enumeration
- We are given a page like below.
![[Pasted image 20220808155903.png]]
- Does not contain anything interesting and the HTTP traffic intercepted did not show anything that is interesting.
![[Pasted image 20220808160009.png]]
- Therefore the only way we can induce error would be to go to a product and key in an incorrect parameter.
![[Pasted image 20220808155833.png]]
- As can be seen below, the vulnerable Apache version is revealed and with the verbose output we also know that Java is used at the backend. 
![[Pasted image 20220808155823.png]]
![[Pasted image 20220808160228.png]]

## Information disclosure on debug page
- This lab contains a debug page that discloses sensitive information about the application. To solve the lab, obtain and submit the SECRET_KEY environment variable. 
### Enumeration
- We are given a webpage below and it seems like there is not much functions to it.
![[Pasted image 20220808161954.png]]
![[Pasted image 20220808161934.png]]
- Webpage does not seem to have any `robots.txt`.
![[Pasted image 20220808162346.png]]
- However, when we view source, we notice a comment with reference to a phpinfo page.
![[Pasted image 20220808161907.png]]
- We visited the directory and found the phpinfo page
![[Pasted image 20220808162139.png]]
![[Pasted image 20220808162156.png]]
- Searched for `SECRET_KEY` in the environment variable table.
![[Pasted image 20220808162225.png]]
- Submitted the `SECRET_KEY`.
![[Pasted image 20220808162300.png]]

## Source code disclosure via backup files
- This lab leaks its source code via backup files in a hidden directory. To solve the lab, identify and submit the database password, which is hard-coded in the leaked source code. 
### Enumeration
- We are given a webpage like below, does not look like it has anything special in the home and product page.
![[Pasted image 20220808162848.png]]
![[Pasted image 20220808162904.png]]
- The page source shows that there are directories that can be directly accessed.
![[Pasted image 20220808163043.png]]
- We check if there is `robots.txt` and there is. `robots.txt` shows that there is a disallowed directory called `/backup`. 
![[Pasted image 20220808162835.png]]
- Gobuster output confirms the existence of `/backup`.
![[Pasted image 20220808163230.png]]
- We visit the /`backup` page and it is a web index with a file named `ProductTemplate.java.bak`. which seems like a backup file.
![[Pasted image 20220808163303.png]]
![[Pasted image 20220808163438.png]]
- Upon inspection of the file, we see that it is a backup source code that displays product.
- We can focus more on `readObject` function which contains a declaration of a `connectionBuilder` object. 
- The parameters passed into the object are `postgresql`, `localhost`, `5432` and `oi8qw3vzd5yjvwd2l5c5equzlwupo9vh`.
- These parameters seems to be the hard-coded configuration to connect to the database (postgresql in localhost and at port 5432) `oi8qw3vzd5yjvwd2l5c5equzlwupo9vh` is the database password. 

## Authentication bypass via information disclosure
- This lab's administration interface has an authentication bypass vulnerability, but it is impractical to exploit without knowledge of a custom HTTP header used by the front-end.

- To solve the lab, obtain the header name then use it to bypass the lab's authentication. Access the admin interface and delete Carlos's account. You can log in to your own account using the following credentials: `wiener:peter`. 

### Enumeration
- We are given the website as below. The products and the webpage itself as well as their HTML source did not have anything interesting 
![[Pasted image 20220808165630.png]]
![[Pasted image 20220808165837.png]]
![[Pasted image 20220808165808.png]]
![[Pasted image 20220808165905.png]]
![[Pasted image 20220808165919.png]]
- We go a step further to enumerate HTTP methods used by the web server, we have written a script to manually test each HTTP methods (nmap can be used to do this as well) :

```python
import requests
from bs4 import BeautifulSoup

HTTP_Methods = ['GET','POST','TRACE','OPTIONS','PUT','DELETE','CONNECT','HEAD']

for i in HTTP_Methods:
        response = requests.request(i, 'https://0a97005a039d3ab9c0bd9ef2001e006c.web-security-academy.net/')
        soup = BeautifulSoup(response.text, 'html.parser') 
        print('Method Used: ' + i + '\n' + '-'*20)
        print(soup.prettify())

```

![[Pasted image 20220808183510.png]]
- From the `TRACE` request we realise that there is a `X-Custom-IP-Authorization` header which points to our own IP address.

### Vulnerability Assessment / Exploitation
- Hence it seems like this `X-Custom-IP-Authorization` header is used to determine whether the request is made from localhost or remote host. 
- Using Burp Repeater, we repeated the HTTP requests with the new header like below and we managed to bypass the Header Authentication.
![[Pasted image 20220808184500.png]]
![[Pasted image 20220808184510.png]]
- We can make use of Burp Proxy Options "Match and Replace" to constantly add the header into our future requests.
![[Pasted image 20220808184724.png]]
![[Pasted image 20220808184709.png]]
- We remove carlos to complete the lab. 
