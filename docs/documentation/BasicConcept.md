# NPL Basic Concept

This section provides a more detailed description to NPL's language feature.

## A Brief History of Computer Language
Computer programming language is essentially a language in itself, like English and Chinese; but it is mainly written by humans to instruct computers. Our current computers has its unique way to process language. In general, it is a kind of command-like IO system, with input and output. Any piece of code can always be statically mapped to nodes and connections, where each connection has a single-unique direction from output to input; when a computer executes the code, it moves data along those connections. This is similar to how our brain works by our biological neural network. The difference is that we do not know how, when and where data is passed along. Since parallelism in our brain can also be mapped to a certain kind of node-connection relationship; the only thing differs is computational speed. So when designing computer language, let us first take concurrent programming out, and focus on making it easier and efficient to create all kinds of nodes and connections, and passing data along the connections.

Let us list all major patterns: 
* Nearly all languages provides functions:
  * Functional language like `C` provides defining custom input/output patterns with functions. Let us regard `if,else,expressions`, etc as build-in functions.
* In some languages like `javascript`, function is first-class object, allowing one to create interesting node-connection patterns. 
* Some like `C++, java, php` supports object-oriented programming, which is essentially a filter-like input/output patterns, that moves data as a whole more easily. 
* Some is weakly typed like `javascript`, `python` allowing a node to accept input from more versatile input, at some expense. Some strongly-typed language like `C++` use template-programming to simulate this feature at the cost of more code generated.
* Some is statically scoped like `lua`, that automatically captured variables for any asynchronous callback, and some dynamically scoped language also simulates this feature with anonymous function and captures in some clumsy way like 'C#'.
* Some features establishing node and connections across multiple threads or computers on the network.

So what is the language that support all the above, while still maintaining high-performance? 
A close answer is `lua`; the complete answer is `NPL`.   

  
## What NPL is Not
* `meta programming` is a technique to translate text code from first domain to second one, and then compile second one and execute. It is common for some library to use this technique to provide a more concise way of coding with the library. Examples:
   * `qt.io` in C++
   * `php` which pre-process code-mixed hyper text into complete programming code.
   * `Scala` to 'Java'
   * `CoffeeScript` to `Javascript`
* NPL is NOT `meta programming`. But [[NPL web server|WebServer]] library use a similar technique to make coding server web pages that generate HTML5/javascript on the client more easily.  
* NPL is not syntactic sugar or meta-programming for concurrent programming library. It is the programmer's job to write or use libraries that support concurrent programming, rather than introducing some non-intuitive syntax to the language itself. 

## NPL Runtime Architecture
This section helps you to understand the internals of NPL runtime. One of the major things that one need to understand is its message passing model. 

### Message Passing Model
In NPL, any file on disk can receive messages from other files. A file can send messages to other files via `NPL.activate` function. This function is strictly asynchronous. The details is below:

Message processing always has two phases: 
* phase1: synapse data relay phase
* phase2: neuron file response phase

When you send a message via `NPL.activate`, the message is in phase 1. Message is not processed immediately, but cached in a queue. For example, if the target neuron file is on the local machine, the message is pushed (cached) directly to local thread queue; if the target is on remote machine, NPL runtime will automatcally establish TCP connection, and it will ensure the message is pushed(cached) on the target machine's specified thread queue. 

Message processing takes place in phase2. Each NPL runtime process may contain one or more NPL runtime states, each corresponds to one system-level thread with its own message queue, memory, and code. NPL runtime states are logically separated from each other. In each NPL state, it processes messages in its message queue in heart-beat fashion. Therefore it is the programmer's job to finish processing the message within reasonable time slice, otherwise the message queue will be full and begin to lose early messages. 

### Code Overview
This section helps you to understand the open source code of NPL.

#### incoming connections
```
incoming connection:  acceptor/dispatcher thread
	-> CNPLNetServer::handle_accept 
		-> CNPLNetServer.m_connection_manager.start(a new CNPLConnection object) : a new CNPLConnection object is added to CNPLConnectionManager 
			-> CNPLConnection.start() 
				-> automatically assign a temporary nid to this unauthenticated connection, temp connection is called ~XXX.
					-> CNPLDispatcher::AddNPLConnection(temporary nid, CNPLConnection)
				-> begin reading the connection sockets
				-> LATER: Once authenticated, the scripting interface can call (CNPLConnection)NPL.GetConnection(nid)::rename(nid, bAuthenticated) 
```
if any messages are sent via the new incomming connection, the message field should contain msg.nid and msg.tid, 
where tid is temporaray nid. If there is no nid, it means that the connection is not yet authenticated. 
In NPL, one can call `NPL.accept(tid, nid)`


#### incoming messages
```
incoming message
	-> CNPLConnection::handle_read 
		-> CNPLConnection::handleReceivedData
			-> CNPLConnection::m_parser.parse until a complete message is read
				-> CNPLConnection::handleMessageIn()
					-> CNPLConnection::m_msg_dispatcher.DispatchMsg((NPLMsgIn)this->m_input_msg)
						-> CNPLRuntimeState = GetNPLRuntime()->GetRuntimeState(NPLMsgIn::m_rts_name)
						-> if CNPLDispatcher::CheckPubFile(NPLMsgIn::m_filename)
							-> new NPLMessage(from NPLMsgIn)
							-> CNPLRuntimeState::Activate_async(NPLMessage);
								-> CNPLRuntimeState::SendMessage(NPLMessage) -> insert message to CNPLRuntimeState.m_input_queue
						-> or return access denied	
```

#### outgoing messages
```
from NPL script NPL.activate()
	-> CNPLRuntime::NPL_Activate
		-> if local activation, get the runtime state.
			-> CNPLRuntimeState::Activate_async(NPLMessage) -> CNPLRuntimeState::SendMessage(NPLMessage) -> insert message to CNPLRuntimeState.m_input_queue
		-> if remote activation with nid, send via dispatcher	
			-> CNPLDispatcher::Activate_Async(NPLFileName, code)
				-> CNPLConnection = CNPLDispatcher::CreateGetNPLConnectionByNID(nid) 
					-> if found in CNPLDispatcher::m_active_connection_map(nid, CNPLConnection), return
					-> or if found in CNPLDispatcher::m_pending_connection_map(nid, CNPLConnection), return
					-> or if found in CNPLDispatcher::m_server_address_map(host, port, nid): we will actively establish a new connection to it. 
						-> CNPLDispatcher::CreateConnection(NPLRuntimeAddress)
							-> CNPLNetServer::CreateConnection(NPLRuntimeAddress)
								-> new CNPLConnection() 
								-> CNPLConnection::SetNPLRuntimeAddress(NPLRuntimeAddress) 
								-> (CNPLNetServer::m_connection_manager as CNPLConnectionManager)::add(CNPLConnection)
								-> CNPLConnection::connect() -> CNPLConnection::m_resolver->async_resolve -> 
									-> CNPLConnection::handle_resolve -> CNPLConnection::m_socket.async_connect 
									-> CNPLConnection::handle_connect 
										-> CNPLConnection::handleConnect() : for custom behavior
										-> CNPLConnection::start()
											-> CNPLDispatcher::AddNPLConnection(CNPLConnection.m_address.nid, CNPLConnection)
												-> add nid to CNPLDispatcher::m_active_connection_map(nid, CNPLConnection), remove nid from m_pending_connection_map(nid, CNPLConnection)
											-> begin reading the connection sockets
							-> Add to CNPLDispatcher::m_pending_connection_map if not connected when returned, return the newly created connection
					-> or return null.	
				-> CNPLConnection::SendMessage(NPLFilename, code)
					-> new NPLMsgOut( from NPLFilename, code)
					-> push to CNPLConnection::m_queueOutput() and start sending first one if queue is previously empty.
```

#### NPL Message Format
NPL message protocol is based on TCP and HTTP compatible. More information is in [NPLMsgIn_parser.h](https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/NPL/NPLMsgIn_parser.h)

*quick sample*
```
A (g1)script/hello.lua NPL/1.0
rts:r1
User-Agent:NPL
16:{"hello world!"}
```

### NPL Quick Guide
In NPL runtime, there can be one or more runtime threads called NPL runtime states. 
Each runtime state has its own name, stack, thread local memory manager, and message queue. 
The default NPL state is called (main), which is also the thread for rendering 3D graphics. It is also the thread to spawn other worker threads.

To make a NPL runtime accessible by other NPL runtimes, one must call 
```lua
	NPL.StartNetServer("127.0.0.1", "60001");
```
This must be done on both client and server. Please note that NPL does not distinguish between client and server. If you like, you may call the one who first calls `NPL.activate` to be client. For private client, the port number can be "0" to indicate that it will not be accessible by other computers, but it can always connect to other public NPL runtimes.

To make files within NPL runtime accessible by other NPL runtimes, one must add those files to public file lists by calling 
```lua
	NPL.AddPublicFile("script/test/network/TestSimpleClient.lua", 1);
	NPL.AddPublicFile("script/test/network/TestSimpleServer.lua", 2);
```
the second parameter is id of the file. The file name to id map must be the same for all communicating computers. This presumption may be removed in futhure versions.  

To create additional worker threads (NPL states) for handling messages, one can call
```lua
	for i=1, 10 do
		local worker = NPL.CreateRuntimeState("some_runtime_state"..i, 0);
		worker:Start();
	end
```

Each NPL runtime on the network (either LAN/WAN) is identified by a globally unique `nid` string. 
NPL itself does not verify or authenticate nid, it is the programmers' job to do it. 

To add a trusted server address to nid map, one can call 
```lua
	NPL.AddNPLRuntimeAddress({host="127.0.0.1", port="60001", nid="this_is_server_nid"})
```
so that NPL runtime knows how to connection to the nid via TCP endpoints. host can be ip or domain name.
	
To activate a file on a remote NPL runtime(identified by a nid), one can call 
```lua
while( NPL.activate("(some_runtime_state)this_is_server_nid:script/test/network/TestSimpleServer.lua", 
 {some_data_table}) ~=0 ) do 
end
```
Please note that `NPL.activate()` function will return non-zero if no connection is established for the target, 
but it will immediately try to establish a new connection if the target address of server nid is known. 
The above sample code simply loop until activate succeed; in real world, one should use a timer with timeout or use one of the helper function such as `NPL.activate_with_timeout`. 

When a file is activated, its activation function will be called, and data is inside the global msg table. 
```lua
	local function activate()
		if(msg.nid) then
		elseif(msg.tid) then
		end
		NPL.activate(string.format("%s:script/test/network/TestSimpleClient.lua", msg.nid or msg.tid), {some_date_sent_back})
	end
	NPL.this(activate)
```
`msg.nid` contains the nid of the source NPL runtime. `msg.nid` is nil, if connection is not authenticated or nid is not known. In such cases, `msg.tid` is always available; it contains the local temporary id. When the connection is authenticated, one can call following function to reassign nid to a connection or reject it. NPL runtime will try to reuse the same existing TCP connect between current computer and each nid or tid target, for all communcations between the two endpoints in either direction.  

```lua
	NPL.accept(msg.tid, nid)  -- to rename tid to nid, or 
	NPL.reject(msg.tid) -- to close the connection. 
```

> only the main thread support non-thread safe functions, such as those with rendering. For worker thread(runtime state), please only use NPL functions that are explicitly marked as thread-safe in the documentation. 

### NPL Extensibility and Scripting Performance
NPL provides three extensibility modes: (1) Lua scripting runtime states (2) Mono .Net runtimes (3) C++ plugin interface
All of them can be used simultanously to create logics for game applications. All modes support cross-platform and multi-threaded computing. 

Lua runtime is natively supported and has the most extensive ParaEngine API exposed. It is both extremely light weighted and having a good thread local memory manager. 
It is suitable for both client and server side scripting. It is recommended to use only lua scripts on the client side. 

Mono .Net runtimes is supported via NPLMono.dll (which in turn is a C++ plugin dll). The main advantage of using .Net scripting is its rich libraries, popular language support(C#,Java,VB and their IDE) and compiled binary code performance.
.Net scripting runtime is recommended to use on the server side only, since deploying .Net on the client requires quite some disk space. And .Net (strong typed language) is less effective than lua when coding for client logics. 

C++ plugins allows us to treat dll file as a single script file. It has the best performance among others, but also is the most difficult to code. 
We would only want to use it for performance critical tasks or functions that make extensive use of other third-party C/C++ libraries. For example, NPLRouter.dll is a C++ plugin for routing messages on the server side. 

### Cross-Runtime Communication
A game project may have thousands of lua scripts on the client, mega bytes of C# code contained in several .NET dll assemblies on the server,  and a few C++ plugin dlls on both client and server. (On linux, *.dll will actually find the *.so file)
All these extensible codes are connected to the ParaEngine Runtime and have full access to the exposed ParaEngine API.
Hence, source code written in different language runtimes can use the ParaEngine API to communicate with each other as well as the core game engine module.  

More over, `NPL.activate` is the single most versatile and effective communication function used in all NPL extensibility modes. 
In NPL, a file is the smallest communication entity. Each file can receive messages either from the local or remote files. 
   * In NPL lua runtime, each script file with a activate() function assigned can receive messages from other scripts. 
   * In NPL .Net runtime, each C# file with a activate() function defined inside a class having the same name as the file name can receive messages from other scripts. (namespace around class name is also supported, see the example)
   * In NPL C++ plugins, each dll file must expose a activate() function to receive messages from other scripts. 

Following are example scripts and NPL.activate() functions to send messages to them. 
```
---------------------------------------
File name:	script/HelloWorld.lua  (NPL script file)
activate:	NPL.activate("script/HelloWorld.lua", {data})
---------------------------------------
local function activate()
end
NPL.this(activate)

---------------------------------------
File name:	HelloWorld.cs inside C# mono/MyMonoLib.dll
activate:	NPL.activate("mono/MyMonoLib.dll/HelloWorld.cs", {data})  
			NPL.activate("mono/MyMonoLib.dll/ParaMono.HelloWorld.cs", {data}) 
---------------------------------------
class HelloWorld
{
	public static void activate(ref int type, ref string msg)
	{
	}
}
namespace ParaMono
{
	class HelloWorld
	{
		public static void activate(ref int type, ref string msg)
		{
		}
	}
}
---------------------------------------
File name:	HelloWorld.cpp inside C++ MyPlugin.dll
activate:	NPL.activate("MyPlugin.dll", {data})
---------------------------------------
CORE_EXPORT_DECL void LibActivate(int nType, void* pVoid)
{
	if(nType == ParaEngine::PluginActType_STATE)
	{
	}
}
```

The `NPL.activate()` can use the file extension (*.lua, *.cs, *.dll) to distinguish the file type.

All `NPL.activate()` communications are asynchronous and each message must go through message queues between the sending and receiving endpoints. 
The performance of NPL.activate() is therefore not as efficient as local function calls, so they are generally only used to dispatch work tasks to different worker threads or communicating between client and server.  
Generally, 30000 msgs per second can be processed for 100 bytes message on 2.5G quad core CPU machine. 
For pure client side game logics, we rarely use NPL.activate(), instead we use local function calls whereever possible. 

### High Level Networking Programming Libraries
In real world applications, we usually need to build a network architecture on top of NPL, which usually contains the following tool sets and libraries.
   * gateway servers and game(application) servers where the client connections are kept. 
   * routers for dispaching messagings between game(app) servers and database servers. 
   * database servers that performances business logics and query the underlying databases(My SQLs)
   * memory cache servers that cache data for database servers to minimize access to database.
   * Configuration files and management tools (shell scripts, server deployingment tools, monitoring tools, etc)
   * backup servers for all above and DNS servers.