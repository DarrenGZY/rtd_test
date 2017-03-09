Multi-Threading
----------------------
In NPL, multi-threading is handled in the same way as networking communication. 
In other words, activating scripts in other local threads is virtually the same as calling scripts running on another computer. You simply communicate with remote script file with the `NPL.activate` in the same way for both local threads and remote computers. The only difference is that the target script url is different. 

For example, to activate a script on local thread `A`, one can use 
```
NPL.activate("(A)helloworld.lua", {});
```

To activate the script in a remote computer `B`'s `C` thread, one can use
```
NPL.activate("(C)B:helloworld.lua", {});
```

For more information, please see [[file activation|ActivationFile]]

## Creating NPL Worker Thread
NPL worker thread must be created, before messages to it can be processed by script running in that thread. 

`NPL.activate("(A)helloworld.lua", {});` does not automatically create thread `A`. You need to call following code to create NPL runtime on thread `A`. 

```lua
NPL.CreateRuntimeState("A", 0):Start();
```
After that, messages sent to `(A)helloworld.lua` will be processed in a real system-level thread called `A`.


## Example Project
Now let us create a real multi-threaded application with just a single file `script/test/TestMultithread.lua`. 
The application print `helloworld` in 5 threads simultaneously. 

```lua
NPL.load("(gl)script/ide/commonlib.lua");

local function Start()
   for i=1, 5 do
      local thead_name = "T"..i;
      NPL.CreateRuntimeState(thead_name, 0):Start();
      NPL.activate(format("(%s)script/test/TestMultithread.lua", thead_name), {
	   text = "hello world", 
	   sleep_time = math.random()*5,
      });
   end
end

local isStarted;
local function activate()
   if(msg and msg.text) then
      -- sleep random seconds to simulate heavy task
      ParaEngine.Sleep(msg.sleep_time);
      LOG.std(nil, "info", "MultiThread", "%s from thread %s", msg.text,  __rts__:GetName());
   elseif(not isStarted) then
      -- initialize on first call 
      isStarted = true;
      Start();
   end
end

NPL.this(activate);
```

To run above file, use 
```
NPL.activate("(gl)script/test/TestMultithread.lua");
```
or from command line
```
npl script/test/TestMultithread.lua
```

The output will be something like below
```
2016-03-16 6:22:00 PM|T1|info|MultiThread|hello world from thread T1
2016-03-16 6:22:01 PM|T3|info|MultiThread|hello world from thread T3
2016-03-16 6:22:03 PM|T2|info|MultiThread|hello world from thread T2
2016-03-16 6:22:03 PM|T5|info|MultiThread|hello world from thread T5
2016-03-16 6:22:04 PM|T4|info|MultiThread|hello world from thread T4
```

## Advanced Examples
Following NPL modules utilize multiple local threads.
 * `script/apps/DBServer/DBServer.lua`:  a database server, each thread for processing SQL logics, a thread monitor is used to find the most free thread to route any sql query. 

## Other Options
By design, threading should be avoided to simplify software design and debugging. 
In addition to real threads, it is usually preferred to use architecture to avoid using thread at all. 
* `Timer/callbacks/events/signals` are good candidates for asynchronous tasks in a single thread. With `NPL.activate`, it even allows you to switch implementation without changing any code; and you can boost performance or debug code in a single thread more easily. 
* `Coroutine` is a lua language feature, which is also supported by NPL. In short, it uses a single thread to simulate multiple virtual threads, allowing you to share all data in the same thread without using locks, but still allowing the developer to `yield` CPU resource to other virtual threads at any time. The interactive debugging module in NPL is implemented with coroutines. Please see `script/ide/Debugger/IPCDebugger.lua` for details.




