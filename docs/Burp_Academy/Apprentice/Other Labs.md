# OS Injection
## OS command injection, simple case
 - This lab contains an OS command injection vulnerability in the product stock checker.
 - The application executes a shell command containing user-supplied product and store IDs, and returns the raw output from the command in its response.
 - To solve the lab, execute the whoami command to determine the name of the current user. 
### Enumeration
- We visit the website and randomly visit a product page to see where the vulnerable function is.
![[Pasted image 20220726230343.png]]
- Below, we can see that there is a "Check stock" button and we attempt to try the function out while intercepting it with burp suite.
![[Pasted image 20220726230404.png]]
- From the request below, we can see that "Check stock" will send a `POST` request and the `POST` request will contain the parameters.
![[Pasted image 20220730153427.png]]
![[Pasted image 20220801105320.png]]
- It seems like `storeId` can be manipulated.
### Exploitation / Vulnerability Assessment
- Hence we try to see if we can elicit a command by doing a `ls` command
![[Pasted image 20220801105454.png]]
- From the above response, it seems like we are able to inject command into the query.
![[Pasted image 20220730154224.png]]
- We try again by reading `stockreport.sh` and it seems to be the `sh` file that is running the backend for the stock query.
- This confirms that using `|` will allow us to run commands.
![[Pasted image 20220730154305.png]]
- Hence, we do a `| whoami` to solve the lab.

# Directory Traversal
## File path traversal, simple case
- This lab contains a file path traversal vulnerability in the display of product images.
- To solve the lab, retrieve the contents of the /etc/passwd file. 
### Enumeration
- We are brought to the webpage below
![[Pasted image 20220726225513.png]]
- We then navigate to view details of a random product
![[Pasted image 20220726225535.png]]
- We viewed the page source and realised that the image is querying the webserver for a file `6.jpg`. 
![[Pasted image 20220726225449.png]]
### Vulnerability Assessment / Exploitation
- We try to query the file directly by using the "Open Image in New Tab"
![[Pasted image 20220726225604.png]]
- True enough, we see that the image appears, seems like `?filename=6.jpg` does a file read operation.
![[Pasted image 20220726225628.png]]
- Using burp repeater, we intercept a request with `../../../../etc/passwd` parameter and look out for the response.
![[Pasted image 20220726225820.png]]
- As can be seen above, in the response captured, the `/etc/passwd` of the host is read.

# Websockets
## Manipulating WebSocket messages to exploit vulnerabilities
-  This online shop has a live chat feature implemented using WebSockets. Chat messages that you submit are viewed by a support agent in real time.
- To solve the lab, use a WebSocket message to trigger an alert() popup in the support agent's browser. 
- Web socket exploitation is further mentioned [here](https://portswigger.net/web-security/websockets)
### Enumeration
- We access the lab website and on the top right hand corner we see "Live chat"
![[Pasted image 20220801110013.png]]
- We visit the "Live chat"  page and below is the page in question.
![[Pasted image 20220801110127.png]]
- We inspect the page source to see the element responsible for the interactive chat below.
![[Pasted image 20220801110344.png]]
- As can be seen, `wss://0ae600df04a148c4c08ec9a400e00087.web-security-academy.net/chat` is used. `wss` suggests the use of websockets.
- This is further confirmed with Websockets History on burp suite
![[Pasted image 20220801180137.png]]
- We then continued on to inspect `resources/js/chat.js` which gives us the javascript responsible for displaying and sending the message.
![[Pasted image 20220801141707.png]]
- In the function above, we can see that there is another function `writeMessage()` that is called which we can see what it does below. 
- The `writeMessage()` function takes in 3 parameters, `className`, `user` and `content`.
![[Pasted image 20220801142051.png]]
- As message content is the only thing we can manipulate, our focus will be on lines that contains variables `content` and `contentCell`.
- `document.createElement("td")`,  `contentCell.innerHTML = content` and `row.appendChild(contentCell)` will create HTML element `<td>content</td>` and append it in row so that messages will appear like below.
![[Pasted image 20220801175651.png]]
- The send message function below. We notice that there is a `foreach` loop that places data into an `object[]` array and does the function `htmlEncode()`.
![[Pasted image 20220801142109.png]]
- `htmlEncode` seem to be an input santisation function, however, it does not seem to sanitise `'(' and ')'`.
![[Pasted image 20220801141547.png]]
### Vulnerability Assessment / Exploitation
- Thus a payload that we can try is as below : 

```html
<img src=1 onerror='alert(1)'>
```

- To bypass html encoding, we have to use burp repeater to change out the content below to our payload above.
![[Pasted image 20220801180818.png]]
- After hitting the 'Send' button, we can see that `alert()` is successfully triggered.
![[Pasted image 20220801180726.png]]

# Insecure Deserialization
## Modifying serialized objects
### Enumeration
- This lab uses a serialization-based session mechanism and is vulnerable to privilege escalation as a result. To solve the lab, edit the serialized object in the session cookie to exploit this vulnerability and gain administrative privileges. Then, delete Carlos's account.
- You can log in to your own account using the following credentials: `wiener:peter` 
- Insecure deserialisation is explained [here](https://portswigger.net/web-security/deserialization) and [this](https://portswigger.net/web-security/deserialization/exploiting) explains how to test for and exploit insecure deserialisation. 
### Enumeration
- We first visit the website and see that there is a "My account" tab. Where we find a login form.
![[Pasted image 20220801194343.png]]
![[Pasted image 20220801194501.png]]
- We use the credentials provided and attempt to login. The intercepted POSt request is as such and in the `Set-Cookie` parameter, we see what looks like a base64 encoded string.
![[Pasted image 20220801183053.png]]
- We decode the string to get what looks like a PHP serialisation format. 
![[Pasted image 20220801183113.png]]
### Vulnerability Assessment
- To get to the admin panel it seems like we simply need to change `b:0` to `b:1` as below and encode it in base64.

```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:5:"admin";b:1;}7
```

![[Pasted image 20220801195256.png]]
- The manipulation can be done by intercepting the GET request and modifying the cookie.
![[Pasted image 20220801195708.png]]
- As can be seen below, the "Admin panel" tab is revealed.
![[Pasted image 20220801195427.png]]
![[Pasted image 20220801195544.png]]
![[Pasted image 20220801195609.png]]
![[Pasted image 20220801195814.png]]

# OAuth Authentication
## Authentication bypass via OAuth implicit flow 
- This lab uses an OAuth service to allow users to log in with their social media account. Flawed validation by the client application makes it possible for an attacker to log in to other users' accounts without knowing their password.
- To solve the lab, log in to Carlos's account. His email address is `carlos@carlos-montoya.net`. You can log in with your own social media account using the following credentials: `wiener:peter`. 
- More about OAuth can be found [here](https://portswigger.net/web-security/oauth) and [this](https://portswigger.net/web-security/oauth/grant-types) will explain more about OAuth grant types. Information on OAuth implicit flow can be found [here](https://oauth.net/2/grant-types/implicit/#:~:text=The%20Implicit%20flow%20was%20a,extra%20authorization%20code%20exchange%20step.).
### Enumeration
- We first visit the website and we find a "My account" tab which leads to a "social media" sign in redirect.
![[Pasted image 20220801213247.png]]
- Below is the redirected page and we subsequently went logged in with the credentials provided.
![[Pasted image 20220801224919.png]]
![[Pasted image 20220801212758.png]]
![[Pasted image 20220801213205.png]]
- The requests were proxied and we investigated the exchanges using burp suite proxy HTTP history.
![[Pasted image 20220801225638.png]]
- The interesting request will be the `authenticate` POST request that comes after the `oauth-callback` GET request. 
![[Pasted image 20220801225930.png]]
- The email parameter in the request above was changed to `carlos@carlos-montoya.net` and sent to repeater.
![[Pasted image 20220801230240.png]]
- We get a 302 redirect which we right click and choose "request in browser". 
![[Pasted image 20220801230611.png]]
![[Pasted image 20220801230737.png]]
- Upon successful authentication, when we go to "My account" the `GET` request was to `id=carlos`.
![[Pasted image 20220801230400.png]]
- We are logged in as Carlos.
![[Pasted image 20220801230414.png]]
