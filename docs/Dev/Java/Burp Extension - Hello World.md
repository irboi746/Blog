# Burp Extension - Hello World 
Lao Tzu once said "A journey of a thousand miles begins with a single step" and that single step to our journey in creating a Burp extension or any development endeavour will be the "Hello World"! 

In this tutorial, we will create a Hello World using the [Hello World Example](https://github.com/PortSwigger/burp-extensions-montoya-api-examples/blob/main/helloworld/src/main/java/example/helloworld/HelloWorld.java) from Portswigger.
## Setting Up
We will start with environment set-up. In our set-up we will be using Eclipse as our IDE.
### Install Burp
In our case, we will be using Burp Community and we will follow [this Burp Suite's official guide](https://portswigger.net/burp/documentation/desktop/getting-started/download-and-install) to install Burp Community.
### Install Eclipse
We then follow [this guide](https://itsfoss.com/install-latest-eclipse-ubuntu/) to install eclipse but the version installed is 2023-09 (4.29.0).
### Add external Jar 
Following [n3utr1n0's guide](https://n3utr1n0.medium.com/steps-to-create-burp-extension-easily-eclipse-183ca3f3c891), we will need to add an external Burpsuite jar file, however there are some deviations from the guide like the use of BurpSuite Community and a newer version of Eclipse 2023-09 (4.29.0).

After creating a new project, we can configure build path  `burpsuite-community.jar` using the following :

Step 1: Right-click on the explorer

![[Pasted image 20231111110324.png]]

Step 2: Click on "Classpath" and select "Add External JARs"

![[Pasted image 20231111110926.png]]

Step 3 : Browse to the BurpSuite Folder and select `burpsuite_community.jar`.

![[Pasted image 20231111111543.png]]

## Creating Extension
After setting up we will now start creating the extension.
### Create Package
Now, we go to right-click on "src", select "New" and the "Package" as shown below.

![[Pasted image 20231111112418.png]]

We will name the Name the same as the project name.

![[Pasted image 20231111112607.png]]

When package is created, we create a new Class as follows.

![[Pasted image 20231111112814.png]]

### Exporting
Then we can just copy and paste the Hello World example and prepare to export as jar using the following steps:

Step 1 : Select Export

![[Pasted image 20231111120138.png]]

Step 2 : In the Export Wizard select "JAR file"

![[Pasted image 20231111120211.png]]

Step 3 : Select Destination

![[Pasted image 20231111120255.png]]

Step 4 : Uncheck "Export class files with compile warnings" then "finish"!

![[Pasted image 20231111120321.png]]

## Misc. Issues faced
We might face an issue like below where the burp imports are "not accessible" even when we have already configured the build path to import `burpsuite_community.jar`.

![[Pasted image 20231111115655.png]]

Based on the following [eclipse forum](https://www.eclipse.org/forums/index.php/t/1103208/) response we simply need to delete the `module-info.java` file and the error will be gone.

> Delete the module-info.java file. It's only used if you're using Java's built-in module system.

Alternatively, when creating a project, we can uncheck create `module-info.java`.

![[Pasted image 20231111143737.png]]

## Loading Extension
Firstly navigate to the "Extensions" tab,  then load the extension by clicking  "Add".

![[Pasted image 20231111175304.png]]

Browse for the `.jar` file that has been exported

![[Pasted image 20231111175415.png]]

As can be seen the extension has been loaded successfully. 

![[Pasted image 20231111175449.png]]

## Reference
- https://n3utr1n0.medium.com/steps-to-create-burp-extension-easily-eclipse-183ca3f3c891
- https://bishopfox.com/blog/power-pen-tests-with-montoya-api