## Useful Code Snippets in Paracraft

### Paracraft Filters
[Filters](FiltersAndEvents) is the recommended way for plugin/app developers to extend or modify paracraft. 

> [Click here to see all Paracraft Filters](https://github.com/NPLPackages/paracraft/blob/master/script/apps/Aries/Creator/Game/ParacraftFilters.md)

Mod interface also use filters to implement all plugin interface, therefore [Mod Manager](https://github.com/NPLPackages/paracraft/blob/master/script/apps/Aries/Creator/Game/Mod/ModManager.lua) is an example of how to add filters to various integration points in paracraft. To extend paracraft, one can either use filters or overwrite virtual functions on mod interface. It is therefore possible for an application to add high priority filters to disable/modify data after all mod interface filters. 

> It is good practice to install filters only once at the very beginning. A good place to install filters is before `Game.Start` is called or in the `mod:init` function. One can also dynamically add/remove filters at runtime, just be sure to add your filters before they are applied. 

### What Is the Filter To Use For World Login
It is not trivial to define a filter (event) when a world is fully loaded, because everything is asynchronously loaded.
The following filters are applied at various stage of world loading:
- "ActivateDesktop", bIgnoreDefaultDesktop, mode: called when desktop mode is changed. This is the place to initialize your custom GUI. 
- "handleLogin", packet_login: whenever client received confirmed login packet from server. (only called in remote world) 
- **"PlayerHasLoginPosition", nil, x,y,z**: called whenever the player is at its spawn position in both local or remote world.This filter is most close to when a world is fully loaded. 

> One may start a one-time timer in any of these events to check if the world is fully loaded. If so, stop the timer and do whatever you want. 

### Register a New Block
Please see [Test Sample](https://github.com/LiXizhi/ParaCraftSDK/blob/master/_mod/Test/Mod/Test/DemoItem.lua) in ParacraftSDK's `_mod` folder.

```lua
-- register a new block item, id < 512 is internal blocks, which is not recommended to modify. 
GameLogic.GetFilters():add_filter("block_types", function(xmlRoot) 
	local blocks = commonlib.XPath.selectNode(xmlRoot, "/blocks/");
	if(blocks) then
		blocks[#blocks+1] = {name="block", attr={
			id = 512, threeSideTex = "true",
			text = "Demo Item", name = "DemoItem",
			texture="Texture/blocks/bookshelf_three.png",
			obstruction="true", solid="true", cubeMode="true",
		}}
		LOG.std(nil, "info", "DemoItem", "a new block is registered");
	end
	return xmlRoot;
end)
```

### Run Command

```lua
GameLogic.RunCommand("/startserver");
```

> If you are inside a 3D world, one can call script using the `SpawnPoint` item.


### Create Win32 TitleBar
Sometimes, we need to hide the native win32 title bar and create our owner-draw title bar.

![image](https://cloud.githubusercontent.com/assets/94537/19351410/3f286224-918e-11e6-97d8-c2dbc06deb68.png)

The following mcml v1 code will do the job. see also [test code](https://github.com/NPLPackages/paracraft/blob/master/script/kids/3DMapSystemApp/mcml/test/test_window_title_bar.html)

```html
<pe:mcml>
<pe:container name="titleBar" alignment="_mt" style="width:0px;height:64px;background:url(Texture/alphadot.png)" >
    <div style="position:relative;float:left;margin:5px;">drag this title bar to move the native win32 window</div>
    <pe:container name="winButtons" alignment="_rt" style="position:relative;padding:5px;margin:5px;width:100px;background:url(Texture/whitedot.png)">
        <input type="button" name="toggleBtn" tooltip="press Z key to toggle" onclick="toggleTitleBar" value="ToggleTitle" class="button"/>
    </pe:container>
    <pe:hotkey name="2" hotkey="DIK_Z" onclick="toggleTitleBar" />
</pe:container>
<%
-- This line will allow the user to drag this container to move the window.
Page:FindControl("titleBar"):SetField("EnableClientTest", true);

function toggleTitleBar()
    local bShowTitle = ParaEngine.GetAttributeObject():GetField("ShowWindowTitleBar", true)
    ParaEngine.GetAttributeObject():SetField("ShowWindowTitleBar", not bShowTitle);
end
%>
</pe:mcml>
``` 

### "DefaultContext" filter
Can be used to replace the default scene context in paracraft via paracraft mod interface. 

```lua
NPL.load("(gl)script/ide/System/Core/SceneContext.lua");
local MySceneContext = commonlib.inherit(commonlib.gettable("System.Core.SceneContext"), commonlib.gettable("System.Core.MySceneContext"));
function MySceneContext:ctor()
	self:EnableAutoCamera(true);
end

function MySceneContext:mouseReleaseEvent(event)
	_guihelper.MessageBox("clicked")
end

-- when mod is loaded, register the filter
GameLogic.GetFilters():remove_all_filters("DefaultContext")
GameLogic.GetFilters():add_filter("DefaultContext", function(context)
   return MySceneContext:CreateGetInstance("MyDefaultSceneContext");
end);
```

> see also: [TestMod](https://github.com/LiXizhi/ParaCraftSDK/blob/master/_mod/Test/Mod/Test/DemoSceneContext.lua) for a working sample.


### Client/Server GUI

In many occasions, one needs to click a button in client side world to display some GUI and send result to server. But it is not trivial to implement. 
The logic is very similar to traditional web browser. But one can think of paracraft as a `3D web browser and server`. The server side database includes all blocks, NPCs, timelines, etc, which are shared (automatically synchronized) by all clients. However, each connected client has its own unique 2d/3d view into the server side database. For example, the changing of player positions automatically generate requests to fetch shared data from the server, so it is with GUI pages, except that the process is not automatic.  

The process of CS GUI request is like below:
- A client make a request to the server, such as clicking a button in 3d world, which will cause server side script to execute.
- The server side script may send back the proper GUI page content or url to display such as via `/runat @p` command to just the requested client.
- The client renders the GUI page to collect information from the user and send result back to server possibly via `/runat @admin` or other commands or messages.
- The server updated its database, such as changing blocks, modifying NPC positions, and updating data properties, etc. which will automatically be synchronized with other connected clients.

> Tips: 

  - To send a message from client to server: `/runat @admin /any_command_here`
  - To send a message from server to client: `/runat @p /any_command_here`

> Example 1: Simple C/S GUI

![image](https://cloud.githubusercontent.com/assets/94537/19636519/5d44e022-99fc-11e6-8ca1-442e6e6983ce.png)
 
Put `/runat @p /tip hi, username` inside a command block on server side. It will display a simple tip GUI whenever a client user clicks the button. 

Alternatively, one can also put `/runat @p /open test.html` inside a command block on server side. It will display any mcml GUI page on the client computer who triggers the button. `/open test.html` can [open mcml GUI page](https://github.com/LiXizhi/ParaCraft/wiki/cmd_open) relative to world or root directory. 

> Example 2: Custom C/S GUI

Custom GUI code must be pre-installed on both client and server, such as via a paracraft mod interface. 
The following code must be run on both client and server. 
```lua
-- register a new show GUI command 
GameLogic.GetFilters():add_filter("show", function(name, bIsShow) 
	if(name=="MyApp.TestCSGUI") then
               _guihelper.MessageBox("hello from custom gui", function()
                       GameLogic.RunCommand("/runat @admin /tip client pressed Yes");
               end)
	end
end)
```
Then put `/runat @p /show MyApp.TestCSGUI` inside a command block on server side as in previous example. It will display a message box GUI whenever a client user clicks the button, and if the user pressed OK, it will send a message back to the server. 

> Note: For security reasons, only the admin user can edit command blocks, some simple blocks like Sign and image block can be edited by both client and admin user. Changes to blocks are automatically applied to all connected client. 

## Chunk Generators

There can be many custom chunk providers deriving from [this base class](https://github.com/NPLPackages/paracraft/blob/master/script/apps/Aries/Creator/Game/World/ChunkGenerator.lua). 

```
Virtual functions:
	Init(world, seed)
	GenerateChunkImp(chunk, x, z, external)
	PostGenerateChunkImp(chunk, x, z)
	OnExit()

	IsSupportAsyncMode
	GetClassAddress
	GenerateChunkAsyncImp
```

#### Synchronous generator
By default generator runs in the main thread, so that your generator can safely use all API. 
Simply derive from `ChunkGenerator` and overwrite `GenerateChunkImp`, and register the provider by name. 

> see [`FlatChunkGenerator.lua`](https://github.com/NPLPackages/paracraft/blob/master/script/apps/Aries/Creator/Game/World/generators/FlatChunkGenerator.lua) for example. 

#### Asynchronous generator
Some generators may take a long time to compute and generate large number of blocks. 
It becomes impossible to run it in the main thread without dropping render frame rates. 
In such occasions, one needs to use asynchronous generator, which runs in one or more worker threads. 
Actually, you can turn your synchronous generator into asynchronous ones by simply 
overwriting `IsSupportAsyncMode` and `GetClassAddress` method. 

If `IsSupportAsyncMode` returns true, the chunk provider will send each chunk request 
to one of the worker threads for processing (see `SetWorkerThreadCount` function). 
The worker thread picks up the request and recreate the chunk provider instance 
based on `GetClassAddress` in that thread and pass a fake in-memory `Chunk` object (see also Chunk.lua)
to its `GenerateChunkImp` function. Finally it sends back the data in `Chunk` object in compressed
binary format to the main thread, which will apply the chunk data to the real `ChunkCpp` object (see also ChunkCpp.lua) in the main thread. 
BTW, applying all chunk data to chunk column in one API may be one-hundred times faster than setting individual blocks in the chunk. 

However, there is limitations of asynchronous generator. The `GenerateChunkImp` should only 
set or get chunk data based on the requested chunk only. It can not query or set entities or advanced information 
that is only available in the main thread. However, one can overwrite `PostGenerateChunkImp` to generate 
some add-on blocks/entities in the main thread, because `PostGenerateChunkImp` is always called for both async and sync mode in the main thread.

> See `NatureV1ChunkGenerator` generator for example. 

### Programmatically Create Blocks
Remember everything in paracraft originates from an `item`. 
So strictly speaking, one needs to call `item:TryCreate` to create an item in Paracraft. 

However, for very simple block based items, we offer other easier ways like command [`/setblock`](https://github.com/LiXizhi/ParaCraft/wiki/cmd_setblock) or `BlockEngine:SetBlock(x,y,z,block_id, block_data, flag, entity_data)`. 

### Change Default Player Appearance
One can change the main player appearance after world is loaded using the `Entity:SetMainAssetPath` function or `/avatar` command. 

Another way is to change the `block_types.xml` of the corresponding item's model configuration, like below. 
```xml
<block id="30002" name="player" text="主角" searchkey="" tooltip="" item_class="ItemNPC" icon="Texture/blocks/items/skull_steve.png" hp="100" respawn_time="300000">
    <model assetfile="character/CC/01char/char_male.x" skin="Texture/blocks/human/boy_blue_shirt01.png" ></model>
  </block>
```
One can do this via `filters` when your application starts. Both `x` and `fbx` files are supported for assetfile.

