# MCML Markup Language For 2D GUI
MCML or Micro Cosmos Markup Language is a meta language written in NPL since 2008, since then it has become the standard of writing 2D user interface in NPL. MCML is inspired by early version of ASP.net, however it uses local architecture for local user interface. 

### Quick Sample

Here is an example of mcml page in a `.html` file, such as `source/HelloWorldMCML/mcml_window.html`. The file extension only makes it possible for you to preview the page in other html code editor. 

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
To render the page, one can launch it with a window like below. See [[System.Window]] for details.
```lua
	NPL.load("(gl)script/ide/System/Windows/Window.lua");
	local Window = commonlib.gettable("System.Windows.Window")
	local window = Window:new();
	window:Show({
		url="source/HelloWorldMCML/mcml_window.html", 
		alignment="_lt", left = 300, top = 100, width = 300, height = 400,
	});
```

### MCML V1 vs V2
There two implementations of MCML: `v1` and `v2`. The example above is given by v2, which is the preferred implementation. However, v1 is mature and stable implementation, v2 is new and currently does not have as many build-in controls as v1. We are still working on v2 and hope it can be 100% compatible and powerful with v1.  This article is only about v2. See last section for v1.

**Difference between v1 and v2**
- v1 uses NPL's build-in GUI objects written in C++, i.e. ParaUIObject like `button, container, editbox`. Their implementation is hidden from NPL scripts and user IO is exposed to NPL script via callback functions. 
- v2 uses NPL script to draw all the GUI objects using 3d-triangle drawing API and handles user input in NPL script. Therefore the user has complete control over the look of their control in NPL script. One can read the source code of v2 in main package. 


### Mcml tags
There are a number of tags which you can use to create interactive content. Besides, one can also create your own custom tags with UI controls, see [[System.Window]] for details. All mcml build-in tags are in the `pe:` xml namespace, which stands for ParaEngine or pe.

- [pe:div](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_div.lua)
- [pe:button](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_button.lua)
- [pe:container](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_container.lua)
- [pe:custom](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_custom.lua)
- [pe:editbox](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_editbox.lua)
- [pe:identicon](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_identicon.lua)
- [pe:if](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_if.lua)
- [pe:input](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_input.lua)
- [pe:repeat](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_repeat.lua)
- [pe:script](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_script.lua)
- [pe:span](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_span.lua)
- [pe:text](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/Elements/pe_text.lua)

Standard html tags are mapped as follows:
- The `div` tag is mapped to `pe:div` class.  
- plain text is mapped to `pe:text`. 
- `input type=button` is mapped to `pe:button`, etc. 

### Examples
There are many mcml examples where you browse in the paracraft package. Most of them are written in mcml v1, however the syntax should be the same for mcml v2. Click to [search paracraft package for mcml](https://github.com/NPLPackages/paracraft/search?q=mcml&type=Code&utf8=%E2%9C%93)

Following is just some quick examples:
```html
<pe:mcml>
<script type="text/npl" refresh="false">
<![CDATA[
globalVar = "test global var";
function OnButtonClick()
    _guihelper.MessageBox("refresh page now?", function()
        Page:SetValue("myBtn", "refreshed");
        Page:Refresh(0);
    end);
end

repeat_data = { {a=1}, {a=2} };
function GetDS()
    return repeat_data;
end
]]>
</script>
    <div align="right" style="background-color:#ff0000;margin:10px;padding:10px;min-width:400px;min-height:150px">
        <div align="right" style="padding:5px;background:url(Texture/Aries/Common/ThemeKid/btn_thick_hl_32bits.png:7 7 7 7)">
            <div style="float:left;background-color:#000000;color:#ffd800;font-size:20px;">
                hello world中文<font color="#ff0000">fontColor</font>
            </div>
            <div style="float:left;background-color:#0000ff;margin-left:10px;width:20px;height:20px;">
            </div>
        </div>
        <div valign="bottom">
            <div style="float:left;background-color:#00ff00;width:60px;height:20px;">
            </div>
            <div style="float:left;background-color:#0000ff;margin-left:10px;width:20px;height:20px;">
            </div>
        </div>
    </div>
    <div>
        hello world中文<font color="#ff0000">fontColor</font>
        <span color="#ff0000">fontColor</span>
    </div>
    <span color="#ff0000">
        <%=Eval('globalVar')%>
        <%=document.write('hello world')%>
    </span>
    <pe:container style="padding:5px;background-color:#ff0000">
        button:<input type="button" name="myBtn" value="RefreshPage" onclick="OnButtonClick" style=""/>
        editbox:<input type="text" name="myEditbox" value="" style="margin-left:2px;width:64px;height:25px"/><br/>
        pe_if: <pe:if condition='<%=globalVar=="test global var"%>'>true</pe:if>
    </pe:container>
    <pe:repeat value="item in repeat_data" style="float:left">
        <div style="float:left;"><%=item.a%></div>
    </pe:repeat>
    <pe:repeat value="item in GetDS()" style="float:left">
        <div style="float:left;"><%=item.a%></div>
    </pe:repeat>
    <pe:repeat DataSource='<%=GetDS()%>' style="float:left">
        <div style="float:left;"><%=a%></div>
    </pe:repeat>
</pe:mcml>
```

### MCML Layout
Please see [[System.Window]] first for basic alignment types. There are over 12 different alignment types, which supports automatic resizing of controls according to parent window size. 

- mcml v1 have a `<pe:container>` control which supports all alignment types via `alignment` attribute.
- mcml v1 and v2's `<div>` supports a limited set of alignment type via `align` and `valign` attribute, where you can align object to left or right.  Please note, due to mcml v1's one pass rendering algorithm, `right` alignment only works when parent width is fixed sized. mcml v2 does not have this limitation because it uses two pass algorithm. A workaround for mcml v1 is to use `<pe:container>` instead of `<div>` and use `alignment` for right alignment. 

#### Relative positioning
Relative position is a way to position controls without occupying any space. It can be specified with `style="position:relative;"`. If all controls in a container is relatively positioned, they can all be accurately positioned at desired location relative to their parent. 

However, relative position is NOT recommended, because a single change in child control size may result in changing all other child controls' position values and if parent control's size changes, the entire UI may be wrong. The recommended way of position is "float:left" or line based position. `<div>` defaults to line based position, in which it will automatically add `line break` at end. Specify "float:left" to make it float. Many other controls default to "float:left", such as `<input>`, `<span>`. Some UI designer in special product may still prefer relative position despite of this.


### MCML vs HTML
MCML only supports a very limited subset of HTML. The most useful layout method in mcml is `<div>` tag. 

### NPL Code Behind and Embedding
In MCML page, there are three ways to embed NPL code. 
- Use `<script src="helloworld.lua">`, this will load the npl file relative to the current page file. 
- Embed NPL code section inside `<script>` anywhere in the page file
- Embed code in xml attribute in quotations like this `attrname='<%= %>'`. Please note it depends on whether the tag implementation support it. For each attribute you can not mix code with text. Unlike in NPL web server page, all code in MCML are evaluated at runtime rather than preprocessed. This is very important. 

```html
<script type="text/npl" src="somefile.lua" refresh="false">
<![CDATA[
globalVar = "test global var";
function OnButtonClick()
    _guihelper.MessageBox("refresh page now?", function()
        Page:SetValue("myBtn", "refreshed");
        Page:Refresh(0);
    end);
end
repeat_data = { {a=1}, {a=2} };
function GetDS()
    return repeat_data;
end
]]>
</script>
<div style='<%=format("width:%dpx", 100) %>'></div>
```

#### Page Variable Scoping
Global variables or functions defined in inline script are local to the page and not visible outside the page.
So consider using a separate code behind file if one wants to expose values to external environment or simply your code is too long. 

There is a predefined variable called `Page`, which is accessible to the page's sandbox's environment. 

#### Two-Way Databinding
Many HTML developers have a `jquery` mindset, where they like to manipulate the DOM (Document Object Model or XML node) directly, which is **NOT** supported in mcml. Instead, one should have a `databinding mindset` like in `angularjs`. In MCML v1, `DOM != Controls`. DOM is only a static template for creating controls in a page. Controls can never modify static template unless you write explicit code, which is really rare. 

Most mcml tags only support one-way data binding from static DOM template to controls only at the time when controls are created (i.e. Page Refresh, see below). To read value back from controls, one must either listen to `onchange` event of individual control or manually call `Page:GetValue(name)` for a collection of controls in a callback function, such as when user pressed `OK` button. 

Two-way data-binding is useful but hard to implement in any system. Some HTML framework like `angularjs` does it by using a timer to periodically check watched data model and hook into every control's `onchange` event. One can simulate it manually in mcml. In mcml v2, DOM is almost equal to controls, hence it is relatively easy to implement two-way data binding in v2 than in v1. 

> Automatic two-way data-binding is not natively supported in both v1 and v2. But we are planning to support it in the near future. The following example shows how to do manual two-way binding in mcml.

```lua
<pe:mcml>
<p>Two-way databinding Example in mcml:</p>
<script type="text/npl" refresh="false"><![CDATA[
-- nodes are generated according to this data source object, ds is global in page scope so that it is shared between page refresh
ds = {
    block1 = nil,
    block2 = nil,
};

function asyncFillBlock1()
    PullData();
    -- use timer to simulate async API, such as fetching from a remote server.
    commonlib.TimerManager.SetTimeout(function()  
        -- simulate some random data from remote server
        ds.block1 = {};
        for i=1, math.random(2,8) do
            ds.block1[i] = {key="firstdata"..i, value=false};
        end
        ds.block2 = nil;
        ApplyData();
    end, 1000)
end

function asyncFillBlock2()
    PullData();
    commonlib.TimerManager.SetTimeout(function()  
        ds.block2 = {};

        -- count is based on the number of checked item in data1(block1)
        local data1_count = 0;
        for i, data in pairs(ds.block1) do
            data1_count = data1_count + (data.value and 1 or 0);
        end
        
        -- generate block2 according to current checked item in data1(block1)
        for i=1, data1_count do
            ds.block2[i] = {key="seconddata"..i, value=false};
        end
        ApplyData();
    end, 1000)
end

-- apply data source to dom
function ApplyData()
    Page("#block1"):ClearAllChildren();
    Page("#block2"):ClearAllChildren();

    if(ds.block1) then
        local parentNode = Page("#block1");
        for i, data in pairs(ds.block1) do
            local node = Page("<span />", {attr={style="margin:5px"}, data.key });
            parentNode:AddChild(node);
            local node = Page("<input />", {attr={type="checkbox", name=data.key, checked = data.value and "true"} });
            parentNode:AddChild(node);
        end
        local node = Page("<input />", {attr={type="button", value="FillBlock2", onclick = "asyncFillBlock2"} });
        parentNode:AddChild(node);
    end

    if(ds.block2) then
        local parentNode = Page("#block2");
        for i, data in pairs(ds.block2) do
            local node = Page("<span />", {attr={style="margin:5px"}, data.key });
            parentNode:AddChild(node);
            local node = Page("<input />",  {attr={type="text", EmptyText="enter text", name=data.key, value=data.value, style="width:80px;"} });
            parentNode:AddChild(node);
        end
    end

    Page:Refresh(0.01);
end

-- pull data from DOM
function PullData()
    if(ds.block1) then
        for i, data in pairs(ds.block1) do
            data.value = Page:GetValue(data.key);
        end
    end
    if(ds.block2) then
        for i, data in pairs(ds.block2) do
            data.value = Page("#"..data.key):value(); -- just another way of Page:GetValue()
        end
    end
end

function OnClickSubmit()
    -- TODO: one can also use a timer and hook to every onchange event of controls to automatically invoke Apply and Pull Data. 
    -- Here we just need to manually call it. 
    PullData();
    _guihelper.MessageBox(ds);
end
]]></script>
<div>
    <input type="button" value="FillBlock1" onclick="asyncFillBlock1()"/><br />
    block1: <div name="block1"></div>
    block2: <div name="block2"></div>
</div>
<input type="button" value="Submit" onclick="OnClickSubmit()" /><br />
</pe:mcml>
```

In above example, When user clicks `FillBlock1` button, some random data is filled in `ds.block1`, each data is displayed as checkbox, and the user can check any number of them and click the `FillBlock2` button. This time we will fill `ds.block2` with same number of checked items in `ds.block1`, and each item is displayed as a text input box. When user clicks `Submit` button, the current `ds` is displayed in a message box with user filled data. 

We call `ApplyData` to rebuild all related DOM whenever `ds` is modified externally, and we call `PullData` in the first line of each callback function so that our `ds` data is always up to date with mcml controls. The above example is called manual two-way binding because we have to call `ApplyData` and `PullData` manually. One can also use a timer and hook to every onchange event of controls to automatically invoke `ApplyData` and `PullData`, so that we do not have to write them everywhere in our callback functions. 

> There is an old two-way data binding code called [BindingContext](https://github.com/NPLPackages/main/blob/master/script/ide/DataBinding.lua), which is only used in `<form>` tag in mcml, it may cause some strange behaviors when you are manipulating DOM. 

#### Page Refresh
For dynamic pages, it is very common to change some variable in NPL script and refresh the page to reflect the changes. 

`<script refresh='false'>` means that the code section will not be reevaluated when page is refreshed. 
Also note that global variables or functions are shared between multiple script sections or multiple page refreshes.

Please also note that code changes in inline script blocks are usually reparsed when window is rebuild. However, if one use code behind model, npl file is cached and you need to restart your application to take effect. It is good practise to write in inline script first, and when code is stable move them to a code behind page. This will also make your page load faster during frequent page refresh or rebuild. 

### Writing Dynamic Pages
MCML has its own way of writing dynamic pages. In short, mcml uses frequent full `page refresh` to build responsive UI layout. Please note that each mcml page refresh does NOT rebuild (reparse) the entire DOM tree again, it simply reuses the old DOM tree and re-execute any embedded code on them each time. If the code is marked as `refresh="false"` (see last section), its last calculated value will be used.

Generally, MCML v1 uses a single pass to render the entire web page. MCML v2 uses two passes (click [here](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/PageElement.lua) to see details). 

Unlike HTML/Javascript(especially jquery) where programmers are encouraged to manipulate the document tree directly, in mcml, you are NOT advised to change the DOM with your own code (although we do provide a jquery like interface to its DOM). Instead, we encourage you to data-bind your code and create different code paths using conditional component like `<pe:if>`, when everything is set you do a full page refresh with `Page:refresh()`. This is a little bit like `AngularJS`, but we do it with real-time executed inline code. The difference is that `AngularJS` will compile all tags and convert everything from angular DOM to HTML DOM, but mcml just does one pass rendering of the mcml tags and execute the code as it converts DOM to real NPL controls. 

Another thing is that unlike in AngularJS where data changes are watched and page is automatically refreshed, in mcml, one has to manually call `Page:refresh(delayTimeSeconds)` to refresh the whole page, such as when user pressed a button to change the layout. Please note, changing the DOM does not automatically refresh the page, one still need to call the refresh function.  Mcml code logics is much simpler to understand and learn, while angularJS, although powerful, but with too many hidden dark-magic that programmers will never figure out on their own.

Finally, mcml page is NOT preprocessed into a second DOM tree before rendering like what NPL web server page or PHP does. Every tag in mcml is a real mcml control or [PageElement](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/PageElement.lua) and the DOM tree is persistent across page refresh. MCML uses a single pass to render all of them. This also makes full page `Page:refresh()` much faster than other technologies. Please note, mcml v2 has even faster page refresh than mcml v1, this is because in mcml v2, NPL controls are pure script objects, while in v1 they are ParaUIObject in ParaEngine. The former is much faster to create/destroy or even shared during page refresh depending on each PageElement's implementation.

#### MCML Controls
Remember mcml is meant to be replacement for tedious control creation in the whole application, not a stateless and isolated web page.  Each mcml page is an integral part of the application, they run in the same thread and have access to everything in that thread. They usually have direct interactions (or data-binding) with the 3D scene and other UI controls. To get the underlying control, one can use `Page:FindControl(name)`

#### MCML Page:Refresh(DelayTime)
`Page:Refresh(DelayTime)` refresh the entire page after DelayTime seconds. Please note that during the delay time period, 
if there is another call to this function with a longer delay time, the actual refresh page activation will be further delayed 
Note: This function is usually used with a design pattern when the MCML page contains asynchronous content such as pe:name, etc. whenever an asynchronous tag is created, it will first check if data for display is available at that moment. if yes, it will just display it as static content; if not, it will retrieve the data with a callback function. In the callback function, it calls this Refresh method of the associated page with a delay time. Hence, when the last delay time is reached, the page is rebuilt and the dynamic content will be accessible by then. 

- @param DelayTime: if nil, it will default to self.DefaultRefreshDelayTime (usually 1.5 second). 
- tip: If one set this to a negative value, it may causes an immediate page refresh. 


### Page Styling and Themes
MCML has limited support of inline css in `style` or `class` parameter. 
MCML v1 and v2 support style tag, in which we can define inline or file based css in the form of a standard NPL table.
Please note, the syntax is not `text/css`, but `text/mcss`. It is simply a table object with key name, value pairs. The key name can be tag class name or class name, compound names with `tag, id, class` filtering is not supported. 

Here is an example of defining styles with different class names, all mcml tags can have zero or more class names.
```xml
<pe:mcml>
<style type="text/mcss" src="script/ide/System/test/test_file_style.mcss">
{
    color_red = { color = "#ff0000" },
    color_blue = { color = "#0000ff", ["margin-left"] = 10 },
    bold = { ["font-weight"]="bold"    }
}
</style>
<span class="color_blue bold">css class</span>
</pe:mcml>
```

External css files are loaded and cached throughout the lifetime of the application process. Its content is also NPL table, such as `script/ide/System/test/test_file_style.mcss`:

```lua
-- Testing mcml css: script/ide/System/test/test_file_style.mcss
{
["default"] = { color="#ffffff" },
["mobile_button"] = {
	background = "Texture/Aries/Creator/Mobile/blocks_UI_32bits.png;1 1 34 34:12 12 12 12", 
},
color_red = { color = "#ff0000" },
color_blue = { color = "#0000ff", ["margin-left"] = 10, ["font-size"] = 16 },
bold = { ["font-weight"] = "bold"    },
["pe:button"] = {padding = 10, color="#00ff00"},
}
```
In most cases, we can style the object directly in style attributes using 9 tiled image file.  See next section.

To change the style of unthemed tags or buildin classes, one must progammatically modify the following table:
- In mcml v1, there is a global table like [this one](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries/Creator/Game/DefaultTheme.mc.lua)
- In mcml v2, there is a class called [StyleDefault](https://github.com/NPLPackages/main/tree/master/script/ide/System/Windows/mcml/StyleDefault.lua)


### 9-Tile Image
MCML supports a feature called image tiling. It can use a specified section of an image and divide the subsection into 9 images and automatically stretch the 5 middle images as the target resizes (4 images at the corner are unstretched).  This is very useful to create good looking buttons with various sizes.

The syntax is like this: 
`<div style="background:url(someimage_32bits.png#0 0 64 21: 10 10 10 10);" >`
- `#left top width height` is optional, which specifies a sub region. if ignored it means the entire image. 
- `:left top to_right to_bottom` means distance to left, top, right and bottom of the above sub region. 
- `_32bits.png` means that we should use 32bits color, otherwise we will compress the color to save video memory
- Image size must of order of 2, like 2,4,8,16,64, 128, 256, etc. Because `directX/openGL` only supports order of 2 images. If your image file is not order of 2, they will be stretched and become blurred when rendered. 

Please note, in css file or `<style>` tag, `#` can also be `;` when specifying a sub region.

### Setting Key Focus
For mcml v1, one can set key focus by following code. It only works for text box.  
```lua
local ctl = Page:FindControl(name);
if(ctl) then
   ctl:Focus();
end
```
For input text box, one can also use. 
```
<input type="text" autofocus="true"  ... />
```

But there can only be one such control in a window. If there are multiple ones, the last one get focus. For example
```xml
<pe:mcml>
    <div>
        Text: <input type="text" name="myAutoFocusEditBox" />
    </div>
    ... many other div here...
    <!-- This should be the last in the page -->
    <script type="text/npl" refresh="true">
        local ctl = Page("#myAutoFocusEditBox"):GetControl()
        ctl:Focus();
        ctl:SetCaretPosition(-1);
    </script>
</pe:mcml>
```

### Conclusion
MCML v2 implementation is not very complete yet.  You are welcome to contribute your own tags or controls to our main package. 

### FAQ 
> Why large font size appears to be blurred in mcml?

MCML uses 3D API to do all the graphical drawing. For each font size, an image must be loaded into video memory. Thus by default, mcml may scale an existing font to prevent creating unnecessary image. To prevent this behavior, one can use `style="base-font-size:30;font-size:30"` this will always create a base font image for your current font setting. In general, an application should limit the number of font settings used in your graphical application. 


#### References
##### Documentation For MCML V1
- [mcml v1 Source Code](https://github.com/NPLPackages/paracraft/tree/master/script/kids/3DMapSystemApp/mcml), such as:
   - [pe_html](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_html.lua)
   - [pe_editor](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_editor.lua)
   - [pe_html_input](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_html_input.lua)
   - [pe_gridview](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_gridview.lua): [more examples](pe_gridview_doc)
   - [pe_treeview](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_treeview.lua)
   - [pe_design](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_design.lua)
   - [pe_component](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_component.lua)
   - [pe_script](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/pe_script.lua)
- [Test cases(Examples) for mcml v1](https://github.com/NPLPackages/paracraft/tree/master/script/kids/3DMapSystemApp/mcml/test)
   - [test_pe_gridview](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/test/test_pe_gridview.html)

> More over, many existing GUI of paracraft are actually created with mcml v1. Search the source code for any HTML pages for examples.
