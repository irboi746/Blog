# Node.js
- This is document is based on a Node.js [crash course](https://www.youtube.com/watch?v=fBNz5xF-Kx4).
- Node.js is a Javascript runtime (not a language or framework) built on V8 Javascript engine written in C++.
- Node.js allows Javascript to be used as a server side language thus the language used server-side and client-side can be the same. (No need for a python,PHP webserver backend etc. all using Javascript).
- Node is not suitable for application that requires CPU intensive calculations.
- Key feature of Node.js is it's [non-blocking I/O model](https://nodejs.org/en/docs/guides/blocking-vs-non-blocking/).

>Non-blocking I/O operations allow a single process to serve multiple requests at the same time. Instead of the process being blocked and waiting for I/O operations to complete, the I/O operations are delegated to the system, so that the process can execute the next piece of code. 

- Non-blocking I/O can be seen as a workaround for Javascripts inability to do multi-threading. Hence Node.js relies alot on async and non blocking I/O to support concurrent connections.
- More [Sync vs Async](https://www.geeksforgeeks.org/synchronous-and-asynchronous-in-javascript/) in javascript.

## Node's Event Loop
- Below is a diagram of a Node.js thread. As can be seen below, EventEmitters fires off multiple events (processes) but the events are queued and handled in a single thread in a loop.

![[Pasted image 20220909212315.png]]

- More details on [event loop](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/).

## NPM (Node Package Manager)
- Used to install 3rd party packages
- Packages are stored in "node_modules" folder
- Dependencies for projects are listed in "package.json" file

```bash
# To generate a package.json file
npm init

# To install packages locally 
npm install <package name>

# To install packages globally (on system)
npm install -g <package name>

# To install dev dependecies in project
npm install --save-dev <package name>
npm install -D <package name>

# To install dependencies that is used by a project
npm install
```

### Package.json
- A file created by `npm init`, where all the dependencies for the project are listed in.
- Dependencies are automatically updated in `Package.json` file.
- There are 2 types of dependencies : `dependencies` and `devDependencies`.
- `devDependencies` are dependencies used only for development.
- `package-lock.json` is created when npm installs a dependency and it tracks all the dependencies and its versions.

#### Script Function
- Below is an example of a Package.json file and within the red box is the script function.

![[Pasted image 20220910130945.png]]

- The script function allow us to use `npm` to run scripts enclosed within the script object denoted by their identifiers. 

```bash
# Syntax 
npm run <identifier>

# Example
npm run dev
npm run start
```

### Node Modules
- Something like libraries in other programming languages.
- 3 types of modules include : Core Mods, 3rd Party Mods installed or Custome Mods
- As Node uses CommonJS, ES6 method of declaring modules is not applicable.
```bash
# how to declare
const module = require('path_to_module')
```

## Sample Project
- Objective : With the modules introduced below, create a Webserver that serves various web pages and content (index, about, images and 404).

### Installing on Debian/Ubuntu
- We can install Node and NPM on Debian/Ubuntu using apt package manager.

```bash
sudo apt install nodejs npm
```

### Starting a Node Project in VS Code
```bash
# Create a new VS Code Project File
mkdir <dir_name>
cd <dir_name> && code <dir_name>
ctrl + shift + `

# Create a new node project (In VSCode Terminal)
npm init
```

### Basic Components
#### Running a js file with node.
```bash
node <js_file>
nodemon <js_file>
```

#### Core Mod : 
- [Path](https://www.youtube.com/watch?v=fBNz5xF-Kx4)
- [FS-File system](https://nodejs.org/api/fs.html)
- [OS](https://nodejs.org/api/os.html)
- [URL](https://nodejs.org/api/url.html)
- [Events](https://nodejs.org/api/events.html)
- [HTTP](https://nodejs.org/api/http.html)

#### Dev Mod : 
- [nodemon](https://www.npmjs.com/package/nodemon) : a tool that helps develop Node.js based applications by automatically restarting the node application when file changes in the directory are detected.

###  Sample Project Code
```
Sample (WIP)
```
