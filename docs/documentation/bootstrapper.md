# Bootstrapping

Bootstrapping is to specify the first script file to load when starting NPL runtime, it will also be activated every 0.5 seconds. There are several ways to do it via [[command line|NPLCommandLine]]. 

The recommended way is to specify it explicitly with `bootstrapper=filename` parameters. Like below. 
```
npl bootstrapper="script/gameinterface.lua"
```
If no file name is specified, `script/gameinterface.lua` will be used. A more concise way is to specify the file name as first parameter, like below. 
```
npl helloworld.lua
```

Internally, the boot order of NPL is like below:

* When the NPL runtime starts, it first reads all of its command line arguments as name, value pairs and save them to an internal data structure for later use.
* NPL recognizes several predefined command names, one of them is bootstrapper. It specifies the main loop file.
* NPL initializes itself. This can take several seconds, involving reading configurations, graphics initialization, script API bindings, etc.
* When everything is initialized, the majority of NPL's Core API is available to the scripting interface and NPL loads the main loop script specified by the bootstrapper and calls its activation function every 0.5 seconds.
* Anything happens afterwards is up to the progammer who wrote the main loop script. 

In addition to script file, one can also use XML file as the bootstrapper.
The content of the xml should look like below.
```xml
<?xml version="1.0" ?>
<MainGameLoop>(gl)script/apps/Taurus/main_loop.lua</MainGameLoop>
```

To start it, we can use 
```
npl script/apps/Taurus/bootstrapper.xml
```

### Understanding File Path
When programming with NPL, the code will reference other script or asset file by relative file name. The search order of a file is as follows:
* check if file exists in the current development directory (in release time, this is the same as the working directory)
* check if file exists in the search paths (by default it is the current working directory)
* check if file exists in any loaded archive files (mainXXX.PKG or world/plugins ZIP files) in their loading order.
* if the file is NPL script file, look for precompiled binary file at ./bin/_filename_.o also in above places and order.
* if the file is listed in `asset_manifest.txt`, we will download it or open it from `./temp/cache` if already downloaded.
* check if disk file exists in all search paths of [[npl_packages]].
* if none of above places are found, we will report file not found. 
Please also note, some API allows you to use specify search order, temporarily disable some places, or include certain directory like the current writable directory. Please refer to the usage of each API for details. 

