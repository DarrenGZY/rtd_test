# NPL Code Wiki

NPL Code Wiki is a web site which can be served directly from your local computer using NPL Runtime. 
It provides you a way to communicate with NPL Runtime via any web browser. 
* NPL Code Wiki is designed to help people learn and practice NPL programming. 
* It provides a way to inspect or debug NPL Runtime via HTTP protocol. 
* Providing an interactive console to edit/debug NPL code at runtime.

## Start Code Wiki
### Start With NPL Runtime
Run from console
```
npl script/apps/WebServer/WebServer.lua
```
The default web address is [[http://127.0.0.1:8099/]]. For more information, please see [[WebServer]] section.

### Start With Paracraft
To launch NPL Code Wiki, you need to 
* install [Paracraft](http://www.paracraft.cn) 
* create an empty world
* press F11 to launch. NPL Code wiki will be opened using your default web browser.

You can then enter any NPL code in the console. See below:

![nplconsole](https://cloud.githubusercontent.com/assets/94537/13699921/8e9b88b6-e7b8-11e5-8558-a310642919e9.png)

NPL Code Wiki Source Code is at [script/apps/WebServer](https://github.com/LiXizhi/ParaCraftSDK/tree/master/src/script/apps/WebServer)

Alternatively, you can start it via following code. Press `F12` in any world in Paracraft to enter code.
```
NPL.load("(gl)script/apps/WebServer/WebServer.lua");
WebServer:Start("script/apps/WebServer/admin");
```
## NPL Debugger
NPL debugger is a HTTP protocol based debugger. It can be opened from `menu::tools::NPL debugger`, or with `Ctrl+Alt+I` in paracraft.
![npldebugger](https://cloud.githubusercontent.com/assets/94537/14600035/5c626b10-058d-11e6-886f-be5954010bf4.png)

Because it is based on HTTP and served from the same NPL runtime you are debugging, you can debug your running application from any modern web browser. For example, you can debug apps running on linux server or mobile device remotely from a windows client. However, you must make sure that the apps are started with full source code available in its running directory. 

### How to Attach 
There is no need to start your app in debug mode. Instead, just launch NPL code wiki, and follow the steps:
* In the web page, open your script files and press F9 or click the left margin of the line to set breakpoints. 
* Click `Attach` button to begin debugging. 
  * You can then press F10, F11, Shift+F11 or press the buttons to step through your code. 
  * In the watcher window, you can evaluate any code or variable. Just type the variable name or code and click `evaluate`
* When debugger is attached, your program may run pretty slow, so once you are finished, click `Stop` button to detach.

### Limitations
Currently only the main thread can be debugged, if you want to debug other thread, you can write your own debugger in NPL, the complete source code of the NPL debugger web site is only 400 lines. 

## NPL Code Editor
This is a multi-tab code editor for NPL. It can be opened from menu::view::code editor.

* Enter file name and press enter to open the file.
* Click the `sync` button on left top corner to navigate to the active document's directory.
* Even if you close your computer, your last opened files will be remembered when you open it again.
* Right click on the file name tab for a list of commands, such as `open containing folder`, `reload`, etc. 
* Modified files will be marked with `*` in its tab, and need to be confirmed before closing it. 
* `Ctrl+S` to save to disk.
* Filter files as you type in the left top's directory text box. 

![nplcodeeditor](https://cloud.githubusercontent.com/assets/94537/14739623/5772a710-08bb-11e6-95b0-06d569df897a.png)


## Object Inspector
Click `Menu::View::ObjectInspect` to open `object inspector`. Here you can view and edit all objects in NPL Runtime.

![nplcodewikiobjectinspector](https://cloud.githubusercontent.com/assets/94537/13727921/f45f0af4-e940-11e5-8bea-99486cc686a1.png)

Once you edited a property, a command string is generated on top of the property window. The command string is equivalent to your last operation. In above image, the command string to change the window title is: 
```
/property -all WindowText Paracraft
```
> The server-side NPL code that realized the `Object Inspector` page can be viewed by clicking the `view source` link on the bottom of the web page. This provides you a way to study the NPL implementation of whatever you see in NPL Code wiki web site. 

## The Console Page
Click `Menu::Tools::Console` to open `Console Window`. Here you can enter any NPL code or NPL web page code to be evalulated at runtime. 

For example, enter following code and press `Run as Code` button:
```
print("hello world")
``` 
By default, text is output to `log.txt` file in the current working directory. 

```
alert("hello world")
``` 
The above code will display a message box, if code wiki is launched with Paracraft. 

## The Log Page
Click `Menu::Tools::Log` to open `Log Window`. 
Both the console page and the log page can display content in `log.txt`. 

![image](https://cloud.githubusercontent.com/assets/94537/19139724/831513b8-8bb9-11e6-9121-a736b100fcb2.png)

- One can leave this window open even when your application restarts. It will automatically scroll to the newest log for you. 
- Check `preserve log` to preserve logs between each application restarts.
- If there are critical errors or warnings, an error sign with a text number will be displayed in red on top of the log window, click it to navigate to the line of interest in the log file. 
