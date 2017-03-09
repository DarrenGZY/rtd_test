# NPL C/C++ Architecture
This section covers cross-platform modules written in C/C++. 
These modules are exposed via [NPL scripting API](https://codedocs.xyz/LiXizhi/NPLRuntime/) so that they are called via NPL.

## C++ Modules
### NPL Scripting Engine:
   * NPL state (or NPL virtual code environment): a single NPL thread, has its own memory allocators and manages all files it load.
     * NPL state can load NPL/Lua script or C++ dll.
     * Mono state: can load C# dll. 
   * NPL Networking: Manage all local or remote NPL states via NPL HTTP/TCP connections.
### Graphics Engine:
All GUI objects must be created in the main NPL thread, which is the same as renderer thread.
2D/3D Engine are all object oriented. All 2D objects are origanized in a parent/child tree. 
3D objects are organized in a quad-tree in additional to parent/child tree. 

#### Video/audio Renderer
A static/dynamic buffer, textures, draw 3d/2d api, fonts, etc. 
* DirectX fully supported renderer. 
* OpenGL renderer: a cross-platform renderer(limited functions used in our linux/android build)

#### PaintEngine
It is like a GDI engine used by 2D Engine for drawing 2D and simple 3d objects, like lines, rectangles, etc.
#### 2D Engine: GUIBase is the base class to all 2d object.
* GUIRoot: root node of all GUI objects.
* GUIContainer, GUIButton, GUIText, etc
#### 3D Engine: BaseObject is the base class to all 3D object.
* ViewportManager: manages 2D viewport into which we can render 3d or 2d objects.
* SceneObject: root node of all 3D objects. It manages all objects like adding, deleting, searching, rendering, physics, etc
* TerrainTileRoot: it is a `quadtree` container of all 3D objects for fast object searching according to their 3d locations and current camera frustum. 
* CameraObject: camera frustum. several derived class like AutoCamera, etc
* MeshObject: base class static triangle mesh objects in the 3d object. 
* MeshPhysicsObject: it is a static mesh with physics.
* BipedObject: it represents an animated object.
* MinisceneGraph: it is a simplified version of SceneObject, which usually manages a small number of 3d objects which can be rendered separately in to a 2D texture. 
* TerrainEngine: infinitely large terrain with a heightmap, and multiple texture layers. It is rendered with dynamic level-of-detail algorithm.
* Other Scene objects:
* [[BlockEngine]]: it manages rendering of `32000x32000x256` blocks. 
   * BlockRegion: manages `512x512x256` blocks, which are saved into a single file
   * ChunkColumn: `16x16x256`
   * Chunk: `16x16x16`, a static renderable object.

### Physics Engine
It uses the open source Bullet physics engine for static mesh objects in the scene. 
Block physics is handled separately by the [[BlockEngine]] itself. 

### Asset Management:
Usually each static asset file is a asset entity. `AssetEntity` is the based class to all assets in the system. Assets provides data and sometimes rendering methods to be used any other 2d/3d objects. 
All assets are loaded asynchronously by default from either IO disk or remote network. 
  * TextureEntity: 2d textures
  * MeshEntity: static mesh file like `.x`
  * ParaXEntity: animated mesh file which can be loaded from `.x`, `.fbx`
  * Audio, bmax model, database, font, etc. 
  * EffectFile: shader files, which can be loaded from directX `.fx`.

### IO and Util
  * FileManager: manages all files and search paths.
  * CParaFile: manages a single file read/write.
  * math: 2d/3d math libs like vectors and matrices

### ParaScripting API: 
   * C++ --> NPL: exposing all above functions and modules to NPL scripting environment. 
   * See all C++ bindings in [NPL scripting reference](https://codedocs.xyz/LiXizhi/NPLRuntime/)
