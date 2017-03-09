NPL Web Server 
--------------------
> Click [here](TutorialWebSite) for step-by-step tutorial to build a NPL web site.

NPL Runtime provides a build-in self-contained framework for writing web server applications in a way similar to [PHP](http://php.net/), yet with asynchronous API like [NodeJs](https://nodejs.org/). 

## Why Write Web Servers in NPL?

### Turning Async Code Into Synchronous Ones
Many tasks on the server side has asynchronous API like database operations, remote procedure call, background tasks, timers, etc. Asynchronous API frees the processing thread from io-bound or long running task. Using asynchronous API (like those in `NodeJs`) makes web server fast, but difficult to write, as there are too many function callbacks that breaks the otherwise sequential and flat programming code. 

In NPL page file, any asynchronous API can be used as synchronous ones via `resume/yield`, so that constructing a web page is like writing a document sequentially like in `PHP`. Under the hood, it is still asynchronous API and the same worker thread can process thousands of requests per second even some of them takes a long time to finish. Following is a quick example show the idea.

```lua
<?
local value = "Not Set"
-- any async function with callback
local mytimer = commonlib.Timer:new({callbackFunc = function(timer)
   value = "Set";
   resume();
end})
-- start the timer after 1000 milliseconds
mytimer:Change(1000, nil)

-- async wait when job is done
yield();
assert(value == "Set");
```

Please note, `resume/yield` is NOT the ones as in multithreaded programming where locks and signals are involved. It is a special `coroutine` internally; the code is completely lock-free and the thread is `NOT` waiting but processing other URL requests. Please see [[NPLServerPage]] for details. 

### Self-Contained Client/Server Solution

NPL Web Server is fast and very easy to deploy on all platforms (no external dependencies even for databases). You may have a very sophisticated 3d client application, which may also be servers and/or web servers, such as [Paracraft](http://www.paracraft.cn). 

We believe, every software either running on central server farm or endpoints like home or mobile devices should be able to provide services via TCP connection and/or HTTP (web server). In many of our applications, a single software acts both as client and server. Very few existing programming language provides an easy way or architecture for sharing client and server side code or variables in a lock-free fashion due to multi-threaded (multi-process) nature of those web frameworks. Moreover, a server application is usually hard to deploy and config, making it difficult to install on home computers or mobile devices. 

NPL is a language to solve these problems and provides rich client side API as well as a web server framework that can service as many requests as professional web servers while sharing the entire runtime environment for your client and server code. See below.

## Programming Model of NPL web server
NPL Runtime provides a build-in framework for writing web server applications in a way similar to [PHP](http://php.net/). 

> NPL Web Server recommends the async programming model like [Nodejs](https://nodejs.org), but without losing the simplicity of synchronous mixed code programming like in PHP. 

Following is an example `helloworld.page` server page. 

|    |query database and wait for database result            | MVC Render |
|----|-------------------------------------------------------|------------|
| duration |95%                                              | 5%         |

The processing of a web page usually consists of two phases. 
* One is fetching data from database engine, which usually takes over 95% of the total time. 
* The other is page rendering, which is CPU-bound and takes only 5% of total request time. 

With NPL's `yield` method, it allows other web requests to be processed concurrently in the 90% interval while waiting database result on the `same` system-level thread. See following code to see how easy to mix async-code with template-based page rendering code. This allows us to serve `5000 requests/sec` in a single NPL thread concurrently, even if each request takes 30ms seconds to fetch from database.

```lua
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html>
<head>
  <title>NPL server page test. </title>
</head>
<body>
<p>
	<?npl
		echo "Hi, I'm a NPL script!";
	?>
</p>

<?
-- connect to TableDatabase (a NoSQL db engine written in NPL)
db = TableDatabase:new():connect("database/npl/", function() end);

-- insert 5 records to database asynchronously.
local finishedCount = 0;
for i=1, 5 do
	db.TestUser:insertOne({name=("user"..i), password="1"}, function(err, data)
		finishedCount = finishedCount + 1;
		if(finishedCount == 5) then
			resume();
		end
	end);
end
yield(); -- async wait when job is done

-- fetch all users from database asynchronously.
db.TestUser:find({}, function(err, users)  resume(err, users); end);
err, users = yield(); -- async wait when job is done
?>

<?npl for i, user in ipairs(users) do ?>
	i = <?=i?>, name=<? echo(user.name) ?> <br/>
<?npl end ?>

<p>
	1.  <?npl echo 'if you want to serve NPL code in XHTML or XML documents, use these tags'; ?>
</p>
<p>
	2.  <? echo 'this code is within short tags'; ?>
    Code within these tags <?= 'some text' ?> is a shortcut for this code <? echo 'some text' ?>
</p>
<p>
	3.  <% echo 'You may optionally use ASP-style tags'; %>
</p>

<% nplinfo(); %>

<% print("<p>filename: %s, dirname: %s</p>", __FILE__, dirname(__FILE__)); %>

<%
some_global_var = "this is global variable";

-- include file in same directory
include("test_include.page");
-- include file relative to web root directory. however this will not print, because we use include_once, and the file is already included before.
include_once("/test_include.page");
-- include file using disk file path
local result = include(dirname(__FILE__).."test_include.page");

-- we can also get the result from included file
echo(result);
%>

</body>
</html>

<% 
--assert(false, "uncomment to test runtime error");

function test_exit()
	exit(); 
end
test_exit();
assert(false, "can not reach here");
%>
```

Content of `test_include.page` referenced in above server page is:
```php
<?npl

echo("<p>this is from included file: "..(some_global_var or "").."</p>");

-- can also has return value. 
return "<p>from test_include.page</p>";
```

***

The output of above server page, when viewed in a web browser, such as `http://localhost:8099/helloworld` is 
```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html>
<head>
  <title>NPL server page test. </title>
</head>
<body>
<p>
	Hi, I'm a NPL script!</p>

	i = 1, name=user1 <br/>
	i = 2, name=user2 <br/>
	i = 3, name=user3 <br/>
	i = 4, name=user4 <br/>
	i = 5, name=user5 <br/>
<p>
	1.  if you want to serve NPL code in XHTML or XML documents, use these tags</p>
<p>
	2.  this code is within short tags    Code within these tags some text is a shortcut for this code some text</p>
<p>
	3.  You may optionally use ASP-style tags</p>

<p>NPL web server v1.0</p><p>site url: http://localhost:8099/</p><p>your ip: 127.0.0.1</p><p>{
<br/>["Host"]="localhost:8099",
<br/>["rcode"]=0,
<br/>["Connection"]="keep-alive",
<br/>["Accept"]="text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
<br/>["Accept-Encoding"]="gzip, deflate, sdch",
<br/>["method"]="GET",
<br/>["body"]="",
<br/>["tid"]="~1",
<br/>["url"]="/helloworld.page",
<br/>["User-Agent"]="Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/48.0.2564.116 Safari/537.36",
<br/>["Upgrade-Insecure-Requests"]="1",
<br/>["Accept-Language"]="en-US,en;q=0.8",
<br/>}
<br/></p><p>filename: script/apps/WebServer/test/helloworld.page, dirname: script/apps/WebServer/test/</p><p>this is from included file: this is global variable</p><p>this is from included file: this is global variable</p><p>from test_include.page</p></body>
</html>
```

### Web Server Source Code
 * [script/apps/WebServer](https://github.com/LiXizhi/ParaCraftSDK/tree/master/src/script/apps/WebServer): implementation of a web server (like Apache) in NPL. 
 * [script/apps/WebServer/WebServer.lua](https://github.com/LiXizhi/ParaCraftSDK/tree/master/src/script/apps/WebServer/WebServer.lua): entry file
 * [script/apps/WebServer/admin](https://github.com/LiXizhi/ParaCraftSDK/tree/master/src/script/apps/WebServer/admin): A php-like web site framework in NPL. See [[AdminSiteFramework]] for details


## Introduction
The NPL web server implementation uses NPL's own messaging system, and is written in pure NPL scripts without any external dependencies. It allows you to create web site built with NPL Runtime on the back-end and `JavaScript/HTML5` on the client.

`NPL language service` plugins for visual studio (community edition is free) provides a good coding environment. 
See below:
![npllanguageservice_page](https://cloud.githubusercontent.com/assets/94537/13956032/eaf41412-f081-11e5-92dd-0ccd2e091bd1.png)

## Website Examples
   - [WebServerExample](https://github.com/NPLPackages/WebServerExample): a sample web site with NPL code wiki
   - [Wikicraft](https://github.com/tatfook/wikicraft): full website example
   - `script/apps/WebServer/admin` is a NPL based web site framework similar to the famous `WordPress.org`. It is recommended that you xcopy all files in it to your own web root directory and work from there. See [[AdminSiteFramework]] for details.
   - `script/apps/WebServer/test` is a test site, where you can see the test code. 

## starting a web server programmatically
Starting server from a web root directory:
```lua
	NPL.load("(gl)script/apps/WebServer/WebServer.lua");
	WebServer:Start("script/apps/WebServer/test", "0.0.0.0", 8099);
```
Open `http://localhost:8099/helloworld.lua` in your web browser to test. See `log.txt` for details.

There can be a `webserver.config.xml` file in the web root directory. see below.
if no config file is found, [`default.webserver.config.xml`](https://github.com/NPLPackages/main/blob/master/script/apps/WebServer/default.webserver.config.xml) is used, which will redirect all request without extension to `index.page` and serve *.lua, *.page, and all static files in the root directory and it will also host NPL code wiki at [[http://127.0.0.1:8099]]. 

#### starting a web server from command line
You can bootstrap a webserver using the buildin file, run following commmand:
```
npl bootstrapper="script/apps/WebServer/WebServer.lua"  port="8099"
```
  * config: can be omitted which defaults to `config/WebServer.config.xml` in current directory

### Web Server configuration file
More info, see `script/apps/WebServer/test/webserver.config.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- web server configuration file: this node can be child node, thus embedded in shared xml -->
<WebServer>
  <!--which HTTP ip and port this server listens to. -->
  <servers>
    <!-- @param host, port: which ip port to listen to. if * it means all. -->
    <server host="*" port="8099" host_state_name="">
      <defaultHost rules_id="simple_rule"></defaultHost>
      <virtualhosts>
        <!-- force "http://127.0.0.1/" to match to iternal npl_code_wiki site for debugging  -->
        <host name="127.0.0.1:8099" rules_id="npl_code_wiki" allow='{"127.0.0.1"}'></host>
      </virtualhosts>
    </server>
  </servers>
  <!--rules used when starting a web server. Multiple rules with different id can be defined. --> 
  <rules id="simple_rule">
    <!--URI remapping example-->
    <rule match='{"^[^%.]+$", "robots.txt"}' with="WebServer.redirecthandler" params='{"/index.page"}'></rule>
    <!--npl script example-->
    <!--<rule match="%.lua$" with="WebServer.makeGenericHandler" params='{docroot="script/apps/WebServer/test", params={}, extra_vars=nil}'></rule>-->
    <rule match='{"%.lua$", "%.npl$"}' with="WebServer.npl_script_handler" params='%CD%'></rule>
    <!--npl server page example-->
    <rule match="%.page$" with="WebServer.npl_page_handler" params='%CD%'></rule>
    <!--filehandler example, base dir is where the root file directory is. %CD% means current file's directory-->
    <rule match="." with="WebServer.filehandler" params='{baseDir = "%CD%"}'></rule>
  </rules>
</WebServer>
```

Each server can have a default host and multiple virtual hosts.  Each host must be associated with a rule_id.
The rule_id is looked up in the rules node, which contains multiple rules specifying how request url is mapped to their handlers. `npl_code_wiki` is an internal rule_id which is usually used for host `http://127.0.0.1:8099/` for debugging purposes only, see above example code. If you changed your port number, you need to update the config file accordingly, otherwise `npl_code_wiki` will not be available. 

Each rule contains three attributes: match, with and params. 
   * "match" is a regular expresion string or a array table of reg strings like shown in above sample code. 
   * "with" is a handler function which will be called when url matches "match". 
More precisely, it is handler function maker, that returns the real handler function(request, response) end.
When rules are compiled at startup time, these maker functions are called just once to generate the actual handler function. 
   * "params" is an optional string or table that will be passed to the handler maker function to generate the real handler function.

The rules are applied in sequential order as they appeared in the config file. 
So they are usually defined in the order of redirectors, dynamic scripts, file handlers. 

## Handler Makers
Some common handlers are implemented in `common_handlers.lua`. Some of the most important ones are listed below

## WebServer.redirecthandler
It is the url redirect handler. it generate a handler that replaces "match" with "params"
```xml
	<rule match="^[^%./]*/$" with="WebServer.redirecthandler" params='{"readme.txt"}'></rule>
```
The above will redirect `http://localhost:8080/` to `http://localhost:8080/readme.txt`.

## WebServer.filehandler  
It generate a handler that serves files in the directory specified in "params" or "params.baseDir".
```xml
	<rule match="." with="WebServer.filehandler" params='{baseDir = "script/apps/WebServer"}'></rule>
	alternatively:
	<rule match="." with="WebServer.filehandler" params='script/apps/WebServer'></rule>
```
The above will map `http://localhost:8080/readme.txt` to the disk file `script/apps/WebServer/readme.txt`

## WebServer.npl_script_handler 
Currently it is the same as WebServer.makeGenericHandler. 
In future we will support remote handlers that runs in another thread asynchrounously. 
So it is recommended for user to use this function instead of WebServer.makeGenericHandler
serving request using npl files to generate dynamic response. This is very useful for REST-like web service in XML or json format. 
```xml
	<rule match="%.lua$" with="WebServer.npl_script_handler" params='script/apps/WebServer/test'></rule>
	alternatively:
	<rule match="%.lua$" with="WebServer.npl_script_handler" params='{docroot="script/apps/WebServer/test"}'></rule>
```
The above will map `http://localhost:8080/helloworld.lua` to the script file `script/apps/WebServer/test/helloworld.lua`

## Caching on NPL Web Server

1. We use `File Monitor API` to monitor all file changes in web root directory.
2. For dynamic page files: recompile dynamic page if changed. All compiled *.page code is always in cache. So multiple requests calling the same page file only compile once. 
3. For static files: everything is cached in either original or gzip-format ready for direct HTTP response. All static files are served using a separate NPL thread to prevent IO affecting the main thread. 

> for more advanced file caching, one may consider using `nginx` as the gateway load balancer. 

### Caching in Application Code
For caching in local thread, there is a helper class called [mem_cache](https://github.com/LiXizhi/ParaCraftSDK/tree/master/src/script/apps/WebServer/mem_cache.lua). 
```lua
NPL.load("(gl)script/apps/WebServer/mem_cache.lua");
local mem_cache = commonlib.gettable("WebServer.mem_cache");
local obj_cache = mem_cache:GetInstance();
obj_cache:add("name", "value")
obj_cache:replace("name", "value1")
assert(obj_cache:get("name") == "value1");
assert(obj_cache:get("name", "group1") == nil);
obj_cache:add("name", "value", "group1")
assert(obj_cache:get("name", "group1") == "value");
```
#### Caching in different computer
TODO: `mem_cache` can be configured to read/write data from a remote computer.  