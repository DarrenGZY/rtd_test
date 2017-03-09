# 3D Scene 
There are a number of 3D objects which one can create via scripting API, such as mesh, physics mesh, sky box, camera, biped character, light, particles, containers, overlays, height map terrain, blocks, animations, shaders, 3D assets, etc.


## Triangle Limitations
One can set the maximum number of total triangles of animated characters in the 3d scene per frame, like below
```lua
ParaScene.GetAttributeObject():SetField("MaxCharTriangles", 150000);
```
Faraway objects exceeding this value will be skipped during rendering. Another way of reducing triangle count is via level-of-detail (LOD) when loading mesh or characters. Currently, one must manually provide all LOD of mesh when loading a 3d file. 

## Rendering Pipeline
Currently all fixed function, shaders and deferred shading share the same predefined rendering pipelines. The render order can be partially affected by `RenderImportance`, `RenderOrder`, several `Shader files` property. But in general, it is a fixed general purpose rendering pipeline suitable in most situations. 

The source code of the pipeline is hard-coded in `SceneObject.cpp's AdvanceScene()` function. 

Here is what it does:

For each active viewport, we do the following:
- traverse the scene and quad-tree tile manager and call `PrepareRender()` for each visible scene object, which will insert the object into a number of predefined global render queues. 
- `PIPELINE_3D_SCENE`
   - draw `MiniSceneGraph` with local cameras and render into textures
   - draw owner-draw objects like render targets.
   - draw all m_mirrorSurfaces into local textures
   - tesselate `global terrain` according to current camera
   - render shadow map for shadow casters queue, and other global object like blocks that cast shadows. 
   - draw block engine multiframe world texture
   - render `global terrain`
   - draw opaque blocks in block engine
   - draw alpha-test enabled blocks in block engine
   - draw static meshes in big mesh queue from front to back
   - draw static meshes in small mesh queue from back to front
   - draw sprite objects queue
   - draw animated characters queue
   - render selection queue
   - render current sky object
   - draw block engine multiframe world texture on sky
   - draw missile object queue
   - draw block engine water reflection pass
   - block engine deferred shading post processing for opaque objects
   - draw batched transparent particles
   - block engine draw all alpha blended blocks
   - draw transparent animated characters in transparent biped queue
   - block engine deferred shading post processing for alpha blended object
   - draw block engine deferred lights
   - draw all head-on display
   - draw simulated global ocean surface
   - draw transparent static mesh objects in transparent mesh queue
   - draw transparent triangle face groups
   - draw objects in `post render queue`
   - draw particle systems
   - invoke custom post rendering shaders in NPL script for 3d scene
   - draw water waves
   - draw helpers for physics world debugging 
   - draw helpers for bounding boxes of scene objects
   - draw portal system
   - draw overlays
- `PIPELINE_UI`
   - render all visible GUI objects by traversing the `GUIRoot` object. 
- `PIPELINE_POST_UI_3D_SCENE`
   - only `MiniSceneGraph` whose `GetRenderPipelineOrder() == PIPELINE_POST_UI_3D_SCENE` will be rendered in this pipeline
- `PIPELINE_COLOR_PICKING`
   
### Pipeline customization
- `RenderImportance` property of ParaObject only affects render order in a given queue. 
- `RenderOrder` property of ParaObject will affect which queue the object goes into, as well as the order inside the queue. However, only `RenderOrder>100` is used to insert objects to `post render queue`. 

For example, the following code will ensure the two objects are rendered last and ztest is disabled. 
```lua
	local asset = ParaAsset.LoadStaticMesh("","model/common/editor/z.x")
	local obj = ParaScene.CreateMeshPhysicsObject("blueprint_center", asset, 1,1,1, false, "1,0,0,0,1,0,0,0,1,0,0,0");
	obj:SetPosition(ParaScene.GetPlayer():GetPosition());
	obj:SetField("progress",1);
	obj:GetEffectParamBlock():SetBoolean("ztest", false);
	obj:SetField("RenderOrder", 101)
	ParaScene.Attach(obj);

	local player = ParaScene.CreateCharacter ("MyPlayer1", ParaAsset.LoadParaX("","character/v3/Elf/Female/ElfFemale.x"), "", true, 0.35, 0, 1.0);
	local x,y,z = ParaScene.GetPlayer():GetPosition()
	player:SetPosition(x+1,y,z);
	player:SetField("RenderOrder", 100)
	player:GetEffectParamBlock():SetBoolean("ztest", false);
	ParaScene.Attach(player);
```

