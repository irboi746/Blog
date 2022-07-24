# Access Control Vulnerabilities
- Access control vulnerabilities is explained in this [link](https://portswigger.net/web-security/access-control). 
## Unprotected Admin Panel
- This lab has an unprotected admin panel. Solve the lab by deleting the user carlos.

### Enumeration
- First we run gobuster with `common.txt` as wordlist to enumerate .
![[Pasted image 20220723175738.png]]
- And from the enumeration we see that there is a robots.txt file. From the robots.txt file, we can see that there is an administration-panel that is disallowed.
![[Pasted image 20220723174552.png]]
### Vulnerability Assessment / Exploitation
- We attempt to checkout the administration panel and realised that no authentication needed for administration panel
![[Pasted image 20220723174617.png]]
- Proceeds to delete carlos.

## Unprotected admin functionality with unpredictable URL
 - This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application.
 - Solve the lab by accessing the admin panel, and using it to delete the user carlos.
### Enumeration
![[Pasted image 20220723181830.png]]
- View Page Source and found a script that reveals admin panel if certain conditions are met.
![[Pasted image 20220723180151.png]]
- Ran gobuster to enumerate the directories with `common.txt` as wordlist. Nothing interesting was found here.
![[Pasted image 20220723183901.png]]
### Vulnerability Assessment/Exploitation
- Using burp suite, intercept the response traffic and change `var isAdmin = false` to `var isAdmin=false`
![[Pasted image 20220723182139.png]]
- The page we will see is as below, an Admin panel tab is revealed. The directory to administration panel is `/admin-1t5ka4`
![[Pasted image 20220723181712.png]]
- We proceed to go to the administration panel and delete `carlos`.
![[Pasted image 20220723181754.png]]

## User role controlled by request parameter
 - This lab has an admin panel at `/admin`, which identifies administrators using a forgeable cookie.

- Solve the lab by accessing the admin panel and using it to delete the user carlos. You can log in to your own account using the following credentials: `wiener:peter`.

### Enumeration
- We intercept the traffic when navigating to the `/admin` directory and in the intercepted `GET` request we found that there is a cookie value `Admin=false`.
![[Pasted image 20220723184946.png]]
When the corresponding cookie value is used, the below response is returned.
![[Pasted image 20220723185007.png]]
### Vulnerability Assessment / Exploitation
- We attempt to reach the administration panel by manipulating the cookie value `Admin=false` to `Admin=true`.
![[Pasted image 20220723185050.png]]
- We arrive at the adminstration panel. However, we will need to constantly manipulate this `GET` request. 
![[Pasted image 20220723185108.png]]
- We proceed to delete carlos and change the cookie value from `Admin=false` to `Admin=true`
![[Pasted image 20220723185135.png]]

## User role can be modified in user profile
 - This lab has an admin panel at `/admin`. It's only accessible to logged-in users with a `roleid` of 2.

- Solve the lab by accessing the admin panel and using it to delete the user carlos. You can log in to your own account using the following credentials: `wiener:peter`

### Enumeration
- We first conduct our enumeration by querying `/admin` to see if there is any  `GET` request parameter that can be manipulated. 
![[Pasted image 20220723185836.png]]
![[Pasted image 20220723185854.png]]
As can be seen above, there is nothing interesting here. Just that, we know that we do not have the correct authorization.
- We then proceed to login with `wiener:peter` and we are lead to the page below.
![[Pasted image 20220723200915.png]]
- We used burp to intercept the traffic that is sent and threw it to repeater to check the response. 
- Oddly enough, the proxy was not able to catch the response below. 
![[Pasted image 20220723200838.png]]
- From the above response on the right, we see that there is the vulnerable parameter `roleid`. It seems like, the POST request on the right will be processed and redirected by the webserver.

### Vulnerability Assessment / Exploitation
- Therefore, we should be able to add the additional parameters into the original `POST` request and change `roleid: 1` to `roleid: 2` as seen below. We passed it to the HTTP request intercepted at the proxy.
![[Pasted image 20220723213243.png]]
- Below, we can see that we have successfully bypassed the authorization, an we now have the `Admin panel` tab.
![[Pasted image 20220723213300.png]]
- After this, we proceed to delete `carlos`
![[Pasted image 20220723213334.png]]

## User ID controlled by request parameter
-  This lab has a horizontal privilege escalation vulnerability on the user account page.
- To solve the lab, obtain the API key for the user `carlos` and submit it as the solution. You can log in to your own account using the following credentials: `wiener:peter`

### Enumeration
- Logged into the account and saw that the URL has a parameter `my-account?id=wiener`.
![[Pasted image 20220723214825.png]]
- Additionally, the link to `My Account` tab is referenced to `/my-account?id=wiener`
![[Pasted image 20220723215022.png]]
![[Pasted image 20220723214957.png]]
### Vulnerability Assessment / Exploitation
- Hence we just simply need to change `wiener` to `carlos`.
![[Pasted image 20220723215113.png]]
- The API key `koxW0InLMGBNCkwGej2BduZUgmSi8NAF` is leaked.
![[Pasted image 20220723215129.png]]

## User ID controlled by request parameter, with unpredictable user IDs
 - This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with `GUIDs`.

- To solve the lab, find the `GUID` for `carlos`, then submit his API key as the solution. You can log in to your own account using the following credentials: `wiener:peter`. 

### Enumeration
- Upon looking at the intercepted response, we see that the account id is no longer the name but a string of numbers.
![[Pasted image 20220723215751.png]]
- However, despite that, it is still vulnerable to the attack used above, just that we will need to find out the account ID of `carlos`. 
- Upon further exploration of the blog, we found out that the users post on the blog and their user account is referenced with a link.
![[Pasted image 20220724164457.png]]
- On top of that, the account ID is in the URL when we clicked on the referenced link.
![[Pasted image 20220724164533.png]]
- We found a post written by carlos and therefore also found his account ID `c5899ede-9ba5-4237-811d-ec61698c2799`
![[Pasted image 20220724164713.png]]
### Vulnerability Assessment / Exploitation
Now to exploit this, we simply need to append `/my-account?id=c5899ede-9ba5-4237-811d-ec61698c2799` like below and we will get to the my-account page.
![[Pasted image 20220724164954.png]]
![[Pasted image 20220724165035.png]]
API key : `LGd3ou8CqEGN9g2LU57t5usEbRVoXmvt`

## User ID controlled by request parameter with data leakage in redirect
 - This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.
- To solve the lab, obtain the API key for the user carlos and submit it as the solution. You can log in to your own account using the following credentials: `wiener:peter` 
### Enumeration
- We initially logged in as `wiener` and realised that the account ID is simply the name and it is parsed through the URL.
![[Pasted image 20220724170000.png]]
### Vulnerability Assessment / Exploitation
- With this, we attempted to simply just change the account id to carlos. 
![[Pasted image 20220724170204.png]]
- In the response, before the redirection to the Login Page, we get a leaked `carlos` account page as seen below.
![[Pasted image 20220724170306.png]]
API Key is : `1b27kaCW9WcohYtc9HktRdA01s16ILOl`

## User ID controlled by request parameter with password disclosure
- This lab has user account page that contains the current user's existing password, prefilled in a masked input.
- To solve the lab, retrieve the administrator's password, then use it to delete carlos. You can log in to your own account using the following credentials: `wiener:peter`. 

### Enumeration
- We log in with the given credentials wiener:peter and we get to this page where we see that there is a mask for the password.
![[Pasted image 20220724183230.png]]
- However, upon inspecting page source, we see that the password is in plain.
![[Pasted image 20220724183333.png]]
- Additionally, we found that the `href` of 'My Account' tab is 
 ![[Pasted image 20220724183701.png]]

### Vulnerability Assessment / Exploitation
- Hence we might be able to bypass authentication and head straight to `/my-account?id=administrator`
![[Pasted image 20220724184055.png]]
- We view page source to reveal the `administrator` password : `pz8t43px3spymeww275t`
![[Pasted image 20220724183837.png]]
- We login as administrator and we go proceed to delete carlos from Admin Panel
![[Pasted image 20220724184020.png]]

## Insecure Direct Object References
-  This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.
- Solve the lab by finding the password for the user carlos, and logging into their account. 

### Enumeration
- We navigate to the "Live Chat" page from the "Live Chat" tab.
![[Pasted image 20220724203234.png]]
- We see two functions "Send" and "View Transcript".
![[Pasted image 20220724205134.png]]
- We visit `viewTranscript.js` and get the below script. The script seems to be suggesting that there is a download action.
![[Pasted image 20220724205225.png]]
- As we can see below, a `POST` HTTP request is sent
![[Pasted image 20220724203440.png]]
- Then we get a `GET` request to a web directory `/download-transcript/3.txt` 
![[Pasted image 20220724203456.png]]
- Then a download pop-up pops out. Seems like, the "View Transcript" function is downloading from that particular directory.
- There also seems to be a pattern in the transcript numbering i.e `3.txt`
### Vulnerability Assessment / Exploitation
- We try to see if there are any text files prior to out transcript.
![[Pasted image 20220724210400.png]]
- It seems like there is a file by the number of `1.txt`.
![[Pasted image 20220724210416.png]]
- Content of 1.txt is below : 
![[Pasted image 20220724210501.png]]
- So it seems like there is a user whose password is : `i2iotr5lot10za1mf9pb`
- We can assume it's carlos for lab purposes and true enough, it is carlos's account password.
![[Pasted image 20220724210632.png]]
