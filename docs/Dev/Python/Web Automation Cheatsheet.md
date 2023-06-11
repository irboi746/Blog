# Web Automation Cheatsheet
A cheatsheet inspired and based on [Rizemon's Exploit Writing for OSWE](https://github.com/rizemon/exploit-writing-for-oswe). "exploit writing" goes beyond OSWE and what was taught is also useful in engagements especially when you need to automate the tedious stuff, hence this cheatsheet so that you can spin up a script real quick. Will add [[#Code Snippets]] for other libraries as I go along.

Of course no matter how quick you can script, it is also worth to note the Return of Investment (ROI) of automation:

> Time saved with Automation > Time spent to Code

## Code Snippets
### Useful Imports
```
# For sending HTTP requests
import requests

# For Base64 encoding/decoding
from base64 import b64encode, b64decode, urlsafe_b64encode, urlsafe_b64decode

# For getting current time or for calculating time delays
from time import time

# For regular expressions
import re

# For running shell commands
import subprocess

# For running a HTTP server in the background
import threading
from http.server import HTTPServer, BaseHTTPRequestHandler

# For quick creation of arguments and help menu
import argparse
```

## `requests` Library
### HTTP Header
#### Set headers
```
headers = {
    "X-Forwarded-For": "127.0.0.1"
}

requests.get("https://github.com", headers=headers)
```
#### Set Cookies
```
cookies = {
    "PHPSESSID": "fakesession"
}

requests.get("https://github.com", cookies=cookies)
```

### Requests
```
# GET method
requests.get("https://github.com")

# POST method
requests.post("https://github.com")

# PUT method
requests.put("https://github.com")

# PATCH method
requests.patch("https://github.com")

# DELETE method
requests.delete("https://github.com")
```

#### Requests thru Proxy
```
PROXY = {
   'http':'http://127.0.0.1:8080'
   'https':'http://127.0.0.1'
}
requests.get("https://github.com",proxies=PROXY)
```

#### Disable Redirect
```
requests.post("https://github.com/login", allow_redirects=False)
```

#### Requests Body
##### Sending data as a query string in the URL 
```
params = {
    "foo": "bar"
}

requests.get("https://github.com", params=params)
```

##### Sending data as a query string in the body 
```
data = {
    "foo": "bar"
}
# sends data as "data" e.g "foo=bar"
requests.post("https://github.com", data=data)

# sends data as "json" e.g "{'foo': 'bar'}" 
requests.post("https://github.com", json=data)
```

##### Sending a File in the body
```
with open('readme.txt','r') as f:
   requests.post("https://github.com", files=f)
```

#### Sessions
##### Creating Session
```
session = requests.Session()
session.get("https://github.com")
```

##### Set Session Header
```
session = requests.Session()
session.headers["Authorization"] = "Basic 123"
```

##### Update Session Cookie
```
session = requests.Session()
session.cookies.update({"PHPSESSID": "fakesession"})
```

### Response
```
resp_obj = requests.get("https://github.com")

# HTTP status code (e.g 404, 500, 301)
resp_obj.status_code

# HTTP response headers (e.g Location, Content-Disposition)
resp_obj.headers["Location"]

# Body as bytes
resp_obj.content

# Body as a string
resp_obj.text

# Body as a dictionary (if body is a JSON)
resp_obj.json()
```

## Reusable Code
### Start template
```
import argparse
import requests

def main():
   parser = argparse.ArgumentParser()
   parser.add_argument('-i','--IP',help='IP address',required=True)
   parser.add_argument('-p','--proxy',help='ip address of proxy',required=False)
   args = parser.parse_args()
   
   print("Hello World!")

if __name__ == "__main__":
   main()
```

## Tips
### Meaningful Messages 
```
print('[+] Exploit has succeeded') 
print('[-] Exploit has failed')
print('[=] Running Process')
```
