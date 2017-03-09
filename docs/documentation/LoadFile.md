## NPL Load File
In NPL code, one may call something like `NPL.load("script/ide/commonlib.lua")` to load a file. 
All NPL code needs to be compiled in order to run. `NPL.load` does following things automatically for you. 
- find the correct file either in NPL zip package or according to NPL search path defined in [[npl_packages]] and automatically use precompiled binary version (`*.o`) if available (see [[DeployGuide]]).
   - relative path is supported, but only recommended for private files, such as `NPL.load("./A.lua") or NPL.load("../../B.lua")`
   - when using relative path, file extension can be omitted, where *.npl and *.lua are searched, such as `NPL.load("./A") or NPL.load("../../B")`
- automatically resolve recursion.(such as File A load File B, while B also load A).
- compiles the script code if not yet compiled
   - if file extension is `*.npl`, NPL [meta compiler](MetaProgramming) will be invoked.
- The first time code is compiled, it also execute the code chunk in protected mode with `pcall`. In most cases,  this means injecting new code to the global or exported table, so that one can use them later. 
- it can also be used to load `C++/Mono C#/NPL packages`
- return the exported file module if any. See `file based module` section below for details. 

```cpp
/**
load a new file (in the current runtime state) without activating it. If the file is already loaded,
it will not be loaded again unless bReload is true. 
IMPORTANT: this function is synchronous; unlike the asynchronous activation function. 
LoadFile is more like "include in C++".When the function returns, contents in the file is loaded to memory. 
@note: in NPL/lua, function is first class object, so loading a file means executing the code chunk in protected mode with pcall, 
in most cases,  this means injecting new code to the global table. Since there may be recursions (such as A load B, while B also load A), 
your loading code should not reply on the loading order to work. You need to follow basic code injection principles.
For example, commonlib.gettable("") is the the commended way to inject new code to the current thread's global table. 
Be careful not to pollute the global table too much, use nested table/namespace. 
Different NPL applications may have their own sandbox environments, which have their own dedicated global tables, for example all `*.page` files use a separate global table per URL request in NPL Web Server App.
@note: when loading an NPL file, we will first find if there is an up to date compiled version in the bin directory. if there is, 
we will load the compiled version, otherwise we will use the text version.  use bin version, if source version does not exist; use bin version, if source and bin versions are both on disk (instead of zip) and that bin version is newer than the source version. 
e.g. we can compile source to bin directory with file extension ".o", e.g. "script/abc.lua" can be compiled to "bin/script/abc.o", The latter will be used if available and up-to-date. 
@param filePath: the local relative file path. 
If the file extension is ".dll", it will be treated as a plug-in. Examples:
	"NPLRouter.dll"			-- load a C++ or C# dll. Please note that, in windows, it looks for NPLRonter.dll; in linux, it looks for ./libNPLRouter.so 
	"plugin/libNPLRouter.dll"			-- almost same as above, it is reformatted to remove the heading 'lib' when loading. In windows, it looks for plugin/NPLRonter.dll; in linux, it looks for ./plugin/libNPLRouter.so
@param bReload: if true, the file will be reloaded even if it is already loaded.
   otherwise, the file will only be loaded if it is not loaded yet. 
@remark: one should be very careful when calling with bReload set to true, since this may lead to recursive 
	reloading of the same file. If this occurs, it will generate C Stack overflow error message.
*/
NPL.load(filePath, bReload)
```

### Code Injection Model
Since, in NPL/lua, table and functions are first class object, we use a very flexible code injection model to manage all dynamically loaded code during the life time of an application. 

To inject new code, we use method like `commonlib.gettable`, `commonlib.inherit`, please see [[ObjectOriented]] for details. The developer should ensure they inject their code with unique names (such as `CompanyName.AppName.ModuleName` etc). In other words, do not pollute the global table. 

To reference these code, the user needs to call `NPL.load` as well as `commonlib.gettable` to create a local stub variable in the file scope. Please note, due to `NPL.load` order, the stub may be created before the actual code is injected into it. 

For example, you have a class file in `script/MyApp/MyClass.lua` like below
```lua
local MyClass = commonlib.inherit(nil, commonlib.gettable("MyApp.MyClass"));

MyClass.default_param = 1;

-- this is the constructor function.
function MyClass:ctor()
   self.map = {};
end

function MyClass:init(param1)
   self.map.param1 = param1;
   return self; 
end

function MyClass:Clone()
   return MyClass:new():init(self:GetParam());
end

function MyClass:GetParam()
   return self.map.param1;
end
```

To use above class,  one may use
```lua
NPL.load("(gl)script/MyApp/MyClass.lua")
local MyClass = commonlib.gettable("MyApp.MyClass");

local UserClass = commonlib.inherit(nil, commonlib.gettable("MyApp.UserClass"));

function UserClass:ctor()
   self.map = MyClass:new();  -- use another class
end
```

> please see [[ObjectOriented]] for details. 


## File-based Modules
File-based modules use the containing source code filename to store exported object. So programmers do not need to explicitly specify object namespace in code, this solves several issues:
- code written in this style is independent of its containing file location. 
- code looks more isolated.
- multiple versions of the same code can coexist.

> One not-so-convenient thing is that the user must explicitly load every referenced external modules written in this way in every file. This is the primary reason why the common NPL library are NOT written in this way. 

### Defining file-based module
There are three ways to define a file-based module. Suppose, you have a file called `A.npl` and you want to export some object from it. 
- One way is to call `NPL.export` when file is loaded. 
```lua
-- this is A.npl file
local A = {};
NPL.export(A);

function A.print(s)
  echo(s);
end
```

Following is a better and recommended way, which works with cyclic dependencies. See `Module Cyclic Reference` section for details.
```lua
-- this is A.npl file
local A = NPL.export();
function A.print(s)
  echo(s);
end
```

- Another way is simply return the object to be associated with the current file at the last line of code, like below.
This is also a lua-compatible way. It does not work when there are cyclic dependencies.
```lua
-- this is A.npl file
local A = {};
return A;
```

- finally, there is an advanced manual way to simply add to the `_file_mod_` table.
```lua
-- this is A.npl file
local A = {};
_file_mod_[NPL.filename():gsub("([^/]*$)", "").."A.npl"] = A;
```

As you can see, file-based module simply automatically stores the mapping from the module's full filename to its exported object in a hidden global table. If one changes the filename or file location, the mapping key also changes. This is why you can load multiple versions of the same module and let the user to choose which one to use in a given source file. 

#### Module Cyclic Reference
When you write file modules that depends on each file, there are cyclic dependencies. One workaround is to modify the default exported object, which is always an empty table, instead of creating a local table. See below. We have two files A and B that reference each other. 
```lua
-- A.lua
local B = NPL.load("./B.lua")
local A = NPL.export();
function A.print(s)
  B.print(s)
end
``` 

```lua
-- B.lua
local A = NPL.load("./A.lua")
local B = NPL.export();
function B.print(s)
  echo(s..type(A));
end
``` 

> `NPL.export()` is the core of file module system in NPL, it will return the default empty table representing the current file. This is the same way as how `commonlib.gettable()` solve cyclic dependency.

#### Modules with Object Oriented Class Inheritance
The following shows an example of two class table with inheritance, and also cyclic dependency.
```lua
-- base_class.lua
local derived_class = NPL.load("./derived_class.lua")
local base_class = commonlib.inherit(nil, NPL.export());
function base_class:ctor()
   derived_class:cyclic_dependency_test();
end
``` 

```lua
-- derived_class.lua
local base_class = NPL.load("./base_class.lua")
local B = commonlib.inherit(base_class, NPL.export());
function B:ctor()
end
function B:cyclic_dependency_test()
end
``` 

### Loading file-based module With module name
To import a file-based module, one calls `NPL.load(modname)`. `NPL.load` is a versatile function which can load either standard source files or file-based modules and return the exported module object. 

> A `modname` is different from filename, a string is treated as `modname` if and only if it does NOT contain file extension or begin with "./" or "../". such as `NPL.load("sample_mod")`


`NPL.load(modname)` will automatically find the source file of the module by trying the following locations in order until it finds one. Using module name in NPL.load is less efficient than using explicit filename, because it needs to do the search every time it is called, so only use it at file-load time, such as at the beginning of a file. 

- if the code that invokes `NPL.load` is from `npl_packages/[parentModDir]/`
   - try: `npl_packages/[parentModDir]/npl_mod/[modname]/[modname].lua|npl`
   - try: `npl_packages/[parentModDir]/npl_packages/[modname]/npl_mod/[modname]/[modname].lua|npl`
- try: `npl_mod/[modname]/[modname].npl|lua`
- try: `npl_packages/[modname]/npl_mod/[modname]/[modname].npl|lua` 

Example 1: NPL.load("sample_mod") will test following file locations for both *.npl and *.lua file:
- if code that invokes it is from `npl_packages/[parentModDir]/`
   - `npl_packages/[parentModDir]/npl_mod/sample_mod/sample_mod.npl`
   - `npl_packages/[parentModDir]/npl_packages/sample_mod/npl_mod/sample_mod/sample_mod.npl`
- `npl_mod/sample_mod/sample_mod.npl`
- `npl_packages/sample_mod/npl_mod/sample_mod/sample_mod.npl` 

Example 2: NPL.load("sample_mod.xxx.yyy") will test following file locations for both *.npl and *.lua file:
- if code that invokes it is from `npl_packages/[parentModDir]/`
   - `npl_packages/[parentModDir]/npl_mod/sample_mod/xxx/yyy.npl`
   - `npl_packages/[parentModDir]/npl_packages/sample_mod/npl_mod/sample_mod/xxx/yyy.npl`
- `npl_mod/sample_mod/xxx/yyy.npl`
- `npl_packages/sample_mod/npl_mod/sample_mod/xxx/yyy.npl` 

As you can see, the `npl_packages/` and `npl_mod/` are two special folders to search for when loading file based module.
code inside npl_packages/xxx/ folder always use local `npl_mod` and local npl_packages before searching for global ones, this allow multiple versions of the same npl_mod to coexist in different npl_packages. It is up to the developer to decide how to package and release their application or modules. 

Here is an [example of npl_mod](https://github.com/NPLPackages/main/tree/master/npl_mod/sample_mod) in main package

For more information on NPL packages, please see [[npl_packages]]

### When to Use `require`
`require` is Lua's way to load a file based module. NPL hooks into this function to always call `NPL.load` with the same input before passing down. More specifically, NPL injects a custom loader (at index 2) into the lua's `package.loaders`. So calling `require` is almost identical to calling `NPL.load` so far, except that multiple code versions can not coexist.

#### Do not Use `require`
Since NPL.load now hooks original lua's require function. Facts in this section may not be true, but we still do not recommend using `require` in your own code, unless you are reusing third-party lua libraries. The reason is simple, NPL.load is backward-compatible with the original require, but the original require does not support features provided by NPL.load. Using require in your own code may confuse other lua developers. Therefore, code written for NPL developers should always use NPL.load. 

One should only use `require` to load lua C plugins, do not use it in your own NPL scripts. `require` is rated low in community. Its original implementation is same, but use a similar global hidden table, whose logic you never know, in a flat manner. It also does not support cyclic dependency so far. 

Moreover, some application may need to create sandbox environment to isolate code execution (code are injected to specified global table), using `require` give you no control of code injection. 

Different NPL applications may have their own sandbox environments, which have their own dedicated global tables, for example all `*.page` files use a separate global table per URL request in NPL Web Server App.

Finally, require` is slow and uses different search paths than NPL; while NPL.load is fast and honors precompiled code either in package zip file or in search paths defined in [[npl_packages]], and consistent on all platforms. 
