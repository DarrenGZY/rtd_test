## NPL vs Lua
NPL embeds the small Lua/luajit compiler, and adds tons of native C/C++ API and new programming paradigms, making NPL a general purpose language/runtime. 

Lua is an embedded language with the world fastest JIT compiler. The entire lua runtime is written in 6000 lines of code, making it very small and memory efficient to be embedded in another host language/runtime. 

## Host Languages vs Embedded Compilers
A high-level language contains its syntax, native API and libraries. 
All dynamic language runtimes must embed one or more compiler implementations in it. Compilers contains low-level implementation of compiling text code to bytecode and then to machine code. 

For example, some version of JAVA, python, mono C#, objective C are first compiled into [LLVM](http://llvm.org/) bytecode, and then the LLVM runtime compiles the bytecode to machine code.  LLVM byte code is similar to lua or luajit byte code. Java/Python/MonoC# runtime embeds LLVM compiler in the same way as we embed lua/luajit in NPL. The difference is that LLVM is large and based on C/C++ syntax; but luajit is really small, and based on lua syntax. Languages like Java/Python/C# also have tons of host API in addition to LLVM in the same way of NPL hosting lua. This is why lua is designed to be an [embedded language](https://www.lua.org/ddj.html) over which other host languages can be devised. 

### Facts about NPL vs Lua:
- NPL provides its own set of libraries written in NPL scripts, whose syntax is 100% compatible with Lua. This means that one can use any existing Lua code in NPL script, but the reverse is NOT possible. 
- NPL runtime has over 700 000 lines of C/C++ [NPL Host API](https://codedocs.xyz/LiXizhi/NPLRuntime/) code (not counting third-party library files), on which all  NPL library files depends on. In NPL, we advise writing NPL modules in our recommend way, rather than using Lua's `require` function to load any external third-party modules. 
- NPL library has over 1 000 000 lines of NPL script code covering many aspects of general-purpose programming. They are distributed as open source [[NPL packages|npl_packages]]. You are welcome to contribute your own NPL package utilizing our rich set of host API and libraries.
- NPL provides both non-preemptive and [[preemptive programming paradigm|Concurrency]] that is not available in Lua. 
- NPL has built-in support for multi-threading and message passing system that is not available in lua.
- Finally, NPL loves Lua syntax. By default all NPL script files has `.lua` extension for ease of editing. Whenever possible, NPL will not deviate its syntax from lua. However, in case, NPL extends Lua syntax, NPL script file will use a different file extension. For example, `*.page`, `*.npl` are used just in case the script uses Non-Lua compatible syntax. 

> NPL is not a wrapper of lua. It hosts lua's JIT compiler to provide its own programming API and language features, in addition to several other compilers, like mono C#, see below.

### Compilers in NPL
NPL runtime state can run on top of three compiler frameworks simultaneously: 
- Lua/Luajit compiler
- C/C++
- Mono/C# compiler
Which means that you can write code with all NPL API and framework features in either of the above three compilers, and these code can communicate with one another with `NPL.activate` in the same NPL runtime process.  Please see [[NPL Architecture|NPLArchitecture]] for details. 

> Code targeting C/C++ should be pre-compiled. The other two can be compiled on the fly. 

## References
Figure 1.0: [NPL Architecture Stack](https://sketchboard.me/NzZQDl1sCrGu#/)

![NPL Architecture Stack](https://github.com/LiXizhi/wiki/blob/master/images/nplarchitecturestack.png)






