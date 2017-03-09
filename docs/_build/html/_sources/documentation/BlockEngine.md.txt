# Block Engine
The block engines can manage multiple `BlockWorld` object. 

## BlockWorld
Each `BlockWorld` represents a block world with at most `32000x32000x256`, where 256 is height of the world.
Each BlockWorld will dynamically and asynchronously load `BlockRegion` on demand. 

## BlockRegion
It manages 512x512x256 blocks, which are saved into a single file.

## BlockChunk
It caches model and light Data for `16x16x16` region. Each chunk is converted and added to a queue into `BlockRenderTask` for sorting and rendering.

## BlockLightGrid
It calculates sun and block lighting in a separate thread and save the result into `BlockChunk` for rendering. 

## BlockModel
BlockModel is usually cube 3D model, but it is not a 3D object directly used in rendering, instead it is actually used in `BlockTemplate` to provide rendering and physics data.



