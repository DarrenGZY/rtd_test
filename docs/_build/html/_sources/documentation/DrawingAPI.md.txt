# Drawing With 2D API
There are two low-level ways to draw 2d graphical objects. 

- One is via creating ParaUIObject controls (like buttons, containers, editbox). These controls are managed in C++ and created in NPL scripts. 
- The other is creating owner draw ParaUIObject, and do all the drawings with Painting API in NPL scripts, such as draw rect, lines, text, etc. 

Moreover, there is a number of high-level graphical libraries written in NPL which allows you to create 2D user interface more easily. 

- IDE controls is NPL wrapper of the low-level C++ API. It provides more versatile controls than the raw ParaUIObjects.
   - One old implementation is based on ParaUIObject controls.
   - One new implementation is based on Painting API,  in `System.Window.UIElement` namespace. 
- [MCML/NPL](mcml) is a HTML/JS like mark up language to create user interface. 
   - One old implementation is based on ParaUIObject controls.
   - One new implementation is based on Painting API,  in `System.Window.mcml` namespace. 

> This article is only about low-level drawing API in C++ side, which are exposed to NPL. However, one should always use [MCML](mcml) or [Windows.UIElement](System.Window) instead of following raw API. 

## button
Creating a button with a texture background and text. The root element is a container see next section.
Please place a png texture `Texture/alphadot.png` at the working directory. 

```lua
NPL.load("(gl)script/ide/System/System.lua");

local clickCount = 1;
local function CreateUI()
   local _this = ParaUI.CreateUIObject("button", "MyBtn", "_lt", 10, 10, 64, 22);
   _this.text= "text";
   _this.background = "Texture/alphadot.png";
   _this:SetScript("onclick", function(obj) 
      obj.text = "clicked "..clickCount;
      clickCount = clickCount + 1;
   end)
   _this:AttachToRoot();
end
CreateUI();
```

## container
Create a container and a child button inside it. 

```lua
	local _parent, _this;
	_this = ParaUI.CreateUIObject("container", "MyContainer", "_lt", 10, 110, 200, 64);
	_this:AttachToRoot();
	_this.background = "Texture/alphadot.png";
	_parent = _this;

	-- create a child
	_this = ParaUI.CreateUIObject("button", "b", "_rt", -10-32, 10, 32, 22);
	_this.text= "X"
	_this.background = "";
	_parent:AddChild(_this);
```

## text and editbox
```lua
```

## System Fonts
In ParaEngine/NPL, we support loading fonts from `*.ttf` files.
One can install custom font files at `fonts/*.ttf`, and use them with filename, such as `Verdana;bold`.
Please note, all controls in ParaEngine uses "System" font by default. "System" font maps to the system font used by the current computer by default. However, one can change it to a different custom font installed in `fonts/*.ttf`. 
Suppose you have a font file at `fonts/ParaEngineThaiFont.ttf`, you can map default System font to it by calling. 
```lua
Config.AppendTextValue("GUI_font_mapping","System");	
Config.AppendTextValue("GUI_font_mapping","ParaEngineThaiFont");
```

It is a pair of function calls, the first is name, the second is value. 
We recommend doing it in `script/config.lua`, it is loaded before any UI control is created. Please see that file for details.

Please note, for each font with different size, weight, and name, we will create a different internal temporary image file for rendering. So it is recommended to use just a few fonts in your application. 

