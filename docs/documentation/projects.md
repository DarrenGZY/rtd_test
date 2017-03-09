# What is NPL?

NPL is short for neural parallel language.  
* NPL is a general-purpose open-source language. Its syntax is 100% compatible with [Lua](http://www.lua.org/). 
* NPL provides essential functionality for building `3D/2D/Web/Server` [applications](projects).
* NPL provides rich C/C++ [API](https://codedocs.xyz/LiXizhi/NPLRuntime/) and large collections of [libraries](https://github.com/NPLPackages/main/tree/master/script/ide) written in NPL.
* NPL is a single language solution for advanced and interactive GUI, complex opengl/DirectX 3D graphics, scalable [webserver](WebServer) and distributed software frameworks. It is cross-platform, [high performance](NPLPerformance), extensible and [debuggable](NPLCodeWiki).
* NPL is a language system initially designed to operate like the brain. Nodes and connections are ubiquitous; [threading](MultiThreading) and networking logics are hidden from developers.
* NPL can mix user-mode `preemptive` and `non-preemptive` code. It gives you the [concurrency](Concurrency) of [Erlang](http://www.erlang.org/) and speed of `Java/C++` in the same dynamic and weakly-typed language.

> See [here](projects) for a list of projects written in NPL. 

## Facts You Should Know
* NPL runtime is written in C/C++. It can utilize the [luajit](http://luajit.org/) compiler, which dynamically compiles script/byte code into native machine code, thus it can approach native C/C++ performance with some caveats. 
* NPL runtime native API is a rich collection of C/C++ functions that is callable from NPL script. It includes networking, graphics, io, audio, assets management, and core NPL/ParaEngine infrastructure. [NPL Reference](https://codedocs.xyz/LiXizhi/NPLRuntime) contains its full documentation. However, 99% of time, we never use native API directly in NPL script.  
* Instead, we use NPL libraries, which are pure NPL/lua script files, distributed open sourced or in a zip/pkg file. Over 1 million lines of fully-documented NPL script code are available for use. Try to search in the repository of NPL source code, before writing your own.
* NPL libraries are all written in `object oriented` fashion, code is organized in folders and table namespaces. 
* Because NPL is written in C/C++, it is cross-platform, extensible and easy to integrate with other thirty-party tools. For examples, NPLRuntime is distributed with following built in plugins: [bullet](http://bulletphysics.org/)(a robust 3d physics engine), [mono](http://www.mono-project.com/)(C# scripting module with NPL API), [mysql](https://www.mysql.com/)/[sqlite](https://www.sqlite.org/)(database engine), [libcurl](https://curl.haxx.se/)(a robust http/ssh client). 
* There are two ways to use Lua, one is embedding, the other is extending it. NPL takes the former approach. For example, true preemptive programming would NOT be possible with the second approach. The embedding approach also allows NPL to expose all of its API and language features to other compiler, for example, each NPL runtime states can host Lua/Mono C#/C++ compiler states all together. 

## `Hello World` in NPL
Run in command line `npl helloworld.lua`. 
The content of `helloworld.lua` is like this:
~~~lua
print("hello world");
~~~

Now, there comes a more complicated helloworld. It turns an ordinary `helloworld.lua` file into a neuron file, by associating an `activate` function with it. The file is then callable from any NPL thread or remote computer by its NPL address(url). 
~~~lua
NPL.activate("(gl)helloworld.lua", {data="hello world!"})
local function activate()
   local msg = msg;
   if(msg) then
      print(msg.data or "");
   end
end
NPL.this(activate);
~~~

If your code has `*.npl` file extension, you can use or extend NPL syntax via [meta programming](MetaProgramming).
For example, above code can be written with NPL syntax in `helloworld.npl` as below.
~~~lua
NPL.activate("(gl)helloworld.npl", {data="hello world!"})
this(msg){
   if(msg) then
      print(msg.data or "");
   end
}
~~~

## Why Should I Use NPL?
You may write your next big project in NPL, if you meet any following condition, if not all: 
* If you like truly dynamic language like lua.
* If you care about C/C++ compatibility and [[performance|NPLPerformance]]. 
* Coming from the C/C++ world, yet looking for a replacement language for java/C#, that can handle heavy load of 2d/3d graphical client or server applications, easier than C/C++.
* Looking for a scripting language for both client and server development that is small and easy to deploy on windows and other platforms. 
* Looking for a dynamic language that can be just-in-time compiled and modified at run time. 
* Mix user-mode `preemptive` and `non-preemptive` code for massive concurrent computation.

> Note: calling API between C++/scripting runtime, usually requires a context switch which is computationally expensive most language also involves type translation (such as string parameter), managed/unmanaged environment transition (garbage collection), etc. NPL/LUA has the smallest cost among all languages, and with luajit, type translation is even unnecessary. Thus making it ideal to choose NPL when some part of your responsive app needs to be developed in C/C++.
 
## Background
NPL prototype was designed in 2004, which was then called 'parallel oriented language'. In 2005, it was implemented together with ParaEngine, a distributed 3d computer game engine. 

### Why a New Programming Language? 
NPL is initially designed to write flexible algorithms that works in a multi-threaded, and distributed environment with many computers across the network. More specifically, I want to have a language that is suitable for writing neural network algorithms, 3d simulation and visualization. Lua and C/C++ affinity was chosen from the beginning. 

### Communicate Like The Brain
Although we are still unclear about our brain's exact computational model, however, following fact seems ubiquitous in our brain. 
 * The brain consists of neurons and connections. 
 * Data flows from one neuron to another: it is asynchronously, single-direction and without callback.
Communication in NPL is the same as above. Each file can become a neuron file to receive messages from other neuron files. They communicate asynchronously without callback. As a result, no lock is required because there is no shared data; making it both simple and fast to write and test distributed algorithms and deploy software in heterogeneous environment. 

### Compare NPL With Other Languages
* `C/C++`: 
  * Pros: They are actually the only popular cross-platform language. NPL Runtime is also written in C++. 
  * Cons: But you really need a more flexible scripting language which are dynamically compiled and easier to use.
* `Mono C#`, `Java`: 
  * Pros: Suitable for server side or heavy client. Rich libraries. 
  * Cons: Deployment on windows platform is big. Writing C++ plugins is hard, calling into C++ function is usually slow. Language is strong typed, not really dynamic. 
* `Node.js`, `Electron`: 
  * Pros: Suitable for server side or heavy client. HTML5/javascript is popular. 
  * Cons: Deployment on the client-side is very big. writing C++ plugins is very hard. 
* `Python`, `Ruby`, `ActionScript` and many others:
  * Pros: Popular and modular
  * Cons: Performance is not comparable to C/C++; standard-alone client-side deployment is very big; syntax too strict for a scripting language. Some of them looks alien for C/C++ developers.
* `Lua`:
  * `Pros`: NPL is 100% compatible with it. Really dynamic language, highly extensible by C/C++, and syntax is clear for C/C++ developers.
  * `Cons`: Born as an embedded language, it does not have a standalone general-purpose runtime with native API support and rich libraries for complex client/server side programming. This is where NPL fits in. 
* [[Concurrency Model|Concurrency]]: in-depth compare with erlang, GO, scala

## References
Click the links, if you like to read the old history:
* [NPL prototype doc in 2005](https://github.com/LiXizhi/lixizhi.github.io/blob/master/downloads/2.0.ParaEngine.and.NPL.doc)
* [A thesis on NPL/ParaEngine in 2005](https://github.com/LiXizhi/lixizhi.github.io/blob/master/downloads/DisGameEngineThesis.pdf) 
* [A book on ParaEngine in 2006](https://github.com/LiXizhi/lixizhi.github.io/blob/master/downloads/DisGameEngineVol1_review.pdf)