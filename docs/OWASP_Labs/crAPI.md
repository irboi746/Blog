# crAPI
Trying this lab out to come up with templates and template generation logic for [apimap](https://github.com/team-s0l1d1ty/apimap). As such, not all challenges are attempted. crAPI is also used to demonstrate how [apimap template for crAPI](https://github.com/team-s0l1d1ty/apimap/tree/main/Sample_Templates/crAPI_Sample) can be used for API testing, PoC writing. 

We also did not come up with templates for brute-forcing of fuzzing. While it is possible to do such tasks with [apimap](https://github.com/team-s0l1d1ty/apimap) we believe that it is not optimal and there are more effective tools out there.

## BOLA Vulnerabilities
### Challenge 1 - Access details of another user’s vehicle
>To solve the challenge, you need to leak sensitive information of another user’s vehicle.
>
>Since vehicle IDs are not sequential numbers, but GUIDs, you need to find a way to expose the vehicle ID of another user. Find an API endpoint that receives a vehicle ID and returns information about it.

Upon logging in we noticed that there is an `/identity/api/v2/vehicle/{uuid}/location` API. 
![[Pasted image 20230701191944.png]]
This API gets the coordinates of the car.
![[Pasted image 20230701192002.png]]
We can test this by keying another valid UUID. To get another UUID we would need to rely on [[#Challenge 4 - Find an API endpoint that leaks sensitive information of other users]].

Using a valid `vehicleid` we attempt to check vehicle location unauthenticated.
![[Pasted image 20230701202409.png]]
Seems like we will be given a 401 Error, with "Invalid Token"
![[Pasted image 20230701202421.png]]

We then replayed the request for our own vehicle with our own token.
![[Pasted image 20230701202529.png]]
![[Pasted image 20230701202542.png]]

Using the same Authorization token, we attempt to replay another valid `vehicleid`
![[Pasted image 20230701203528.png]]
As can be seen from the response, we are able to get another car's information. 
![[Pasted image 20230701203539.png]]

### Challenge 2 - Access mechanic reports of other users
>crAPI allows vehicle owners to contact their mechanics by submitting a "contact mechanic" form. This challenge is about accessing mechanic reports that were submitted by other users.

We start our analysis of mechanic report by first submitting a report of our own.
![[Pasted image 20230701184832.png]]
We noticed in the response, there is an `/api/mechanic/mechanic_report?report_id=6`. This is not in the user interface we have.
![[Pasted image 20230701184901.png]]
We visited the link, and it seems like we are able to access our own report with no token needed!
![[Pasted image 20230701185028.png]]
![[Pasted image 20230701185141.png]]
We noticed that the report number is sequential. Hence we attempted to brute-force used intruder to retrieve all other reports.
![[Pasted image 20230701185239.png]]
Retrieving `report_id=2`
![[Pasted image 20230701185607.png]]
![[Pasted image 20230701185705.png]]

Seems like there are only 6 reports.
![[Pasted image 20230701185747.png]]
![[Pasted image 20230701185737.png]]

## Broken User Authentication
### Challenge 3 - Reset the password of a different user
>Find an email address of another user on crAPI
>
>Brute forcing might be the answer. If you face any protection mechanisms, remember to leverage the predictable nature of REST APIs to find more similar API endpoints.

We tried the forget password function and it seems like there are API calls are as follows:
![[Pasted image 20230709143010.png]]

**/identity/api/auth/forget-password**
Shown below is Request/Response when a valid email is sent
![[Pasted image 20230709143133.png]]
![[Pasted image 20230709143152.png]]

Then we tried to send an invalid email and the following is the Response.
![[Pasted image 20230709143313.png]]
This in itself is an issue. The error message is overly verbose and it can help in brute-forcing the email.

**/identity/api/auth/v3/check-otp**
Shown below is the Request/Response of a successful OTP password change.
![[Pasted image 20230709143441.png]]
![[Pasted image 20230709143534.png]]

Then a response when we sent invalid OTP.
![[Pasted image 20230709143619.png]]

We can skip the brute-forcing of valid email since [[#Challenge 4 - Find an API endpoint that leaks sensitive information of other users]] is one such API that leaks the email address of other users.

As such, we will only need to send a password reset request for an email and brute-force the OTP (4 numeric digits - 10000 possibilities).

However, there seem to be some protection mechanism.
![[Pasted image 20230709144305.png]]

After number of attempts exceeded, when we used the correct OTP below.
![[Pasted image 20230709144540.png]]
![[Pasted image 20230709144605.png]]

We get an ERROR response. The OTP has been invalidated.
![[Pasted image 20230709144640.png]]

But we do notice something peculiar.
![[Pasted image 20230709144732.png]]
Most of the endpoints we have are `/v2/` however this is `/v3/`. Could there have been a `/identity/api/auth/v1/check-otp` or a `/identity/api/auth/v2/check-otp` endpoint?

Going by this, we sent request to the respective paths. We found out that `/identity/api/auth/v1/check-otp` is non-existent
![[Pasted image 20230709145002.png]]
![[Pasted image 20230709145015.png]]

However, `/identity/api/auth/v2/check-otp` is a valid and working endpoint!

![[Pasted image 20230709144923.png]]
![[Pasted image 20230709144933.png]]

![[Pasted image 20230709145210.png]]
![[Pasted image 20230709145221.png]]

And even after sending 20 payloads, we are not locked out! Seems like we can brute-force using this endpoint!
![[Pasted image 20230709145530.png]]

As can be seen below, we have successfully changed the password of the user!
![[Pasted image 20230709145746.png]]

### Challenge 14 - Find an endpoint that does not perform authentication checks for a user.

Through [[#Challenge 2 - Access mechanic reports of other users]], we know that `/api/mechanic/mechanic_report?report_id={id}` endpoints does not require authentication.


## Excessive Data Exposure
### Challenge 4 - Find an API endpoint that leaks sensitive information of other users
We notice that API `/community/api/v2/community/posts/recent` called when we browse to `http://localhost:8888/forum` reveals more than just title and content of the post but also `vehicleid`
![[Pasted image 20230701202009.png]]
![[Pasted image 20230701202223.png]]
 
## Rate-limiting
### Challenge 6 - Perform a layer 7 DoS using ‘contact mechanic’ feature

Owing to our [[#Challenge 11 - Make crAPI send an HTTP call to "www[.]google[.]com" and return the HTTP response.]] discovery we attempt to perform a layer 7 DoS by simply changing the url of `mechanic_api` to an invalid one, set `repeat_request_if_failed` to `true` and `number_of_repeats` to a large enough number.

![[Pasted image 20230703215553.png]]
From the response, it seems like we have caused a layer 7 DoS.
![[Pasted image 20230703215747.png]]

## Mass Assignment
### Challenge 8 - Get an item for free

>crAPI allows users to return items they have ordered. You simply click the "return order" button, receive a QR code and show it in a USPS store. To solve this challenge, you need to find a way to get refunded for an item that you haven’t actually returned.
>
>Leverage the predictable nature of REST APIs to find a shadow API endpoint that allows you to edit properties of a specific order.

Analysing the business logic flow with burp, we realised that order returns are related to APIs with /orders in them.
![[Pasted image 20230701235142.png]]

Again using the apimap, we extract the relevant endpoints from the API specs.
![[Pasted image 20230701235055.png]]

An interesting API is the `/workshop/api/shop/orders/{order_id}` where there is a `GET` and a `PUT` method.  To get a valid `order_id`, we would first need to purchase an item and this would use the `POST` method on `/workshop/api/shop/orders/` endpoint
![[Pasted image 20230702000658.png]]
![[Pasted image 20230702000722.png]]

Shown below is a valid GET request, after we have purchased the item.

![[Pasted image 20230702000513.png]]
Notice we only bought 1 and the variable quantity is 1.
![[Pasted image 20230702000531.png]]

We analyse the API schema and it seems that we might be able to edit the ProductQuantity with the `PUT` request even after we have bought and paid for the item.

![[Pasted image 20230701235714.png]]

We test this out by sending the request as follows.

![[Pasted image 20230702000948.png]]
We get an 200 OK response with the quantity altered.
![[Pasted image 20230702001049.png]]

We send the corresponding `GET` request again to double confirm the change.
![[Pasted image 20230702001151.png]]
![[Pasted image 20230702001202.png]]

We are also able to do this to items that are return "pending status".
![[Pasted image 20230702002932.png]]
![[Pasted image 20230702002943.png]]

However, this did not increase our credit. Instead, within the API specs we found a peculiar error message.

![[Pasted image 20230702003849.png]]

Based on the request/response below, even though it is not documented, it seems like we might just be able to manipulate "status"

![[Pasted image 20230702003957.png]]
![[Pasted image 20230702004007.png]]

True enough, we were able to change status from "pending return" back to "delivered".

![[Pasted image 20230702004131.png]]
![[Pasted image 20230702004153.png]]

That being the case, perhaps if we change it to "returned" we might get more credit back than expected?

We tested this idea out and upon changing "status" to "returned" 
![[Pasted image 20230702004326.png]]
![[Pasted image 20230702004340.png]]

We were able to see an increase in our credit through the `GET /workshop/api/shop/products`.

![[Pasted image 20230702004416.png]]
From $60.0 to $160.0
![[Pasted image 20230702004450.png]]
![[Pasted image 20230702004503.png]]


### Challenge 9 - Increase your balance by $1,000 or more

>After solving the [[crAPI#Challenge 8 - Get an item for free]] challenge, be creative and find a way to get refunded for an item you never returned, but this time try to get a bigger refund.

To do this we will just need to increase the quantity of items to > 100 and then change the status from "delivered"/"pending return" to returned, just like how it was done in [[#Challenge 8 - Get an item for free]].

Another way to do this would be to continuously alter the "status" variable from "delivered"/"pending return" to "returned" This will be demonstrated in apimap's sample template!

## SSRF
### Challenge 11 - Make crAPI send an HTTP call to "www[.]google[.]com" and return the HTTP response.

There is something peculiar about the endpoint `/workshop/api/merchant/contact_mechanic` .  
![[Pasted image 20230703210026.png]]

In the highlighted portion we noticed that the request JSON contains fields `mechanic_api`,  `repeat_request_if_failed` and `number_of_repeats`. They seem to be configurations for sending request.

![[Pasted image 20230703210351.png]]
The data within the response `response_from_mechanic_api` is not helping too! Seems like this is an API that calls another API!

We attempted to repeat this request but changing the `mechanic_api` to `postman-echo[.]com`.

![[Pasted image 20230703212710.png]]

Request seem to be successful, and it seems like `response_from_mechanic_api` returns the response of the website(postman-echo)! Additionally, we seem to be able to use arbitrary information.

![[Pasted image 20230703212742.png]]

We tested the other request methods of postman-echo and it seems like only `GET` request is valid.

![[Pasted image 20230703211812.png]]

The API makes a `GET` request and returns the response from the GET request. Therefore when we change the `mechanic_api` to `google.com`, we will get back the response of google.com!
![[Pasted image 20230703212025.png]]
![[Pasted image 20230703212051.png]]

## Injection
### Challenge 12 - Find a way to get free coupons without knowing the coupon code. (NoSQL)

![[Pasted image 20230709150602.png]]
![[Pasted image 20230709150625.png]]

We looked at the coupon validation API and used a [noSQL payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/NoSQL%20Injection/Intruder/NoSQL.txt) to fuzz. We get mostly 422 Erros, where it is finding fault with our inputs,  indicating further that the database used is likely noSQL as it is processing the JSON.

![[Pasted image 20230709151221.png]]
When `{"$gt":""}` payload was used, for some reason a 200 OK response was the coupon code was displayed!
![[Pasted image 20230709151417.png]]

## Challenges Not Attempted
### (Excessive Data Exposure) Challenge 5 - Find an API endpoint that leaks an internal property of a video
>In this challenge, you need to find an internal property of the video resource that shouldn’t be exposed to the user. This property name and value can help you to exploit other vulnerabilities.

### (BFLA) Challenge 7 - Delete a video of another user

### (Mass Assignment) Challenge 10 - Update internal video properties
>After solving the [[#Challenge 5 - Find an API endpoint that leaks an internal property of a video]] challenge, try to find an endpoint that would allow you to change the internal property of the video. Changing the value can help you to exploit another vulnerability.

### (Injection) Challenge 13 - Find a way to redeem a coupon that you have already claimed by modifying the database (SQL)

### (Broken Authentication) Challenge 15 - Find a way to forge valid JWT Tokens

