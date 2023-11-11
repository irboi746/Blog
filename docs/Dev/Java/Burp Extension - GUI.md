# Burp Extension - GUI
The appeal of writing a Burp Extension is to visualise our actions during the pentest. Therefore, the next important thing to do will be do learn how to create a GUI and interact with it before dwelling into writing the functions. We will be continuing from the [[Burp Extension - Hello World]].
## Setting Up 
Burp uses the Java Swing Library for the GUI hence, we will need to install this as a plugin to our Eclipse IDE.
### Installing relevant dependencies
Step 1: Navigate to the "Help" tab and the "Install New Software" button

![[Pasted image 20231111164052.png]]

Step 2: Select the relevant site to download from. In our case we choose the first site available.
![[Pasted image 20231111164138.png]]

Step 3: We then filter for "Swing" and "Window" and select everything in that filter.  

![[Pasted image 20231111164245.png]]

![[Pasted image 20231111164956.png]]

After installation and restart we will be ready for the next step.
## Creating GUI
In GUI creation, we will explore a the GUI feature to register a tab. 
### Register a Tab
The following is the documentation for `registerSuiteTab()`, it seems like all we need to do is provide it with a string for the `title` and a `Component` object. 

![[Pasted image 20231111230828.png]]

The following code is a simple example that will give us a "Hello World" Tab within it will have a `JLabel()` with "Hello World". 
```Java
package TestExtension;

import javax.swing.JLabel;

import burp.api.montoya.BurpExtension;
import burp.api.montoya.MontoyaApi;

public class TestExtension implements BurpExtension{
    @Override
    public void initialize(MontoyaApi api)
    {
        // set extension name
        api.extension().setName("Hello world extension");
        
        //Register a Tab
        api.userInterface().registerSuiteTab("Hello World", new JLabel("Hello, World!"));

        // throw an exception that will appear in our error stream
        throw new RuntimeException("Hello exception.");
    }
}
```

As can be seen below, we have registered a tab and the panel contains the Hello World Text.

![[Pasted image 20231111231347.png]]

### Creating JPanel() Component
But of course, we would definitely want to do more than that and we can make use of `JPanel()` for this. To do so we will first need to create a new `.java` file. It is always wise to separate the GUI `.java` from any logic `.java`. 

For the simplicity sake, we will explore just changing the appearance of the GUI and logging the changes in Extension Logs.

Step 1: We right-click on TestExtension -> New -> Other, or we can simply `Ctrl + N`.

![[Pasted image 20231111231645.png]]

Step 2 : In the creation wizard, we select "JPanel"

![[Pasted image 20231111231755.png]]

Step 3 : We can name the `JPanel()` object however we want and in this case we will use the name `TestGUI`

![[Pasted image 20231111231814.png]]

Step 4 : While it is entire possible to design the GUI with code, but to keep myself sane, I decided to use the "Design" feature of eclipse which can be found at the bottom left corner.

![[Pasted image 20231111233636.png]]

Step 5 : For this demonstration we used `GridBagLayout`, seen below by navigating to "Properties" panel and clicking onto the "Layout" tab.

![[Pasted image 20231111233941.png]]

Step 6 : As can be seen below, we can choose the Components to add from the Palette section and `GridBagLayout` allows us to place the Components in a grid.

![[Pasted image 20231111234145.png]]

Step 7 : After adding all the relevant labels and button, we can choose what happens when the button is clicked by right clicking on the button then "Add event handler" ->  "action" -> "actionPerformed".

![[Pasted image 20231111235540.png]]

Step 8 : We add the following action to change the text displayed when selected and logging it through the Montoya API.

```Java
//[...Truncated swing related imports...]
import burp.api.montoya.MontoyaApi;
import burp.api.montoya.logging.Logging;

//[...Truncated other GUI related code...]

public TestGUI(MontoyaApi api){}
tglbtnDisabled.addActionListener(new ActionListener() {
			public void actionPerformed(ActionEvent arg0) {
				if(tglbtnDisabled.isSelected()) {
					tglbtnDisabled.setText("Enabled");
					logging.logToOutput("Extension Enabled");
				}
				else {
					logging.logToOutput("Extension Disabled");
				}
			}
		});
//[...Truncated other GUI related code...]
}
```

Step 9 : We will need to update the following line in our `TestExtension.java` and we are ready to export the extension.
```java
api.userInterface().registerSuiteTab(HELLOW, new TestGUI(api));
```

### Loading the Extension
There we have it! Our first Burp Extension that has a tab a JPanel 

![[Pasted image 20231112001852.png]]

Has buttons that can be toggled On and Off

![[Pasted image 20231112001931.png]]![[Pasted image 20231112001948.png]]

The On/Off actions are logged in output log.

![[Pasted image 20231112002057.png]]

## References 
- https://systemweakness.com/writing-your-first-extension-in-burp-suite-part-2-c61622f73db3
- https://bishopfox.com/blog/power-pen-tests-with-montoya-api
- https://portswigger.github.io/burp-extensions-montoya-api/javadoc/burp/api/montoya/ui/UserInterface.html