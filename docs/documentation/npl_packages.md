# NPL Packages
NPL Package is a special folder under `npl_packages/[package_name]/`. Files in it are always organized as if they are relative to the working directory. So this folder can be used as a search path directly, or zipped to a `*.zip|pkg` to be used as an [archive file](DeployGuide), or loaded by [file module name](LoadFile) all at the same time.  

NPL package serves following purposes:
- it provides a way to release your software modules to be used by someone else.
- it can be used as additional search path for third-party plugins, which I will explain later. 
- it provides a way to allow different versions of the same file module to coexist by putting them in different npl packages in a single application. See [[LoadFile]]
- it provides a way to install third-party modules at development time. 


## Package As Search Path
Files in `npl_packages/[package_name]` are usually relative to the root working directory. 

Therefore, developer who wants to use other people's modules can simply add `npl_packages/[package_name]` to global search path by calling following code:
```lua
NPL.load("npl_packages/some_test_module/")
```
The trick is to end with `/` in file name. What this function does is actually find the package folder
in a number of possible locations (see next section), and then add it to the global search path.
By default, when NPL starts, it will always try to load the official 'npl_packages/main/' package. 
So you do not need to call `NPL.load("npl_packages/main/")` in order to use the rich set of open source NPL libraries.

Another way is to load via command line, such as below. See [[NPLCommandLine]]
```
npl loadpackage="npl_packages/paracraft/" dev="/"
```

## How NPL Locate Packages
NPL locates package folder given by its relative path in following order:
* search in current working directory
* search in current executable directory
* search recursively for 5 parent directories of the executable directory.

For example, suppose:
* your current working directory is `/home/myapp/`, 
* and your executable directory is `/opt/NPLRuntime/redist/bin64/`, 

then `NPL.load("npl_packages/main/")` will search following directories until one exists and add it to the search path.
* /home/myapp/npl_packages/main/
* /opt/NPLRuntime/redist/bin64/npl_packages/main/
* /opt/NPLRuntime/redist/npl_packages/main/
* /opt/NPLRuntime/npl_packages/main/
* /opt/npl_packages/main/
* /npl_packages/main/

### What Happened After Loading A Package
The short answer is `nothing happens`, because loading a package only add its folder to the global search path. 
You still need to load any module files in the package folder with `NPL.load`.
For example, support you have `NPL.load("npl_packages/test/")` successfully loaded at '/home/myapp/npl_packages/test/' 
and there is module file at `/home/myapp/npl_packages/test/script/any_module_folder/test.lua`
Then you can load it with relative file path. `NPL.load("script/any_module_folder/test.lua")`
However, if there is a file at your working directory such as `/home/myapp/script/any_module_folder/test.lua`, this file will be loaded rather than the one in the global search path. 

### File Search Order
Files in current working directory are always searched first (including those in zipped archive files) before we resolve to global search path. Moreover, search path added last is actually searched first.
There is one exception, if NPL is run with dev directory in its command line `dev="dev_folder"`, NPL packages in `dev_folder` are loaded before zipped archive files. This allows one to use the latest source code during development.

For example: 
```lua
NPL.load("npl_packages/A/")
NPL.load("npl_packages/B/")
NPL.load("test.lua");
```
`test.lua` is searched first in current working directory and any loaded archive (zip, pkg) file. 
If not exist, it will search in `npl_packages/B/` and then in `npl_packages/A/`. 

### Usage

#### For package developers: 
> NPL packages are mostly used at development time by other developers (not end users). 

i.e. If you want other developers' to use your code, you can upload your working directory to git, so that other developers can clone your project to their `npl_packages/[your module name]`. Package can include not only source code, but also binary asset files, like images, textures, sound, 3d models, etc.

#### For other developers: 
> Other developers should merge all used npl_packages to working directory, before releasing their software. 

i.e. It is the developer's job to resolve dependencies, in case multiple versions of the same file exist among used npl_packages. At release time, it is recommended NOT to redistribute the npl_package folder, but copy/merge the content in them to the working directory, pre-compile all source code and package code and/or assets in one or multiple archive files. Please see the [[DeployGuide]] for details. However, if different versions of the same file must coexist, we can use [file-based modules](LoadFile) to distribute in separate `npl_packages` folders.

## Where To Find NPL Packages
Each repository under [NPLPackages](https://github.com/NPLPackages/) is a valid npl_package managed by the community.
> Click [here](https://github.com/LiXizhi/NPLRuntime/wiki/npl_packages) for more details on NPL packages.

If one want to upload their own package here, please make an [issue here](https://github.com/NPLPackages/NPLPackages/issues), and provide links to its code.

## How To Install A Package
Simply create a folder under your development's working directory, create a sub folder called `npl_packages`.
And then run git clone from there. Like this:
```
cd npl_packages
git clone https://github.com/NPLPackages/main.git
```

See also [paracraft package](https://github.com/NPLPackages/paracraft/wiki) for another example.

## File-based Modules
See [[LoadFile]].

## How to Contribute
It is NOT advised to modify or add files in the ./npl_packages folder, instead create a similar directory structure in your project's development directory if you want to add or modify package source code. If you do want to contribute to any npl packages, please fork it on github and send pull requests to its author on github.

For example, if you want to modify or add a file like `./npl_packages/main/.../ABC.lua`
Instead of modify it in the npl package folder, you simply create a file at the root development folder with the same directory structure like this `./.../ABC.lua`. At runtime, your version of file will be loaded instead of the one in npl package folder. 

When your code is mature, you may consider fork the given npl_package in another place, and merge your changed files and send the author a pull request. If the author responds fast, he or she may accept your changes and you can later get rid of your changed files in your original project. 
