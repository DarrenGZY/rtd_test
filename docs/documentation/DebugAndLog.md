# Debugging And Logs
There are many ways to debug your NPL scripts, including printing logs, via [[NPL Code Wiki|NPLCodeWiki]], or via IDE like visual studio or vscode, etc.  

## Logs in NPL
`log.txt` file is the most important place to look for errors. When your application starts, NPL runtime will create a new `log.txt` in the current working directory. 

### Rule of Silence
The entire Unix system is built on top of the philosophy of [rule of silence](http://www.linfo.org/rule_of_silence.html), which advocate every program to output nothing to standard output unless absolutely necessary. 

So by default, NPL runtime output nothing to its standard output. All NPL output API like `log`, `print`, `echo` write content to `log.txt` instead of standard output. There is only one hidden API `io.write` which can be used to write to standard output. 

By default, NPL runtime and its libraries output many info to log files. This log file can grow big if your server runs for days. There are two ways to deal with it: one is to use a timer (or external shell script) to backup or redirect `log.txt` to a different file everyday; second is to change the default log level to `error`, which only output fatal errors to `log.txt`. Application developers can also write their own logs to several files simultaneously, please see `script/ide/log.lua` for details.

When application starts, one can also change the default log file name, or whether it will overwrite existing `log.txt` or append to it. Please refer to log interface C++ API for details.

### Use Logs For Debugging
The default log level of NPL is `TRACE`, which will write lots of logs to `log.txt`. At development time, it is good practice to keep this file open in a text editor that can automatically refresh when file content is changed. 

When you feel something goes wrong, search for `error` or `runtime error` in `log.txt`. NPL Code Wiki provide a log page from `menu:view:log`, where you can filter logs. See below:

![image](https://cloud.githubusercontent.com/assets/94537/19139724/831513b8-8bb9-11e6-9121-a736b100fcb2.png)

Many script programmers prefer to insert `echo({params, ...})` in code to output debug info to `log.txt`, and when code is working, these `echo` lines are commented out. 

A good practice is to insert permanent log line with debug level set to `debug` like `LOG.std(nil, "debug", "modulename", "some text", ...)`.

## Use A Graphical Debugger

### HTTP Debugger in NPL Code Wiki
Sometimes inserting temporary log lines into code can be inefficient. One can also use NPL HTTP debugger to debug code via HTTP in a HTML5 web browser with nice GUI. NPL HTTP Debugger allows you to debug remote process, such as debugging linux/android/ios NPL process on a windows or mac computer. See [[NPLCodeWiki]] for details.

![npldebugger](https://cloud.githubusercontent.com/assets/94537/14600035/5c626b10-058d-11e6-886f-be5954010bf4.png)

If you have installed [NPL/Lua language service for visual studio](https://visualstudiogallery.msdn.microsoft.com/7782dc20-924a-4726-8656-d876cdbb3417), one can set HTTP-based breakpoints directly inside visual studio. 

In visual studio, open your NPL script or page file, right click the line where you want to set breakpoint, and from the context menu, select `NPL Set Breakpoint Here`.  

![setbreakpoint](https://cloud.githubusercontent.com/assets/94537/16826877/ccc6e736-49b3-11e6-9937-a72703a84ef2.png)

If you have started your NPL process, visual studio will open the HTTP debugger page with your default web browser. This is usually something like `http://localhost:8099/debugger`. Breakpoints will be automatically set and file opened in the browser.

![httpdebugger](https://cloud.githubusercontent.com/assets/94537/16826881/d7df6e04-49b3-11e6-881b-7c07324994d4.png)

### Debugger Plugin for Visual Studio 
We also provide open source NPL debugger plugins that is integrated with visual studio and vscode, see [[InstallGuide]] for details.However, it is a little complex to install visual studio debugger plugin, please follow instruction [here: How To Install NPL Debugger for VS](https://visualstudiogallery.msdn.microsoft.com/7ebe665c-4f1d-41fd-91e1-52176cf2d9db)

Once installed, you can debug by selecting from visual studio menu `Tools::Launch NPL Debugger...`

![vsdebugger](https://i1.visualstudiogallery.msdn.s-msft.com/7ebe665c-4f1d-41fd-91e1-52176cf2d9db/image/file/188983/1/npldebugger2.png)



