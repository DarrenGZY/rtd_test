Welcome to the NPL wiki!
-------------------------
NPL or Neural Parallel Language is an open source, fast, general purpose scripting language. Its syntax is 100%-compatible with [lua](http://www.lua.org). NPL runtime provides essential functionality for building `3D/2D/Server` applications that runs on `windows/Mac/linux/android/iOS`. NPL can mix user-mode preemptive and non-preemptive code. It gives you the concurrency of [Erlang](http://www.erlang.org/) and speed of `Java/C++` in the same dynamic and weakly-typed language.

> NPL技术交流QQ群 543474853

### Install Guide
```
git clone https://github.com/LiXizhi/NPLRuntime.git
git submodule init && git submodule update
./build_linux.sh
```
See [Install Guide](InstallGuide) for details

### Getting Started
* [[What is NPL?|WhatIsNPL]]
* [[Tutorial: HelloWorld|TutorialHelloWorld]]
* [[Source Code Overview|SourceCodeOverview]]

### Example code
```lua
local function activate()
   if(msg) then
      print(msg.data or "");
   end
   NPL.activate("(gl)helloworld.lua", {data="hello world!"})
end
NPL.this(activate); 
```

### Why a New Programming Language?
NPL prototype was designed in 2004, which was then called 'parallel oriented language'. NPL is initially designed to write flexible algorithms that works in a multi-threaded, and distributed environment with many computers across the network. More specifically, I want to have a language that is suitable for writing neural network algorithms, 3d simulation and visualization. Lua and C/C++ affinity was chosen from the beginning. 

### Usage
To run with GUI, use:
``` 
    npl [parameters...]
```    
To run in server mode, use:
```	
    npls [filename] [parameters...]
```    
For example:
```	
    npls hello.lua
```    

