# Video Tutorial: Create Paracraft Mod
[video](#)

In this video tutorial, you will learn how to create 3D applications with NPL. Make sure you have read our previous video [[TutorialWebSite]]. This is also going to be a very long video, be prepared and patient!

### 3D Graphics Powered By ParaEngine
NPL's graphics API is powered by a built-in 3D computer game engine called ParaEngine, which is written as early as 2005 (NPL was started in 2004). Its development goes side-by-side with NPL in the same repository. If you read the [source code](https://github.com/LiXizhi/NPLRuntime) of NPLRuntime, you will notice two namespaces called `NPL` and `ParaEngine`.

In the past years, I have made several attempts to make the ParaEngine portion more modular, but even so, I did not separate it from the NPLRuntime code tree, because I want the minimum build of NPLRuntime to support all 3D API, just like JAVA, C# and Chrome/NodeJS does. However, if you build NPLRuntime with no `OpenGL/DirectX` graphics, such as on Linux server, these 3D APIs are still available to NPL script but will not draw anythings to your screen.

### Introducing Our 3D World Editor: Paracraft
First of all, creating 3D applications isn't easy in general. Fortunately, NPL offers a free and [open source](https://github.com/NPLPackages/paracraft) NPL package called Paracraft, which is 3d world editor written in NPL. 

Paracraft itself is also a standard alone software product for every one (including kids above 7 years old) to create 3D world, animated characters, movies and codes all in a single application. See [[http://www.paracraft.cn/]] for its official website. Actually there are hundreds of interactive 3D worlds, movies and games created by paracraft users without the need of any third-party software. 

![image](https://cloud.githubusercontent.com/assets/94537/19154692/2fad2be8-8c0e-11e6-938e-fa0d5ea202ef.png)

_Fig. Screen shot of Paracraft_

> Note: Yes, the look of it are inspired by a great sandbox game called `minecraft` (I love it very much), but using blocks is just the methodology we adopt for crafting objects almost exclusively. Blocks in paracraft can be resized and turned into animated characters. The block methodology (rather than using triangles) allows everyone (including kids) to create models, animations, and even rigging characters very easily, and it is lots of fun. Remember you can still import polygon models from `MAYA/3dsmax`, etc, if you are that professional. 

Let us look at some features of Paracraft.
- Everything is a block, entity or item, a block or entity can contain other items in it. 
- Every operation is a command and most of them support undo and redo.  
- There are items like scripts, movie blocks, bones, terrain brush, physics blocks, etc. 
- Use NPL Code Wiki to view and debug.
- Network enabled, you can host your own private world and website in your world directory. 

### Creating Paracraft Plugin(Mod)
If you want to control more of the logic and looks of your 3D world, you need to write NPL code as professionally as we wrote paracraft itself. This is what this tutorial is really about. 

#### Why Using Mod?
Because paracraft is also released as an [open source NPL package](https://github.com/NPLPackages/paracraft), you can just modify the source code of Paracraft and turn it into anything you like or even write everything from scratch. 

However, this is not the recommended way, because paracraft is maintained by us and regularly updated. If you directly modify paracraft without informing us, it may be very difficult to stay up to date with us. Unless you find a bug or wrote a new feature which we can merge into our main branch, you are recommended to use Paracraft Mod to create plugins to extend paracraft or even applications that looks entirely different from it. 

Paracraft Mod or plugin system exposes many extension points to various aspects of the system. One can use it to add new blocks, commands, web page in NPL code wiki, or even replace the entire user interface and IO of paracraft. 

![image](https://cloud.githubusercontent.com/assets/94537/19157826/25c0966e-8c19-11e6-86db-6f6760076f8b.png)

_Fig. Above Image is an [online game](http://ps.61.com/) created via Paracraft Mod by one of our partners. _

You can see how much difference there is from paracraft. 

#### Creating Your First Paracraft Mod

##### Install ParacraftSDK
* Install [Git](https://git-scm.com/) so that you can call `git` command from your cmd line.
* Download [ParacraftSDK](https://github.com/LiXizhi/ParaCraftSDK/archive/master.zip)
  * or you can clone it by running: 
```
git clone https://github.com/LiXizhi/ParaCraftSDK.git
```
   * Make sure you have updated to the latest version, by running `./redist/paracraft.exe` at least once
* Download and install [Visual Studio Community Edition](https://www.visualstudio.com/): the free version of `visual studio`
  * In visual studio, select `menu::Tools::Extensions`, and search online for `npl` or `lua` to install following plugins:
[NPL/Lua language service for visual studio](https://visualstudiogallery.msdn.microsoft.com/7782dc20-924a-4726-8656-d876cdbb3417): this will give you all the syntax highlighting, intellisense and debugging capability for NPL in visual studio. 

see [[Install Guide|InstallGuide]] and [ParaCraftSDK Simple Plugin](https://github.com/LiXizhi/ParaCraftSDK/wiki/TutorialSimplePlugin) for more information.

##### Create `HelloWorld` Mod
- Run `./bin/CreateParacraftMod.bat` and enter the name of your plugin, such as `HelloWorld`. A folder at `./_mod/HelloWorld ` will be generated.
- Run `./_mod/HelloWorld/Run.bat` to test your plugin with your copy of paracraft in the ./redist folder.

The empty plugin does not do anything yet, except printing a log when it is loaded. 

##### Command line and runtime environment
If you open Run.bat with a text editor, you will see following lines.
```
@echo off 
pushd "%~dp0../../redist/" 
call "ParaEngineClient.exe" dev="%~dp0" mod="HelloWorld" isDevEnv="true"  
popd 
```
It specifies following things:
* the application is started via ParaEngineClient.exe in `./redist` folder
* the `dev` parameter specifies the development directory (which is the plugin directory ./_mod/HelloWorld/), basically what this means is that NPLRuntime will always look for files in this folder first and then in working directory (`./redist` folder). 
* the `mod` and `isDevEnv` parameters simply tells paracraft to load the given plugin from the development directory, so that you do not have to load and enable it manually in paracraft GUI.

##### Install Source Code From NPL Package
All APIs of paracraft are available as source code from NPL package. 
Run `./_mod/HelloWorld/InstallPackages.bat` to get them or you can create `./_mod/HelloWorld/npl_packages` and install manually like this.
```
cd npl_packages
git clone https://github.com/NPLPackages/main.git
git clone https://github.com/NPLPackages/paracraft.git
```
You may edit `InstallPackages.bat` to install other packages, and run it regularly to stay up-to-date with git source.  

> it is NOT advised to modify or add files in the ./npl_packages folder, instead create a similar directory structure in your mod directory if you want to add or modify package source code.  Read [[npl_packages]] for how to contribute.

##### Setup Project Files
NPL is a dynamic language, it does not require you to build anything in order to run, so you can create any type of visual studio project you like and begin adding files to it. Open `ModProject.sln` with visual studio in your mod directory, everything should already have been setup for you. You can `Ctrl+F5` to run. 

To manually create project solution file, follow following steps:
- create an empty visual studio project: such as a C# library and remove all cs files from it.
- add `npl_packages/main/*.csproj` and `npl_packages/paracraft/*.csproj` to your solutions, so that you have all the source code of NPL core lib and paracraft, where you can easily search for documentation and implementation. 
- configure the project property to tell visual studio to start NPLRuntime with proper command line parameters when we press Ctrl+F5. The following does exactly the same thing as the `./Run.bat` in mod folder. 
   - For the external program: we can use `ParaEngineClient.exe`, see below
   - external program: `D:\lxzsrc\ParaCraftSDKGit\redist\ParaEngineClient.exe`
   - command line parameters: `mod="HelloWorld" isDevEnv="true"  dev="D:/lxzsrc/ParaCraftSDKGit/redist/_mod/HelloWorld/"`
   - working directory: `D:/lxzsrc/ParaCraftSDKGit/redist/_mod/HelloWorld/`
     - Note: the `dev` param and working directory should be the root of your mod project folder. 

Please see my previous [video tutorial for web server](TutorialWebSite) for more details about installing NPL language service for visual studio.

#### Understanding Paracraft Mod

When paracraft starts, it will load the file at `./Mod/_plugin_name_/main.lua` for each enabled plugin. It is the entry point of any plugin, everything else is up to the developer. 

In our example, open the entry file `./_mod/HelloWorld/Mod/HelloWorld/main.lua` (see below). Because development directory is set to `_mod/HelloWorld` by startup command line, the relative path of above file is `./Mod/HelloWorld/main.lua`. This is why Paracraft can find the entry file. 

```
local HelloWorld = commonlib.inherit(commonlib.gettable("Mod.ModBase"),commonlib.gettable("Mod.HelloWorld"));

function HelloWorld:ctor()
end

-- virtual function get mod name
function HelloWorld:GetName()
	return "HelloWorld"
end

-- virtual function get mod description 
function HelloWorld:GetDesc()
	return "HelloWorld is a plugin in paracraft"
end

function HelloWorld:init()
	LOG.std(nil, "info", "HelloWorld", "plugin initialized");
end

function HelloWorld:OnLogin()
end

-- called when a new world is loaded. 
function HelloWorld:OnWorldLoad()
end

-- called when a world is unloaded. 
function HelloWorld:OnLeaveWorld()
end

function HelloWorld:OnDestroy()
end
```

Most plugin will register a class in the "Mod" table, like above. The class usually inherits from `Mod.ModBase`. The class will be instantiated when paracraft starts, and its virtual functions are called at appropriate time. 

For example, if your plugin contains custom commands or block items, you can register them in the `init()` method.

### How to write plugin
There are three recommended ways to extend paracraft
* create your own command: command is a powerful way to interact with the user. The rule of thumb is that **before you have any GUI dialog, consider writing a command first**. Command is easy to test, integrate, and capable of interacting with other commands. Graphic User Interface(GUI) is good, however, it can only interact with humans in strict ways, commands can interact with both human and other commands or programs. 
* create custom items: custom items include entities, items, or blocks. This ensures that your code is modular and independent. The best way to add new functions to paracraft is by adding new block or item types. Consider the color block and movie block in paracraft, which contains both GUI and complex logics. 
* add filters: filters is a useful way to inject your code at predefined integration points in Paracraft. 

The least recommended way to extend paracraft is to clone the source code files of paracraft to development directory and modify them. Because plugin directory is searched before the main paracraft source code pkg file, it is possible to overwrite source code of paracraft in this way, but you are highly decouraged to do so. If you are requesting a feature that can not be added via the recommended ways, please inform the developers of paracraft to add a new filter for you, instead of modifying our source code directly in your plugin. 

#### Extending Command
see `_mod/Test/Mod/Test/DemoCommand.lua`

#### Extending GUI
see `_mod/Test/Mod/Test/DemoGUI.lua`

#### Extending Item
TODO: 

#### Extending NPL Code Wiki
Users of Paracraft knows about `NPL Code Wiki`, if you want to use latest web technology (HTML5/js) to create user interface that can interact with your 3D application, you can extend the NPL code wiki. 

Now let us create a new web page and add a menu item in NPL Code wiki

First of all, you need to place your server page file in NPL code wiki's web root directory. So we can create a file at 
`./_mod/HelloWorld/script/apps/WebServer/admin/wp-content/pages/HelloWorld.page`

its content may be 
```
<h1>Hello World from plugin!</h1>
```

In the `./_mod/HelloWorld/Mod/HelloWorld/main.lua` file's init function, we add a menu filter to attach a menu item.
```lua
function HelloWorld:init()

	-- add a menu item to NPL code wiki's `Tools:helloworld`
	NPL.load("(gl)script/apps/WebServer/WebServer.lua");
	WebServer:GetFilters():add_filter( 'wp_nav_menu_objects', function(sorted_menu_items)
		sorted_menu_items[sorted_menu_items:size()+1] = {
			url="helloworld",
			menu_item_parent="Tools",
			title="hello world",
			id="helloworld",
		};
		return sorted_menu_items;
	end);
end
```
Now restart paracraft, and press F11 to activate the NPL code wiki, you will notice the new menu item in its tools menu. 

### Documentation
The best documentation is the source code of paracraft which is in the [paracraft package](https://github.com/NPLPackages/paracraft) and several plugin demos in the `./_mod/test` folder. Since code is fully documented, use search by text in source code for documentations, as if you are googling on the web.

### Debugging
For detailed information, please see [NPL Debugging and Logs](https://github.com/LiXizhi/NPLRuntime/wiki/DebugAndLog)

* Use `./redist/log.txt` to print and check log file.
* press `F12` to bring up the buildin debugger.
* use `Ctrl+F3` to use the MCML browser for building GUI pages.
* use GUI debuggers for NPL in visual studio. 

### Plugin Deployment
To deploy and test your plugin in production environment, you need to package all the files in your plugin's development directory (i.e. `_mod/HelloWorld/*.*`) into a zip file called `HelloWorld.zip`, and place the zip file to './redist/Mod/HelloWorld.zip'. It is important that the file path in the zip file resembles the relative path to the development directory (i.e. `Mod/HelloWorld/main.lua` in the `HelloWorld.zip`).

Once you have deployed your plugin, you can start paracraft normally, such as running `./redist/Paracraft.exe`, enable your plugin in Paracraft and test it. 

You can redistribute your plugins in any way you like. We highly recommend that you code and release on github and inform us (via email or pull request) if you have just released some cool plugin. We are very likely to fork your code and review it. 

### Further Readings
* Check out all sample plugins in `./_mod` folder. 
* Check out all the SDK video tutorials by the author of Paracraft on the website.
* Read NPL Runtime's [official wiki](https://github.com/LiXizhi/NPLRuntime/wiki)

### Why are you doing this?
Lots of people and even our partners asked me why are you doing this `NPL/Paracraft/ParaEngine` thing and make all of them open source? To make the long story short, I want to solve two problems:
- Create a self-learning based educational platform so that everyone can learn computer science as my own childhood does.
   - We hope in 7 years, this could be the best present that we can give to our children. Right now, I really cannot find any material even close to it on the Internet. 
- Create a online 3D simulated virtual world in which we can test new artificial intelligence algorithms with lots of humans. Using blocks is obviously the most economical way.

Thanks to our generous investor at `Tatfook`, we are able to sponsor more independent programmers and support more teams and partners using NPL to fulfill the above two goals. Please read [this](https://github.com/LiXizhi/ParaCraft/wiki/NPLUnion) if you are interested in joining us and be sponsored. 