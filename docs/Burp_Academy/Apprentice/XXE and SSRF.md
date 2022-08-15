# XXE and SSRF
- From the embedded link we can understand a little more about [XXE](https://portswigger.net/web-security/xxe/xml-entities) (XML External Entity) as well as [XXE injection](https://portswigger.net/web-security/xxe). 
- It is said also that XXE attack can be escalated and used to compomise server side or back-end infrastructure like [SSRF](https://portswigger.net/web-security/ssrf) (Server Side Request Forgery).

## Exploiting XXE using external entities to retrieve files
-  This lab has a "Check stock" feature that parses XML input and returns any unexpected values in the response.
- To solve the lab, inject an XML external entity to retrieve the contents of the `/etc/passwd` file. 

### Enumeration
- We are given a website like below. It does not have muc functions but a "Home" button at the top and "View Details" button which will lead us to the product.

![[Pasted image 20220814210605.png]]

- In the product page, we see a "Check stock" button and its relevant form and javascript.

![[Pasted image 20220814210621.png]]
![[Pasted image 20220814210710.png]]
![[Pasted image 20220814210936.png]]
![[Pasted image 20220814210954.png]]
- Below is an intercepted `POST` request and we see that the generated XML entity query. 
![[Pasted image 20220814211308.png]]

### Vulnerability Assessment / Exploitation
- We can manipulate the query as below to call for an external entity `xxe` to read, parse and store the `/etc/passwd` as `xxe`. 

![[Pasted image 20220814211537.png]]

- XXE attack was successful and the `/etc/passwd` file of the server was revealed.

![[Pasted image 20220814211603.png]]

## Exploiting XXE to perform SSRF attacks
- This lab has a "Check stock" feature that parses XML input and returns any unexpected values in the response.
- The lab server is running a (simulated) EC2 metadata endpoint at the default URL, which is `http://169.254.169.254/`. This endpoint can be used to retrieve data about the instance, some of which might be sensitive.
- To solve the lab, exploit the XXE vulnerability to perform an SSRF attack that obtains the server's IAM secret access key from the EC2 metadata endpoint. 

### Enumeration
- We get a web page with similar function as the one given above.
![[Pasted image 20220814212648.png]]
![[Pasted image 20220814212851.png]]
![[Pasted image 20220814212903.png]]
![[Pasted image 20220814212952.png]]

- So we get to this familiar `POST` request and we send it to Burp Repeater to manipulate the request.

![[Pasted image 20220814213015.png]]

### Vulnerability Assessment / Exploitation
- We can see an excerpt from the [link](https://www.emergingdefense.com/blog/2019/1/16/abusing-aws-metadata-service) below that Instance Metadata Service is a standard API.
![[Pasted image 20220814213409.png]]
- Hence we will need to craft our payload like below for an XXE attack that will result into SSRF which will enumerate the users that have the credentials.

```html
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials"> ]>
```

- From below, we can see that there is one user `admin` with credentials.

![[Pasted image 20220814214049.png]]

- Hence we craft the next payload as below to reveal the credentials that is stored.

```html
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/admin"> ]>
```

- Credentials are stolen.

![[Pasted image 20220814214146.png]]
![[Pasted image 20220814214157.png]]

## Basic SSRF against the local server
- This lab has a stock check feature which fetches data from an internal system.
- To solve the lab, change the stock check URL to access the admin interface at http://localhost/admin and delete the user carlos. 

### Enumeration
- We are given a website like below. It has a few functions which is a "My account" for logging in, "Home" to go back to Home page and "View details" to see the details of each product.


![[Pasted image 20220815103626.png]]

- We are not given credentials, and there is no registration or forget password hence logging in might not be the way

![[Pasted image 20220815104625.png]]

- We go to view details to a random product and we see "Check stock" function.

![[Pasted image 20220815103652.png]]

- We intercepted the `POST` request and in the Body of the request we see the below parameter `stockApi` transmitted.

![[Pasted image 20220815104714.png]]

- The data is url encoded and it leads to an internal webserver. It seems like this `stockApi` parameter is able to do a request to a webserver.

![[Pasted image 20220815104725.png]]

### Vulnerability Assessment / Exploitation
- As the admin function is within the localhost, we appended the following line below to delete the user `carlos`.

![[Pasted image 20220815105057.png]]

- Request succeeded and carlos was deleted.

![[Pasted image 20220815105129.png]]

## Basic SSRF against another back-end system
 - This lab has a stock check feature which fetches data from an internal system.
- To solve the lab, use the stock check functionality to scan the internal `192.168.0.X` range for an admin interface on port 8080, then use it to delete the user carlos. 

### Enumeration
- We receive a similar webpage as above, hence we will just go to the intercepted HTTP traffic.

![[Pasted image 20220815105822.png]]

- The "Login" traffic is still the same, nothing special.

![[Pasted image 20220815105751.png]]

- However, of note would be the 'Check stock' function again, where this time, instead of a domain name, an IP address is called instead. We know that this IP address does not have the admin page (stated by the question).

![[Pasted image 20220815105804.png]]

- Hence we will need to enumerate the IP addresses with `stockApi`. We first used Burp Repeater to find out the difference in reponse between a valid web server and a webserver that does not exist.

![[Pasted image 20220815110044.png]]

- As can be seen below, "Could not connect to external stock check service" is printed on the screen and the portswigger default page is displayed if server does not exist.

![[Pasted image 20220815110104.png]]

- Armed with this knowledge, we sent this `POST` request to Burp Intruder and set the end of the IP address as the variable payload.

![[Pasted image 20220815111223.png]]

- We created a python script below to generate IP addresses from 1-255.

```python
for x in range(1,256): 
   f = open("range.txt","a")
   f.write(str(x) + "\n")
   f.close()
```

- Then we loaded the text file as payload.

![[Pasted image 20220815111325.png]]

- As can be seen below, if webserver does not exist, Reponse status 500 would be printed and if webserver exists, it will either be 200 or 404.

![[Pasted image 20220815111344.png]]

- With the above results, we know that `192.168.0.31` is the likely target. send a POST request to the `/admin` page and as can be seen below, we see the admin panel.

![[Pasted image 20220815111428.png]]

### Vulnerability Assessment / Exploitation
- We proceed to delete the user `carlos`.

![[Pasted image 20220815111503.png]]