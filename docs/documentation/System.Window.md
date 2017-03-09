# 2D GUI Windows

A `System.Windows.Window` is the primary container for 2D GUI. There are multiple ways to create windows. 
The most common way is to use `mcml` markup language. The programing model is like writing `HTML/js` web page. 

### Quick Sample

First you need to create a html file, such as  `source/HelloWorldMCML/mcml_window.html`.

```html
<pe:mcml>
<script refresh="false" type="text/npl" src="mcml_window.lua"><![CDATA[
    function OnClickOK()
        local content = Page:GetValue("content") or "";
        _guihelper.MessageBox("你输入的是:"..content);
    end
]]></script>
<div style="color:#33ff33;background-color:#808080">
    <div style="margin:10px;">
        Hello World from HTML page!
    </div>
    <div style="margin:10px;">
        <input type="text" name="content" style="width:200px;height:25px;" />
        <input type="button" name="ok" value="确定" onclick="OnClickOK"/>
    </div>
</div>
</pe:mcml>
```

To create and show the window using the html file, we simply do following code. 
```lua
	NPL.load("(gl)script/ide/System/Windows/Window.lua");
	local Window = commonlib.gettable("System.Windows.Window")
	local window = Window:new();
	window:Show({
		url="source/HelloWorldMCML/mcml_window.html", 
		alignment="_lt", left = 300, top = 100, width = 300, height = 400,
	});
```


### Window Alignment

Notice the alignment parameter specify the relative position to its parent or the native screen window.
```
		* @param alignment: can be one of the following strings or nil or left out entirely: 
		*	- "_lt" align to left top of the screen
		*	- "_lb" align to left bottom of the screen
		*	- "_ct" align to center of the screen
		*	- "_ctt": align to center top of the screen
		*	- "_ctb": align to center bottom of the screen
		*	- "_ctl": align to center left of the screen
		*	- "_ctr": align to center right of the screen
		*	- "_rt" align to right top of the screen
		*	- "_rb" align to right bottom of the screen
		*	- "_mt": align to middle top
		*	- "_ml": align to middle left
		*	- "_mr": align to middle right
		*	- "_mb": align to middle bottom
		*	- "_fi": align to left top and right bottom. This is like fill in the parent window.
		*
		*	the layout is given below:
		*	_lt _mt _rt	
		*	_ml _ct _mr 
		*	_lb _mb _rb 
```

`left, top, width, height` have different meanings for different alignment type. The most simple one is "_lt" which means left top alignment. 

- center alignment:
  - `alignment="_ct", left = 0, top = -100, width = 300, height = 400` will display window at center. 
Normally, if you want to center a window with given `width` and `height`, one should use:
`left = -width/2, top = -height/2, width, height`.  The left, top means relative to the center of the parent window. "_ctt" means center top. 
- middle alignment:
  - "_mt" middle top: it means that the pixels relative to left and right border of the parent window should be `fixed`.In other words, if the parent resizes, the width of the window also resize. 
  - `_mt`: x is coordinate from the left. y is coordinate from the top, width is the coordinate from the right and height is the height
  - `_mb`: x is coordinate from the left. y is coordinate from the bottom, width is the coordinate from the right and height is the height
  - `_ml`: x is coordinate from the left. y is coordinate from the top, width is the width and height is the coordinate from the bottom
  - `_mr`: x is coordinate from the right. y is coordinate from the top, width is the width and height is the coordinate from the bottom

### MCML V1 vs V2
There two implementations of MCML: `v1` and `v2`. The example above is given by v2, which is the preferred implementation. However, v1 is mature and stable implementation, v2 is new and currently does not have as many build-in controls as v1. We are still working on v2 and hope it can be 100% compatible and powerful with v1. To launch the same mcml page with v1, one can use 

```lua
	NPL.load("(gl)script/kids/3DMapSystemApp/mcml/PageCtrl.lua");
	local page = System.mcml.PageCtrl:new({url="source/HelloWorldMCML/mcml_window.html"});
	page:Create("testpage", nil, "_lt", 0, 0, 300, 400);
```

**Difference between v1 and v2**
- v1 uses NPL's build-in GUI objects written in C++, i.e. ParaUIObject like `button, container, editbox`. Their implementation is hidden from NPL scripts and user IO is exposed to NPL script via callback functions. 
- v2 uses NPL script to draw all the GUI objects using 3d-triangle drawing API and handles user input in NPL script. Therefore the user has complete control over the look of their control in NPL script. One can read the source code of v2 in main package. 


### UI Elements (Controls)
In mcml v2 implementation, mcml page are actually translated to UI Elements at runtime. 
UI Element contains a collection of GUI controls written completely in NPL. They all derive from [UIElement](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/UIElement.lua), which contains virtual functions to paint the control and handle user IO event.

The following are buildin-in UI elements or controls:
- [Button](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/Controls/Button.lua)
- [EditBox](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/Controls/EditBox.lua)
- [Label](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/Controls/Label.lua)
- [Canvas](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/Controls/Canvas.lua)
- [Window](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/Window.lua)

### Creating GUI Without MCML
In most cases, we should use mcml to create GUI, however, there are places when you want to create your custom control or custom mcml tag, you will then need to write your own control.

Following are some examples:

```lua
function test_Windows:TestCreateWindow()
	
	-- create the native window
	local window = Window:new();

	-- test UI element
	local elem = UIElement:new():init(window);
	elem:SetBackgroundColor("#0000ff");
	elem:setGeometry(10,0,64,32);

	-- test create rectangle
	local rcRect = Rectangle:new():init(window);
	rcRect:SetBackgroundColor("#ff0000");
	rcRect:setGeometry(10,32,64,32);

	-- test Button
	local btn = Button:new():init(window);
	btn:SetBackgroundColor("#00ff00");
	btn:setGeometry(10,64,64,32);
	btn:Connect("clicked", function (event)
		_guihelper.MessageBox("you clicked me");
	end)
	btn:Connect("released", function(event)
		_guihelper.MessageBox("mouse up");
	end)
	
	-- show the window natively
	window:Show("my_window", nil, "_mt", 0,0, 200, 200);
end

function test_Windows:TestMouseEnterLeaveEvents()
	-- create the native window
	local window = Window:new();
	window.mouseEnterEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"window enter", event:localPos()}));	
	end
	window.mouseLeaveEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"window leave"}));	
	end

	-- Parent1
	local elem = UIElement:new():init(window);
	elem:SetBackgroundColor("#0000ff");
	elem:setGeometry(10,0,64,64);
	elem.mouseEnterEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"parent1 enter", event:localPos()}));	
	end
	elem.mouseLeaveEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"parent1 leave"}));	
	end

	-- Parent1:Button1
	local btn = Button:new():init(elem);
	btn:SetBackgroundColor("#ff0000");
	btn:setGeometry(0,0,64,32);
	btn.mouseEnterEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"btn1 enter", event:localPos()}));	
	end
	btn.mouseLeaveEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"btn1 leave"}));	
	end

	-- Button2
	local btn = Button:new():init(window);
	btn:SetBackgroundColor("#00ff00");
	btn:setGeometry(10,64,64,32);
	btn.mouseEnterEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"btn2 enter", event:localPos()}));	
	end
	btn.mouseLeaveEvent = function(self, event)
		Application:postEvent(self, System.Core.LogEvent:new({"btn2 leave"}));	
	end
	
	-- show the window natively
	window:Show("my_window1", nil, "_mt", 0,200, 200, 200);
end

function test_Windows:TestEditbox()
	
	-- create the native window
	local window = Window:new();

	-- test UI element
	local elem = EditBox:new():init(window);
	elem:setGeometry(60,30,64,25);
	-- elem:setMaxLength(6);
	-- show the window natively
	window:Show("my_window", nil, "_lt", 0,0, 200, 200);
end
```

### Creating Custom MCML v2 Controls 

This is our custom control `MyApp.Controls.MyCustomControl`, whose implementation is defined in `test_pe_custom.lua`
```html
<pe:mcml>
<script type="text/npl" refresh="false">
<![CDATA[
]]>
</script>
    <pe:custom src="test_pe_custom.lua" classns="MyApp.Controls.MyCustomControl" style="width:200px;height:200px;">
    </pe:custom>
</pe:mcml>
```

Now in `test_pe_custom.lua`, we create its implementation. 
```lua
local MyCustomControl = commonlib.inherit(commonlib.gettable("System.Windows.UIElement"), commonlib.gettable("MyApp.Controls.MyCustomControl"));

function MyCustomControl:paintEvent(painter)
	painter:SetPen("#ff0000");
	painter:DrawText(10,10, "MyCustomControl");
end
```

### Preview and Debug Layout
Sometimes, we want to change the code and preview the result without restarting the whole application. If your windows contains no external logics and uses mcml v1 exclusively, you can preview using MCML browser, simply press `Ctrl+F3`. 

In most cases, you may need to write temporary or persistent test code for your new window class with data connected. 
This usually involves delete the old window object and create a new one like below. Run it repeatedly in NPL code wiki's console to preview your updated mcml code.
```lua
	-- remove old window
	local window = commonlib.gettable("test.window")
	if(window and window.CloseWindow) then
		window:CloseWindow(true);
	end

	-- create a new window
	NPL.load("(gl)script/ide/System/Windows/Window.lua");
	local Window = commonlib.gettable("System.Windows.Window")
	local window = Window:new();
	window:Show({
		url="script/ide/System/test/test_mcml_page.html", 
		alignment="_lt", left = 0, top = 0, width = 200, height = 400,
	});
	-- keep a reference for refresh
	test.window = window;
```
