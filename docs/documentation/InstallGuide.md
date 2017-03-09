#Install NPL Runtime

You can redistribute NPLRuntime side-by-side with your scripts and resource files on windows/linux/iOS, etc.
However, this section is only about installing NPL runtime to your development computer and setting up a recommended coding environment for NPL. 

## Install on Windows
### Install from Windows Installer 
It is very easy to install on windows. The win32 executable of NPL runtime is currently released in two forms, choose one that is most suitable to you:
- 32bits/64bits most recent development version (recommended for standalone application or module dev). [Click to download](https://github.com/LiXizhi/NPLRuntime/releases)
- Official stable version released in ParacraftSDK (recommended for paracraft mod dev). See below.

[Paracraft](http://www.paracraft.cn) is an IDE written completely in NPL. 
Follow the steps:
* Download [ParacraftSDK](https://github.com/LiXizhi/ParaCraftSDK/archive/master.zip)
  * or you can clone it with following commands (you need [git](https://git-scm.com/) installed): 
```bash
# clone repository & sub modules
git clone https://github.com/LiXizhi/ParaCraftSDK.git
cd ParaCraftSDK
git submodule init
git submodule update
# Go into the repository
cd ParaCraftSDK
# install paracraft
redist\paracraft.exe
# install NPLRuntime
NPLRuntime\install.bat
```
 * `redist\paracraft.exe` will install the most recent version of `Paracraft` to  `./redist` folder
   * if you failed to install, please manually download from [Paracraft](http://www.paracraft.cn) and copy all files to `redist` folder.
 * Inside `./NPLRuntime/install.bat`
   * The NPL Runtime will be copied from `./redist` folder to `./NPLRuntime/win/bin`
   * The above path will be added to your `%path%` variable, so that you can run npl script from any folder. 

#### Update NPL Runtime to latest version
- If you use NPLRuntime installer, simply [download and reinstall it here](https://github.com/LiXizhi/NPLRuntime/releases)
- If you use paracraftSDK, we regularly update `paracraft` (weekly) on windows platform, you can update to latest version of NPL Runtime following following steps. 
  - run `redist\paracraft.exe` to update paracraft's NPL runtime in `redist` folder
  - run `NPLRuntime\install.bat` to copy NPL runtime executable from `redist` to `NPLRuntime/win/bin` folder
- Compiling from source code is always the latest version (recommended for serious developers), see next section.

#### On 64bits version
To build 64bits version of NPLRuntime, you need to build from source code (see next section), or download prebuild version from [[http://www.paracraft.cn]]. 

Our current build target for public users is still 32bits on windows platform. 

> Please note that all compiled and non-compiled NPL/lua code can be loaded by both 32bits/64bits version of NPL runtime across all supported platforms (windows/linux/mac/android/ios) etc. However, if you use any C++ plugins (such as mysql, mono, etc), they must be built for each platform and CPU architecture. 

### Install on Windows From Source
> see [.travis.yml](https://github.com/LiXizhi/NPLRuntime/blob/master/appveyor.yml) or our [build output](https://ci.appveyor.com/project/DarrenGZY/nplruntime-e8wud) for reference

* You need to download and build [boost](http://www.boost.org/) from source code. Version `1.55.0` and `1.60.0` are tested,but the latest version should be fine.
Suppose you have unzipped the boost to `D:\boost`, you can build with following command.
```
bootstrap
b2 --build-type=complete
```
and then add a system environment variable called `BOOST_ROOT`, with value `D:\boost`(or where your boost is located), so that our build-tool will find boost for you.
* Download [NPLRuntime source code](https://github.com/LiXizhi/NPLRuntime), or clone it by running:
```
git clone https://github.com/LiXizhi/NPLRuntime.git
```
* Download [cmake](https://cmake.org/), and build either `server` or `client` version. They share the same code base but with different `cmakelist.txt`; `client` version has everything `server` has plus graphics API. 
   * `Server` version: run `NPLRuntime/NPLRuntime/CMakeList.txt`  in `NPLRuntime/bin/win32` folder (any folder is fine, but do not use the `cmakelist.txt` source folder), please refer to `./build_win32.bat` for details
   * `Client` version: run `NPLRuntime/Client/CMakeList.txt`  in `NPLRuntime/bin/Client` folder (any folder is fine, but do not use the `cmakelist.txt` source folder)
       * **REQUIRED:** Install latest DirectX9 SDK in default location [here](https://github.com/LiXizhi/NPLRuntime/blob/master/NPLRuntime/ParaEngineClient/cmake/DirectX.cmake). 
       * There is also a cmake option called `ParaEngineClient_DLL` which will build NPLRuntime as shared library, instead of executable. 
* If successful, you will have visual studio solution files generated. Open it with visual studio and build.
* The final executable is in `./ParaWorld/bin/` directory.  

#### How to Debug NPLRuntime C++ source code 
When you build NPLRuntime from source code, all binary file(dlls) are automatically copied to `./ParaWorld/bin/` directory. All debug version files have `_d` appended to the end of the dll or exe files, such as `sqlite_d.dll`, etc. 

To debug your application, you need to start your application using the executable in `./ParaWorld/bin/`, such as `paraengineclient_d.exe`. You also need to specify a working directory where your application resides, such as `./redist` folder in `ParacraftSDK`. Alternatively, one can also copy all dependent `*_d.dll` files from `./ParaWorld/bin/` to your application's working directory.

Applications written in NPL are usually in pure NPL scripts. So you can download and install any NPL powered application (such as [this one](http://www.paracraft.cn) ), and set your executable's working directory to it before debugging from source code. `ParacraftSDK` also contains a good collection of example NPL programs. Please see the video at [[TutorialWebSite]] for install guide.
 
## Install on Linux
> Click here to download precompiled [NPL Runtime linux 64bits release](https://github.com/LiXizhi/NPLRuntime/releases).

Under linux, it is highly recommended to build NPL from source code with latest packages and build tools on your dev/deployment machine. 
* Download [NPLRuntime source code](https://github.com/LiXizhi/NPLRuntime), or clone it by running:
```
git clone https://github.com/LiXizhi/NPLRuntime.git
git submodule init && git submodule update
```
* Install third-party packages and latest build tools. 
  * see [.travis.yml](https://github.com/LiXizhi/NPLRuntime/blob/master/.travis.yml) or our [build output](https://travis-ci.org/LiXizhi/NPLRuntime) for reference (on debian/ubuntu). In most cases, you need `gcc 4.9 or above, cmake, bzip2, boost, libcurl`. Optionally, you can install `mysql, mono, postgresql` if want to build NPL plugins for them. 
* Run [./build_linux.sh](https://github.com/LiXizhi/NPLRuntime/blob/master/build_linux.sh) from the root directory. Be patient to wait for build to complete. 
```
./build_linux.sh
```

> Please note, you need to have at least 2GB memory to build with gcc, or you need to step up [swap memory](http://www.cyberciti.biz/faq/linux-add-a-swap-file-howto/) 
  
* The final executable is in `./ParaWorld/bin64/` directory. A symbolic link to the executable is created in `/usr/local/bin/npl`, so that you can run NPL script from anywhere.

## Install on MAC OSX
We use [cmake](https://cmake.org/) to build under MAC OSX, it is very similar to `install on linux`, but you should use 
```
./build_mac_server.sh
```
Currently, only server version is working on mac, client version is still under development.

## Editors and Debuggers
You can use your preferred editors that support lua syntax highlighting. 
NPL Runtime has a built-in web server called [[NPL Code Wiki|NPLCodeWiki]], it allows you to edit and debug via HTTP from any web browser on any platform.
* [[NPL Code Wiki|NPLCodeWiki]]: build-in debugger and editor.

It is though highly recommended that you install NPL Runtime on windows, and use following editors:
* [Visual Studio Community Edition](https://www.visualstudio.com/): the free version of `visual studio`. This one is what I use.
* [Visual Studio Code](https://code.visualstudio.com/): a very good free/cross-platform editor based on `Atom`


We have developed several useful open-source NPL language plugins for visual studio products:
In visual studio, select `menu::Tools::Extensions`, and search online for `npl` to install following plugins:

* [NPL/Lua language service for visual studio](https://visualstudiogallery.msdn.microsoft.com/7782dc20-924a-4726-8656-d876cdbb3417): used by thousands of developers worldwide.
   * Download [Source code](https://github.com/LiXizhi/NPL)
* [NPL Debugger for visual studio](https://visualstudiogallery.msdn.microsoft.com/7ebe665c-4f1d-41fd-91e1-52176cf2d9db): set break points and attach to running NPL process.
   * Download [Source code](https://github.com/LiXizhi/NPL)
* [NPL/ParaEngine Tools](https://visualstudiogallery.msdn.microsoft.com/accd022b-ec37-4614-8c08-30e291d28bd5) for visual studio: misc tools

![NPL language service](https://i1.visualstudiogallery.msdn.s-msft.com/7782dc20-924a-4726-8656-d876cdbb3417/image/file/168690/1/npl_language_service.png)

![Editors](https://visualstudiogallery.msdn.microsoft.com/site/view/file/136918/1/peeditor2.png)

## Example Code
[ParacraftSDK](https://github.com/LiXizhi/ParaCraftSDK) contains many example projects written in NPL. 
* See [ParacraftSDK wiki](https://github.com/LiXizhi/ParaCraftSDK/wiki) site for more details.
* You may now get your hand dirty by this [tutorial](https://github.com/LiXizhi/ParaCraftSDK/wiki/TutorialSimplePlugin) or continue reading here.