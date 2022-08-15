# JWT
- JWT stands for JSON Web Token.
- Basics of JWT is explained [here](https://portswigger.net/web-security/jwt) and the use of burp suite to work with JWT is explained [here](https://portswigger.net/web-security/jwt/working-with-jwts-in-burp-suite). 

## JWT authentication bypass via unverified signature
 - This lab uses a JWT-based mechanism for handling sessions. Due to implementation flaws, the server doesn't verify the signature of any JWTs that it receives.
- To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`. You can log in to your own account using the following credentials: `wiener:peter`. 

### Enumeration
- As this challenge is access-token challenge, we will jump straight to login page and login with the given credentials `wiener:peter`.

![[Pasted image 20220815121154.png]]

- Upon logging in we are given a `/my-account` page like below.

![[Pasted image 20220815121207.png]]

- we attempted to access the admin interface with peter's account but is given the following page.

![[Pasted image 20220815121136.png]]

- We analyse the HTTP traffic and we realise that after login success we are given a response with a `set-cookie` like below.

![[Pasted image 20220815125150.png]]

- The cookie value looks like that of a JWT encoded in base64 and it seems to be used as an authentication mechanism.

![[Pasted image 20220815125127.png]]

- The decoded cookie value is show below.

![[Pasted image 20220815125449.png]]

### Vulnerability Assessment / Exploitation
- As can be seen above, the value `"sub"` is `wiener`. It might possibly be used as the identifier for the account used.

![[Pasted image 20220802111354.png]]

- We can try this by sending the `GET` request for `/my-account` to burp repeater highlight the relevat text to edit and use the inspector tab to edit `"sub" : "wiener"` to `"sub" : "administrator"` like above. 

![[Pasted image 20220815144304.png]]

- As can be seen, we are now logged in as an administrator. From burp repeater we go to the `GET` request and right click to choose "request in browser".

![[Pasted image 20220815144807.png]]
![[Pasted image 20220815144937.png]]
- We go to burp proxy, "Match and Replace" and then put the cookie value that we want to match and replace. 
- When rule is enabled, we can simply delete carlos from the admin panel

![[Pasted image 20220802111900.png]]

## JWT authentication bypass via flawed signature verification
- This lab uses a JWT-based mechanism for handling sessions. The server is insecurely configured to accept unsigned JWTs.
- To solve the lab, modify your session token to gain access to the admin panel at /admin, then delete the user carlos.
- You can log in to your own account using the following credentials: `wiener:peter `

### Enumeration
- We are given a webpage like below : 

![[Pasted image 20220815145533.png]]

- In similar vein to the challenge above, we will skip straight to the login page and the analysis of the login function. We login with the credentials given below.

![[Pasted image 20220815145558.png]]

- We are sent to this `/my-account` page.

![[Pasted image 20220815145636.png]]

- And it seems like we are not able to log in as administrator as there seem to be some checks.

![[Pasted image 20220815145625.png]]

- We analyse the HTTP traffic and we see that there is a JWT issued as a cookie.

![[Pasted image 20220815150311.png]]
![[Pasted image 20220815150326.png]]
![[Pasted image 20220815150342.png]]

- We decoded the JWT cookie and we realise that it is the same as the above, where `"sub" : "wiener"` is used to identify the user.

![[Pasted image 20220815150256.png]]

### Vulnerability Assessment / Exploitation
- We attemp to use the technique used above, but it seems like we are unable to bypass the authentication mechanism.

![[Pasted image 20220815150522.png]]
![[Pasted image 20220815152436.png]]

- We downloaded the "JWT Editor" extension and in Burp Repeater, we highlight the token and select the 'JSON Web Token' tab.

![[Pasted image 20220815151023.png]]

- We will be led to the editor and we go to the payload section where we change the `"sub:"` to `"sub" : "adminstrator"` again.

![[Pasted image 20220815151041.png]]

- However, this time, we go to the bottom of the editor and select "Attack" and choose "none Signing Algorithm".

![[Pasted image 20220815151100.png]]

- We will get the JWT token below.

![[Pasted image 20220815151116.png]]

- From repeater, we send the request and we get to the admin panel.

![[Pasted image 20220815150927.png]]

- Hence like above, we simply do a "Match and Replace" at the proxy.

![[Pasted image 20220815151232.png]]

- The user `carlos` can be deleted.

![[Pasted image 20220815153046.png]]
![[Pasted image 20220815151444.png]]
