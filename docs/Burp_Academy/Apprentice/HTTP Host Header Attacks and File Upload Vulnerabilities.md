# HTTP Host Header Attacks & File Upload Vulnerabilities
## HTTP Host Header Attacks
- More on host header can be understood [here](https://portswigger.net/web-security/host-header) and testing for host header vulnerabilities can be understood [here](https://portswigger.net/web-security/host-header/exploiting).
### Basic Password Reset Poisoning
 - This lab is vulnerable to password reset poisoning. The user `carlos` will carelessly click on any links in emails that he receives. To solve the lab, log in to Carlos's account.
- You can log in to your own account using the following credentials: `wiener:peter`. Any emails sent to this account can be read via the email client on the exploit server. 
#### Enumeration
- As this is a password reset challenge, we will skip the thorough mapping of the website. Below is the "Home" page.
![[Pasted image 20220806133032.png]]
![[Pasted image 20220806133051.png]]
We attempted to login with the credentials given and we get to the `my-account` page below. Which does not seem to have anything special.
![[Pasted image 20220806133111.png]]
- Next we try the "Forgot password" button which brings us to this link. We supply it with `wiener` as the user and we get the email that can be seen below.
![[Pasted image 20220806133139.png]]
![[Pasted image 20220808113937.png]]
- We notice that `/forgot-password` is a POST request and passes the username in the body. 
![[Pasted image 20220806133322.png]]
![[Pasted image 20220806133205.png]]
- As we can see from the link, a unique token is generated for each reset and it is passed in as a parameter `?temp-forgot-password-token=<token>` to `/forget-password` .

#### Vulnerability Assessment / Exploitation
- We visited the page and intercepted the POST request of the password reset page.
![[Pasted image 20220806133247.png]]
![[Pasted image 20220806133632.png]]
- Initially, we thought that there was someting special in the cookie. But upon decoding with cyberchef, we did not see anything that is of note.
![[Pasted image 20220806133755.png]]
- So next we will test the HTTP Host Header. Based on the above link for testing HTTP Host Header vulnerabilities, we will now add a abitrary value to the Host Header and check what happens.
![[Pasted image 20220808114300.png]]
![[Pasted image 20220808114241.png]]
- It seems like the reset email is still sent and we see that the link for password reset is `google.com` appended with the `/forgot-password?temp-forgot-password-token=<token>` query.
- The next step will be to trick user `carlos` into clicking on the reset email and steal his token.
- This can be done by changing the body parameter of the `forgot-password` POST request to `carlos` and Host Header to the exploit server host name, and when `carlos` clicks on the link, he will make a request to the exploit server with the token in his GET request.
![[Pasted image 20220808114742.png]]
![[Pasted image 20220808114833.png]]
- As we can see above, in the access log of the exploit server, we can see that there is a error 404 request to `/forgot-password` with the token appended.
![[Pasted image 20220808115233.png]]
- We solve the challenge by logging into carlos with the new password `peter1`.
![[Pasted image 20220808115310.png]]

### Host Header Authentication Bypass
- This lab makes an assumption about the privilege level of the user based on the HTTP Host header.
- To solve the lab, access the admin panel and delete Carlos's account. 

#### Enumeration
- We are given a website below, but we do not have any credentials.
![[Pasted image 20220808121618.png]]
![[Pasted image 20220808121710.png]]
- It is evident that we are unable to login with the usual credentials `wiener:peter`.
![[Pasted image 20220808121749.png]]
- Going into the details of the posts product does not give us valuable information.
- We attempt to see if the page has `robots.txt`. 
![[Pasted image 20220808120458.png]]
- As can be seen above, `robots.txt` reveals that there is an admin panel which can be seen below.
![[Pasted image 20220808120535.png]]

#### Vulnerability Assessment / Exploitation
- The clue given by the error message above is that the interface is only available for local users.
- The original GET request is as such.
![[Pasted image 20220808122231.png]]
- Hence we used Burp Repeater and modified the Host Header to `localhost`.
![[Pasted image 20220808121542.png]]
- As can be seen below, we succesfully logged into the adminsitrator panel.
![[Pasted image 20220808121551.png]]
- We can proceed to delete `carlos` to solve the challenge.
![[Pasted image 20220808122525.png]]

## File Upload Vulnerabilities
### RCE via Webshell Upload
- This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.
- To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.
- You can log in to your own account using the following credentials: `wiener:peter`. 

#### Enumeration
- First we will need to do some webserver enumeration. Using wappalyzer we find out that PHP is used and the webserver running is Apache on a Linux distribution Ubuntu. 
![[Pasted image 20220808142008.png]]
- Next we login with the credentials given and we are greeted with a page that allows us to upload an image for our avatar.
![[Pasted image 20220808140837.png]]
- We check if there is any restriction to content by uploading a `test.txt `file.
![[Pasted image 20220808140957.png]]
- The file was sucessfully uploaded and it seems like the server is executing the file.
![[Pasted image 20220808141017.png]]
![[Pasted image 20220808141032.png]]
- Below, we can see the directory to the resource `/files/avatars/`
![[Pasted image 20220808141050.png]]
- We are able to visit the page where `test.txt` was uploaded.
![[Pasted image 20220808141127.png]]
#### Vulnerability Assessment / Exploitation
- Therefore, we send the POST request responsible for uploading to Burp Repeater and injected a PHP one liner execution shell running OS command `whoami`.
![[Pasted image 20220808143639.png]]
- We continue to query the directory where the php shell is located and now we know that the user account that is behind this web instance is `carlos`.
![[Pasted image 20220808143712.png]]
- We do the same as above, just that we changed the command to `cat /home/carlos/secret` to read the secret file we are supposed to find.
![[Pasted image 20220808143856.png]]
- We get the secret code and submitted it.
![[Pasted image 20220808143917.png]]
![[Pasted image 20220808143957.png]]


### Web shell upload via Content-Type restriction bypass
- This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

- To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

- You can log in to your own account using the following credentials: `wiener:peter`

#### Enumeration
- The first few steps of enumeration is the same as the above Lab, and we are then given this login page upon logging in with the correct credentials.
![[Pasted image 20220808145056.png]]
- Attempted to upload `test.txt` and `websh.php` but there seem to be some file checking in place thus preventing the upload. 
![[Pasted image 20220808145119.png]]
![[Pasted image 20220808145136.png]]
#### Vulnerability Assessment / Exploitation
- Upon closer inpection at the POST request, we realise that the check is done using the `Content-Type` header.
- Hence to bypass this, we intercepted the POST request and changed the `Content-Type` to the `Content-Type` that the server accepts `image/jpeg` and `test.txt` was sucessfully uploaded in the `/files/avatars` directory.
![[Pasted image 20220808145308.png]]
- Then we continue to upload `websh.php` which does a `shell_exec('whoami')` and the user that it is running as is `carlos`
![[Pasted image 20220808145523.png]]
![[Pasted image 20220808145611.png]]
![[Pasted image 20220808145709.png]]
- Next we upload the websh.php with the one liner that does `cat /home/carlos/secret`.
![[Pasted image 20220808145733.png]]
- Execute it and we obtain the secret code for submission.
![[Pasted image 20220808145749.png]]
![[Pasted image 20220808145812.png]]