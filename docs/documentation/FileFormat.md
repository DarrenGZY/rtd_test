# File Format
ParaEngine/NPL support several build-in file format. Some is used for 3d models, some for 3d world

## 3D Model file format

ParaEngine supports following built-in or standard file format. 
- ParaX format (*.x): This is our build-in file format for advanced animated characters, particles, etc. One needs to download and install ParaEngine exporter plugin for 3dsmax 9 (32bits/64bits). However, we are not supporting the latest version of 3dsmax. Please consider using FBX file format.
- FBX format (`*.fbx`): FBX is the universal file format for AutoDesk product like MAYA/3dsmax. It is one of the most supported lossless file format in the industry. Click [here for how to use FPX](https://github.com/LiXizhi/ParaCraftSDK/wiki/FBXModel). (Please note model size and embedded texture matters)

> we only support FBX version 2013 or above. Version 2014, 2015 are tested. 

- BMAX (*.bmax): This is our built-in block max file format, which is used exclusively by Paracraft application for storing static and animated blocks. 
- STL format (*.stl): This is a file format used for 3d printing.

### BMax file format
BMAX is short for Block Max, it is a file format used exclusively in Paracraft for storing and exchanging block data. 
In paracraft, the world is made up of blocks, each block may contain `x,y,z, block_id, block_data, custom_data`
- `x,y,z` is block position
- `block_id` is type id of the block, see [block_types.xml](https://github.com/NPLPackages/paracraft/blob/master/config/Aries/creator/block_types.xml) for a complete list of block ids. 
- `block_data` is a 32bits data of the block, it usually denotes the orientation or type of the block. 
- `custom_data` can be any NPL table object that stores additional data of the block. Most static blocks does not contain custom_data, however, blocks like movie block, command blocks will save animation data and commands in this place. 

You can select some blocks in paracraft and save them to bmax file to examine a real bmax file, it should be self-explanatory. Following is an example bmax file(containing a button, 2 movie blocks, a repeater and a wire):

![image](https://cloud.githubusercontent.com/assets/94537/21293073/901298a4-c554-11e6-8967-14fbb914ff55.png)

```xml

<pe:blocktemplate>
	<pe:blocks>{
{-2,0,-3,105,5,},

{-2,0,-2,228,[6]={{name="cmd","/t 6 /end",},{{"{timeseries={lookat_z={times={0,5348,},data={20001.56125,20004.65625,},ranges={{1,2,},},type=\"Linear\",name=\"lookat_z\",},eye_liftup={times={0,5348,},data={0.28568,0.26068,},ranges={{1,2,},},type=\"Linear\",name=\"eye_liftup\",},lookat_x={times={0,5348,},data={20000.51959,20000.58203,},ranges={{1,2,},},type=\"Linear\",name=\"lookat_x\",},eye_rot_y={times={0,5348,},data={-3.11606,-3.11668,},ranges={{1,2,},},type=\"LinearAngle\",name=\"eye_rot_y\",},is_fps={times={0,5348,},data={0,0,},ranges={{1,2,},},type=\"Discrete\",name=\"is_fps\",},lookat_y={times={0,5348,},data={-127.08333,-127.08333,},ranges={{1,2,},},type=\"Linear\",name=\"lookat_y\",},eye_dist={times={0,5348,},data={8,8,},ranges={{1,2,},},type=\"Linear\",name=\"eye_dist\",},has_collision={times={0,5348,},data={1,1,},ranges={{1,2,},},type=\"Discrete\",name=\"has_collision\",},eye_roll={times={0,5348,},data={0,0,},ranges={{1,2,},},type=\"LinearAngle\",name=\"eye_roll\",},},}",name="slot",attr={count=1,id=10061,},},{"{timeseries={time={times={},data={},ranges={},type=\"Linear\",name=\"time\",},music={times={},data={},ranges={},type=\"Discrete\",name=\"music\",},tip={times={},data={},ranges={},type=\"Discrete\",name=\"tip\",},movieblock={times={},data={},ranges={},type=\"Discrete\",name=\"movieblock\",},cmd={times={},data={},ranges={},type=\"Discrete\",name=\"cmd\",},blocks={times={},data={},ranges={},type=\"Discrete\",name=\"blocks\",},text={times={},data={},ranges={},type=\"Discrete\",name=\"text\",},},}",name="slot",attr={count=1,id=10063,},},name="inventory",{"{timeseries={blockinhand={times={},data={},ranges={},type=\"Discrete\",name=\"blockinhand\",},x={times={0,},data={20000.51959,},ranges={{1,1,},},type=\"Linear\",name=\"x\",},pitch={times={},data={},ranges={},type=\"LinearAngle\",name=\"pitch\",},y={times={0,},data={-127.08333,},ranges={{1,1,},},type=\"Linear\",name=\"y\",},parent={times={},data={},ranges={},type=\"LinearTable\",name=\"parent\",},roll={times={},data={},ranges={},type=\"LinearAngle\",name=\"roll\",},block={times={},data={},ranges={},type=\"Discrete\",name=\"block\",},scaling={times={},data={},ranges={},type=\"Linear\",name=\"scaling\",},gravity={times={},data={},ranges={},type=\"Discrete\",name=\"gravity\",},HeadUpdownAngle={times={},data={},ranges={},type=\"Linear\",name=\"HeadUpdownAngle\",},anim={times={},data={},ranges={},type=\"Discrete\",name=\"anim\",},bones={R_Forearm_rot={times={0,34,1836,},data={{0.00024,0.00175,-0.01261,0.99992,},{0.00024,0.00175,-0.01261,0.99992,},{0.00022,-0.05946,0.42959,0.90107,},},ranges={{1,3,},},type=\"Discrete\",name=\"R_Forearm_rot\",},R_UpperArm_rot={times={0,34,1836,},data={{-0.01933,-0.00286,0.03036,0.99935,},{-0.01802,-0.03834,0.32796,0.94375,},{-0.0137,-0.01077,0.14324,0.98954,},},ranges={{1,3,},},type=\"Discrete\",name=\"R_UpperArm_rot\",},isContainer=true,},speedscale={times={},data={},ranges={},type=\"Discrete\",name=\"speedscale\",},assetfile={times={0,},data={\"actor\",},ranges={{1,1,},},type=\"Discrete\",name=\"assetfile\",},skin={times={},data={},ranges={},type=\"Discrete\",name=\"skin\",},z={times={0,},data={20001.56125,},ranges={{1,1,},},type=\"Linear\",name=\"z\",},facing={times={},data={},ranges={},type=\"LinearAngle\",name=\"facing\",},HeadTurningAngle={times={},data={},ranges={},type=\"Linear\",name=\"HeadTurningAngle\",},name={times={0,},data={\"actor3\",},ranges={{1,1,},},type=\"Discrete\",name=\"name\",},opacity={times={},data={},ranges={},type=\"Linear\",name=\"opacity\",},},tooltip=\"actor3\",}",name="slot",attr={count=1,id=10062,},},},name="entity",attr={bz=19201,bx=19200,class="EntityMovieClip",item_id=228,by=5,},},},

{-2,0,-1,197,3,},
{-2,0,0,189,},
{-2,0,1,228,[6]={{name="cmd","/t 30 /end",},{name="inventory",{name="slot",attr={count=1,id=10061,},},},name="entity",attr={bz=19204,bx=19200,class="EntityMovieClip",item_id=228,by=5,},},},}

	</pe:blocks>
</pe:blocktemplate>
```

For user manual of using movie blocks, please see the courses in
- http://www.paracraft.cn/learn/movieblockcourses?lang=zh

## ParaX File format
ParaX is binary file format used exclusively in NPL Runtime for animated 3D character asset.  
ParaX is like a concise version of FBX file (FBX is like BMAX file used by autodesk 3dsmax/maya, ...)
NPL Runtime also support reading FBX file and load as ParaXModel object in C++, for user guide see [FBX](https://github.com/LiXizhi/ParaCraftSDK/wiki/FBXModel)

- BMAX to ParaX converter in C++ (Lacking support for animation data in movie block)： https://github.com/LiXizhi/NPLRuntime/tree/master/Client/trunk/ParaEngineClient/BMaxModel
- ParaX models: 
  - https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/ParaXModel/XFileCharModelParser.h
  - https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/3dengine/ParaXSerializer.h
  - https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/3dengine/ParaXSerializer.h

#### BMAX movie blocks to ParaX File format Conversion Details
- locate all movie blocks in BMAX file and ignore all other blocks. Movie blocks needs to be in the same order of 3D space (as repeater and wires indicates)
- The first movie block is always animation id 0 (idle animation)
- The second movie block is always animation id 4 (walk animation)
- The animation id of the third or following movie blocks can be specified in its movie block command
- The BMAX model(s) in the first movie block is used for defining bones and mesh (vertices, etc), please see [this code](https://github.com/LiXizhi/NPLRuntime/tree/master/Client/trunk/ParaEngineClient/BMaxModel) for how to translate blocks into bones, vertices and sub meshes. BMAX models in all other movie blocks are ignored. 
- Animation data in movie blocks are used to generate bone animations for each animation id in the final ParaXModel. 


## 3D World File Format
ParaEngine/NPL can automatically save or load 3D scene object to or from disk files. 

To create an empty world, one can use
```lua
	local worldpath = "temp/clientworld";
	ParaIO.DeleteFile(worldpath.."/");
	ParaIO.CreateDirectory(worldpath.."/");
	ParaWorld.NewEmptyWorld(worldpath, 533.3333, 64);
```
3D world is tiled. Each tile is usually 512*512 meters. For historical reasons, it is 533.3333 by default. 
You will notice three files, after calling above function. 

`temp/clientworld/worldconfig.txt` which is the entry file for 3d world, its content is like 
```
-- Auto generated by ParaEngine 
type = lattice
TileSize = 533.333313
(0,0) = temp/clientworld/flat.txt
(0,1) = temp/clientworld/flat.txt
...
```
It contains a mapping from tile (x,y) to tile configuration file. 

`temp/clientworld/flat.txt` is configuration file for a single terrain tile.
```
-- auto gen by ParaEngine
Heightmapfile = temp/clientworld/flat.raw
MainTextureFile = terrain/data/MainTexture.dds
CommonTextureFile = terrain/data/CommonTexture.dds
Size = 533.333313
ElevScale = 1.0
Swapvertical = 1
HighResRadius = 30
DetailThreshold = 50.000000
MaxBlockSize = 64
DetailTextureMatrixSize = 64
NumOfDetailTextures = 0
```

## Global Terrain Format
Global Terrain Format is a LOD triangle tile based terrain engine. The logical tile size is defined by 3d world configuration file, such as 533.333, however, its real dimension is always 512*512 and saved as a `*.raw` file. 

## Block World File Format
ParaEngine has a build-in block engine for rendering 3D blocky world. 
Block world is also tiled. The logical tile size is defined by 3d world configuration file, such as 533.333, however, its real dimension is always `512x512` and saved as a binary block region file in `*.raw` extension. Please note that block world region file `*.raw` is different from global terrain's `*.raw` file. The latter is just height maps. 

Currently, each block may contain `x,y,z, block_id, block_data[, custom_data]`, currently `custom_data` is not supported in block region `*.raw` file. Block region file is encoded (compressed) using `increase-by-integer` algorithm, which will save lots of disk space if blocks are of the same type in the tiled region. 

See [BlockRegion.cpp](https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/BlockEngine/BlockRegion.h) for details

## References
- http://www.paracraft.cn/  : see bmax model tutorial video 
- https://github.com/LiXizhi/NPLRuntime/wiki
- https://github.com/LiXizhi/STLExporter
- http://wikicraft.cn/wiki/mod/packages : see STLExporter
- BMAX to ParaX converter in C++ (Lacking support for animation data in movie block)： https://github.com/LiXizhi/NPLRuntime/tree/master/Client/trunk/ParaEngineClient/BMaxModel

- ParaX models: 
  - https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/ParaXModel/XFileCharModelParser.h
  - https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/3dengine/ParaXSerializer.h

