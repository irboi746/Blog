# Business Logic Vulnerabilities
- In this [link](https://portswigger.net/web-security/logic-flaws) we can understand what is meant by business logic vulnerabilities and its impact. This link [here](https://portswigger.net/web-security/logic-flaws/examples) gives a more detailed and more example driven explanation of what is meant by business logic vulnerabilities.

## Excessive trust in client-side controls
-  This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. 
- To solve the lab, buy a "Lightweight l33t leather jacket". You can log in to your own account using the following credentials: `wiener:peter`
### Enumeration
- We are given the webpage below, where it is an ecommerce shop, with a shopping cart.

![[Pasted image 20220810172139.png]]

- We can view products and even add to cart like below.

![[Pasted image 20220810172215.png]]

- There is user login

![[Pasted image 20220810172306.png]]

- When we login, we can see that there is a "Store Credit" as well as a user cart which is different from an anonymous cart.

![[Pasted image 20220810172328.png]]

- There is a `/cart` page where products that are added will show. 

![[Pasted image 20220810172415.png]]
### Vulnerability Assessment / Exploitation
- The vulnerable function in this case is the way where products are added to cart.
- As can be seen below, a product is added to cart and rather than fetching the price from the database, the product details are sent as a `POST` request.

![[Pasted image 20220810172540.png]]
![[Pasted image 20220810172710.png]]

- Therefore it seems like we can manipulate the price of the jacket simply by changing the price in the `POST` request like below.

![[Pasted image 20220810173026.png]]

- As can be seen below, the product is added to cart and we sucessfully checked out with a significantly low price for the product.

![[Pasted image 20220810173049.png]]
![[Pasted image 20220810173108.png]]
![[Pasted image 20220810173127.png]]

## High-level logic vulnerability
 - This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. To solve the lab, buy a "Lightweight l33t leather jacket".

- You can log in to your own account using the following credentials: `wiener:peter`.

### Enumeration
- We are given an ecommerce page that looks similar to the one above, with very similar functionalities with regards to the checkout and login.

![[Pasted image 20220810175640.png]]

- The difference in this web application is that the price is no longer sent in the POST request.
![[Pasted image 20220810175757.png]]
![[Pasted image 20220811102701.png]]
- As can seen above, the only variable that can be manipulated here is the quantity.

### Vulnerability Assessment / Exploitation
- We then realised that quantity can actually be set to `-1`. 

![[Pasted image 20220811102817.png]]

- We then intercepted the POST request and tested if the value can be set to more than `-1`.

![[Pasted image 20220811102935.png]]

- As can be seen below, the quantity is set to `-3`

![[Pasted image 20220811103007.png]]

- Additionally, we realised that "Add to cart" button is not the only button vulnerable to this. The "-",  "+" and "remove" buttons are vulnerable to this too.

![[Pasted image 20220811103717.png]]
![[Pasted image 20220811103631.png]]

- It seems like there is a check for negative total value.
- We can get our item "Lightweight 'l33t' Leather Jacket" with a huge "discount" by setting other items to be a negative value and the leather jacket to a positive value.

![[Pasted image 20220811104126.png]]
![[Pasted image 20220811104143.png]]

## Inconsistent security controls
- This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. 
- To solve the lab, access the admin panel and delete Carlos. 

### Enumeration
- We do an initial directory enumeration and find that there is an `/admin` directory but it returns us a `401 Unauthorized` error.

![[Pasted image 20220811105218.png]]

- Concurrently, we attached burp proxy and surfed the website normally trying all the functionalities

![[Pasted image 20220811104646.png]]

- It seems like there is no "Add to cart" function. 

![[Pasted image 20220811104702.png]]

- There is a login page, but we are not given credentials. However, we also notice a register page.

![[Pasted image 20220811104632.png]]

- In this register page, it seems like anyone can register but we notice that for workers that work for DontWannaCry, their email domain is `@dontwannacry.com`. We are not given a donwannacry.com email but it might be useful later.

![[Pasted image 20220811104619.png]]

- Registering for a new user witht the email given to us.

![[Pasted image 20220812101426.png]]
![[Pasted image 20220811104828.png]]

- We go to the email given and followed the link to complete the registration.

![[Pasted image 20220811104856.png]]
![[Pasted image 20220811104931.png]]
![[Pasted image 20220811104955.png]]

- We are brought to the following page upon login, where the only function is to change our email address.

![[Pasted image 20220811105014.png]]

- It seems like the error message for `/admin` page is that we are not a DontWannaCry user.

![[Pasted image 20220811104730.png]]

### Vulnerability Assessment / Exploitation
- Based on the information above, where DontWannaCry users have email address of `@dontwannacry.com` perhaps, we can bypass the authentication by simply having a dontwannacry email.

![[Pasted image 20220812104230.png]]

- We changed our email domain to `dontwannacry.com` and in the next picture, we can see that the admin panel appears.

![[Pasted image 20220812104438.png]]
- We proceed to the `/admin` page and delete the user `carlos`.

![[Pasted image 20220812104453.png]]

## Flawed enforcement of business rules
 - This lab has a logic flaw in its purchasing workflow. To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".
- You can log in to your own account using the following credentials: `wiener:peter`

### Enumeration
- We visit the website and we get one like below, where they give out coupon and there is a cart, thus suggesting that the ecommerce function is working in this application.

![[Pasted image 20220812161520.png]]
- We scroll to the bottom of the page to see this "Sign up to our newsletter" function.

![[Pasted image 20220812163910.png]]

- Based on the client side source code, it seems like this sends a POST request with email account as a parameter to the `/sign-up` page. 

![[Pasted image 20220812164048.png]]

- Upon sucessful sign up, the section changed into a javascript that prompts another discount code.

![[Pasted image 20220812164215.png]]
![[Pasted image 20220812164231.png]]
![[Pasted image 20220812163922.png]]

- There is indeed an add to cart function.

![[Pasted image 20220812161632.png]]

- There is also a login function and this time credentials are given and hence we will need to log in with the given credentials.

![[Pasted image 20220812161649.png]]

- So this is a view of the user `/account` page.

![[Pasted image 20220812161711.png]]

- Below is a picture of the user cart.

![[Pasted image 20220812161811.png]]

- When a coupon is applied it will be appended to a HTML table like below.

![[Pasted image 20220812161847.png]]

- There is checks to make sure the same coupon cannot be applied twice.

![[Pasted image 20220812161939.png]]

- There is also a check to make sure invalid coupon cannot be used.

![[Pasted image 20220812162136.png]]

- Below are the GET redirects when a coupon is applied sucessfully, or unsucessfully respectively. 

![[Pasted image 20220812163223.png]]
![[Pasted image 20220812163144.png]]
### Vulnerability Assessment / Exploitation
- Even though there are checks for the same coupon used consecutively, it seems like if a new coupon is added, we are able to stack the discount.

![[Pasted image 20220812164519.png]]

- And the best part is we can stack as many discounts as we want as long as they are not consecutive.

![[Pasted image 20220812164608.png]]
![[Pasted image 20220812164659.png]]

- Hence we stacked the coupon till the price of "Lightweight 'l33t' Leather Jacket" is zero before we make the purchase.

![[Pasted image 20220812164755.png]]
![[Pasted image 20220812164811.png]]
