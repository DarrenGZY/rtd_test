# NPL Server Page
NPL server page is a mixed HTML/NPL file, usually with the extension `.page`. 

## How Does It Work?
At runtime time, server page is preprocessed into pure NPL script and then executed. 
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

### Sandbox Environment
When a HTTP request come and redirected to NPL page handler, a special sandbox environment table is created, all page scripts related to that request is executed in this newly created sandbox environment. So you can safely create global variables and expect them to be uninitialized for each page request. 

However, the sandbox environment also have read/write access to the global per-thread NPL runtime environment, where all NPL classes are loaded. 

#### `NPL.load` vs `include`
In a page file, one can call `NPL.load` to load a given NPL class, such as `mysql.lua`; or one can also use the page command `include` to load another page file into sandbox environment. The difference is that classes loaded by `NPL.load` will be loaded only once per thread; where `include` will be loaded for every HTTP request handled by its worker thread. Moreover, NPL web server will monitor file changes for all page files and recompile them when modified by a developer;  for files with `NPL.load`, you need to restart your server, or use special code to reload it. 

### Mixing Async Code with Yield/Resume

The processing of a web page usually consists of two phases. 
* One is fetching data from database engine, which usually takes over 95% of the total time. 
* The other is page rendering, which is CPU-bound and takes only 5% of total request time. 

|    |query database and wait for database result            | MVC Render |
|----|-------------------------------------------------------|------------|
| duration |95%                                              | 5%         |

With NPL's `yield` method, it allows other web requests to be processed concurrently in the 90% interval while waiting database result on the `same` system-level thread. See following code to see how easy to mix async-code with template-based page rendering code. This allows us to serve `5000 requests/sec` in a single NPL thread concurrently, even if each request takes 30ms seconds to fetch from database.

Following is excerpt from our `helloworld.page` example.
```lua
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
err, users = yield(true); -- async wait when job is done
?>

<?npl for i, user in ipairs(users) do ?>
	i = <?=i?>, name=<? echo(user.name) ?> <br/>
<?npl end ?>
```
**Code Explanation**:
When the first `yield()` is called, the execution of page rendering is paused. It will be resumed by the result of a previous async task. In our case, when all five users have been inserted to our database, we will call `resume()`, which will immediately resume page execution from last yield (paused) code position. 

Then we started another async task to fetch all users in the database, and called yield immediately to wait for its results. This time we passed some parameter `resume(err, users)`,  everything passed to resume will be returned from yield() function. So `err, users = yield(true)` will return the `users` table when it returns. 

Please note, we recommend you pass a boolean `err` as first parameter to `resume`, since all of our async API follows the same rule. Also note that we passed true to the second `yield` function, which tells to page renderer to output error and stop execution immediately. If you want to handle the error yourself, please pass nothing to yield like `err, users = yield()`

## Page Commands
The following commands can only be called from inside an NPL page file. They are shortcut to long commands. 

```
Following objects and functions can be used inside page script:
	request:   current request object: headers and cookies
	response:   current response object: send headers or set cookies, etc.
	echo(text):   output html
	__FILE__: current filename
	page: the current page (parser) object
	_GLOBAL: the _G itself

following are exposed via meta class:
	include(filename, bReload):  inplace include another script
	include_once(filename):  include only once, mostly for defining functions
	print(...):  output html with formated string.   
	nplinfo():   output npl information.
	exit(text), die():   end the request
	dirname(__FILE__):   get directory name
	site_config(): get the web site configuration table
	site_url(path, scheme): 
	addheader(name, value):
	file_exists(filename):
	log(obj)
	sanitize(text)  escape xml '<' '>' 
	json_encode(value, bUseEmptyArray)   to json string
	json_decode(str)  decode from json string
	xml_encode(value)    to xml string
	include_pagecode(code, filename):  inplace include page code. 
	get_file_text(filename) 
	util.GetUrl(url, function(err, msg, data) end): 
	util.parse_str(query_string): 
	err, msg = yield(bExitOnError)  pause execution until resume() is called.
	resume(err, msg)  in async callback, call this function to resume execution from last yield() position.
```

see [script/apps/WebServer/npl_page_env.lua](https://github.com/NPLPackages/main/tree/master/script/apps/WebServer/npl_page_env.lua) for detailed documentation.

## Request/Response Object
- [request object](https://github.com/NPLPackages/main/tree/master/script/apps/WebServer/npl_request.lua): current request object: headers and cookies 
- [response object](https://github.com/NPLPackages/main/tree/master/script/apps/WebServer/npl_response.lua): current response object: send headers or set cookies, etc.

## Memory Cache and Global Objects
NPL web server has a built-in simple local memory cache utility. One can use it to store objects that is shared by all requests on the main thread. In future it may be configured to run on a separate server like `memcached`.

See below:
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

Alternatively, one can also create global objects that is shared by all requests in the NPL thread, by using `commonlib.gettable()` method. Table objects created with `commonlib.gettable()` is different from `gettable()` in page file. The latter will create table on the local page scope which lasts only during the lifetime of a given http request.

## File Uploader
NPL web server supports `multipart/form-data` by which one can upload binary file to the server. It is recommended to use a separate server for file upload, because it is IO bound and consumes bandwidth when file is very large. 

Click [here](https://github.com/NPLPackages/main/blob/master/script/apps/WebServer/admin/wp-content/pages/fileuploader.page) for a complete example of file uploader. 

Here is the client side code to upload file as `enctype="multipart/form-data"`
```html
<form name="uploader" enctype="multipart/form-data" class="form-horizontal" method="post" action="/ajax/fileuploader?action=upload">
  <input name="fileToUpload" id="fileToUpload" type="file" class="form-control"/>
  <input name="submit" type="submit" class="btn btn-primary" value="Upload"/>
</form>
```

Here is an example NPL server page code, the binary contents of the file is in `request:getparams()["fileToUpload"].contents`. The maximum file upload size allowed can be configured by NPL runtime attribute. 
The default value is 100MB. 

```lua
local fileToUpload = request:getparams()["fileToUpload"]
if(fileToUpload and request:getparams()["submit"] and fileToUpload.name and fileToUpload.contents) then
	local target_dir = "temp/uploads/" .. ParaGlobal.GetDateFormat("yyyyMMdd") .. "/";
	local target_file = target_dir .. fileToUpload.name;
	local fileType = fileToUpload.name:match("%.(%w+)$"); -- file extension
	
	-- check if file already exists
	if(file_exists(target_file)) then
		response:send({err = "Sorry, file already exists."});
		return
	end
	-- check file size
	if (fileToUpload.size and fileToUpload.size> 5000000) then
		response:send({err = "Sorry, your file is too large."});
		return
	end

	-- Allow certain file formats
	if(false and fileType ~= "jpg" and fileType ~= "png" and fileType~="txt" ) then
		response:send({err = "Sorry, only JPG, PNG & TXT files are allowed."});
		return
	end

	-- if everything is ok, try to save file to target directory
	ParaIO.CreateDirectory(target_dir);
	local file = ParaIO.open(commonlib.Encoding.Utf8ToDefault(target_file), "w");
	if(file:IsValid()) then
		file:write(fileToUpload.contents, #fileToUpload.contents);
		file:close();
		response:send({contents = target_file, name = fileToUpload.name, size = fileToUpload.size, ["content-type"] = fileToUpload["content-type"], });
		return;
	else
		response:send({err = "can not create file on disk. file name invalid or disk is full."});
	end
end
response:send({err = "unknown err"});
```

### Server Side Redirection
See also [RFC standard](https://en.wikipedia.org/wiki/Server-side_redirect)
```
<?npl
response:set_header("Location", "http://www.paracraft.cn/")
response:status(301):send_headers();
```