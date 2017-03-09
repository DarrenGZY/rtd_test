# NPL Common Libraries
NPL common Libraries contains a rich and open source collection of libraries covering: io, networking, 2d/3d graphics, web server, data structure, and many other software frameworks. 

## Library Installation
NPL common libraries can be installed in `pkg` or `zip` or in plain source code. When you deploy your application, you can choose to use our pre-made zip file or deploy only used files in a zip package by yourself.

If you installed NPL runtime via ParaCraftSDK on windows platform, you will already have NPL common library installed in `ParaCraftSDKGit/NPLRuntime/win/packages/main.pkg`.

If you installed NPL runtime from source code on linux. It does not have NPL common libraries preinstalled. Instead you need to copy or link the `script` folder to your project's working directory. 
The folder to link to can be found [here](https://github.com/LiXizhi/ParaCraftSDK/tree/master/src/script), which contains thousands of open source and documented files all written in NPL.

## Major Libraries

For minimum server-side application development, include this:
```
NPL.load("(gl)script/ide/commonlib.lua");
```
or 
```
NPL.load("(gl)script/ide/System/System.lua");
```

For full-fledged heavy 2d/3d application development, include this:
```
NPL.load("(gl)script/ide/IDE.lua");
```
or 
```
NPL.load("(gl)script/kids/ParaWorldCore.lua");
```

## Documentation
NPL libraries are themselves written in pure NPL scripts. They usually have no external dependencies, but on the low level NPL API provided by NPL runtime. These API is in turn implemented in C/C++ in cross-platform ways. 

> The documentation for low level API is here [[https://codedocs.xyz/LiXizhi/NPLRuntime/modules.html]]

However, it is not particularly useful to read low level API. All you need to do it is to glance over it, and then dive into the source code of NPL libraries [here](https://github.com/LiXizhi/ParaCraftSDK/tree/master/src/script). 



