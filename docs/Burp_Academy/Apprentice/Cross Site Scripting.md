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

## DOM XSS in `document.write` sink using source `location.search`
### Enumeration
### Vulnerability Assessment
### Exploitation


## aaa
### Enumeration
### Vulnerability Assessment
### Exploitation