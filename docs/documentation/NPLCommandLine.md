# Build-in NPL CommandLine

## Native Params
The following command line is built-in to NPL executable.

* `bootstrapper="myapp/main.lua"`: set bootstrapper file
* [-d|D] [filename]: -d to run in background (daemon mode), -D to force run in fore ground.
* `dev="C:/MyDev/"`: set dev directory, if empty or "/", it means the current working directory. 
* `servermode="true"`: disable 3D rendering and force running in server mode.
* `logfile="log2016.5.20.txt"`: change default `log.txt` location
* `single="true"`: force only one instance of the executable can be running in the OS.
* `loadpackage="folder1,folder1,..."`: commar separated list of packages to load before bootstrapping. Such as `loadpackage="npl_packages/paracraft/"`

## NPL System Module Params
The following params are handled by the NPL system module
* `debug="main"` : enable IPC debugging to the main NPL thread. No performance penalties.
* `resolution="1020 680"`: force window size in case of client application

## Paracraft Module Params
The following params are handled by the Paracraft Package
* `world="worlds/DesignHouse/myworld"` : Load a given world at start up.
* `mc="true"`: force paracraft (in case of running from MagicHaqi directory)