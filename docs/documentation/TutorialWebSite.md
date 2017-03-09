## Video Tutorial: Create A Web Site

[video](http://pan.baidu.com/s/1c20GEi)

In this video tutorial, you will learn how to create a web site like [this](https://github.com/tatfook/wikicraft) with NPL. The following topics will be covered:
- setup NPL web development environment
- NPL server page programming model
- Debugging and logs
- Use table database
- MVC frameworks for serious web development

> For complete explanation behind the scene, please read [[WebServer]]

### Setup NPL web development environment
For more information, see [[Install Guide|InstallGuide]]. The following is only about setting up the recommended IDE under win32. 

* Install [Git](https://git-scm.com/) so that you can call `git` command from your cmd line.
* Download [ParacraftSDK](https://github.com/LiXizhi/ParaCraftSDK/archive/master.zip)
  * or you can clone it by running: 
```
git clone https://github.com/LiXizhi/ParaCraftSDK.git
```
   * Make sure you have updated to the latest version, by running `./redist/paracraft.exe` at least once
   * Run `./NPLRuntime/install.bat`
      * The NPL Runtime will be automatically copied from `./redist` folder to `./NPLRuntime/win/bin`
      * The above path will be added to your `%path%` variable, so that you can run npl script from any folder.
* Download and install [Visual Studio Community Edition](https://www.visualstudio.com/): the free version of `visual studio`
  * In visual studio, select `menu::Tools::Extensions`, and search online for `npl` or `lua` to install following plugins:
[NPL/Lua language service for visual studio](https://visualstudiogallery.msdn.microsoft.com/7782dc20-924a-4726-8656-d876cdbb3417): this will give you all the syntax highlighting, intellisense and debugging capability for NPL in visual studio. 

#### Create a New Visual Studio Project 
NPL is a dynamic language, it does not require you to build anything in order to run, so you can create any type of visual studio project you like and begin adding files to it. 

There are two ways to do it. Please use `the lazy way` but read `the manual one`. 
##### The lazy way
- run `git clone https://github.com/tatfook/wikicraft.git`
- cd into the source directory and run `./update_packages.bat`
- open the `sln` file with visual studio
- run `start.bat`, you will see the npl server started. 
   - visit [[http://localhost:8099]] for the main site.
   - visit [[http://127.0.0.1:8099]] for the NPL code wiki web site. 
- rename the project to your preferred name and begin coding in the `./www` folder. (see next chapter) 
- finally, see the `configure the project property` step in `the manual one` section.

##### The manual one
- create an empty visual studio project: such as a C# library or VC++ empty project.  
- install the main [npl package](npl_packages) into your project root directory. 
```
mkdir npl_packages
cd npl_packages
git clone https://github.com/NPLPackages/main.git
```
- configure the project property to tell `visual studio` to start NPLRuntime with proper command line parameters when we press `Ctrl+F5`. 
   - For the external program: we can use either `npl.bat` or `ParaEngineClient.exe`, see below
   - external program: `D:\lxzsrc\ParaCraftSDKGit\NPLRuntime\win\bin\ParaEngineClient.exe`
   - command line parameters: `bootstrapper="script/apps/WebServer/WebServer.lua" port="8099" root="www/" servermode="true" dev="D:/lxzsrc/wikicraft/"`
   - working directory: `D:/lxzsrc/wikicraft/`
![image](https://cloud.githubusercontent.com/assets/94537/18578343/05c96428-7c22-11e6-8f98-fffcd3d22c78.png)
     - Note: the `dev` param and working directory should be the root of your web project directory. 
- add main project at `npl_packages/main/*.csproj` to your newly created solution file, so that you have all the source code of NPL and NPL web server for debugging and documentation. **Reopen your visual studio solution file for NPL documentation intellisense to take effect.** 
- create `www/` directory and add `index.page` file to it. 
- Press `Ctrl+F5` in visual studio, and you will launch the web server. 
   - visit [[http://localhost:8099/index.page]] in your browser.

### NPL server page programming model
NPL web server itself is an open source application written completely in NPL. For its complete source code, please see `script/apps/WebServer/WebServer.lua` in the [main](https://github.com/NPLPackages/main/) npl_package.

So to start a website in a given folder such as `./www/`, we need to call 
```
npl bootstrapper="script/apps/WebServer/WebServer.lua" port="8099" root="www/"
```
Normally, we also need to place a file called `webserver.config.xml` in the web root folder.
If no config file is found, [`default.webserver.config.xml`](https://github.com/NPLPackages/main/blob/master/script/apps/WebServer/default.webserver.config.xml) is used, which will redirect all request without extension to `index.page` and serve `*.npl`, `*.page`, and all static files in the root directory and it will also host NPL code wiki at [[http://127.0.0.1:8099]]. For more information, please see [[WebServer]]

#### Play with Mixed Code Web Programming
NPL server page is a mixed HTML/NPL file, usually with the extension `.page`. You can now play with it by creating new `*.page` file in `./www/` directory under visual studio. For more information, see [[NPLServerPage]].

**Intellisense in visual studio** 

If you have installed NPL language service plugin and added main package csproj to your solution, you should already noticed intellisense in your visual studio IDE for all `*.lua, *.npl, *.page` files. 
Everything under `./Documentation` folder is used for auto-code completion.  Please see `npl_packages/main/Documentation/`, you can also add your own intellisense by adding xml files to your own project's './Documentation' folder. For more information, please see [[NplVisualStudioIDE]].

![NPL language service](https://visualstudiogallery.msdn.microsoft.com/site/view/file/205772/1/NPLLanguageService2.png)

#### Toggle Page Editing Mode 
- One can toggle between two editing mode when writing `*.page` file in visual studio
   - right click the page file in solution manager, and select `open with...`
   - the default text editor will highlight NPL code in the page file, while all HTML text are greyed out
   - the HTML editor will highlight all html tags and all npl code are greyed out. (see below)

![image](https://cloud.githubusercontent.com/assets/94537/18735242/f9c59ce6-80ae-11e6-919e-f6d6c5bac157.png)

You can toggle between these mode depending on whether you are working on HTML or code. 

#### How Does It Work?
At runtime, server page is preprocessed into pure NPL script and then executed. 

For example
```php
<html><body>
<?npl  for i=1,5 do ?>
   <p>hello</p>
<?npl  end ?>
</body></html>
```
Above server page will be pre-processed into following NPL page script, and cached for subsequent requests.
```lua
echo ("<html><body>");
for i=1,5 do
 echo("<p>hello</p>")
end
echo ("</body></html>");
```
When running above page script, `echo` command will generate the final HTML response text to be sent back to client. 

#### Internals and Performance
Normally, a single NPL server can serve over 1000 dynamic page requests per seconds and with millions of concurrent connections (limited by your OS). It can serve over 40000 req/sec if it is very simple dynamic page without database query. See [[NPLPerformance]] and [[WebServer]] for details. 

Unlike other preprocessed page language like `PHP`, database queries inside npl page file are actually invoked asynchronously, but written in the more human-friendly synchronous fashion. Please see `Use table database` chapter for details.

### Debugging and logs
Most page errors are rendered to the HTTP stream, which is usually directly visible in your browser. For complete or more hidden errors, please keep `./log.txt` in your working directory open, so that you can read most recent error logs in it. Your error is usually either a `syntax` error or a `runtime` error.  

**To debug with NPL Code wiki**

- Launch your website 
- Right click on a line in visual studio and select `NPL set breakpoint here` in the context menu. 
- The `http://127.0.0.1:8099/debugger` page will be opened.
   - Please note, your actual web site is at `http://localhost:8099`. They are two different websites hosted on the same port by the same NPL process, you can use the first `127.0.0.1` website to debug the second `localhost` website, because both web sites share the same NPL runtime stack/memory/code. 

![setbreakpoint](https://cloud.githubusercontent.com/assets/94537/16826877/ccc6e736-49b3-11e6-9937-a72703a84ef2.png)

Please see [[DebugAndLog]] for general purpose NPL debugging. 

### Use table database
Table database is written in NPL and is the default and recommended database engine for your website. It can be used without any external dependency or configuration. 

```lua
<html>
<body>
<?
-- connect to TableDatabase (a NoSQL db engine written in NPL)
db = TableDatabase:new():connect("database/npl/", function() end);

db.TestUser:findOne({name="user1"}, resume);
local err, user = yield(); -- async wait when job is done

if(not user) then
	-- insert 5 records to database asynchronously.
	local finishedCount = 0;
	for i=1, 5 do
		db.TestUser:insertOne({name=("user"..i)}, {name=("user"..i), password="1"}, function(err, data)
			finishedCount = finishedCount + 1;
			if(finishedCount == 5) then
				resume();
			end
		end);
	end
	yield(); -- async wait when job is done

	echo("<p>we just created "..finishedCount.." users in `./database/npl/TestUser.db`</p>")
else
	echo("<p>users already exist in database, print them</p>")
end

-- fetch all users from database asynchronously.
db.TestUser:find({}, function(err, users)  resume(err, users); end);
err, users = yield(); -- async wait when job is done
?>

<?npl for i, user in ipairs(users) do ?>
	i = <?=i?>, name=<? echo(user.name) ?> <br/>
<?npl end ?>

</body>
</html>
```
Try above code. See [[UsingTableDatabase]] for details.

### MVC frameworks for serious web development
There are many ways to write MVC framework with NPL. For robust web development, we will introduce the MVC coding style used in [this](https://github.com/tatfook/wikicraft) sample web site. 

The pros of using MVC framework is that:
- Making your website robust and secure
- Reuse the same code for both client/server side API calls. 
- Writing less code for robust http `rest API`

**It begins with a router page**

All dynamic page requests are handled by a single file [`index.page`](https://github.com/tatfook/wikicraft/blob/master/www/index.page), which calls [`routes.page`](https://github.com/tatfook/wikicraft/blob/master/www/wiki/routes.page). 

The router page will check whether the request is a ajax rest API request or a normal page request according to its url. 
For example, every request url that begins with `/api/[name]/...` is regarded as an API request, and everything else is a normal page request. 

If it is a ajax API request, we will simply route it by loading the corresponding page file in `models/[name].page`. 
In other words, if you placed a `user.page` file in `models` directory, you have already created a set of ajax HTTP rest API on the url `/api/user/...`. 

Normally, all of your model file should inherit from a carefully-written [`base`](https://github.com/tatfook/wikicraft/blob/master/www/wiki/models/abstract/base.page) model class. So that you can define a robust data-binding interface to your low-level database system or anything else.

For example, the [models/project.page](https://github.com/tatfook/wikicraft/blob/master/www/wiki/models/project.page), defines basic CRUD interface to `create/get/update/delete` records in the `project.db` database table. It also defines a custom ajax api at `project:api_getminiproject()` which is automatically mapped to the url `api/project/getminiproject/` in the router page (i.e. every method that begins with `api_` in the model class is a public rest API).

How you write your model class and define the mapping rules are completely up to you. The example website just gives you an example of writing model class.

As a website developer, you can choose to query model data on the client side with HTTP calls, or on the server side by directly calling model class method. Generally speaking, client side frameworks like `angular`, etc prefer everything to be called on the client side. We also favors client side query in most cases unless it is absolutely necessary to do it on the server side, such as content needs to be rendered on the server so that it is search-engine friendly. 

### Final Word
NPL used to be targeted at `3D/TCP server` development since 2005. NPL for Web development was introduced in early 2016, which is relatively new. So if there are errors or questions, please report on our [issues](https://github.com/LiXizhi/NPLRuntime/issues). 

**Hope you enjoy your NPL programming~**

