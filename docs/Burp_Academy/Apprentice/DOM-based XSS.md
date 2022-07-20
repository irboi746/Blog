# DOM based XSS
## What is DOM?
- Based on this [link](https://www.w3schools.com/js/js_htmldom.asp) and this [portswigger link](https://portswigger.net/web-security/dom-based) will explain what HTML sink is.

## DOM XSS in `document.write` sink using source `location.search`
### Enumeration
Step 1: Randomly type a string to get a search result
![[Pasted image 20220714181624.png]]
Step 2: go to inspect source and we find the script responsible for the `'aaaa'`<br> we find that there is a `document.write()`
![[Pasted image 20220714181743.png]]|

### Vulnerability Assessment
- From the highlighted statement above, we know that the `query` is the entry point for injection. Hence we will need to craft a payload and do a search based on where query is.  

- The goal of this lab is to issue an `alert()` statement hence we will simply edit the payload accordingly to `"><svg onload=alert(1)>` which will result in the query below : 
 
```html
document.write('<img src="/resources/images/tracker.gif?searchTerms='+"><svg onload=alert(1)>+'">');
```

- What happened here is that a svg (scalable vector graphic) is created at runtime and loaded beside `img src` which was closed. Upon loading, `alert()` function will be called. 

![[Pasted image 20220717221947.png]]

## DOM XSS in innerHTML sink using source location.search
### Enumeration
Step 1: Randomly type a string to get a search result
![[Pasted image 20220717234551.png]]
Step 2: Inspect Page source to see what is the script behind it.
![[Pasted image 20220717234512.png]]
### Vulnerability Assessment
- From this [link](https://stackoverflow.com/questions/30661497/xss-prevention-and-innerhtml)we understand how `innerHTML` XSS works.
- The goal of this lab is to issue an `alert()` statement thus our payload will be as such : `<img src=1 onerror=alert('1')>` which will make the statement look like : 
```html
   document.getElementById('searchMessage').innerHTML = `<img src=1 
   onerror=alert('1')>`;
```
![[Pasted image 20220720001428.png]]
- XSS is executed, as there is no such image and hence on error, the alert box will be executed. 

## DOM XSS in jQuery anchor `href` attribute sink using `location.search` source
### Enumeration
- URL before submission
![[Pasted image 20220718111314.png]]
- There is input sanitization.
![[Pasted image 20220718105437.png]]
- There is response when sending feedback.
![[Pasted image 20220718110752.png]]
- The url when feedback is submitted
![[Pasted image 20220718111235.png]]
- Probable vulnerable jQuery
![[Pasted image 20220718110842.png]]

### Vulnerability Assessment
- We can attempt to test the vulnerability based on how this [link](https://portswigger.net/web-security/cross-site-scripting/dom-based) does it.
- Payload used : 
```html
?returnPath=javascript:alert('1')
```
-  After the payload is sent, we will need to click `Back` for the `alert()` to show.
![[Pasted image 20220719232023.png]]
### Exploitation
- The objective of this lab is make the "back" link alert `document.cookie`. 
- Change the payload to :
```html
javascript:alert(document.cookie)
```
![[Pasted image 20220719231913.png]]

## DOM XSS in jQuery selector sink using a hashchange event
The URL hash is everything that follows the pound sign (#) in the URL. The `hashchange` event activates when the URL hash changes from one to another. 

>Example of `hashchange`: 
>From `url.com/#header` to `url.com/#footer`
>

 This lab contains a DOM-based cross-site scripting vulnerability on the home page. It uses jQuery's `$()` selector function to auto-scroll to a given post, whose title is passed via the `location.hash` property.

To solve the lab, deliver an exploit to the victim that calls the `print()` function in their browser. 

### Enumeration
![[Pasted image 20220720003905.png]]
- From the inspect page source, we can see the vulnerable jQuery function.
![[Pasted image 20220720003825.png]]
- We can see that when there is a `hashchange`, it will call a `function()` which will check if the hash component contains values the are part of the  `<h2>` heading in the list below. If it exists, it will scroll to the post.
![[Pasted image 20220720144523.png]]

### Vulnerability Assessment
- To exploit this, we can intercept the response and modify this function to trigger something when there is a `hashchange`.
![[Pasted image 20220720145512.png]]
- From above, we can added 
```html
console.log('test function');
``` 
- As we can see below, `test function` is called when hash changes from `Identit` to `Ident`.
![[Pasted image 20220720145818.png]]
![[Pasted image 20220720145649.png]]
![[Pasted image 20220720145621.png]]
- Another means where we can deliver the exploit is through the URI. When hashchange occurs, the value of the hash that changed is parsed into `decodeURIComponent`. Hence we can deliver the exploit through the URI. 
- We can test this out by appending the payload 
```html
<img src=1 onerror=alert('1')>
``` 
- which will result in the jQuery 
![[Pasted image 20220720150952.png]]
![[Pasted image 20220720150934.png]]
- As can be seen above, an alert popped up. 
### Exploitation
- However, there is a problem. The payload above will not work directly on a victim's browser as to the victim, there is no change in hash. Hence we will need to invoke a `hashchange` for it to work. Additionally, we will also need to  call the `print()` function.
- Since the input is a HTML string, we can make use of the `iframe` HTML tag which embeds another document.
- The idea is to use the `iframe` to load the webpage with one hash and in the same frame load the URL with the payload thus invoking a `hashchange`.
- Payload to use will be :
```html
<iframe src="https://0a3f000403f29a80c05636c400ae007e.web-security-academy.net/#1234" onload="this.src = this.src + '<img src=a onerror=print()>'"/>
```
As can be seen below, the `iframe` is invoked and `print()` function is called.
![[Pasted image 20220720154322.png]]