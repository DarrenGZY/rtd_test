## Serialization, Encoding and Logging

- Serialization: [script/ide/serialization.lua](https://github.com/NPLPackages/main/tree/master/script/ide/serialization.lua)
- Encoding: [script/ide/Encoding.lua](https://github.com/NPLPackages/main/tree/master/script/ide/Encoding.lua)
- Logging: [script/ide/log.lua](https://github.com/NPLPackages/main/tree/master/script/ide/log.lua)
- SHA1: [script/ide/System/Encoding/sha1.lua](https://github.com/NPLPackages/main/tree/master/script/ide/System/Encoding/sha1.lua)

### NPL/Lua Table Serialization
see `script/test/TestNPL.lua`

```lua
local o = {a=1, b="string"};

-- serialize to string
local str = commonlib.serialize_compact(o)
-- write string to log.txt
log(str); 
-- string to NPL table again
local o = NPL.LoadTableFromString(str);
-- echo to output any object to log.txt
echo(o); 
```

### Data Formatting
```lua
log(string.format("succeed: test case %.3f for %s\r\n", 1/3, "hello"));
-- format is faster than string.format, but only support limited place holder like %s and %d. 
log(format("succeed: test case %d for %s\r\n", 1, "hello"));
```

### Data Encoding

- Encoding: [script/ide/Encoding.lua](https://github.com/NPLPackages/main/tree/master/script/ide/Encoding.lua)

```lua
commonlib.echo(NPL.EncodeURLQuery("http://www.paraengine.com", {"name1", "value1", "name2", "中文",}))
```

### UTF8 vs Default Text Encoding
Encoding is a complex topic. The rule of thumb in NPL is to use utf8 encoding wherever possible, such as in source code, XML/html/page files, UI text, network messages, etc. However, system file path must be encoded in the operating system's default encoding, which may be different from utf8, so we need to use following methods to convert between the default and utf8 encoding for a given text to be used as system file path, such as `ParaIO.open(filename)` requires `filename` to be in default encoding. 

```lua
NPL.load("(gl)script/ide/Encoding.lua");
local Encoding = commonlib.gettable("commonlib.Encoding");
commonlib.Encoding.Utf8ToDefault(text)
commonlib.Encoding.DefaultToUtf8(text)
```

### Json Encoding
- Json Encoding: [script/ide/Json.lua](https://github.com/NPLPackages/main/tree/master/script/ide/Json.lua)

```lua
NPL.load("(gl)script/ide/Json.lua");
local t = { 
["name1"] = "value1",
["name2"] = {1, false, true, 23.54, "a \021 string"},
name3 = commonlib.Json.Null() 
}

local json = commonlib.Json.Encode (t)
print (json) 
--> {"name1":"value1","name3":null,"name2":[1,false,true,23.54,"a \u0015 string"]}

local t = commonlib.Json.Decode(json)
print(t.name2[4])
--> 23.54

-- also consider the NPL version
local out={};
if(NPL.FromJson(json, out)) then
	commonlib.echo(out)
end
```

### XML Encoding
- XML Encoding: [script/ide/LuaXML.lua](https://github.com/NPLPackages/main/tree/master/script/ide/LuaXML.lua)

```lua
NPL.load("(gl)script/ide/LuaXML.lua");
function TestLuaXML:test_LuaXML_CPlusPlus()
	local input = [[<paragraph justify="centered" >first child<b >bold</b>second child</paragraph>]]
	local x = ParaXML.LuaXML_ParseString(input);
	assert(x[1].name == "paragraph");
end

function TestLuaXML:test_LuaXML_NPL()
	local input = [[<paragraph justify="centered" >first child<b >bold</b>second child</paragraph>]]
	local xmlRoot = commonlib.XML2Lua(input)
	assert(commonlib.Lua2XmlString(xmlRoot) == input);
	log(commonlib.Lua2XmlString(xmlRoot, true))
end
```

### Binary Data 
To read or write binary data to or from string, we can use the file API with a special filename called `<memory>`, see below. 
- To read binary string, simply call `WritingString` to write input string to a memory buffer file and then `seek(0)` and read data out in any way you like.
- To write binary string, simply call `WritingString`, `WriteBytes` or `WritingInt`, once finished, call `GetText(0, -1)` to get the final output binary string. 

```lua
function test_MemoryFile()
	-- "<memory>" is a special name for memory file, both read/write is possible. 
	local file = ParaIO.open("<memory>", "w");
	if(file:IsValid()) then	
		file:WriteString("hello ");
		local nPos = file:GetFileSize();
		file:WriteString("world");
		file:WriteInt(1234);
		file:seek(nPos);
		file:WriteString("World");
		file:SetFilePointer(0, 2); -- 2 is relative to end of file
		file:WriteInt(0);
		file:WriteString("End");
		file:WriteBytes(3, {100, 0, 22});
		-- read entire binary text data back to npl string
		echo(#(file:GetText(0, -1)));
		file:close();
	end
end
```

### XPath query in XML 
- XPath: [script/ide/XPath.lua](https://github.com/NPLPackages/main/tree/master/script/ide/XPath.lua)

```lua
NPL.load("(gl)script/ide/XPath.lua");

local xmlDocIP = ParaXML.LuaXML_ParseFile("script/apps/Poke/IP.xml");
local xpath = "/mcml:mcml/mcml:packageList/mcml:package/";
local xpath = "//mcml:IPList/mcml:IP[@text = 'Level2_2']";
local xpath = "//mcml:IPList/mcml:IP[@version = 5]";
local xpath = "//mcml:IPList/mcml:IP[@version < 6]"; -- only supported by selectNodes2
local xpath = "//mcml:IPList/mcml:IP[@version > 4]"; -- only supported by selectNodes2
local xpath = "//mcml:IPList/mcml:IP[@version >= 5]"; -- only supported by selectNodes2
local xpath = "//mcml:IPList/mcml:IP[@version <= 5]"; -- only supported by selectNodes2

local xmlDocIP = ParaXML.LuaXML_ParseFile("character/v3/Pet/MGBB/mgbb.xml");
local xpath = "/mesh/shader/@index";
local xpath = "/mesh/boundingbox/@minx";
local xpath = "/mesh/submesh/@filename";
local xpath = "/mesh/submesh";

--
-- select nodes to an array table
--
local result = commonlib.XPath.selectNodes(xmlDocIP, xpath);
local result = XPath.selectNodes(xmlDocIP, xpath, nMaxResultCount); -- select at most nMaxResultCount result

--
-- select a single node or nil
--
local node = XPath.selectNode(xmlDocIP, xpath);

--
-- iterate on all nodes. 
--
for node in commonlib.XPath.eachNode(xmlDocIP, xpath) do
	commonlib.echo(node[1]);
end
```

### Logging

> `log.txt` may take a few seconds to be flushed to disk file.

- Logging: [script/ide/log.lua](https://github.com/NPLPackages/main/tree/master/script/ide/log.lua)

```lua
-- write formatted logs to log.txt
LOG.std(nil,"info", "sub_system_name", "some formated message: %s", "hello"); 
LOG.std(nil,"debug", "sub_system_name", {a=1, c="any table object"}); 
LOG.std(nil,"error", "sub_system_name", "error code here"); 
LOG.std(nil,"warn", "sub_system_name", "warning"); 
```

Log redirection and configurations:
It is also possible to redirect `log.txt` to different files on startup. see [[NPLCommandLine]]
```lua
NPL.load("(gl)script/ide/log.lua");
commonlib.log("hello %s \n", "paraengine")
commonlib.log({"anything"})
local fromPos = commonlib.log.GetLogPos()
log(fromPos.." babababa...\n");
local text = commonlib.log.GetLog(fromPos, nil)
log(tostring(text).." is retrieved\n")

commonlib.applog("hello paraengine"); --> ./log.txt --> 20090711 02:59:19|0|hello paraengine|script/shell_loop.lua:23: in function FunctionName|
commonlib.applog("hello %s", "paraengine") 

commonlib.servicelog("MyService", "hello paraengine"); --> ./MyService_20090711.log --> 2009-07-11 10:53:27|0|hello paraengine||
commonlib.servicelog("MyService", "hello %s", "paraengine");

-- set log properties before using the log
commonlib.servicelog.GetLogger("no_append"):SetLogFile("log/no_append.log")
commonlib.servicelog.GetLogger("no_append"):SetAppendMode(false);
commonlib.servicelog.GetLogger("no_append"):SetForceFlush(true);
commonlib.servicelog("no_append", "test");

-- This will change the default logger's file position at runtime.
commonlib.servicelog.GetLogger(""):SetLogFile("log/log_2016.5.19.txt");
```
