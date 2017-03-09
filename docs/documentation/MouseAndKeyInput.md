# Mouse and Key Input
This section is about handling mouse and key events in a window application. 

## Low level event handler
At the lowest level of NPL, mouse and keyboard event handlers can be registered to `ParaUIObject` and the global `ParaScene`. The ParaScene is a global singleton to handle all events not handled by any GUI objects. 
Please note, there is a another `AutoCameraController` which is enabled by default to handle mouse and key events before passing to ParaScene. So the order of event filtering in NPLRuntime is like below
- GUI events: controls that have focus always get event first
- `AutoCameraController`: optionally enabled for handling basic `player control`, such as right click to rotate the view, arrow keys to move the main player. Please note, this can be disabled if one wants to handle everything in NPL script. Trust me, it can cost you over 2000 lines of code if you do it manually. 
- ParaScene: finally mouse and key events are handled by the 3d scene. 

It is NOT recommended to use low level NPL api directly, see next section.

## Use NPL libraries to handle event

### 2D events
For 2D GUI events, one can handle via window controls or MCML page.
There is also a mcml v1 tag called `<pe:hotkey>` for handling simple key event when window is visible. 

### 3D Scene Context
For 3D or global mouse/key, one should use [SceneContext](https://github.com/NPLPackages/main/blob/master/script/ide/System/Core/SceneContext.lua).

At most one scene context can be selected at any time. Once selected, the scene context 
will receive all key/mouse events in the 3D scene. One can derive from this class to 
write and switch to your own scene event handlers. 

The computational model for global key/mouse event in C++ Engine is:
```
	key/mouse input--> 2D --> (optional auto camera) --> 3D scene --> script handlers
```
This class simplified and modified above model with following object oriented model:
```
	key/mouse input--> 2D --> one of SceneContext object--> Manipulator container -->  Manipulators --> (optional auto camera manipulator)
```
Please note that both model can coexist, however SceneContext is now the recommended way to handle any scene event. 
SceneContext hide all dirty work of hooking into the old C++ engine callback interface, and offers more user friendly way 
of event handling. 

Virtual functions:
```
	mousePressEvent(event)
	mouseMoveEvent
	mouseReleaseEvent
	mouseWheelEvent
	keyReleaseEvent
	keyPressEvent
	OnSelect()
	OnUnselect()
```
use the lib:

```lua
NPL.load("(gl)script/ide/System/Core/SceneContext.lua");
local MySceneContext = commonlib.inherit(commonlib.gettable("System.Core.SceneContext"), commonlib.gettable("System.Core.MySceneContext"));
function MySceneContext:ctor()
	self:EnableAutoCamera(true);
end

function MySceneContext:mouseReleaseEvent(event)
	_guihelper.MessageBox("clicked")
end

-- method 1:
local sContext = MySceneContext:new():Register("MyDefaultSceneContext");
sContext:activate();
-- method 2:
MySceneContext:CreateGetInstance("MyDefaultSceneContext"):activate();
```
#### Scene Context Switch
Context is usually associated with a tool item, such as when user click to use the tool, its context is activated and once user deselect it, the context is switched back to a default one. 

Scene context allows one to write modular code for each tool items with complex mouse/key input without affecting each other. It is common for an application to derive all of its context from a base context to support shared global key events, etc. 

For an example of using context, please see Paracraft's [context](https://github.com/NPLPackages/paracraft/blob/master/script/apps/Aries/Creator/Game/SceneContext) folder. 

> See also [here](ParacraftSnippets) for how to replace the default context in paracraft's mod interface with filters.

### Manipulators
Scene context can contain manipulators. [Manipulator](https://github.com/NPLPackages/main/blob/master/script/ide/System/Scene/Manipulators/Manipulator.lua) is a module to manipulate complex scene objects, like the one below.

![image](https://cloud.githubusercontent.com/assets/94537/19374481/74358f6c-91fe-11e6-9141-b1abde786d28.png)

Manipulator is the base class used for creating user-defined manipulators.
A manipulator can be connected to a depend node instead of updating a node attribute directly
call AddValue() in constructor if one wants to define a custom manipulator property(plug) 
that can be easily binded with dependent node's plug. 

Manipulator can also draw complex 3D overlay objects with 3d picking support. In above picture, the three curves are drawn by the [Rotate manipulator](https://github.com/NPLPackages/main/blob/master/script/ide/System/Scene/Manipulators/RotateManip.lua).