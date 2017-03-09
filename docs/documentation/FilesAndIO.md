## Files API

### Search Files
- Files: [script/ide/Files.lua](https://github.com/NPLPackages/main/tree/master/script/ide/Files.lua)

```lua
NPL.load("(gl)script/ide/Files.lua");
local result = commonlib.Files.Find({}, "model/test", 0, 500, function(item)
	local ext = commonlib.Files.GetFileExtension(item.filename);
	if(ext) then
		return (ext == "x") or (ext == "dds")
	end
end)

-- search zip files using perl regular expression. like ":^xyz\\s+.*blah$"
local result = commonlib.Files.Find({}, "model/test", 0, 500, ":.*", "*.zip")

-- using lua file system
local lfs = commonlib.Files.GetLuaFileSystem();
echo(lfs.attributes("config/config.txt", "mode"))
```

### Read/Write Binary Files

```lua
	local file = ParaIO.open("temp/binaryfile.bin", "w");
	if(file:IsValid()) then	
		local data = "binary\0\0\0\0file";
		file:WriteString(data, #data);
		-- write 32 bits int
		file:WriteUInt(0xffffffff);
		file:WriteInt(-1);
		-- write float
		file:WriteFloat(-3.14);
		-- write double (precision is limited by lua double)
		file:WriteDouble(-3.1415926535897926);
		-- write 16bits word
		file:WriteWord(0xff00);
		-- write 16bits short integer
		file:WriteShort(-1);
		file:WriteBytes(3, {255, 0, 255});
		file:close();

		-- testing by reading file content back
		local file = ParaIO.open("temp/binaryfile.bin", "r");
		if(file:IsValid()) then	
			-- test reading binary string without increasing the file cursor
			assert(file:GetText(0, #data) == data);
			file:seekRelative(#data);
			assert(file:getpos() == #data);
			file:seek(0);
			-- test reading binary string
			assert(file:ReadString(#data) == data);
			assert(file:ReadUInt() == 0xffffffff);
			assert(file:ReadInt() == -1);
			assert(math.abs(file:ReadFloat() - (-3.14)) < 0.000001);
			assert(file:ReadDouble() == -3.1415926535897926);
			assert(file:ReadWord() == 0xff00);
			assert(file:ReadShort() == -1);
			local o = {};
			file:ReadBytes(3, o);
			assert(o[1] == 255 and o[2] == 0 and o[3] == 255);
			file:seek(0);
			assert(file:ReadString(8) == "binary\0\0");
			file:close();
		end
	end
```

### In-Memory File And Binary Buffer
To write binary data to string, we can use the file API with a special filename called `<memory>`, see below

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

### Read/Write Files

see `NPL.load("(gl)script/test/ParaIO_test.lua");`

```lua
-- tested on 2007.1, LiXizhi
local function ParaIO_FileTest()
	
	-- file write
	log("testing file write...\r\n")
	
	local file = ParaIO.open("temp/iotest.txt", "w");
	file:WriteString("test\r\n");
	file:WriteString("test\r\n");
	file:close();
	
	-- file read
	log("testing file read...\r\n")
	
	local file = ParaIO.open("temp/iotest.txt", "r");
	log(tostring(file:readline()));
	log(tostring(file:readline()));
	log(tostring(file:readline()));
	file:close();
	
end

-- tested on 2007.6.7, LiXizhi
local function ParaIO_ZipFileTest()
	local writer = ParaIO.CreateZip("d:\\simple.zip","");
	writer:ZipAdd("temp/file1.ini", "d:\\file1.ini");
	writer:ZipAdd("temp/file2.ini", "d:\\file2.ini");
	writer:ZipAddFolder("temp");
	writer:AddDirectory("worlds/", "d:/temp/*.", 4);
	writer:AddDirectory("worlds/", "d:/worlds/*.*", 2);
	writer:close();
end

-- tested on 2007.6.7, LiXizhi
local function ParaIO_SearchZipContentTest()
	-- test case 1 
	log("test case 1\n");
	local search_result = ParaIO.SearchFiles("","*.", "d:\\simple.zip", 0, 10000, 0);
	local nCount = search_result:GetNumOfResult();
	local i;
	for i = 0, nCount-1 do 
		log(search_result:GetItem(i).."\n");
	end
	search_result:Release();
	-- test case 2 
	log("test case 2\n");
	local search_result = ParaIO.SearchFiles("","*.ini", "d:\\simple.zip", 0, 10000, 0);
	local nCount = search_result:GetNumOfResult();
	local i;
	for i = 0, nCount-1 do 
		log(search_result:GetItem(i).."\n");
	end
	search_result:Release();
	-- test case 3 
	log("test case 3\n");
	local search_result = ParaIO.SearchFiles("","temp/*.", "d:\\simple.zip", 0, 10000, 0);
	local nCount = search_result:GetNumOfResult();
	local i;
	for i = 0, nCount-1 do 
		log(search_result:GetItem(i).."\n");
	end
	search_result:Release();
	-- test case 4 
	log("test case 4\n");
	local search_result = ParaIO.SearchFiles("temp/","*.*", "d:\\simple.zip", 0, 10000, 0);
	local nCount = search_result:GetNumOfResult();
	local i;
	for i = 0, nCount-1 do 
		log(search_result:GetItem(i).."\n");
	end
	search_result:Release();
	-- test case 5 
	log("test case 5\n");
	local search_result = ParaIO.SearchFiles("","temp/*.*", "d:\\simple.zip", 0, 10000, 0);
	local nCount = search_result:GetNumOfResult();
	local i;
	for i = 0, nCount-1 do 
		log(search_result:GetItem(i).."\n");
	end
	search_result:Release();	
end

local function ParaIO_SearchPathTest()
	ParaIO.CreateDirectory("npl_packages/test/")
	local file = ParaIO.open("npl_packages/test/test_searchpath.lua", "w");
	file:WriteString("echo('from test_searchpath.lua')")
	file:close();

	ParaIO.AddSearchPath("npl_packages/test");
	ParaIO.AddSearchPath("npl_packages/test/"); -- same as above, check for duplicate
	
	assert(ParaIO.DoesFileExist("test_searchpath.lua"));
	
	ParaIO.RemoveSearchPath("npl_packages/test");
	
	assert(not ParaIO.DoesFileExist("test_searchpath.lua"));

	-- ParaIO.AddSearchPath("npl_packages/test/");
	-- this is another way of ParaIO.AddSearchPath, except that it will check for folder existence.
	-- in a number of locations.
	NPL.load("npl_packages/test/");
	
	-- test standard open api
	local file = ParaIO.open("test_searchpath.lua", "r");
	if(file:IsValid()) then
		log(tostring(file:readline()));
	else
		log("not found\n");	
	end
	file:close();

	-- test script file
	NPL.load("(gl)test_searchpath.lua");
	
	ParaIO.ClearAllSearchPath();
	
	assert(not ParaIO.DoesFileExist("test_searchpath.lua"));
end

-- TODO: test passed on 2008.4.20, LiXizhi
function ParaIO_PathReplaceable()
	ParaIO.AddPathVariable("WORLD", "worlds/MyWorld")
	if(ParaIO.AddPathVariable("userid", "temp/LIXIZHI_PARAENGINE")) then
		local fullpath;
		commonlib.echo("test simple");
		fullpath = ParaIO.DecodePath("%WORLD%/%userid%/filename");
		commonlib.echo(fullpath);
		fullpath = ParaIO.EncodePath(fullpath)
		commonlib.echo(fullpath);
		
		commonlib.echo("test encoding with a specified variables");
		fullpath = ParaIO.DecodePath("%WORLD%/%userid%/filename");
		commonlib.echo(fullpath);
		commonlib.echo(ParaIO.EncodePath(fullpath, "WORLD"));
		commonlib.echo(ParaIO.EncodePath(fullpath, "WORLD, userid"));
		
		commonlib.echo("test encoding with inline path");
		fullpath = ParaIO.DecodePath("%WORLD%/%userid%_filename");
		commonlib.echo(fullpath);
		fullpath = ParaIO.EncodePath(fullpath)
		commonlib.echo(fullpath);
		
		
		commonlib.echo("test nested");
		fullpath = ParaIO.DecodePath("%userid%/filename/%userid%/nestedtest");
		commonlib.echo(fullpath);
		fullpath = ParaIO.EncodePath(fullpath)
		commonlib.echo(fullpath);
		
		commonlib.echo("test remove");
		if(ParaIO.AddPathVariable("userid", nil)) then
			fullpath = ParaIO.DecodePath("%userid%/filename");
			commonlib.echo(fullpath);
			fullpath = ParaIO.EncodePath(fullpath)
			commonlib.echo(fullpath);
		end
		
		commonlib.echo("test full path");
		fullpath = ParaIO.DecodePath("NormalPath/filename");
		commonlib.echo(fullpath);
		fullpath = ParaIO.EncodePath(fullpath)
		commonlib.echo(fullpath);
	end
end

function ParaIO_SearchFiles_reg_expr()
	-- test case 1 
	local search_result = ParaIO.SearchFiles("script/ide/",":.*", "*.zip", 2, 10000, 0);
	local nCount = search_result:GetNumOfResult();
	local i;
	for i = 0, nCount-1 do 
		log(search_result:GetItem(i).."\n");
	end
	search_result:Release();
end

function test_excel_doc_reader()
	NPL.load("(gl)script/ide/Document/ExcelDocReader.lua");
	local ExcelDocReader = commonlib.gettable("commonlib.io.ExcelDocReader");
	local reader = ExcelDocReader:new();

	-- schema is optional, which can change the row's keyname to the defined value. 
	reader:SetSchema({
		[1] = {name="npcid", type="number"},
		[2] = {name="superclass", validate_func=function(value)  return value or "menu1"; end },
		[3] = {name="class", validate_func=function(value)  return value or "normal"; end },
		[4] = {name="class_name", validate_func=function(value)  return value or "通用"; end },
		[5] = {name="gsid", type="number" },
		[6] = {name="exid", type="number" },
		[7] = {name="money_list", },
	})
	-- read from the second row
	if(reader:LoadFile("config/Aries/NPCShop/npcshop.xml", 2)) then 
		local rows = reader:GetRows();
		echo(rows);
	end



	NPL.load("(gl)script/ide/Document/ExcelDocReader.lua");
	local ExcelDocReader = commonlib.gettable("commonlib.io.ExcelDocReader");
	local reader = ExcelDocReader:new();

	-- schema is optional, which can change the row's keyname to the defined value. 
	local function card_and_level_func(value)  
		if(value) then
			local level, card_gsid = value:match("^(%d+):(%d+)");
			return {level=level, card_gsid=card_gsid};
		end
	end
	reader:SetSchema({
		{name="gsid", type="number"},
		{name="displayname"},
		{name="max_level", type="number" },
		{name="hp", type="number" },
		{name="attack", type="number" },
		{name="defense", type="number" },
		{name="powerpips_rate", type="number" },
		{name="accuracy", type="number" },
		{name="critical_attack", type="number" },
		{name="critical_block", type="number" },
		{name="card1",  validate_func= card_and_level_func},
		{name="card2",  validate_func= card_and_level_func},
		{name="card3",  validate_func= card_and_level_func},
		{name="card4",  validate_func= card_and_level_func},
		{name="card5",  validate_func= card_and_level_func},
		{name="card6",  validate_func= card_and_level_func},
		{name="card7",  validate_func= card_and_level_func},
		{name="card8",  validate_func= card_and_level_func},
	})
	-- read from the second row
	if(reader:LoadFile("config/Aries/Others/combatpet_levels.excel.teen.xml", 2)) then 
		local rows = reader:GetRows();
		log(commonlib.serialize(rows, true));
	end
end
```

### Delete Files

See [ParaIO Reference](https://codedocs.xyz/LiXizhi/NPLRuntime/classParaScripting_1_1ParaIO.html) for more functions

```lua
ParaIO.DeleteFile("temp/*.*");
```

### Compress and Decompress with Zlib and Gzip 

- Files: [script/test/TestNPL.lua](https://github.com/NPLPackages/main/tree/master/script/test/TestNPL.lua)

```lua
-- compress/decompress test
function TestNPL.Compress()
	-- using gzip
	local content = "abc";
	local dataIO = {content=content, method="gzip"};
	if(NPL.Compress(dataIO)) then
		echo(dataIO);
		if(dataIO.result) then
			dataIO.content = dataIO.result; dataIO.result = nil;
			if(NPL.Decompress(dataIO)) then
				echo(dataIO);
				assert(dataIO.result == content);
			end
		end
	end

	-- using zlib and deflate
	local content = "abc";
	local dataIO = {content=content, method="zlib", windowBits=-15, level=3};
	if(NPL.Compress(dataIO)) then
		echo(dataIO);
		if(dataIO.result) then
			dataIO.content = dataIO.result; dataIO.result = nil;
			if(NPL.Decompress(dataIO)) then
				echo(dataIO);
				assert(dataIO.result == content);
			end
		end
	end
end
```

### Running Shell Commands
- source: [script/ide/System/os/run.lua](https://github.com/NPLPackages/main/tree/master/script/ide/System/os/run.lua)

To run external command lines via windows batch or linux bash shell in either `synchronous` or `asynchronous` mode. 

```lua
NPL.load("(gl)script/ide/System/os/run.lua");
if(System.os.GetPlatform()=="win32") then
	-- any lines of windows batch commands
	echo(System.os("dir *.exe \n svn info"));
	-- this will popup confirmation window, so there is no way to get its result. 
	System.os.runAsAdmin('reg add "HKCR\\paracraft" /ve /d "URL:paracraft" /f');
else
	-- any lines of linux bash shell commands
	echo(System.os.run("ls -al | grep total\ngit | grep commit"));
end
-- async run command in worker thread
for i=1, 10 do
	System.os.runAsync("echo hello", function(err, result)   echo(result)  end);
end
echo("waiting run async reply ...")
```
> please note:
- windows and linux have different default shell program. One may need to use `System.os.GetPlatform()=="win32"` to target code to a given platform.
- We can write very long multi-line commands. Internally a temporary shell script file is created and executed with IO redirection. 
- Sometimes, we may prefer async API to prevent stalling the calling thread, such as NPL web server invoking some backend programs for image processing. The async API pushes a message to a processor queue and returns immediately, one can use one or more NPL threads to process any number of queued shell commands. For more details, please see doc in `System.os.runAsync` source file.
- You need to have permissions to run these scripts. Under window 10 or above, we provide `System.os.runAdmin` to automatically popup confirmation dialog to run as administrator. Under linux, there is nothing we can do, you must give execute permission to `./temp` folder. This is the case for `selinux` core. 

### Reading Image File

```lua
function test_reading_image_file()
	-- reading binary image file
	-- png, jpg format are supported. 
	local filename = "Texture/alphadot.png";
	local file = ParaIO.open(filename, "image");
	if(file:IsValid()) then
		local ver = file:ReadInt();
		local width = file:ReadInt();
		local height = file:ReadInt();
		-- how many bytes per pixel, usually 1, 3 or 4
		local bytesPerPixel = file:ReadInt();
		echo({ver, width=width, height = height, bytesPerPixel = bytesPerPixel})
		local pixel = {};
		for y=1, height do
			for x=1, width do
				pixel = file:ReadBytes(bytesPerPixel, pixel);
				echo({x, y, rgb=pixel})
			end
		end
		file:close();
	end
end
``` 