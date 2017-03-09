# NPL Source Code Overview

This is a brief overview of NPL source code. The best way to learn NPL is to read its source code. 

All NPL related source code is distributed as [packages](npl_packages). 

Two important packages for NPL and paracraft developement is at:
- [main package](https://github.com/NPLPackages/main): NPL standard library source code
- [paracraft package](https://github.com/NPLPackages/paracraft): paracraft source code

```
cd npl_packages
git clone https://github.com/NPLPackages/main.git
git clone https://github.com/NPLPackages/paracraft.git
```

When you program, you can search by text in all NPL source code.

## Source code overview

### Core Library Files
 * [script/ide/](https://github.com/NPLPackages/main/tree/master/script/ide)
 * [script/ide/commonlib.lua](https://github.com/NPLPackages/main/tree/master/script/ide/commonlib.lua): common include file for frequently used libraries without GUI.
 * [script/ide/System](https://github.com/NPLPackages/main/tree/master/script/ide/System): Core library
 * [script/ide/STL](https://github.com/NPLPackages/main/tree/master/script/ide/STL): Like C++ stl, simple dData structures
 * [script/ide/timer.lua](https://github.com/NPLPackages/main/tree/master/script/ide/timer.lua): time class
 * [script/ide/math](https://github.com/NPLPackages/main/tree/master/script/ide/math): vector, matrix, etc.

### Object Oriented
* [script/ide/oo.lua](https://github.com/NPLPackages/main/tree/master/script/ide/oo.lua): class inheritance implementation
* [script/ide/System/Core/ToolBase.lua](https://github.com/NPLPackages/main/tree/master/script/ide/System/Core/ToolBase.lua): a powerful base class with signal/property support.

### Web Server Source Code
 * [script/apps/WebServer](https://github.com/NPLPackages/main/tree/master/script/apps/WebServer): implementation of a web server (like Apache) in NPL. 
 * [script/apps/WebServer/WebServer.lua](https://github.com/NPLPackages/main/tree/master/script/apps/WebServer/WebServer.lua): entry file
 * [script/apps/WebServer/admin](https://github.com/NPLPackages/main/tree/master/script/apps/WebServer/admin): A php-like web site framework in NPL

### Paracraft Source Code
Lots of code here. If you are developing Paracraft plugins or 3d client applications in NPL. It is important to read source here. 
* [script/apps/Aries/Creator/Game](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries/Creator/Game): main source code of Paracraft
* [script/apps/Aries/Creator/Game/Areas](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries
/Creator/Game/Areas): desktop GUI
* [script/apps/Aries/Creator/Game/Entity](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries/Creator/Game/Entity): All movable entity types
* [script/apps/Aries/Creator/Game/blocks](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries/Creator/Game/blocks): All block types
* [script/apps/Aries/Creator/Game/Commands](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries
/Creator/Game/Commands): In-Game Commands
* [script/apps/Aries/Creator/Game/Tasks](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries
/Creator/Game/Tasks): Commands with GUI
* [script/apps/Aries/Creator/Game/GUI](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries
/Creator/Game/GUI): editor GUIs
* [script/apps/Aries/Creator/Game/Network](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries
/Creator/Game/Network): Paracraft server implementation
* [script/apps/Aries/Creator/Game/Shaders](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries
/Creator/Game/Shaders): GPU shaders
* [script/apps/Aries/Creator/Game/Mod](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries
/Creator/Game/Mod): Plugin interface
* [script/apps/Aries/Creator/Game/SceneContext](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries/Creator/Game/SceneContext): User input like keyboard and mouse