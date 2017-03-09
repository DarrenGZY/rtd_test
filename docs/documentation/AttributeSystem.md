## Attribute System 
Almost all C++ API objects like `ParaUIObject`, `ParaObject`, and even some global table like `ParaEngine`, expose a data interface via [`ParaAttributeObject`](https://codedocs.xyz/LiXizhi/NPLRuntime/classParaScripting_1_1ParaAttributeObject.html). 

The attribute system allows us to easily get or set data in core C++ object via NPL scripts, like below.

```lua
local attr = ParaEngine.GetAttributeObject();
local value = attr:GetField("IgnoreWindowSizeChange", false);
attr:SetField("IgnoreWindowSizeChange", not value);
```

In [NPL code wiki](NPLCodeWiki), one can open from menu `view::object browser` to inspect all living core objects via the attribute system. 

> Please note [script/ide/System/Core/DOM.lua](https://github.com/NPLPackages/main/blob/master/script/ide/System/Core/DOM.lua) provides a handy interface to iterate over existing core objects or even pure NPL tables. 

