## Asset Manifest & Asynchronous Asset Loading 
A graphical application usually depends on tons of assets (textures, models, etc) to process and render. It is usually not possible deploy all assets to client machine at installation time, instead assets are downloaded from the server usually on first use. 

NPL/ParaEngine has built-in support for asynchronous asset loading via asset manifest system. Basically it resolves two problems:
1. Use a plain text file to look up and download assets in the background in several worker threads. 
2. Most UI and 3d objects in ParaEngine provide a synchronous interface to set assets, as if they are already available. In reality, they will automatically resolves assets (also their dependencies) when those assets are available locally. 

### Asset Manifest Manager
When an application starts, NPLRuntime will read all `Assets_manifest*.txt` file under the root directory. Each file has following content
```
	format is [relative path],md5,fileSize 
```
if the name ends with .z, it is zipped. This could be 4MB uncompressed in size
 md5 is checksum code of the file. fileSize is the compressed file size. 

```	
	audio/music.mp3.z,3799134715,22032
	model/building/tree.dds.z,2957514200,949
	model/building/tree.x.z,2551621901,816
	games/tutorial.swf,1157008036,171105
```
When one of the async loader try to load an application asset(texture, model, etc), it will first search in AssetManifest
using the TO-LOWER-CASED asset path, such as (model/building/tree.x). if will then search the "temp/cache/" directory for a matching file
	
The file matching is done by comparing the line in the asset file with the filename in the cache directory, using their md5 and size. 
```	
	audio/music.mp3.z,3799134715,22032 matches to file 379913471522032
```

**Example Usage:**
```
	AssetFileEntry* pEntry = CAssetManifest::GetSingleton().GetFile("Texture/somefile.dds");
	if(pEntry && pEntry->DoesFileExist())
	{
		// Load from file pEntry->GetLocalFileName();
	}
```