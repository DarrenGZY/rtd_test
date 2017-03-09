File Activation
----------------
NPL can communicate with remote NPL script using the `NPL.activate` function. It is a powerful function, and is available in C++, mono plugin too. 

## Basic Syntax
Like in neural network, all communications are asynchronous and uni-directional without callbacks. Although the function does return a integer value. Please see `NPL Reference` for details. 
```
NPL.activate(url, {any_pure_data_table_here, })
```
* @param `url`: a globally unique name of a NPL file name instance. The string format of an NPL file name is like below. ```[(sRuntimeStateName|gl)][sNID:]sRelativePath[]```
the following is a list of all valid file name combinations: 
  - `user001@paraengine.com:script/hello.lua`	-- a file of user001 in its default gaming thread
  - `(world1)server001@paraengine.com:script/hello.lua`		-- a file of server001 in its thread world1
  - `(worker1)script/hello.lua`			-- a local file in the thread worker1
  - `(gl)script/hello.lua`			-- a glia (local) file in the current runtime state's thread
  - `script/hello.lua`			-- a file in the current thread. For a single threaded application, this is usually enough.
  - `(worker1)NPLRouter.dll`			-- activate a C++ file. Please note that, in windows, it looks for NPLRonter.dll; in linux, it looks for ./libNPLRouter.so 
  - `(worker1)DBServer.dll/DBServer.DBServer.cs`  -- for C# file, the class must have a static activate function defined in the CS file. 

* @param `msg`: it is a chunk of pure data table that will be transmitted to the destination file. 

## Neuron File
Only files associated with an activate function can be activated.  This is done differently in NPL/C++/C# plugin

* In NPL, it is most flexible by using the build-in `NPL.this` function
```lua
local function activate()
   -- input is passed in a global "msg" variable
   echo(msg);
end
NPL.this(activate);
```
* `msg` is passed in a global `msg` variable which is visible to all files, and the `msg` variable  will last until the thread receives the next activation message.
* In NPL's C++ plugin, you need to define a `C` function. Please see ParacraftSDK's example folder for details.
* In NPL's mono C# plugin, you simply define a class with a static `activate` function. Please see ParacraftSDK's example folder for details.

### Input Message, `msg.nid` and `msg.tid`
In above code, `msg` contains the information received from the sender, plus the `source id` of the sender. 
For unauthenticated senders, the source id is stored in `msg.tid`, which is an auto-generated number string like "~1".
The receiver can keep using this temporary id `msg.tid` to send message back, such as 
```lua
local function activate()
   -- input is passed in a global "msg" variable
   NPL.activate(format("%s:some_reply_file.lua", msg.tid or msg.nid), {"replied"});
end
NPL.this(activate);
```
The receiver can also rename the temporary `msg.tid` by calling `NPL.accept(msg.tid, nid_name)`, so the next time if the receiver got a message from the same sender (i.e. the same TCP connection), the `msg.nid` contains the last assigned name and `msg.tid` field no longer exists. We usually use `NPL.accept` to distinguish between authenticated and unauthenticated senders, and reject unauthenticated messages by calling `NPL.reject(msg.tid)` as early as possible to save CPU cycles. 

See following example: 
```lua
local function activate()
   -- input is passed in a global "msg" variable
   if(msg.tid) then
      -- unauthenticated? reject as early as possible or accept it. 
      if(msg.password=="123") then
         NPL.accept(msg.tid, msg.username or "default_user");
      else
         NPL.reject(msg.tid);
      end
   elseif(msg.nid) then
      -- only respond to authenticated messages. 
      NPL.activate(format("%s:some_reply_file.lua", msg.nid), {"replied"});
   end
end
NPL.this(activate);
```
Please note, `msg.tid or msg.nid` always match to a single low-level TCP connection, hence their names are shared to all neuron files in the process. For example, if you accept in one neuron file, all other neuron files will receive messages in `msg.nid` form. 

Please see 
## Neuron File Visibility
For security reasons, all neuron files can be activated by other files in the same process. This includes scripts in other local threads of the same process. See also [[MultiThreading]].

To expose script to remote computers, one needs to do two things. 
* one is to start NPL server by listening to an IP and port: NPL uses TCP protocol for all communications.  
* second is to tell NPL runtime that a given file is a `public neuron file`.

See below: 
```
NPL.StartNetServer("0.0.0.0", 8080);
NPL.AddPublicFile(filename, id);
```
* "0.0.0.0" means all IP addresses, one can also use "127.0.0.1", "localhost" or whatever IP addresses. 
* 8080: is the port number. Pick any one you like.

The second parameter to `NPL.AddPublicFile` is an integer, which is transmitted on behalf of the long filename to save bandwidth. So it must be unique if you add multiple public files. 

> Please note, that file name must be relative to working directory. such as `NPL.AddPublicFile("script/test/test.lua", 1)`. Absolute path is not supported at the moment.

## Activating remote file
Once a server NPL runtime exposes a public file, other client NPL runtime can activate it with the `NPL.activate` function. Please note that, an NPL runtime can be both server and client. The one who started the connection is usually called server. Pure client must also call `NPL.StartNetServer` in order to activate the server. But it can specify `port="0"` to indicate that it will not listen for incoming connections.

However, on the client, we need to assign a local name to the remote server using the `NPL.AddNPLRuntimeAddress`, so that we can refer to this server by name in all subsequent `NPL.activate` call. 

```lua
NPL.AddNPLRuntimeAddress({host = "127.0.0.1", port = "8099", nid = "server1"})
```
Usually we do this during initialization time only once. After that we can activate public files on the remote server like below:

```lua
NPL.activate("server1:helloworld.lua", {})
```
Please note the name specified by `nid` is arbitrary and used only on the client computer to refer to a computer. In other words, different clients can name the same remote computer differently. 

### Message Delivery Guarantees
Please note that the first time that a computer activate a remote file, a TCP connection is automatically established, but the first message is NOT delivered. This is because `NPL.activate()` is asynchronous, it must return a value before the new connection is established. It always returns 0 when your message is delivered via an existing path, and non-zero in case of first message to a remote system. 

If there is no already established path (i.e no TCP connection), NPL will immediately try to establish it. 
However, please note, the message that returns non-zero is NOT delivered, even if NPL successfully established a path to the remote system soon afterwards. Thus it is the programmer's job to activate again until `NPL.activate` returns 0. This guarantees that a message with 0 return value is always delivered at least in the viewpoint of NPL runtime. 

The same mechanism can be used to recover lost-connections. 

To write fault-tolerant message passing code, consider following. 
- When a NPL runtime process start, ping remote process with NPL.activate until it returns 0 to establish TCP connections. This ensures that all subsequent NPL.activate across these two systems can be delivered. 
- Discover Lost Connections: 
  - Method1: Use a timer to ping or listen for disconnect system event and reconnect in case connection is lost. However, individual messages may be lost during these time. 
  - Method2: Use a wrapper function to call NPL.activate, which checks its return value. If it is non-zero, either reconnect with timeout or put message to a pending queue in case connection can be recovered shortly and resend queued messages. 

We leave it to the programmer to handle all occasions when NPL.activate returns non-zero values, since different business logic may use a different approach.

* [Akka Reference](http://doc.akka.io/docs/akka/2.2.5/general/message-delivery-guarantees.html)

### Example Client/Server app
To run the example, call following. 

```lua
npl "script/test/network/SimpleClientServer.lua" server="true"
npl "script/test/network/SimpleClientServer.lua" client="true"
```

The source code of this demo is also included in `ParaCraftSDK/examples` folder.

filename: `script/test/network/SimpleClientServer.lua`

```lua
--[[
Author: Li,Xizhi
Date: 2009-6-29
Desc: start one server, and at least one client. 
-----------------------------------------------
npl "script/test/network/SimpleClientServer.lua" server="true"
npl "script/test/network/SimpleClientServer.lua" client="true"
-----------------------------------------------
]]
NPL.load("(gl)script/ide/commonlib.lua"); -- many sub dependency included

local nServerThreadCount = 2;
local initialized;
local isServerInstance = ParaEngine.GetAppCommandLineByParam("server","false") == "true";

-- expose these files. client/server usually share the same public files
local function AddPublicFiles()
    NPL.AddPublicFile("script/test/network/SimpleClientServer.lua", 1);
end

-- NPL simple server
local function InitServer()
    AddPublicFiles();
    
    NPL.StartNetServer("127.0.0.1", "60001");
	
	for i=1, nServerThreadCount do
		local rts_name = "worker"..i;
		local worker = NPL.CreateRuntimeState(rts_name, 0);
		worker:Start();
	end
    
    LOG.std(nil, "info", "Server", "server is started with %d threads", nServerThreadCount);
end

-- NPL simple client
local function InitClient()
    AddPublicFiles();

    -- since this is a pure client, no need to listen to any port. 
	NPL.StartNetServer("0", "0");
    
	-- add the server address
	NPL.AddNPLRuntimeAddress({host="127.0.0.1", port="60001", nid="simpleserver"})
	
    LOG.std(nil, "info", "Client", "started");
    
	-- activate a remote neuron file on each thread on the server
	for i=1, nServerThreadCount do
		local rts_name = "worker"..i;
		while( NPL.activate(string.format("(%s)simpleserver:script/test/network/SimpleClientServer.lua", rts_name), 
            {TestCase = "TP", data="from client"}) ~=0 ) do
            -- if can not send message, try again.
            echo("failed to send message");
            ParaEngine.Sleep(1);
        end
	end
end

local function activate()
    if(not initialized) then
        initialized = true;
        if(isServerInstance) then
            InitServer();
        else
            InitClient();
        end
	elseif(msg and msg.TestCase) then
    	LOG.std(nil, "info", "test", "%s got a message", isServerInstance and "server" or "client");
		echo(msg);
	end
end
NPL.this(activate);
```
The above server is actually multi-threaded, please see [[MultiThreading]] for details.

> The first time, `NPL.activate` calls a new remote server (with which we have not established TCP connection), the message is dropped and returned a non-zero value. `NPLExtension.lua` contains a number of helper functions to help you for sending a guaranteed message, such as `NPL.activate_with_timeout`.You need to include [[commonlib]] to use it.

## Trusted Connections and NID
In the receiver's activate function, it can assign any name or `nid` to incoming connection's NPL runtime. 

## HelloWorld Example
Now, here comes a more complicated helloworld. It turns an ordinary `helloworld.lua` file into a neuron file, by associating an `activate` function with it. The file is then callable from any NPL thread or remote computer by its NPL address(url). 

~~~lua
local function activate()
   if(msg) then
      print(msg.data or "");
   end
   NPL.activate("(gl)helloworld.lua", {data="hello world!"})
end
NPL.this(activate);
~~~

## Simple Web Server Example
NPL uses a HTTP-compatible protocol, so it is possible to handle standard HTTP request using the same NPL server. 
When NPL runtime receives a HTTP request message, it will send it to a publicly visible file with id `-10`. 
So we can create a simple HTTP web server with just a number of lines, like below:

filename: `main.lua`
```lua
NPL.load("(gl)script/ide/commonlib.lua");

local function StartWebServer()
	local host = "127.0.0.1";
	local port = "8099";
	-- tell NPL runtime to route all HTTP message to the public neuron file `http_server.lua`
	NPL.AddPublicFile("source/SimpleWebServer/http_server.lua", -10);
	NPL.StartNetServer(host, port);
	LOG.std(nil, "system", "WebServer", "NPL Server started on ip:port %s %s", host, port);
end
StartWebServer();

local function activate()
end
NPL.this(activate);
```

filename: `http_server.lua`
```lua
NPL.load("(gl)script/ide/Json.lua");
NPL.load("(gl)script/ide/LuaXML.lua");

local tostring = tostring;
local type = type;

local npl_http = commonlib.gettable("MyCompany.Samples.npl_http");

-- whether to dump all incoming stream;
npl_http.dump_stream = false;

-- keep statistics
local stats = {
	request_received = 0,
}

local default_msg = "HTTP/1.1 200 OK\r\nContent-Length: 31\r\nContent-Type: text/html\r\n\r\n<html><body>hello</body></html>";

local status_strings = {
	ok ="HTTP/1.1 200 OK\r\n",
	created ="HTTP/1.1 201 Created\r\n",
	accepted ="HTTP/1.1 202 Accepted\r\n",
	no_content = "HTTP/1.1 204 No Content\r\n",
	multiple_choices = "HTTP/1.1 300 Multiple Choices\r\n",
	moved_permanently = "HTTP/1.1 301 Moved Permanently\r\n",
	moved_temporarily = "HTTP/1.1 302 Moved Temporarily\r\n",
	not_modified = "HTTP/1.1 304 Not Modified\r\n",
	bad_request = "HTTP/1.1 400 Bad Request\r\n",
	unauthorized = "HTTP/1.1 401 Unauthorized\r\n",
	forbidden = "HTTP/1.1 403 Forbidden\r\n",
	not_found = "HTTP/1.1 404 Not Found\r\n",
	internal_server_error = "HTTP/1.1 500 Internal Server Error\r\n",
	not_implemented = "HTTP/1.1 501 Not Implemented\r\n",
	bad_gateway = "HTTP/1.1 502 Bad Gateway\r\n",
	service_unavailable = "HTTP/1.1 503 Service Unavailable\r\n",
};
npl_http.status_strings = status_strings;

-- make an HTML response
-- @param return_code: nil if default to "ok"(200)
function npl_http.make_html_response(nid, html, return_code, headers)
	if(type(html) == "table") then
		html = commonlib.Lua2XmlString(html);
	end
	npl_http.make_response(nid, html, return_code, headers);
end

-- make a json response
-- @param return_code: nil if default to "ok"(200)
function npl_http.make_json_response(nid, json, return_code, headers)
	if(type(html) == "table") then
		json = commonlib.Json.Encode(json)
	end
	npl_http.make_response(nid, json, return_code, headers);
end

-- make a string response
-- @param return_code: nil if default to "ok"(200)
-- @param body: must be string
-- @return true if send. 
function npl_http.make_response(nid, body, return_code, headers)
	if(type(body) == "string" and nid) then
		local out = {};
		out[#out+1] = status_strings[return_code or "ok"] or return_code["not_found"];
		if(body~="") then
			out[#out+1] = format("Content-Length: %d\r\n", #body);
		end
		if(headers) then
			local name, value;
			for name, value in pairs(headers) do
				if(name ~= "Content-Length") then
					out[#out+1] = format("%s: %s\r\n", name, value);
				end
			end
		end
		out[#out+1] = "\r\n";
		out[#out+1] = body;

		-- if file name is "http",  the message body is raw http stream
		return NPL.activate(format("%s:http", nid), table.concat(out));
	end
end


local function activate()
	stats.request_received = stats.request_received + 1;
	local msg=msg;
	local nid = msg.tid or msg.nid;
	if(npl_http.dump_stream) then
		log("HTTP:"); echo(msg);
	end
	npl_http.make_response(nid, format("<html><body>hello world. req: %d. input is %s</body></html>", stats.request_received, commonlib.serialize_compact(msg)));
end
NPL.this(activate)
```

For a full-fledged build-in HTTP server framework in NPL, please see [[WebServer]]