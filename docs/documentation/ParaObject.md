# ParaObject
ParaObject is the scripting proxy to a 3D object on the C++ engine. In most cases, it could be a 3d mesh, an animated character called biped, a bmax model or any custom 3D objects. 

## Create 3D Object

Use `CreateCharacter` to create animated character based on a ParaX asset file. 
```lua
local player = ParaScene.CreateCharacter ("MyPlayer", ParaAsset.LoadParaX("","character/v3/Elf/Female/ElfFemale.x"), "", true, 0.35, 0, 1.0);
player:SetPosition(ParaScene.GetPlayer():GetPosition());
ParaScene.Attach(player);
```

Use `CreateMeshPhysicsObject` to create a static mesh object based on a mesh asset file
```lua
local asset = ParaAsset.LoadStaticMesh("","model/common/editor/z.x")
local obj = ParaScene.CreateMeshPhysicsObject("blueprint_center", asset, 1,1,1, false, "1,0,0,0,1,0,0,0,1,0,0,0");
obj:SetPosition(ParaScene.GetPlayer():GetPosition());
obj:GetAttributeObject():SetField("progress",1);
ParaScene.Attach(obj);
```

## Get View Parameters
> Please note all asset are async-loaded, when asset is not loaded, object renders nothing, and following parameters may not be correct when associated mesh asset is not async-loaded.
 
```lua
local obj = ParaScene.GetObject("MyPlayer")
local params = {};
param.rotation = obj:GetRotation({})
param.scaling = obj:GetScale();
param.facing = obj:GetFacing();	
param.ViewBox = obj:GetViewBox({});
local x,y,z = obj:GetViewCenter(); 
```

## References:
More Complete API reference, please see [ParaObject Reference](https://codedocs.xyz/LiXizhi/NPLRuntime/classParaScripting_1_1ParaObject.html)
