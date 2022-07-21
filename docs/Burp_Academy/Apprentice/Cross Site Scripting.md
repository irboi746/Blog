# Cross Site Scripting
## Reflected XSS into HTML context with nothing encoded
### Enumeration
- Given this website to test.
![[Pasted image 20220624115118.png]]
- There are only a few functions : Search Box, Home button and View Post.
### Vulnerability Assessment
- As this is a reflected XSS lab, it is likely that input is needed and it is transient (not persistent) hence the first input we can test here is the search box.
 - String is rendered normally when normal input is given. 
 ![[Pasted image 20220624115512.png]]
- Adding `<h1>aa</h1>` gives us below where the HTML is rendered with `H1` heading.
 ![[Pasted image 20220624115531.png]]
- With the above information we know that the search box field is vulnerable to Reflected XSS.
### Exploitation
- To do an `alert()` we will need to encode the html with `<script></script>` hence the injection string will be 
```html
<script>alert('aa')</script>
```
![[Pasted image 20220624115545.png]]
 - There we have it an alert.
 ![[Pasted image 20220624115618.png]]

## Stored XSS into HTML context with nothing encoded
### Enumeration
- We are given a website below which only has multiple posts.
![[Pasted image 20220624120817.png]]
![[Pasted image 20220624120946.png]]
- We find that within each post there is a comment section.
### Vulnerability Assessment
- As this lab is about Stored XSS, there must be fields that can be stored and we can turn our attention to the comment section.
- Seems like there is some input sanitation for Email and Website fields.
![[Pasted image 20220624121120.png]]
![[Pasted image 20220624121157.png]]
- Hence the parameters to test will be `Comment` and `Name` field.
![[Pasted image 20220624121215.png]]
![[Pasted image 20220624121258.png]]
- So it seems like the `Name` field is not vulnerable but the `Comment` field is.
![[Pasted image 20220624121325.png]]
### Exploitation
- Hence we can submit a request like below : 
![[Pasted image 20220624121432.png]]
- And there we have it stored XSS.
![[Pasted image 20220624121503.png]]
![[Pasted image 20220624121514.png]]
- As this is stored XSS, as long as the comment is there, any user that visits the page will receive the pop-up.


## Reflected XSS into attribute with angle brackets HTML-encoded
### Enumeration
![[Pasted image 20220720162800.png]]
![[Pasted image 20220720162746.png]]
### Vulnerability Assessment
- From the above enumeration, we can see that there is some input sanitization where `<>` are changed to `&lt;` and `&gt;` respectively. Hence adding angle brackets will not work.
- We do notice however, that adding `""` appropriately can trigger the javascript behind the form.
![[Pasted image 20220720174319.png]]
- Hence our payload :
```html
"onclick="alert(1)
```
- We can use the event attributes [here](https://www.w3schools.com/tags/ref_eventattributes.asp) and trigger the XSS as per the attribute  used.
![[Pasted image 20220720165418.png]]

## Stored XSS into anchor href attribute with double quotes HTML-encoded
- This lab contains a stored cross-site scripting vulnerability in the comment functionality. 
- To solve this lab, submit a comment that calls the alert function when the comment author name is clicked. 
### Enumeration
- There is some form of input sanitization.

![[Pasted image 20220720175207.png]]
![[Pasted image 20220720175351.png]]
- We see that the author's name linked with a `href` link which is the website field. 
![[Pasted image 20220720212834.png]]
### Vulnerability Assessment
- Inserting `<script>alert()</script>` to the website field does ot seem to work
![[Pasted image 20220720213105.png]]
- With reference to this [link](https://security.stackexchange.com/questions/11985/will-javascript-be-executed-which-is-in-an-href) we can try `javascript:alert(1)`
 ![[Pasted image 20220720213255.png]]
- And the javascript is executed.

## Reflected XSS into a JavaScript string with angle brackets HTML encoded
- This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. 
- The reflection occurs inside a JavaScript string. 
- To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function. 
### Enumeration
- Went to the search box and typed 'aaaa'
![[Pasted image 20220720214322.png]]
- Inspect source and realised that there is a `<script></script>` section where input 'aaaa' is also present. Hence we need to manipulate this input and have `document.write` execute the manipulated input.
![[Pasted image 20220720214259.png]]

### Vulnerability Assessment
- Payload used : 
```html
';<img src=1 onerror=alert()>'
```
- But the result is encoded hence reflected XSS will not work.
![[Pasted image 20220720215209.png]]
- We use another payload 
```html
'-alert()-'
```
- As we can see below, `alert()` is triggered.
![[Pasted image 20220720222417.png]]