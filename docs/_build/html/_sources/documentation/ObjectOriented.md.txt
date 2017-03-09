# Object Oriented Programming
All NPL library source code is written in object oriented way. It is recommended that you do the same for your own code.

> Lua itself can be used as a functional language, which does not enforce object oriented programming. 
It is the programmer's choice of how to write their code. 

NPL has a number of library files to help you code in object oriented ways. 

## Referencing a class
`commonlib.gettable` and `NPL.load` is the primary way for you to include and import other component.
Because all NPL libraries are defined to be compatible with `commonlib.gettable`, it is possible to import the class namespace table without loading it. This also makes the order of importing or loading libraries trivial. 

Remember that NPL/Lua is a statically scoped language and each function is hash value on its containing table, it is important to cache class table on a local variable.

Example of importing `commonlib.LinkedList` into `LinkedList`. 

```lua
local LinkedList = commonlib.gettable("commonlib.LinkedList");

local MyClass = commonlib.gettable("MyApp.MyClass");
function MyClass.Test()
   local list = LinkedList:new();
end
```

By the time, you call the function on the imported table, you need to `NPL.load` its implementation before hand. As you see, most common class is already buddled in common include files, such as `NPL.load("(gl)script/ide/commonlib.lua");`, this include the implementation of `commonlib.gettable` as well. If you are interested, you can read its source code. 
So if you have loaded that file before, such as in your `bootstrapper` file, you do not need to load it in every other files. But you need to call `commonlib.gettable` on the beginning of every file using the given library, for better performance.

> It is good practice to only `NPL.load` non-frequently used files shortly before they are used. So that, your application can boot up faster, and use less memory. 


## Defining a singleton class
For singleton class, which contains only static functions. It is sufficient to use `commonlib.gettable` to define the class. The idea is that you reference the class,and then add some static implementations to it dynamically. 

```lua
local MyClass = commonlib.gettable("MyApp.MyClass");

function MyClass.Method1()

end

function MyClass.Method2()

end
```

## Defining an ordinary class
For class, which you want to create instances from, you need to use the `commonlib.inherit` function. 
For implementation of `commonlib.inherit`, please see [script/ide/oo.lua](https://github.com/NPLPackages/main/blob/master/script/ide/oo.lua).

The following will define a `MyClass` class with `new` method. 
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

To create a new instance of it
```lua
local MyClass = commonlib.gettable("MyApp.MyClass");

local c1 = MyClass:new():init("param1");
local c2 = MyClass:new():init("param1");
```

You can define a derived class like below
```lua
local MyClassDerived = commonlib.inherit(commonlib.gettable("MyApp.MyClass"), commonlib.gettable("MyApp.MyClassDerived"));

-- this is the constructor function.
function MyClassDerived:ctor()
   -- parent class's ctor() have been automatically called
end

function MyClassDerived:init(param1)
   MyClassDerived._super.init(self, param1);
   return self; 
end
```

## Advanced `ToolBase` class
If you want a powerful class with `event/signal/auto property`, and `dynamic reflections`, you can derive your class from `ToolBase` class.

See the examples.
```lua
local Rect = commonlib.gettable("mathlib.Rect");

-- class new class
local UIElement = commonlib.inherit(commonlib.gettable("System.Core.ToolBase"), commonlib.gettable("System.Windows.UIElement"));
UIElement:Property("Name", "UIElement");
UIElement:Signal("SizeChanged");
UIElement:Property({"enabled", true, "isEnabled", auto=true});


function UIElement:ctor()
	-- client rect
	self.crect = Rect:new():init(0,0,0,0);
end

function UIElement:init(parent)
	self:SetParent(parent);
	return self;
end

-- many other functions omitted here
```