# Concurrency Model
This is an in-depth topic about NPL's concurrency model and design principles. It also compares it with other similar models in popular languages, like [erlang](erlang.org), [GO](https://golang.org/), [Scala(java)](http://www.scala-lang.org/), etc. 

## What is Concurrency?
The mental picture of all computer languages is to execute in sequential order. Concurrency is a language feature about writing code that runs concurrently.  Traditional way of doing this is via threads and locks, which is very troublesome to write. [The Actor Model](https://en.wikipedia.org/wiki/Actor_model), which was first proposed by Carl Hewitt in 1973, takes a different approach to concurrency, which should avoid the problems caused by threading and locking. 

There are many implementations of concurrency model in different languages, they differ both in performance under different use cases, and in the programmers' mental picture when writing concurrent code. 

> NPL uses a hybrid approach, which give you the ability to run tens of thousands of tasks in a single thread or across multiple threads. More importantly, you do not need to write any code to spawn a virtual process or write error-prone message loops.  In short, NPL is very fast, scalable and give programmers a mental picture that is close to neurons in the brain.

## Concurrent Activation in NPL
### Preemptive vs Non-Preemptive File Activation
By default NPL activate function is non-preemptive, it is the programmer's job to finish 
execution within reasonable time slice. The default non-preemptive mode gives the programmer 
full control of how code is executed and different neuron files can easily share data using global tables in the same thread.

On the other hand, NPL also allows you to do preemptive activation, in which the NPL runtime will count
virtual machine instructions until it reaches a user-defined value (such as 1000) and then 
automatically yield(pause) the activate function. The function will be resumed automatically
in the next time slice. NPL time slice is by default about 16ms (i.e. 60FPS). 

To make file activate function preemptive, simply pass a second parameter `{PreemptiveCount, MsgQueueSize, [filename|name], clear}` to `NPL.this` like below:
- PreemptiveCount: is the number of VM instructions to count before it yields. If nil or 0, it is non-preemptive. Please note JIT-compiled code does not count as instruction by default, see below.
- MsgQueueSize: Max message queue size of this file, if not specified, it is same as the NPL thread's message queue size. 
- filename|name: virtual filename, if not specified, the current file being loaded is used. 
- clear: clear all memory used by the file, including its message queue. Normally one never needs to clear. 
A neuron file without messages takes less than 100 bytes of memory (mostly depending on the length's of its filename)

```lua
-- this is a demo of how to use preemptive activate function. 
NPL.this(function() 
	local msg = msg; -- important to keep a copy on stack since we go preemptive.
	local i=0;
	while(true) do
		i = i + 1;
		echo(tostring(msg.name)..i);
		if(i==400) then
			error("test runtime error");
		end
	end
end, {PreemptiveCount = 100, MsgQueueSize=10, filename="yourfilename.lua"});
```
You can test your code with:
```lua
NPL.activate("yourfilename.lua", {name="hi"});
```
Facts about preemptive activate function:
- It allows you to run tens of thousands of jobs concurrently in the same system thread. 
Each running job has its own stack and the memory overhead is about 450bytes. 
A neuron file without pending messages takes less than 100 bytes of memory (mostly depending on the length's of its filename).
The only limitation to the number of concurrent jobs is your system memory.
- There is a slight performance penalty on program speed due to counting VM instructions. 
- With preemptive activate function, the programmer should pay attention when 
making changes to shared data in the thread, since your function may be paused at any instruction.
The golden rule here is never make any changes to shared data, but use messages to exchange data.
- C/C++ API call is counted as one instruction, so if you call ParaEngine.Sleep(10), 
it will block all concurrent jobs on that NPL thread for 10 seconds. 
- Code in async callbacks (such as timer, remote api call) in activate function are NOT preemptive. 
Because callbacks are invoked from the context of other concurrent activate functions.

### Test Cases and Examples:
see also `script/ide/System/test/test_concurrency.lua` for more tests cases.
```lua
local test_concurrency = commonlib.gettable("System.Core.Test.test_concurrency");

function test_concurrency:testRuntimeError()
	NPL.this(function() 
		local msg = msg; -- important to keep a copy on stack since we go preemptive.
		local i=0;
		while(true) do
			i = i + 1;
			echo(tostring(msg.name)..i);
			if(i==40) then
				error("test runtime error");
			end
		end
	end, {PreemptiveCount = 100, MsgQueueSize=10, filename="tests/testRuntimeError"});
	NPL.activate("tests/testRuntimeError", {name="1"});
	NPL.activate("tests/testRuntimeError", {name="1000"});
end

function test_concurrency:testLongTask()
	NPL.this(function() 
		local msg = msg; -- important to keep a copy on stack since we go preemptive.
		local i=0;
		while(true) do
			i = i + 1;
			echo(i);
		end
	end, {PreemptiveCount = 100, MsgQueueSize=10, filename="tests/testLongTask"});
	NPL.activate("tests/testLongTask", {name="1"});
end

function test_concurrency:testMessageQueueFull()
	NPL.this(function() 
		local msg = msg; -- important to keep a copy on stack since we go preemptive.
		local i=0;
		for i=1, 1000 do
			i = i + 1;
		end
		echo({"finished", msg});
	end, {PreemptiveCount = 100, MsgQueueSize=3, filename="tests/testMessageQueueFull"});
	for i=1, 10 do
		NPL.activate("tests/testMessageQueueFull", {index=i});	
	end
	-- result: only the first three calls will finish.
end

function test_concurrency:testMemorySize()
	__rts__:SetMsgQueueSize(100000);
	for i=1, 10000 do
		NPL.this(function() 
			local msg = msg; -- important to keep a copy on stack since we go preemptive.
			for i=1, math.random(1,1000) do
				msg.i = i;	
			end
			echo(msg);
		end, {PreemptiveCount = 10, MsgQueueSize=1000, filename="tests/testMemorySize"..i});
		NPL.activate("tests/testMemorySize"..i, {index=i});	
	end
	-- TODO: use a timer to check when it will finish. 
end

function test_concurrency:testThroughput()
	__rts__:SetMsgQueueSize(100000);
	for i=1, 10000 do
		NPL.this(function() 
			local msg = msg;
			while(true) do
				echo(msg)
			end
		end, {PreemptiveCount = 10, MsgQueueSize=3, filename="tests/testThroughput"..i});
		NPL.activate("tests/testThroughput"..i, {index=i});	
	end
end
```
## NPL Message Scheduling
Each NPL runtime can have one or more NPL states/threads (i.e. real system threads).
Each NPL state has a single message queue for input and output with other NPL threads or remote processes. 
Programmer can set the maximum size of this queue, so when it is full, messages are automatically dropped. 

On each time slice, NPL state will process `ALL` messages in its message queue in priority order. 
- If a message belongs to a non-preemptive file, it will invoke its activate function immediately.
- If a message belongs to a preemptive file, it will remove the message from the queue and insert it to the target
file's message queue, which may has a different message queue size. If message queue of the file is full, it will drop 
the message immediately. 

In another native thread timer, all active preemptive files (with pending/half-processed messages) will be processed/resumed in `preemptive` way. I.e. we will count VM(virtual machine) instructions for each activate function and pause it if necessary. 

> Note: all TCP/IP network connections are managed by a single global network IO thread. In this way you can have tens of thousands of live TCP connections without using much system memory. A global NPL dispatcher automatically dispatch incoming network messages to the message queue of the target NPL state/thread. Normally, the number of NPL threads used is close to the number of CPU cores on the computer.

### Causions on Preemptive Programming
Since preemptive code and non-preemptive code can coexit in the same NPL state (thread).
It is the programmers' job to ensure preemptive code does not modify global data.

- Note: when NPL debugger is attached, all preemptive code are paused to make debugging possible.
- Also, please note that JIT-compiled code does NOT call hooks by
default. Either disable the JIT compiler or edit src/Makefile:
`XCFLAGS= -DLUAJIT_ENABLE_CHECKHOOK` This comes with a speed penalty even hook is not set.
Our recommendation is to call `dummy()` or any NPL API functions and count that. 

### Priority of Concurrent Activation
PreemptiveCount is the total number of instructions (or Byte code) executed in an activate function before we pause it. 
In NPL, `PreemptiveCount` can be specified per activate function, thus giving you fine control over how much computation each activate function can run in a given time slice. 

The NPL scheduler for preemptive activation file will guarantee each function will run precisely PreemptiveCount instructions after PreemptiveCount * total_active_process_count instructions every time slice.

So there is absolutely no dead-lock conditions in our user-mode `preemptive` scheduler. The only thing that may break your application is running out of memory. However, each active (running) function only take 400-2000 bytes according to usage. even 1 million concurrently running jobs takes only about 1GB memory. If only half of those jobs are busy doing calculation on average, you get only a little over 500MB memory usage.


## Language Compare: The Mental Picture
We will compare concurrency model in several languages in terms of programmer's mental picture and implementation. 

### Erlang
In [erlang](erlang.org), one has to manually call `spawn` function to create a user-mode virtual process. So it is both fast and memory efficient to create millions of erlang processes in a single thread.
Erlang simulates `preemptive` scheduling among those processes by counting byte code executed. Erlang processes are currently scheduled on a reduction count basis as described [here](http://erlang.org/pipermail/erlang-questions/2001-April/003132.html) and [here](https://hamidreza-s.github.io/erlang/scheduling/real-time/preemptive/migration/2016/02/09/erlang-scheduler-details.html). One reduction is roughly equivalent to a function call. A process is allowed to run until it pauses to wait for input (a message from some other process) or until it has executed 1000 reductions. A process waiting for a message will be re-scheduled as soon as there
is something new in the message queue, or as soon as the receive timer (receive ... after Time -> ... end) expires. It will then be put last in the appropriate queue. Erlang has 4 scheduler queues (priorities): 'max', 'high', 'normal', and 'low'.

- Pros: In the viewpoint of programmers, erlang process is `preemptive`. Underneath it is not true `preemptive`, but as along as it does not call `C` functions, the scheduler is pretty accurate. 
- Cons: The programmers need to manually name, create and manage the process's message loop (such as when message queue grows too big). 

Comparison:
- Similarity: 
    - Both NPL and erlang copies messages when sending them
    - Both support `preemptive` concurrent scheduler.
    - Both use user mode code to simulate processes, so that there is almost no limit to the total number of asynchronous entities created.
- Differences: 
    - Erlang is `preemptive`; NPL can be not `preemptive` and `non-preemptive`. 
    - NPL use source code file name as the entity name, and it does not require the programmer to create and manually write the message loop. Thus the mental picture is `each NPL file has a hidden message loop`. 
    - Erlang does not give you explicit control over which real system thread your process is running (instead, it automatically does it for you). In NPL, each neuron file can be explicitly instanced on one or more system-level thread. 
    - Erlang scheduler does not give you control over Instruction Count per process before it is preemptied. In NPL, one can specify different instruction count per activate function, thus giving you a much finer control over priority.
    - In NPL we count each C API as a single instruction, while Erlang tries to convert C API to the number of ByteCode executed in the same time. 

### Go
To me, [GO](https://golang.org/)'s concurrency model is a wrapper of C/C++ threading model. Its syntax and library can greatly simply the code (if written in C/C++). It uses real system threads, so you can not have many concurrent jobs like erlang or NPL. 

Comparison:
- NPL can be written in the same fashion of GO, because it give you explicit control over real system-level thread.

### Scala
[Scala](http://www.scala-lang.org/) is a meta language over higher general purpose language like java. 
Scala is like a rewrite of Erlang in java. Its syntax is in object-oriented fashion with a mixture of Erlang's functional style. So please refer to erlang for comparision. However, Scala is not-preemptive as erlang, it relies on the programmer to yield in its event based library, while NPL supports `preemptive` mode in additional to `non-preemptive` mode.

### Final Words
- Unlike erlang, go, scala, NPL is a dynamic and weak type language. In NPL, it is faster to invoke C/C++ code, its byte code is as fast as most strongly typed languages. See [[NPLPerformance]] for details.


