## HTTP request

http url request: [script/ide/System/os/GetUrl.lua](https://github.com/NPLPackages/main/tree/master/script/ide/System/os/GetUrl.lua)

```lua
-- return the content of a given url. 
-- e.g.  echo(NPL.GetURL("www.paraengine.com"))
-- @param url: url string or a options table of {url=string, postfields=string, form={key=value}, headers={key=value, "line strings"}, json=bool, qs={}}
-- if .json is true, code will be decoded as json.
-- if .qs is query string table
-- if .postfields is a binary string to be passed in the request body. If this is present, form parameter will be ignored. 
-- @param callbackFunc: a function(rcode, msg, data) end, if nil, the function will not return until result is returned(sync call).
--  `rcode` is http return code, such as 200 for success, which is same as `msg.rcode`
--  `msg` is the raw HTTP message {header, code=0, rcode=200, data}
--  `data` contains the translated response data if data format is a known format like json
--    or it contains the binary response body from server, which is same as `msg.data`
-- @param option: mostly nil. "-I" for headers only
-- @return: return nil if callbackFunc is a function. or the string content in sync call. 
function System.os.GetUrl(url, callbackFunc, option)
end
```

```lua
NPL.load("(gl)script/ide/System/os/GetUrl.lua");
```
- download file or making standard request

```lua
System.os.GetUrl("https://github.com/LiXizhi/HourOfCode/archive/master.zip", echo);`
```
- get headers only with "-I" option. 

```lua
System.os.GetUrl("https://github.com/LiXizhi/HourOfCode/archive/master.zip", function(err, msg, data)  echo(msg) end, "-I");
```
- send form KV pairs with http post
```lua
System.os.GetUrl({url = "http://localhost:8099/ajax/console?action=getparams", form = {key="value",} }, function(err, msg, data)		echo(data)	end);
```
- send multi-part binary forms with http post
```lua
System.os.GetUrl({url = "http://localhost:8099/ajax/console?action=printrequest", form = {name = {file="dummy.html",	data="<html><bold>bold</bold></html>", type="text/html"}, } }, function(err, msg, data)		echo(data)	end);
```
- To send any binary data, one can use 
```lua
System.os.GetUrl({url = "http://localhost:8099/ajax/console?action=printrequest", headers={["content-type"]="application/json"}, postfields='{"key":"value"}' }, function(err, msg, data)		echo(data)	end);
```
- To simplify json encoding, we can send form as json string using following shortcut
```lua
System.os.GetUrl({url = "http://localhost:8099/ajax/console?action=getparams", json = true, form = {key="value", key2 ={subtable="subvalue"} } }, function(err, msg, data)		echo(data)  end);
```

HTTP PUT request:
```lua
System.os.GetUrl({
	method = "PUT",
	url = "http://localhost:8099/ajax/log?action=log", 
	form = {filecontent = "binary string here", }
}, function(err, msg, data)		echo(data)  end);
```

HTTP DELETE request:
```lua
System.os.GetUrl({
	method = "DELETE",
	url = "http://localhost:8099/ajax/log?action=log", 
	form = {filecontent = "binary string here", }
}, function(err, msg, data)		echo(data)  end);
```
On the server side, suppose it is NPL web server, one can get the request body using `request:GetBody()` or `request:getparams()`. The former will only include request body, while the latter contains url parameters as well.

### Debugging HTTP request

In NPL Code Wiki's console window, one can test raw http request by sending a request to `/ajax/console?action=printrequest`, and then check log console for raw request content.

```lua
System.os.GetUrl({url = "http://localhost:8099/ajax/console?action=printrequest", echo);
```


## Local Server
Local server offers a way for you to download files or making url requests with a cache policy. Local database system is used to backup all url requests. The cached files may be in the database or in native file system. 

For more information, please see [localserver](https://github.com/NPLPackages/main/tree/master/script/ide/System/localserver)

### what is a local server?
The LocalServer module allows a web application to cache and serve its HTTP resources locally, without a network connection.

###  Local Server Overview
The LocalServer module is a specialized URL cache that the web application controls. Requests for URLs in the LocalServer's cache are intercepted and served locally from the user's disk.

### Resource stores 
A resource store is a container of URLs. Using the LocalServer module, applications can create any number of resource stores, and a resource store can contain any number of URLs.

There are two types of resource stores:
	- ResourceStore - for capturing ad-hoc URLs using NPL. The ResourceStore allows an application to capture user data files that need to be addressed with a URL, such as a PDF file or an image. 
	- ManagedResourceStore - for capturing a related set of URLs that are declared in a manifest file, and are updated automatically. The ManagedResourceStore allows the set of resources needed to run a web application to be captured. 
For both types of stores, the set of URLs captured is explicitly controlled by the web application.

###  Architecture & Implementation Notes
all sql database manipulation functions are exposed via WebCacheDB, whose implementation is split in WebCacheDB* files. 
localserver is the based class for two servers: ResourceStore and ManagedResourceStore. 

#### Using Local server as a local database
One can use local server as a simple (name, value) pair database with cache_policy functions.

To query a database entry call below, here we will use web service store

```lua
	NPL.load("(gl)script/ide/System/localserver/factory.lua");
	local ls = System.localserver.CreateStore(nil, 2);
	if(not ls) then
		return 
	end
	cache_policy = cache_policy or System.localserver.CachePolicy:new("access plus 1 week");
	
	local url = System.localserver.UrlHelper.WS_to_REST(fakeurl_query_miniprofile, {JID=JID}, {"JID"});
	local item = ls:GetItem(url)
	if(item and item.entry and item.payload and not cache_policy:IsExpired(item.payload.creation_date)) then
		-- NOTE:item.payload.data is always a string, one may deserialize from it to obtain table object.
		local profile = item.payload.data;
		if(type(callbackFunc) == "function") then
			callbackFunc(JID, profile);
		end
	else
		
	end
```	

To add(update) a database entry call below
```lua
	NPL.load("(gl)script/ide/System/localserver/factory.lua");
	local ls = System.localserver.CreateStore(nil, 2);
	if(not ls) then
		return 
	end
	-- make url
	local url = System.localserver.UrlHelper.WS_to_REST(fakeurl_query_miniprofile, {JID=JID}, {"JID"});

	-- make entry
	local item = {
		entry = System.localserver.WebCacheDB.EntryInfo:new({
			url = url,
		}),
		payload = System.localserver.WebCacheDB.PayloadInfo:new({
			status_code = System.localserver.HttpConstants.HTTP_OK,
			data = msg.profile,
		}),
	}
	-- save to database entry
	local res = ls:PutItem(item) 
	if(res) then 
		log("ls put JID mini profile for "..url.."\n")
	else
		log("warning: failed saving JID profile item to local server.\n")
	end
```


#### Lazy writing 

For the URL history, this transaction commit overhead is unacceptably high(0.05s for the most simple write commit). 
On some systems, the cost of committing a new page to the history database was as high as downloading the entire page 
and rendering the page to the screen. As a result, ParaEngine's localserver has implemented a lazy sync system. 

Please see  https://developer.mozilla.org/en/Storage/Performance, for a reference

Localserver has relaxed the ACID requirements in order to speed up commits. In particular, we have dropped durability. 
This means that when a commit returns, you are not guaranteed that the commit has gone through. If the power goes out 
right away, that commit may (or may not) be lost. However, we still support the other (ACI) requirements. 
This means that the database will not get corrupted. If the power goes out immediately after a commit, the transaction 
will be like it was rolled back: the database will still be in a consistent state. 


## Send Email Via SMTP 
It is not trivial to make a SMTP email server. However, it is fairly easy to send an email via an existing email server. 
One can do it manually via [`System.os.GetUrl`](https://github.com/NPLPackages/main/tree/master/script/ide/System/os/GetUrl.lua) with proper options, or we write the following handy function for you.
```lua
System.os.SendEmail({
	url="smtp://smtp.exmail.qq.com", 
	username="lixizhi@paraengine.com", password="XXXXX", 
	-- ca_info = "/path/to/certificate.pem",
	from="lixizhi@paraengine.com", to="lixizhi@yeah.net", cc="xizhi.li@gmail.com", 
	subject = "title here",
	body = "any body context here. can be very long",
}, function(err, msg) echo(msg) end);
```
Here is what I receive in my mail box:

![image](https://cloud.githubusercontent.com/assets/94537/19721841/9ee8ab84-9ba6-11e6-8b5e-a31533c6168c.png)


