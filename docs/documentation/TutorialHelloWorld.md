# Hello World in NPL

This tutorial demonstrates 
* how to setup a simple development environment 
* write a program that output hello world
* explains the basic concept behind the scene

> This article is an aggregates of information you will find in this wiki guide. Skip this tutorial, if you have already read all related pages.

## Setup Development Environment
For more information, see [[Install Guide|InstallGuide]].

* Download [ParacraftSDK](https://github.com/LiXizhi/ParaCraftSDK/archive/master.zip)
  * or you can clone it by running: 
```
git clone https://github.com/LiXizhi/ParaCraftSDK.git
```
* Make sure you have updated to the latest version, by running `./redist/paracraft.exe` at least once
* Run `./NPLRuntime/install.bat`
   * The NPL Runtime will be copied from `./redist` folder to `./NPLRuntime/win/bin`
   * The above path will be added to your %path% variable, so that you can run npl script from any folder.

To write NPL code, we recommend using either or both of following editors (and plugins):
* [Visual Studio Community Edition](https://www.visualstudio.com/): the free version of `visual studio`
  * In visual studio, select `menu::Tools::Extensions`, and search online for `npl` to install following plugins:
     * [NPL/Lua language service for visual studio](https://visualstudiogallery.msdn.microsoft.com/7782dc20-924a-4726-8656-d876cdbb3417)
     * [NPL Debugger for visual studio](https://visualstudiogallery.msdn.microsoft.com/7ebe665c-4f1d-41fd-91e1-52176cf2d9db)
* [Visual Studio Code](https://code.visualstudio.com/): a cross-platform free editor based on [Atom](https://atom.io/)

## Hello World Program

Run in command line `npl helloworld.lua`. 
The content of `helloworld.lua` is like this:
~~~lua
print("hello world");
~~~
You should see a file `./log.txt` in your current working directory, which contains the "hello world" text. 

> If `npl` command is not found, make sure `./NPLRuntime/win/bin` is added to your search %path%; or you can simply use the full path to run your script (see below).


```
__YourSDKPath__/NPLRuntime/win/bin/npl helloworld.lua
```

## Behind the Scenes
ParacraftSDK's `redist` folder contains an installer of [Paracraft](http://www.paracraft.cn). 
Running `Paracraft.exe` from redist directory will install the latest version of Paracraft, which ships with NPLRuntime. `./NPLRuntime/install.bat` copies only NPL runtime related files from redist folder to `./NPLRuntime/win/bin` and `./NPLRuntime/win/packages`

`./NPLRuntime/win/packages` contains pre-compiled source code of NPL libraries. 

To deploy your application, you need to deploy both `bin` and `packages` folders together with your own script and resource files, such as 

```
./bin/       :npl runtime 32bits 
./bin64/     :npl runtime 64bits
./packages/  :npl precompiled source code
./script/    :your own script
./           :the working (root) directory of your application
```
In NPL script, you can reference any file using path relative to the working directory. So you need to make sure you start your app from the root directory.

## Further Readings
[ParacraftSDK](https://github.com/LiXizhi/ParaCraftSDK) contains many example projects written in NPL. 
* See [ParacraftSDK wiki](https://github.com/LiXizhi/ParaCraftSDK/wiki) site for more details.
* See [[Source Code Overview | SourceCodeOverview]]
