#  Embedding NPLRuntime
NPLRuntime comes with a general purpose executable for launching NPL based applications. This is the standard usage of NPLRuntime. 

However, some client application may prefer embedding NPLRuntime in their own executable file, or even as child window of their own application. This is possible via NPLRuntime dll or (`paraengineclient.dll`) under windows platform. 

Please see [[https://github.com/LiXizhi/ParaCraftSDK/blob/master/samples/MyAppExe/MyApp.cpp]] for details. 

```c
#include "PEtypes.h"
#include "IParaEngineApp.h"
#include "IParaEngineCore.h"
#include "INPL.h"
#include "INPLRuntime.h"
#include "INPLRuntimeState.h"
#include "INPLAcitvationFile.h"
#include "NPLInterface.hpp"
#include "PluginLoader.hpp"

#include "MyApp.h"

using namespace ParaEngine;
using namespace NPL;
using namespace MyCompany;

// some lines omitted here ....

INT WINAPI WinMain(HINSTANCE hInst, HINSTANCE, LPSTR lpCmdLine, INT)
{
	std::string sAppCmdLine;
	if (lpCmdLine)
		sAppCmdLine = lpCmdLine;

	// TODO: add your custom command line here
	sAppCmdLine += " mc=true noupdate=true";

	CMyApp myApp(hInst);
	return myApp.Run(0, sAppCmdLine.c_str());
}
```

### Building Custom Executable Application
To build your custom application, you must download NPLRuntime source code (for header files) and set CMAKE include directory properly. 
- Download the sample at [[https://github.com/LiXizhi/ParaCraftSDK/blob/master/samples/MyAppExe]]
- Set CMAKE include directory to [[https://github.com/LiXizhi/NPLRuntime/tree/master/Client/trunk/ParaEngineClient/Core]]
- `paraengineclient.dll` is dynamically loaded at runtime via NPL plugin interface, which means that your executable does not need to link to any library file.  **All you need when embedding NPLRuntime is header files. ** This decouples your host application executable from NPLRuntime library files which may be upgraded separately without rebuilding your host executable. 
   - Main header files: 
      - [IParaEngineCore.h](https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/Core/IParaEngineCore.h)
      - [IParaEngineApp.h](https://github.com/LiXizhi/NPLRuntime/blob/master/Client/trunk/ParaEngineClient/Core/IParaEngineApp.h)

### Build NPLRuntime as Libraries
In most cases, you do not need to build these libraries, simply copy `ParaEngineClient.dll` and other related dll files from the ParacraftSDK's  `./redist` folder to your host application's executable directory. Make sure they are up to date. 

However, one can also build NPL Runtime as Libraries manually.  See `CMake` options when building NPLRuntime from source code. 

