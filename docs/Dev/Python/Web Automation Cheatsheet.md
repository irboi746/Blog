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

### CLI as a configuration
One of the most compelling reason to use this is so that you can organise your commands, subcommands and their respective arguments in a dictionary thus greatly improving readability and maintainability.

This method can be observed in [apimap](https://github.com/team-s0l1d1ty/apimap/tree/main/CLI) where nested subcommands are parsed as a python dictionary and generated dynamically. As a result we can simply change the python dictionary and it will be reflected in the CLI. 

Downside is that we will also need a second dictionary that maps the handler function to the arguments.

Shown below is a sample `cli.py` and `handler.py`.

#### cli.py
python dictionary configuration
```python
commands = {
    'cmd1': {
        'help': 'help for cmd1',
        'args': {
            'cmd1_args': {
                'help': 'help for cmd1_param',
                'type': str
            }
        }
    },
    'cmd2': {
        'help': 'help for cmd2',
        'subcommands': {
            'cmd2_subcmd1': {
                'help': 'help for cmd2_subcmd1',
                'args': {
                    'cmd_subcmd1_args': {
                        'help': 'help for cmd2_subcmd1_args',
                        'type': str
                    }
                }
            },
            'cmd2_subcmd2': {
                'help': 'help for cmd2_subcmd2',
                'args': {
                    'cmd2_subcmd2_args': {
                        'help': 'help for cmd2_subcmd2_args',
                        'type': str
                    }
                }
            }
        }
    },
    'cmd3': {
        'help': 'help for cmd3',
        'subcommands': {
            'cmd3_subcmd1': {
                'help': 'help for cmd3_subcmd1',
                'subcommands': {
                    'cmd3_subcmd1_subcmd1': {
                        'help': 'help for cmd3_subcmd1_subcmd1',
                        'args': {
                            'cmd3_subcmd1_subcmd1_args': {
                                'help': 'help for cmd3_subcmd1_subcmd1_args',
                                'type': str
                            }
                        }
                    },
                    'cmd3_subcmd1_subcmd2': {
                        'help': 'help for cmd3_subcmd1_subcmd2',
                        'args': {
                            'cmd3_subcmd1_subcmd2_args': {
                                'help': 'help for cmd3_subcmd1_subcmd2',
                                'type': str
                            }
                        }
                    }
                }
            },
            'cmd3_subcmd2': {
                'help': 'help for cmd3_subcmd2',
                'subcommands': {
                    'cmd3_subcmd2_subcmd1': {
                        'help': 'help for cmd3_subcmd2_subcmd1',
                        'args': {
                            'cmd3_subcmd2_subcmd1_args': {
                                'help': 'help for cmd3_subcmd2_subcmd1_args',
                                'type': str
                            }
                        }
                    }
                }
            }
        }
    }
}
```

cli.py
```python
def generate_args(args_dict, subparser):
    if 'subcommands' in args_dict:
        subparsers_sub = subparser.add_subparsers(dest='subcommand', title='subcommands')
        for subcommand, subcommand_dict in args_dict['subcommands'].items():
            subparser_sub = subparsers_sub.add_parser(subcommand, help=subcommand_dict['help'])
            generate_args(subcommand_dict, subparser_sub)
    if 'args' in args_dict:
        for arg, arg_dict in args_dict['args'].items():
            subparser.add_argument(arg, **arg_dict)

def parse_arguments():
    parser = argparse.ArgumentParser(description='APIMAP')

    subparsers = parser.add_subparsers(title='subcommands', dest='command')
    subparsers.required = True

    # Generate the command-line arguments based on the dictionary
    for command, command_dict in commands.items():
        command_parser = subparsers.add_parser(command, help=command_dict['help'])
        generate_args(command_dict, command_parser)

    return parser.parse_args()

if __name__ == '__main__':
    args = parse_arguments()
```

#### handler.py
python dictionary configuration
```python
command_handlers = {
    'cmd1': handle_cmd1,
    'cmd2': {
        'cmd2_subcmd1': handle_cmd2_subcmd1,
        'cmd2_subcmd2': handle_cmd2_subcmd2
    },
    'cmd3': {
        'cmd3_subcmd1_subcmd1': handle_cmd3_subcmd1_subcmd1,
        'cmd3_subcmd1_subcmd2': handle_gen_cmd3_subcmd1_subcmd2,
        'cmd3_subcmd2_subcmd1': handle_cmd3_subcmd2_subcmd1
    }
}
```

handler.py
```python
# define the handlers here
# handlers can be imported too

def handle_commands(args):
    command = args.command
    handler = command_handlers.get(command)

    if handler is None:
        print(f"Invalid command: {command}")
        return

    if callable(handler):
        handler(args)
    elif isinstance(handler, dict):
        subcommand = args.subcommand
        subcommand_handler = handler.get(subcommand)

        if subcommand_handler is None:
            print(f"Invalid subcommand: {subcommand}")
            return

        if callable(subcommand_handler):
            subcommand_handler(args)
        else:
            handle_commands(args, subcommand_handler)
    else:
        print(f"Invalid command handler: {handler}")
```

## Tips
### Meaningful Messages 
```
print('[+] Exploit has succeeded') 
print('[-] Exploit has failed')
print('[=] Running Process')
```
