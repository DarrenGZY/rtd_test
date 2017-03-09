# NPL编程语言

## 1. 网站与文档（中文与英文）   
NPL语言有完整的注释以及库，类，函数的解释及使用方法。现在都的在线文档存储在[CodeDocs](https://codedocs.xyz/LiXizhi/NPLRuntime/)。未来的计划可以将文档以MarkDown的格式呈现，更符合现在人们查阅文档的习惯。  
- 建立统一的自动化文档发布机制：包括Core API, NPL packages, etc.  自动生成MarkDown
- 建立NPL官网 www.nplproject.com ：中英文的学习网，引用NPL WIKI文档

## 2. 自动化部署与更新    
NPL语言是一门不断的完善和进化的编程语言。每一次更新我们都会在Windows, Linux平台上自动化的部署，确保运行时环境每一次更新都可以在各个平台上的稳定运行和部署。    
- NPL运行时环境：Windows, Linux, Mac版的自动化部署

## 3. NPL语言的下载与安装，跨平台    
NPL语言的运行时环境现在还是跟ParaCraft整合在一起。我们正在进行的一项工作就是将NPL语言的运行时环境独立，提供独立的安装包，提供一个更轻便的运行时环境。同样，我们也会确保NPL语言的运行时环境在Windows，Linux，MacOS平台上均可以安装。     
- NPL运行时安装包windows(32/64bits), NPL官网增加下载页面: 自动加入Path Env中

## 4. VS(Visual Studio)编辑器与跨平台编辑器相关，模版插件
- NPL ide command line: 所有的UI命令应该都对应NPL命令 
- NPL for visual studio: 增加对基本框架和APP的模版，让初级程序员可以从UI创建初始程序。
- NPL for vs code

## 5. 包管理命令与VS(Visual Studio)插件    
NPL语言作为一门动态的编程语言，当编写前端或后端代码时，往往会依赖一些包或库。
- 我们准备参考现在流行的JavaScript的包管理工具npm，编写一个NPL语言的包管理工具，服务于NPL语言相关的包和库的管理。     
   - 增加require, import独立模块的开发与管理机制
- NPL语言的VS语言服务插件已经是一款成型的插件，提供高亮，语法检查，定义跳转，文件轮廓分析，格式化等常用功能。    
- NPL语言的VS调试插件同样是一款成型的插件，提供断点调试，提供调用堆栈等功能。    

## 6. 常用基础类库框架：WebServer， 2D UI， 3D    
NPL语言有着丰富的基础类库框架，使其作为一款动态脚本语言同样能完成一些大型，结构严谨的工程，比如服务器， 3D场景。    
- WebServer已经提供了npl page 编程模式， 计划再引入一种类似nodejs express, mustache的代码与模版分离的编程方式
- 2D UI： 计划移植qt的大部分UI到NPL中
- 3D： 基于Paracraft编辑器。 

## 7. DSL(Domain-Specific Language)与中文语言
- 我们已经现在NPL语言的基础上，衍生出一款适用于CAD编程的语言NPLCAD，这款语言用编程语言的形式完成3D模型的构建。
- 同样，现在的另外一项工作是设计并实现一款中文编程语言，寻找将中文更好带入编程语言的方式。
- 关于支持js：我们可以考虑提供一种方式将其他语言例如javascript动态翻译为NPL代码。 NPL运行环境和编译器会支持*.js，但是不推荐任何框架类应用使用它编码， 仅仅提供给初级程序员做一些简单的编码或临时性的JS类库的直接使用。 机器翻译的代码，会存在一些性能上的问题。 

