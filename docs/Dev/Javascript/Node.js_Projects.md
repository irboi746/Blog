# NodeJS Projects
## Simple Webpage with Vanilla Node
### Inefficient Webpage implementation
- Below is an implementation of a Node web server that will respond to requests.
- However, this implementation is not efficient and elegant as we will need to create multiple if-else cases in the server object which will clutter the code.   

```Javascript
//import libraries
const http = require('http');
const path = require('path');
const fs = require('fs');

// declare port to listen on - either environment var or pre-defined
const PORT = process.env.port || 5000;

//create server object
const server =http.createServer((req,res)=>{
    //basic request handling
    if (req.url == '/'){
        fs.readFile(path.join(__dirname,'public','hello.html'),
        (err,content)=>
        {
            if (err) throw err;
            res.writeHead(200, {'Content-Type': 'text/html'});
            res.end(content);
        }) 
    }
});

// start server listener
server.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

### Slightly Better Implementation
- Below is a slightly better implementation of the web server but still not the best. It can be more elegant and cleaner by more abstraction.
```Javascript
//import libraries
const http = require('http');
const path = require('path');
const fs = require('fs');

// declare port to listen on - either environment var or pre-defined
const PORT = process.env.port || 5000;

const server =http.createServer((req,res)=>{
    //buildFilePath
     let filePath = path.join(
     __dirname,
     'public',
     req.url === '/' ? 'hello.html' : req.url);

     // Get File Extension
     let extname = path.extname(filePath);
     
     // Set Content-Type
     let contentType = 'text/html';
     //Check for ext and set Content-type
    switch(extname){
        case '.js':
            contentType = 'text/javascript';
            break;
        case '.css':
            contentType = 'text/css';
            break;
        case '.json':
            contentType = 'application/json';
            break;
        case '.png':
            contentType = 'image/png';
            break;
        case '.jpg':
            contentType = 'image/jpg';
            break;
    }
    //Read and serve file
    fs.readFile(filePath,(err,content) => {        
        if(err) {
            //Handle 404 Error
            if(err.code == 'ENOENT'){
                fs.readFile(path.join(__dirname,'public','404.html'), 
                (err,content)=> {
                    res.writeHead(200, {'Content-Type': 'text/html'});
                    res.end(content, 'utf8');
                })
            } 
            // Some other server errors
            else{
                res.writeHead(500);
                res.end(`Server Error: ${err.code}`);
            }
        }
        //Success Response if resource is found
        else{
            res.writeHead(200,{'Content-type':contentType});
            res.end(content,'utf8');

        }
    });

});

server.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

- A simple webpage project will end here. Usually framework like ExpressJS is used to create a simple webserver and using frameworks will make the process easier and cleaner.

## Adding of Portswigger Academy Lab names to Trello board via Trello REST API Automation with NodeJS
- The inspiration of this project comes from the need to automate adding of cards into trello. 
- Adding of cards using the web application is too slow and repititive as we have 200 over cards name to add.
- References : 
	- [Official Trello REST API Documentation](https://developer.atlassian.com/cloud/trello/rest/api-group-cards/#api-cards-post)
	- [Online Guide](https://digitalcommunications.wp.st-andrews.ac.uk/2021/12/15/adding-cards-with-the-trello-api/)

### Cleaning Data from Portswigger
- We looked into the HTML of the portswigger website and noticed that the title is an HTML anchor.
![[Pasted image 20220926181401.png]]
- Hence, with the help of curl and grep we extract only all the anchor texts that contains the lab links.
- We also noticed that all the labs will start with `lab-` in the URL.

```
curl https://portswigger.net/web-security/all-labs | grep 'lab-' > out.txt
```

![[Pasted image 20220926181657.png]]

- We wanted to modify the marker "EXPERT", "PRACTITIONER" and "APPRENTICE" to "E","P","A" respectively and still have the lab title.
- We then make use of cyberchef to get the cleaned ouput. The Cyberchef recipe is as below : 

```json
[{"op":"Strip HTML tags","args":[true,true]},{"op":"Find / Replace","args":[{"option":"Regex","string":"APPRENTICE"},"\\r|A|",true,false,true,false]},{"op":"Find / Replace","args":[{"option":"Regex","string":"PRACTITIONER"},"\\r|P|",true,false,true,false]},{"op":"Find / Replace","args":[{"option":"Regex","string":"EXPERT"},"\\r|E|",true,false,true,false]},{"op":"Remove whitespace","args":[false,false,true,true,true,false]}]
```

Sample output of cleaned data :

![[Pasted image 20220927140704.png]]

### Getting the required Information from Trello
- Based on the trello references above, we know that we will need four variables: 
	- `idlist`: ID of the list we want to add to
	- `key`: API Key for Trello
	- `token`: Server Token for Trello 
	- `name`: Name of the card to be added 

### Node.JS Code
- The idea would be to create a `add_card` function that takes in the name to be added in a `function.js` file 
- Then, using a foreach loop in `index.js` read the text file and pass the variable into the `add_card` function in the iteratively.
- The solution below will result in "ERROR 429"., as there is [rate limiting](https://developer.atlassian.com/cloud/trello/guides/rest-api/rate-limits/) on Atlassian end 
- Ideally we will need to [create a](https://stackoverflow.com/questions/33289726/combination-of-async-function-await-settimeout) promisified `sleep` function that can be awaited. (Maybe Version 2?)

#### Index.js
```javascript
import { add_card,syncReadFile } from './functions.js';

var names = syncReadFile('o.txt','utf-8');
names.forEach(name => add_card(name));
```

#### functions.js
```javascript
import fetch from 'node-fetch';

export async function add_card(card_name){
    try {
        const response = await fetch('https://api.trello.com/1/cards?'+
	        'idList=[list_id]&' +
            'key=[API_KEY]&' +
            'token=[API_Token]&' +
            'name=' + card_name.toString(), {
            method: 'POST',
            headers: {
                'Accept': 'application/json'
            }
        });
        console.log(
            `Response: ${response.status} ${response.statusText}`
        );
        const text = await response.text();
        return console.log(text);
    } catch (err) {
        return console.error(err);
    }
}

export function syncReadFile(filename) {
    const contents = readFileSync(filename, 'utf-8');
  
    const arr = contents.split(/\r?\n/);
  
    console.log(arr); 
  
    return arr;
  }
```


## Real Time Markdown Editor (Re-make)
- The Sample Project is a remake of this this project [real-time markdown viewer](https://www.digitalocean.com/community/tutorials/building-a-real-time-markdown-viewer) from [repo that teaches coding through projects](https://github.com/practical-tutorials/project-based-learning#javascript). This project is a collaborative markdown editor which allows multiple users to edit and view the same markdown together in realtime.
- The old project is using vulnerable dependency (ShareJS) and redis server which in our implementation we do not want to use any of these. As such, there are some differences as compared to the original implementation.
- We will base our project on this [repo](https://github.com/genzyy/Realtime-Markdown-Editor) which is a WYSIWYG markdown editor and build the collaboration portion from here.
- ExpressJS and [EJS](https://www.geeksforgeeks.org/use-ejs-as-template-engine-in-node-js/ )will be the main dependencies used in this project.

#### Initialising Project
```
mkdir RTMDEditor
code RTMDEditor
npm init
npm install express ejs 
```

{WIP}