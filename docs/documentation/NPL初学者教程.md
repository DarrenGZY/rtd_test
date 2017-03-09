# 1.	NPL语言简介
NPL（Neural Parallel Language）是神经元并行语言的缩写，是大富网络技术有限公司开发的一种新的编程语言，语法上和Lua 100%兼容。NPL的设计理念类似人类神经元的结构，将复杂计算分解到节点，简洁高效，特别适合于分布式计算、大型网络应用，3D图形渲染等。

# 2.	NPL运行时环境
NPL是一种解释性语言，包含了一个运行时环境，提供了完善的组件和接口，包括基本组件、网络组件和一套完整的3D引擎系统，并且体积只有10MB，在任何一台计算机上都可以快速部署。

NPL官方文档：[NPLRuntime](https://github.com/LiXizhi/NPLRuntime/wiki)

![NPL Runtime Architecture](https://cloud.githubusercontent.com/assets/22266739/18788825/fbf1e0f6-81da-11e6-9bac-31defc8a3ea8.png)NPL Runtime Architecture

# 3.	NPL运行时环境的安装
## 3.1	下载
直接使用git下载：[git clone](https://github.com/LiXizhi/ParaCraftSDK.git)

或从NPL官方地址下载：[官方下载](https://github.com/LiXizhi/ParaCraftSDK)
![](https://cloud.githubusercontent.com/assets/22266739/18789075/20d1d1a0-81dc-11e6-99cd-600c9a5c0423.png)
 
## 3.2	解压
将下载得到的`ParaCraftSDK-master.zip`解压到任意文件夹下。

## 3.3	更新二进制文件
运行`redist\ParaCraft.exe`，点“开始”会自动更新。
![](https://cloud.githubusercontent.com/assets/22266739/18789225/c97b1640-81dc-11e6-9089-5261fe0fed15.png)

## 3.4	安装
运行`NPLRuntime\install.bat`，安装完成。

## 3.5	设置路径的环境变量
当前版本有个小bug，可能不能成功注册环境变量。此时需要手动将解压路径`\NPLRuntime\win\bin`加入到环境变量中，这样就可以在任何路径下运行NPL程序了。

# 4.	编写第一个NPL程序
## 4.1	用文本编辑器新建`hello.lua`，输入以下内容并保存到`NPLRuntime\win\bin`路径下：
```lua
print("hello world!")
ParaGlobal.Exit(0)
```

## 4.2	打开命令行，进入`NPLRuntime\win\bin`路径，执行

`npl helloworld.lua`

## 4.3	打开`log.txt`，可以看到输出了`“hello world!”`
![](https://cloud.githubusercontent.com/assets/22266739/18789268/f1691d8c-81dc-11e6-8d00-854e94581158.png) 

# 5. Lua基本语法
## 5.1 简介
NPL的语法基于Lua语言。Lua语言是一种高效的脚本语言。官方网站是：http://www.lua.org/

这里是一个快速的语法介绍，旨在让阅读者能够以最快的速度了解基本语法，更详细的内容请参阅官方推荐的教材：[推荐教材](http://www.lua.org/pil/contents.html)

或者此处的中文教材：[中文教材](http://www.runoob.com/lua/lua-tutorial.html)
	
如果你已经对Lua语法很熟悉可以直接跳到第14章。

## 5.2	语句
一行为一个语句：
```lua
a=0
print(a)
```

## 5.3	注释
两个减号后为单行注释：
```lua
--这是一行注释
```
多行注释用--[[开头，

]]结束：
```lua
--[[
多行注释
多行注释
多行注释
]]
```

## 5.4	关键字
以下为Lua保留的关键字，不可用作变量或函数名称：

`and、break、do、else、elseif、end、false、for、function、if、in、local、nil、not、or、repeat、return、then、true、until、while`

# 6. 数据类型与变量
## 6.1 数据类型
Lua中有8个基本类型分别为：`nil、boolean、number、string、userdata、function、thread和table`。

|数据类型    |描述  
|---------------|-------
|nil            |只有值nil属于该类，表示一个无效值（在条件表达式中相当于false）。
|boolean	|包含两个值：false和true。
|number	        |表示双精度类型的实浮点数
|string 	|字符串由一对双引号或单引号来表示
|function	|由 C 或 Lua 编写的函数
|userdata       |表示任意存储在变量中的C数据结构
|thread         |表示执行的独立线路，用于执行协同程序
|table	        |Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。 	
	
## 6.2 变量
Lua是动态类型语言，变量不要类型定义，只需要为变量赋值。 值可以存储在变量中，作为参数传递或结果返回。
```lua
a=10		--a为number类型
b="hello"	--b为string类型
```

## 6.3	全局变量与局部变量
Lua中直接定义的变量全部是全局变量，包括函数中的变量，除非用local显式声明为局部变量。

```lua
a=5
function test()
     b=10
     local c=15
end
test()
print(a,b,c)    --输出为5    10    nil
```

# 7.	运算符
## 7.1	算术运算符
假设`a=10，b=20`

|操作符          |描述 |实例 
|---------------|-------|-------
|+              |加法    |a+b=30 
|-	        |减法	|a-b=-10 
|*	        |乘法	|a*b=200
|/	        |除法	|a/b=0.5 
|%        	|取余	|a%b=10
|^              |乘幂	|a^2=100
|-              |负号	|-a=-10
		
## 7.2	关系运算符
假设`a=10，b=20`

|操作符          |描述     |实例 
|---------------|-------  |-------
|==             |等于      |(a==b)=false
|~=	        |不等于     |(a~=b)=true
|>	        |大于	   |(a>b)=false
|<        	|小于	   |(a<b)=true
|>=             |大于或等于  |(a>=b)=false
|<=             |小于或等于  |(a<=b)=true
 

## 7.3	逻辑运算符
假设`a=true，b=false`

|操作符          |描述   |实例 
|---------------|-------|-------
|and            |逻辑与  |(a and b)=false 
|or	        |逻辑或	|(a or b)=true
|not	        |逻辑非	|not a= false

## 7.4	其它运算符
假设`a="Hello"，b="World"`

|操作符          |描述       |实例 
|---------------|--------------|-------
|..             |连接两个字符串        |a..b=HelloWorld
|#	        |返回字符串或表的长度   |#a=5

## 7.5	运算符优先级
|高             |^
|---------------|--------------
|               |not   -(负号)

|低             |*    /
|---------------|--------------
|               |+   -
|               |..
|               |<    >    <=    >=    ~=    ==
|               |and
|               |or	

# 8.	流程控制
## 8.1	if
语法：
```lua
if (bool表达式) then
    --bool表达式为true时执行的语句
end

if (bool表达式) then
    --bool表达式为true时执行的语句
else
    --bool表达式为false时执行的语句
end

if (bool表达式1) then
    --bool表达式1为true时执行的语句
elseif (bool表达式2) then
    --bool表达式2为true时执行的语句
else
    --以上表达式都为false时执行的语句
end
```

## 8.2	数值for
语法：
```lua
for var=exp1,exp2,exp3 do
     --语句块
end
     --var从exp1变化到exp2，每次变化以exp3为步长，exp3为可选，不指定则为1
```
实例：

```lua
for i=1,10 do
     print(i)
end

for i=10,1,-1 do
     print(i)
end
```

## 8.3	泛型for
语法：
```lua
for i,v in ipairs(a) do
      print(v)
end
    --打印数字a中的所有值
```
实例：
```lua
days = {"Suanday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"}
for i,v in ipairs(days) do
     print(v)
end
```

## 8.4	while
语法：
```lua
while(bool表达式) do
     --bool表达式为true时执行的语句
end
```

实例：
```lua
i=0
while(i<20) do
     print(i)
     i = i+1
end
```

## 8.5	repeat
语法：
```lua
repeat    --语句块
until(bool表达式)
```
实例：
```lua
i=0
repeat
    print(i)
    i=i+1
until(i>20)
```

## 8.6	break
跳出循环。

## 8.7	return
函数返回。

# 9.	函数
## 9.1	定义
```lua
(local) function function_name (arg1, arg2, arg3, … argn)
       函数体
       return 返回值
end     --和变量一样，local为作用域，默认为全局函数，加local为局部函数。
```

## 9.2	参数
多个参数时用逗号隔开，调用时如果传入的参数不足，其余参数为nil
```lua
function test(a, b, c)
   print(a,b,c)
end
test(1,2)      --输出为1   2   nil
```

Lua支持可变参数，函数的参数放在一个名为arg的表中，#arg表示参数个数
```lua
function test(...)
   local arg={...}
   for i,v in ipairs(arg) do
      print(v)
   end
   print(“参数个数=”..#arg)
end
```

## 9.3	返回值
Lua支持多返回值，与相应数目的变量直接赋值即可
```lua
function swap(a,b)
   return b,a
end
```

```lua
i=1
j=2
i,j=swap(i,j)
print("i="..i,"j="..j)    --输出i=2  j=1
```

# 10.	table
Lua中的table实际是一个“Key-Value”结构的关联数组，使用前需要先初始化：
```lua
mytalbe={}		--创建一个空表
```
创建之后可以直接添加元素：
```lua
mytable[1]="Monday"
mytable["hello"]="world"
```
或在创建时直接添加：
```lua
mytable={[1]="Monday", hello="world"}		--与上面的创建方式等价
```
如果用key全用数字（相当于数组）创建时可以简略写成：
```lua
mytable = {"Monday", "Tuesday","Wednesday"}
```
但此时下标是从1开始的，即`mytable[1]="Monday"`

如果key为字符串，可以用"."的形式访问：

`mytable.hello`等价于`mytable["hello"]`

遍历可使用泛型for：
```lua
for k,v in pairs(mytable) do		--一般table
    print(k,v)
end
```

```lua
for i,v in ipairs(mytable) do		--数字下标的table
    print(i,v)
end
```

# 11.	metatable（元表）
metastable可以用来改变table的行为，如__index表示查找元素的行为：
```lua
mt = { foo = 3 }
t = {}
setmetatable(t, { __index = mt }) 
print(t.foo)
```
在执行t.foo时，由于t没有foo这个key对应的value，会到metatable中去查找，metatable中有__index这个key对应的值mt，所以最终会到mt中去
查找foo对应的value，最后得到3。必须理解metatable才能理解下一节中面向对象的实现方法。

# 12.	面向对象
## 12.1	对象
Lua也支持面向对象的编程，实际上是利用table来实现的：
```lua
Account = {balance = 0}
function Account.withdraw (v)
    Account.balance = Account.balance - v
end
```
创建了一个Account对象，它有一个balance的成员和一个withdraw方法。

## 12.2	类
上面的Account是一个对象，如果想定义一个类，添加方法时需要用“:”创建一个self参数，并且需要一个new方法：
```lua
Account = {balance = 0}
function Account:new (o)			--等价于function Account.new(self, o)
   o = o or {}   					--创建一个新的table
   setmetatable(o, self)			--非常重要，请看下面的详细说明
   self.__index = self
   return o
end
function Account:withdraw (v)		--等价于function Account.withdraw(self, v)
    self.balance = self.balance - v
end
```

使用Account类
```lua
a=Account:new()					--创建一个新的对象a
a:withdraw(10)					--调用withdraw方法
```
在Account:new()中的：
```lua
   setmetatable(o, self) 

   self.__index = self

--为新对象o设置了一个metatable，改变了__index的行为，对于：

a=Account:new()

--在执行new函数的时候，o的metatable被设置成了Account，a=o（这时是个空表），那么在：

a:withdraw(10)

--执行的时候，由于a没有withdraw这个方法，结果通过metatable调用了：

Account.withdraw(a, 10)

--即Account的withdraw方法，参数里的self是a本身
```

# 13.	IO
## 13.1	简单模式
简单模式一次处理一个文件，直接使用io操作：
```lua
file = io.open("test.lua", "r")		--只读打开文件

io.input(file)					--设置io的输入文件
print(io.read())				--读取文件第一行并打印
io.close(file)					--关闭文件
```

## 13.2	完全模式
如果同时要处理多个文件就需要使用完全模式，即使用file:function代替io.function：
```lua
file = io.open("test.lua", "r")		--只读打开文件
print(file:read())				--读取文件第一行并打印
file:close()					--关闭文件
```

# 14.	NPL的运行模型
从本章开始介绍NPL的特性。NPL作为一个完整的运行时环境有其独特的运行机制，简单的说是一个分布式的运行调度方式，基于Actor模型。理解了运行模型，才能真正理解NPL。

Actor模型

![image](https://cloud.githubusercontent.com/assets/22266739/18821111/cbc30d90-83d5-11e6-8172-679f5dc99636.png)
 
Actor这个模型由Carl Hewitt在1973年提出，是一个并行计算的数学模型。它的理念非常简单：

- 万物皆Actor，每个Actor是完全独立的，可以同时执行它们的操作

- Actor之间通过发送消息来通信，消息通过一个消息队列来处理

- 每一个Actor是一个计算实体，映射接收到的消息到以下动作

  - 发送有限个消息给其它Actor

  - 创建有限个新的Actor

  - 为下一个接收的消息指定行为


NPL Runtime调度模型
 
![image](https://cloud.githubusercontent.com/assets/22266739/18822568/e8a4bd56-83e3-11e6-8881-7347ff3e2a70.png)

当我们在本地执行：
```lua
npl hello.lua
```
创建了一个主进程，这个进程又创建了一个主线程来执行hello.lua（实际还创建了其它线程用来运行runtime的不同组件），每个线程有一个消息队列，被执行的文件会被加入到消息队列中。


Activate（激活）文件

在程序中，用`NPL.activate(url, {data})`激活一个文件，data是传递的数据：

hello.lua中执行：
```lua
NPL.CreateRuntimeState("T1", 0):Start();	--创建名为T1的线程

NPL.activate("(T1)test1.lua", {"hello"})		--在T1线程中激活test.lua
```
test1.lua代码
```lua
local function activate()
   print(msg[1]);		--打印出"hello"
end
NPL.this(activate);		--设置activate的入口为activate()函数
```
	一个文件被激活后，它将会加入到指定线程中运行，接收消息，处理消息和发送消息。


Activate的本质--发送消息

	NPL.activate的本质是向某个文件发送消息，如果这个文件不在指定线程的消息队列中，

则会将它加入到线程中，如果已经在消息队列中，则只是发送消息到消息队列里。


分布式

`NPL.activate`的第一个参数，即文件的url，可以是本地文件，也可以是一个远程的文件，url参数的格式是：
```lua
[(sRuntimeStateName|gl)][sNID:]sRelativePath[]
```
举例：
```lua
(gl)script/hello.lua
(worker1)script/hello.lua
(world1)server001:script/hello.lua
```

# 15.	多线程
从NPL运行模型可知，多线程是NPL的基本运行方式，使用多线程只需要用`NPL.CreateRuntimeState`创建线程，然后用`NPL.activate`调用相关的文件
加载到相关线程运行就可以了。
```lua
NPL.CreateRuntimeState("T1", 0):Start();
NPL.activate("(T1)test1.lua", {"hello1"})
NPL.CreateRuntimeState("T2", 0):Start();
NPL.activate("(T2)test2.lua", {"hello2"})
```
一般情况下消息驱动的模型只需要将要运行的同类文件加载到同一线程就可以了。

Lua本身还支持一种名为`coroutine（协同程序）`的类似线程的机制，但它实际是用单线程模拟多线程，需要用程序来控制不同coroutine的协同工作，使用比较复杂，也不符合NPL的设计理念，在NPL中不推荐使用。

# 16.	网络
从NPL运行模型可知，网络是NPL的内置属性，用NPL.activate可以无需知道网络的细节，直接根据url进行传输，运行时环境会自动建立连接和传输数据。

下面是一个远程传输的例子：

Server端：
```lua
hello.lua
local function activate()
	print("hello world")
end
NPL.this(activate)

server.lua
print("server start")
NPL.StartNetServer("127.0.0.1", "60001");
NPL.AddPublicFile("hello.lua", 1);
```

Client端：
```lua
client.lua
print("client start")
NPL.StartNetServer("0", "0")
NPL.AddNPLRuntimeAddress({host="127.0.0.1", port="60001", nid="server1"})
while( NPL.activate("server1:hello.lua", {"remote activate"}) ~=0 ) do
    print("failed to send message");
    ParaEngine.Sleep(1);
end
```

测试时打开两个命令行分别执行：
```lua
npl bootstrapper="server.lua" logfile="server.txt"
npl bootstrapper="client.lua" logfile="client.txt"
```

在server.txt中可以看到输出：
```lua
server start				--server运行后的输出
hello world				--client连接上之后激活hello.lua的输出
```

# 17.	使用库
## 17.1	加载库
使用库首先要用`NPL.load`加载`commonlib.lua`和需要的库文件：
```lua
NPL.load("(gl)script/ide/commonlib.lua")
NPL.load("(gl)mylib.lua")
```
## 17.2	使用库中的类
要使用库中的类，先使用`commonlib.gettable`来获取库的类：
```lua
myclass = commonlib.gettable("myclass");
```
以下是一个完整的例子：
```lua
Account.lua：
Account = {balance = 0}
function Account:new (o)
   o = o or {}
   setmetatable(o, self)
   self.__index = self
   return o
end
function Account:withdraw (v)
    self.balance = self.balance - v
end

test.lua：
NPL.load("(gl)script/ide/commonlib.lua");
NPL.load("Account.lua");

local Account = commonlib.gettable("Account");
a=Account:new({balance=100})
a:withdraw(10)
print(a.balance)			--输出90
```

## 17.3	继承
如果要从原有的类派生新类，要使用`commonlib.inherit`：
```lua
newclass = commonlib.inherit(nil, commonlib.gettable("myclass"))
```
	在新的类中ctor()函数为构造函数。


一个例子：
```lua
NPL.load("(gl)script/ide/commonlib.lua");
NPL.load("Account.lua");

local NewAccount= commonlib.inherit(nil, commonlib.gettable("Account"))

function NewAccount:ctor()				--构造函数
	self.balance = 100
end

function NewAccount: deposit(v)			--增加的方法
	self.balance = self.balance+v
end

a=Account:new()
a:deposit(10)
print(a.balance)			--输出110
```

# 18.	Web开发
NPL带有一个Web服务器，它类似一个PHP服务器，可以处理用户请求的页面，运行该页面中内嵌的NPL脚本，并产生结果的html页面返回给用户。
## 18.1	搭建服务器
1)	从git上下载WebServer组件：
[git clone](https://github.com/NPLPackages/main.git)

2)	在`main\script\apps\WebServer`下执行：
```lua
npl bootstrapper="WebServer.lua" port="80" root="test/" servermode="true"
```
	port参数为端口号，root参数为web服务的目录

3)	用浏览器打开 http://localhost/index.page，显示测试页面

![image](https://cloud.githubusercontent.com/assets/22266739/18821251/261b10de-83d7-11e6-964a-45bc876be65a.png)
 
## 18.2	编写页面程序
页面程序是在html页面中嵌入NPL脚本，用户请求该页面时，server会执行脚本并将结果返回给用户：
```lua
<html>
<body>
<?npl
for i=1,5 do
        echo "<p>hello</p>"
      end
?>
</body>
</html>
```
其中
```lua
<?npl  ?>
```
标记间的代码会被服务器端执行，

# 19.	数据库编程
NPL基于Lua语言，因此Lua支持的数据库都可以支持。以下是几种数据库的使用方法：
## 19.1	Table Database（NPL内置）
Table Database是NPL内置的数据库，是一个具有大部分标准数据库功能的小型数据库，简单高效，在数据不是特别复杂的情况下非常适用。

## 19.2	MySql
NPL还内置了MySql驱动，可以直接连接使用MySql数据库，详见[UsingMySql](https://github.com/LiXizhi/NPLRuntime/wiki/UsingMySql)

## 19.3	ODBC, ADO, Oracle, MySql, SQLite, Firebird and PostgreSQL
要连接使用其它第三方数据库，需要使用开源的LuaSQL，详见[官方网站](https://github.com/keplerproject/luasql)，具体使用方法请参考官方文档。