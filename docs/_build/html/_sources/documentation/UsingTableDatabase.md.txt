# Table Database
`Table database` is implemented in NPL and is the default and recommended database engine. It can be used without any external dependency or configuration. 

## Introduction to Table Database
A schema-less, server-less, NoSQL database able to process big data in multiple NPL threads, with extremly intuitive table-like API, 
automatic indexing and extremely fast searching, keeping data and algorithm in one system *just like how the human brain works*.

## Local Storage API vs Full-Featured Database Solution 
The raw table database has no configuration file. Please regard the raw table database as a local storage API provided by NPL, rather than a full-featured database solution. However its performance is good enough to be used as the database engine for medium sized projects. You need to write application-level code to have something like authentication, and data replication/hashing on multiple computers, etc. However, in future, some of these functions may be provided as additional libraries on top of the raw table database to provide enterprise level database solution that runs on multiple nodes.  

## Performance
Following is tested on my Intel-i7-3GHZ CPU. See [test folder](https://github.com/NPLPackages/main/blob/master/script/ide/System/Database/test/test_TableDatabase.lua) for test source code.

### Run With Conservative Mode
Following is averaged value from 100000+ operations in a single thread

* Random non-index insert: `43478 inserts/second`
   * Async API tested with default configuration with 1 million records on a single thread.
* Round trip latency call: 
   * Blocking API: `20000 query/s`
   * Non-blocking API: `11ms` or `85 query/s` (due to NPL time slice)
   * i.e. Round strip means start next operation after previous one is returned. This is latency test.
* Random indexed inserts: `17953 query/s`
   * i.e. start next operation immediately, without waiting for the first one to return.
* Random select with auto-index: `18761 query/s`
   * i.e. same as above, but with findOne operation. 
* Randomly mixing CRUD operations: `965-7518` query/s
   * i.e. same as above, but randomly calling Create/Read/Update/Delete (CRUD) on the same auto-indexed table.
   * Mixing read/write can be slow when database grows bigger. e.g. you can get `18000 CRUD/s` for just 10000 records. 

### Run With Aggressive Mode
One can also use in-memory journal file or ignore OS disk write feedback to further increase DB throughput by 30-100% percent.
See `Store.lua` for details. By default, this feature is off.

Code Examples:
```lua
	NPL.load("(gl)script/ide/System/Database/TableDatabase.lua");
	local TableDatabase = commonlib.gettable("System.Database.TableDatabase");
	-- this will start both db client and db server if not.
	local db = TableDatabase:new():connect("temp/mydatabase/", function() end);
	
	-- Note: `db.User` will automatically create the `User` collection table if not.
	-- clear all data
	db.User:makeEmpty({}, function(err, count) echo("deleted"..(count or 0)) end);
	-- insert 1
	db.User:insertOne(nil, {name="1", email="1@1",}, function(err, data)  assert(data.email=="1@1") 	end)
	-- insert 1 with duplicate name
	db.User:insertOne(nil, {name="1", email="1@1.dup",}, function(err, data)  assert(data.email=="1@1.dup") 	end)
	
	-- find or findOne will automatically create index on `name` and `email` field.
	-- indices are NOT forced to be unique. The caller needs to ensure this see `insertOne` below. 
	db.User:find({name="1",}, function(err, rows) assert(#rows==2); end);
	db.User:find({name="1", email="1@1"}, function(err, rows) assert(rows[1].email=="1@1"); end);
	-- find with compound index of name and email
	db.User:find({ ["+name+email"] = {"1", "1@1"} }, function(err, rows) assert(#rows==1); end);
	
	-- force insert
	db.User:insertOne(nil, {name="LXZ", password="123"}, function(err, data)  assert(data.password=="123") 	end)
	-- this is an update or insert command, if the query has result, it will actually update first matching row rather than inserting one. 
	-- this is usually a good way to force uniqueness on key or compound keys, 
	db.User:insertOne({name="LXZ"}, {name="LXZ", password="1", email="lixizhi@yeah.net"}, function(err, data)  assert(data.password=="1") 	end)

	-- insert another one
	db.User:insertOne({name="LXZ2"}, {name="LXZ2", password="123", email="lixizhi@yeah.net"}, function(err, data)  assert(data.password=="123") 	end)
	-- update one
	db.User:updateOne({name="LXZ2",}, {name="LXZ2", password="2", email="lixizhi@yeah.net"}, function(err, data)  assert(data.password=="2") end)
	-- remove and update fields
	db.User:updateOne({name="LXZ2",}, {_unset = {"password"}, updated="with unset"}, function(err, data)  assert(data.password==nil and data.updated=="with unset") end)
	-- replace the entire document
	db.User:replaceOne({name="LXZ2",}, {name="LXZ2", email="lixizhi@yeah.net"}, function(err, data)  assert(data.updated==nil) end)
	-- force flush to disk, otherwise the db IO thread will do this at fixed interval
    db.User:flush({}, function(err, bFlushed) assert(bFlushed==true) end);
	-- select one, this will automatically create `name` index
	db.User:findOne({name="LXZ"}, function(err, user) assert(user.password=="1");	end)
	-- array field such as {"password", "1"} are additional checks, but does not use index. 
	db.User:findOne({name="LXZ", {"password", "1"}, {"email", "lixizhi@yeah.net"}}, function(err, user) assert(user.password=="1");	end)
	-- search on non-unqiue-indexed rows, this will create index `email` (not-unique index)
	db.User:find({email="lixizhi@yeah.net"}, function(err, rows) assert(#rows==2); end);
	-- search and filter result with password=="1"
	db.User:find({name="LXZ", email="lixizhi@yeah.net", {"password", "1"}, }, function(err, rows) assert(#rows==1 and rows[1].password=="1"); end);
	-- find all rows with custom timeout 1 second
	db.User:find({}, function(err, rows) assert(#rows==4); end, 1000);
	-- remove item
	db.User:deleteOne({name="LXZ2"}, function(err, count) assert(count==1);	end);
	-- wait flush may take up to 3 seconds
	db.User:waitflush({}, function(err, bFlushed) assert(bFlushed==true) end);
	-- set cache to 2000KB
	db.User:exec({CacheSize=-2000}, function(err, data) end);
	-- run select command from Collection 
	db.User:exec("Select * from Collection", function(err, rows) assert(#rows==3) end);
	-- remove index fields
	db.User:removeIndex({"email", "name"}, function(err, bSucceed) assert(bSucceed == true) end)
	-- full table scan without using index by query with array items.
	db.User:find({ {"name", "LXZ"}, {"password", "1"} }, function(err, rows) assert(#rows==1 and rows[1].name=="LXZ"); end);
	-- find with left subset of previously created compound key "+name+email"
	db.User:find({ ["+name"] = {"1", limit=2} }, function(err, rows) assert(#rows==2); end);
	-- return at most 1 row whose id is greater than -1
	db.User:find({ _id = { gt = -1, limit = 1, skip == 1} }, function(err, rows) assert(#rows==1); echo("all tests succeed!") end);
```

## Why A New Database System?
Current SQL/NoSQL implementation can not satisfy following requirements at the same time.
* Keeping data close to computation, much like our brain. 
* Store arbitrary data without schema. 
* Automatic indexing based on query usage.
* Provide both blocking/non-blocking API.
* Multithreaded architecture without using network connections for maximum local performance.
* Capable of storing hundreds of Giga Bytes of data locally.
* Native document storage format for NPL tables. 
* Super easy client API just like manipulating standard NPL/lua tables.
* Easy to setup and deploy with NPL runtime.
* No server configuration, calling client API will automatically start the server on first use. 

## About Transactions
Each `write/insert` operation is by default a write command (virtual transaction). 
We will periodically (default is 50ms, see `Store.AutoFlushInterval`) flush all queued commands into disk. 
Everything during these period will either succeed or fail. 
If you are worried about data lose, you can manually invoke `flush` command, however doing so 
will greatly compromise performance. Please note `flush` command will affect the overall throughput of the entire DB system.
In general, you can only get about `20-100` flush(real transactions) per second. Without enforcing transaction on each command, you can easily get a throughput of `6000` write commands per second (i.e. could be `100 times` faster). 

> There is one solution to get both high throughput and transaction.

The following solution give you ACID of a single command. After issuing a really important group of commands, and you want to ensure that these commands are actually successful like a transaction, the client can issue a `waitflush` command to check if the previous commands are successful. Please note that `waitflush` command may take up to 50ms or `Store.AutoFlushInterval` to return. You can make all calls asynchronous, so 99.99% times user get a fast feed back, but there is a very low chance that user may be informed of failure after 50ms. On client side, you may also need to prevent user to issue next transaction in 50ms,In most cases, users do not click mouse that quick, so this hidden logic goes barely noticed.

The limitation of the above approach is that you can only know whether a group of commands has been successfully flushed to disk, but you can not rollback your changes, nor can you know which commands fail and which are successful in case of multiple commands.  

## Opinion on Software Achitecture
Like the brain, we recommend that each computer manages its own chunk of data. 
As modern computers are having more computing cores, it is possible for a single (virtual) machine 
to manage 100-500GB of data locally or 1000 requests per second. Using a local database engine is the best choice 
in such situation for both performance and ease of deployment. 

To scale up to even more data and requests, we devide data with machines and use higher level
programming logics for communications. In this way, we control all the logics with our own code, 
rather than using general-purpose solutions like memcached, MangoDB(NoSQL), or SQLServer, etc. 

## Implementation Details
* Each data table(collections of data) is stored in a single sqlite database file.
* Each database file contains a indexed mapping from object-id to object-table(document). 
* Each database file contains a schema table telling additional index keys used. 
* Each database file contains a key to object-id table for each custom index key. 
* All DB IO operations are performaned in a single dedicated NPL thread. 

The above general idea can also be found in commercial database engine like [CortexDB](http://cortex-ag.com/), see below.
![tabledb](https://cloud.githubusercontent.com/assets/94537/15766809/45a99964-2975-11e6-8e02-0ee797b1d4ce.png)


## User Guide
Because Table database is schema-less.  It requires some caution of the programmer during development. 

### Auto Key Index
By default, when you are inserting records into the database, it will have only one internal unique key called `_id`.
All subsequent query commands such as `findOne` or `find` or `insertOne`, `deleteOne`, `updateOne` command with query fields, such as `name` will automatically add a new index key with corresponding name such as `name`, and rebuild the index table for all existing records. Please note, all indices are NOT-constraint to be unique, it is up to the caller to ensure index usage. Different records with the same key value are all stored in the index table. 

> One can also force not-to-use index when querying by using array items rather than key,value pairs in the query parameter, see below. 

```lua
	NPL.load("(gl)script/ide/System/Database/TableDatabase.lua");
	local TableDatabase = commonlib.gettable("System.Database.TableDatabase");
	-- this will start both db client and db server if not.
	local db = TableDatabase:new():connect("temp/mydatabase/", function() end);
	
	-- Note: `db.User` will automatically create the `User` collection table if not.
	-- clear all data
	db.User:makeEmpty({}, function(err, count) echo("deleted"..(count or 0)) end);
	-- insert 1
	db.User:insertOne(nil, {name="1", email="1@1",}, function(err, data)  echo(data) 	end)
	-- insert 1 with duplicate name
	db.User:insertOne(nil, {name="1", email="1@1.dup",}, function(err, data)  echo(data) 	end)
	
	-- find or findOne will automatically create index on `name` and `email` field.
	-- indices are NOT forced to be unique. The caller needs to ensure this see `insertOne` below. 
	-- this will return two records using one index table `name`
	db.User:find({name="1",}, function(err, rows) echo(rows); end);
	-- this will return one record using two index table `name` and `email`
	db.User:find({name="1", email="1@1"}, function(err, rows) echo(rows); end);
	
	-- force insert, insertOne with no query parameter will force insertion.
	db.User:insertOne(nil, {name="LXZ", password="123"}, function(err, data)  echo(data) 	end)
	-- this is an update or insert command, if the query has result, it will actually update first matching row rather than inserting one. 
	-- this is usually a good way to force uniqueness on key or compound keys 
	-- so the following will update existing record rather than inserting a new one
	db.User:insertOne({name="LXZ"}, {name="LXZ", password="1", email="lixizhi@yeah.net"}, function(err, data)  echo(data) 	end)
```

All query fields in `findOne`, `updateOne`, etc can contain array items, which performs additional filter checks without automatically create index, like below. `name` is an indexed key, where as `password` and `email` are not, so they need be specified as array items in the query parameter. 

```lua
	-- array fields such as {"password", "1"} are additional checks, but does not use index. 
	db.User:findOne({name="LXZ", {"password", "1"}, {"email", "lixizhi@yeah.net"}}, function(err, user) echo(user);	end)
```
In case, you make a mistake with index during development, please use `DB manager` in NPL code wiki to remove any index that you accidentally create. or simply call `removeIndex` to remove all or given index keys. 

#### Force Key Uniqueness and `Upsert`

If you want to force uniqueness on a given key, one can specify the unique key in query field when inserting a new record, which turns the `insertOne` command into an `Upsert` command. See below:
```lua
-- this is an update or insert command, if the query has result, it will actually update first matching row rather than inserting one. 
-- this is usually a good way to force uniqueness on key or compound keys 
-- so the following will update existing record rather than inserting a new one
db.User:insertOne({name="LXZ"}, {name="LXZ", password="1", email="lixizhi@yeah.net"}, function(err, data)  echo(data) 	end)
```

Some notes:
- Query commands such as `find`, `findOne`, `insertOne` will automatically create index for fields in query parameter, unless field is specified in array format. 
- Query with mixed index and non-index is done by fetching using index first and then filter result with non-indexed fields. If there is no indexed fields, then non-indexed query can be very slow, since it will linearly search on all records, which can be very slow. 
- Query with multiple fields is supported. It will first fetch ids using each index field, and calculate the shared ids and then fetch these rows in a batch. 

### Write-Ahead-Log and Checkpointing
By default, TableDatabase uses [write-ahead-log](https://www.sqlite.org/wal.html). 
We will flush data to WAL file every 50ms; and we will do WAL checkpoint every 5 seconds or 1000 dirty pages. 
One can configure these two intervals, like below. However, it is not recommended to change these values.
```lua
NPL.load("(gl)script/ide/System/Database/Store.lua");
local Store = commonlib.gettable("System.Database.Store");
-- We will wait for this many milliseconds when meeting the first non-queued command before commiting to disk. So if there are many commits in quick succession, it will not be IO bound. 
Store.AutoFlushInterval = 50;
Store.AutoCheckPointInterval = 5000;
```
> Checkpointing may take 10ms-1000ms depending on usage, during which time it will block all write operations, so one may experience a latency in API once in a while.

### Message Queue Size
One can set the message queue size for both the calling thread and db processor thread. 
The default value is good enough for most situations. 
```lua
db.name:exec({QueueSize=10001});
```

### Memory Cache Size
Change the suggested maximum number of database disk pages that TableDatabase will hold in memory at once per open database file. The default suggested cache size is -2000, which means the cache size is limited to 2048KB of memory. You can set it to a much bigger value, since it is only allocated on demand. Negative value is KB, Positive is number of pages, each page is 4KB. 
```lua
db.name:exec({CacheSize=-2000});
```

### Bulk Operations and Message Deliver Guarantee
Table database usually has a high throughout such as 5000-40000 requests/sec. However, if you are doing bulk insertion of millions of records. You should do chunk by chunk using callbacks, otherwise you will either reach the limit of NPL thread buffer or TCP socket buffer(in case it is remote process).  A good way is to keep at most 1000 active requests at any one time (`System.Concurrency` has helper class for this kind of jobs). This way you can get a good throughput with message delivery guarantees.

Following is example of inserting 1 million records (takes about 23 seconds). You can paste following code to NPL Code Wiki's console to test.
```lua
	NPL.load("(gl)script/ide/System/Database/TableDatabase.lua");
	local TableDatabase = commonlib.gettable("System.Database.TableDatabase");

        -- this will start both db client and db server if not.
	local db = TableDatabase:new():connect("temp/mydatabase/");
	
	db.insertNoIndex:makeEmpty({});
	db.insertNoIndex:flush({});
		
	NPL.load("(gl)script/ide/Debugger/NPLProfiler.lua");
	local npl_profiler = commonlib.gettable("commonlib.npl_profiler");
	npl_profiler.perf_reset();

	npl_profiler.perf_begin("tableDB_BlockingAPILatency", true)
	local total_times = 1000000; -- a million non-indexed insert operation
	local max_jobs = 1000; -- concurrent jobs count
	NPL.load("(gl)script/ide/System/Concurrent/Parallel.lua");
	local Parallel = commonlib.gettable("System.Concurrent.Parallel");
	local p = Parallel:new():init()
	p:RunManyTimes(function(count)
		db.insertNoIndex:insertOne(nil, {count=count, data=math.random()}, function(err, data)
			if(err) then
				echo({err, data});
			end
			p:Next();
		end)
	end, total_times, max_jobs):OnFinished(function(total)
		npl_profiler.perf_end("tableDB_BlockingAPILatency", true)
		log(commonlib.serialize(npl_profiler.perf_get(), true));			
	end);
```
> In future version, we may support build-in Bulk API.

### Async vs Sync API
Table database provides both sync and asynchronous API, and they can be use simultaneously. 
However, sync interface is disabled by default. One has to manually enable it, such as during initialization.
In sync interface, if you do not provide a callback function, then the API block until result is returned, otherwise the API return AsyncTask object immediately. See following example:

```lua
-- enable sync mode once and for all in current thread.
db:EnableSyncMode(true);
-- synchronous call will block until data is fetched. 
local err, data = db.User:insertOne(nil, {name="LXZ2", email="sync mode"})
-- async call return a task object immediately without waiting result. 
local task = db.User:insertOne(nil, {name="LXZ2", email="sync mode"}, function(err, data) end)
```

Synchronous call will pause the entire NPL thread, and is `NOT` the recommended to use, that is why we disabled this feature by default. 

Instead, during web development, there is another way to use async API in synchronous way, which is to use coroutine.
In NPL web server, we provide `resume/yield`. See [[NPLServerPage]]'s mixing sync/async code section. 

### Ranged Query and Pagination
Paging through your data is one of the most common operations with TableDB. A typical scenario is the need to display your results in chunks in your UI. If you are batch processing your data it is also important to get your paging strategy correct so that your data processing can scale.

Let’s walk through an example to see the different ways of paging through data in TableDB. In this example, we have a database of user data that we need to page through and display 10 users at a time. So in effect, our page size is 10. Let us first create our db with 10000 users of scheme `{_id, name, company, state}`.

```lua
	NPL.load("(gl)script/ide/System/Database/TableDatabase.lua");
	local TableDatabase = commonlib.gettable("System.Database.TableDatabase");
	local db = TableDatabase:new():connect("temp/mydatabase/");	

	-- add some data
	for i=1, 10000 do
		db.pagedUsers:insertOne({name="name"..i}, 
			{name="name"..i, company="company"..i, state="state"..i}, function() end)
	end
```
#### Approach One: Use `limit` and `skip`
TableDB support `limit` and `offset|skip` key words in query parameter on an indexed field or the internal `_id` field.
`offset or skip` tells TableDB to skip some items before returning at most `limit` number of items. `gt` means `greater than` to select a given range of data.

```lua
	-- page 1
	db.pagedUsers:find({_id = {gt=-1, limit=10, skip=0}}, function(err, users)  echo(users) end);
	-- page 2
	db.pagedUsers:find({_id = {gt=-1, limit=10, skip=10}}, function(err, users)  echo(users) end);
	-- page 3
	db.pagedUsers:find({_id = {gt=-1, limit=10, skip=20}}, function(err, users)  echo(users) end);
```
You get the idea. In general to retrieve page `n` the code looks like this

```lua
	local pagesize = 10; local n = 10;
	db.pagedUsers:find({_id = {gt=-1, limit=pagesize, skip=(n-1)*pagesize}}, function(err, users)  echo(users) end);
```
However as the `offset` of your data increases this approach has serious performance problems. The reason is that every time the query is executed, the server has to walk from the beginning of the first non-skipped row to the specified offset. As your offset increases this process gets slower and slower.  Also this process does not make efficient use of the indexes.  So typically the `skip` and `limit` approach is useful when you have small `offset`. If you are working with large data sets with very big `offset` values you need to consider other approaches.

#### Approach Two: Using `find` and `limit`
The reason the previous approach does not scale very well is the `skip` command. So the goal in this section is to implement paging without using the `skip` command. For this we are going to leverage the natural order in the stored data like a time stamp or an id stored in the document. In this example we are going to use the `_id` stored in each document. `_id` is a TableDB auto-increased internal id.

```lua
	-- page 1
	db.pagedUsers:find({_id = { gt = -1, limit=10}}, function(err, users)  
		echo(users)
		-- Find the id of the last document in this page
		local last_id = users[#users]._id;

		-- page 2
		db.pagedUsers:find({_id = { gt = last_id, limit=10}}, function(err, users)  
		   echo(users)
		   -- Find the id of the last document in this page
		   local last_id = users[#users]._id;
		   -- page 3
		   -- ...
		end);
	end);
```

Notice how the `gt` keywords to select rows whose values is `greater than` the specified value and sort data in ascending order. One can do this for any indexed field, either it is number or string, see below for ranged query in TableDB:

```lua
	NPL.load("(gl)script/ide/System/Database/TableDatabase.lua");
	local TableDatabase = commonlib.gettable("System.Database.TableDatabase");

	-- this will start both db client and db server if not.
	local db = TableDatabase:new():connect("temp/mydatabase/");	

	-- add some data
	for i=1, 100 do
		db.rangedTest:insertOne({i=i}, {i=i, data="data"..i}, function() end)
	end

	-- return at most 5 records with i > 90, skipping 2. result in ascending order
	db.rangedTest:find({ i = { gt = 95, limit = 5, offset=2} }, function(err, rows)
		echo(rows); --> 98,99,100
	end);

	-- return at most 20 records with _id > 98, result in ascending order
	db.rangedTest:find({ _id = { gt = 98, limit = 20} }, function(err, rows)
		echo(rows); --> 99,100
	end);

	-- do a full table scan without using index
	db.rangedTest:find({ {"i", { gt = 55, limit=2, offset=1} }, {"data", {lt="data60"} }, }, function(err, rows)
		echo(rows); --> 57,58
	end);

	db.rangedTest:find({ {"i", { gt = 55} }, {"data", "data60"}, }, function(err, rows)
		echo(rows); --> 60
	end);
```
> Please note that array fields in query object will not auto-create or use any index, instead a full table scan is performed which can be slow. Ranged query is also supported in array fields, but it is implemented via full table scan. 

> Compound key is more suitable for ranged query see Compound keys section below for how to use it. 

#### Count Record 
> Rule of thumb: avoid using `count`!

Implementation of full paging usually requires you to get the count of all items in a database. However, it is not trivial to get accurate count. SQL query like `select count(*) as count from Collection` may take seconds or even minutes to return for tables with many rows like 100 millions. This is because almost every SQL engine uses B-tree, but B-tree does not store count. The same story is true for both TableDB, MySQL, etc. 

For a faster count, you can create a counter table and let your application update it according to the inserts and deletes it does. Or you may not allow the user to scroll to the last page, but show more pages progressively. 

If you do not mind the performance or your table size is small, TableDB provides the `count` query
```lua
	NPL.load("(gl)script/ide/System/Database/TableDatabase.lua");
	local TableDatabase = commonlib.gettable("System.Database.TableDatabase");
	local db = TableDatabase:new():connect("temp/mydatabase/");	
	
	db.countTest:removeIndex({}, function(err, bRemoved) end);

	-- compound keys
	for i=1, 100 do
		db.countTest:insertOne({name="name"..i}, { 
			name="name"..i, 
			company = (i%2 == 0) and "tatfook" or "paraengine", 
			state = (i%3 == 0) and "china" or "usa"}, function() end)
	end

	-- count all rows
	db.countTest:count({}, function(err, count)  
		assert(count == 100) 
	end);
	-- count with compound keys
	db.countTest:count({["+company"] = {"tatfook"}}, function(err, count)  
		assert(count == 50) 
	end);
	-- count with complex query
	db.countTest:count({["+state+name+company"] = {"china", gt="name50"}}, function(err, count)  
		assert(count == 19) 
	end);
```

### Compound Indices
To efficiently query data with multiple key values, one needs to use compound indices. 
Please note a compound key of (key1, key2, key3) can also be used to query (key1, key2) and (key1), so the order of subkeys are very important. 

If you created a compound index say on (key1, key2, key3), then only the right most key in the query can contain ranged constraint, all other keys to its left such as (key1 and key2) must be present (no gaps) and specified in equal constraint. 

To understand compound indices, one needs to understand how index works in standard relational database engine, because a TableDB query usually use only one internal index to do all the complex queries and an index is a B-tree. So compound index can not be used where B-tree is not capable of. Read query planner of sqlite [here](https://www.sqlite.org/optoverview.html) for more details. 

The syntax of using compound key is by prepending "+|-" to one or more field names, such as "+key1+key2+key3", there can be at most 4 sub keys, see below for examples. "+" means ascending order, "-" means descending order. 

Examples:
```lua
	NPL.load("(gl)script/ide/System/Database/TableDatabase.lua");
	local TableDatabase = commonlib.gettable("System.Database.TableDatabase");
	local db = TableDatabase:new():connect("temp/mydatabase/");	
	
	db.compoundTest:removeIndex({}, function(err, bRemoved) end);

	-- compound keys
	for i=1, 100 do
		db.compoundTest:insertOne({name="name"..i}, { 
			name="name"..i, 
			company = (i%2 == 0) and "tatfook" or "paraengine", 
			state = (i%3 == 0) and "china" or "usa"}, function() end)
	end

	-- compound key can also be created on single field "+company". it differs from standard key "company". 
	db.compoundTest:find({["+company"] = {"paraengine", limit=5, skip=3}}, function(err, users)  
		assert(#users == 5)
	end);

	-- create a compound key on +company+name (only the right most key can be paged)
	-- `+` means index should use ascending order, `-` means descending. such as "+company-name"
	-- it will remove "+company" key, since the new component key already contains the "+company". 
	db.compoundTest:find({["+company+name"] = {"tatfook", gt="", limit=5, skip=3}}, function(err, users)  
		assert(#users == 5)
	end);

	-- a query of exact same left keys after a compound key is created will automatically use 
	-- the previously created compound key, so "+company",  "+company+name" in following query are the same.
	db.compoundTest:find({["+company"] = {"tatfook", limit=5, skip=3}}, function(err, users)  
		assert(#users == 5)
	end);

	-- the order of key names in compound index is important. for example:
	-- "+company+state+name" shares the same compound index with "+company" and "+company+state"
	-- "+state+name+company" shares the same compound index with "+state" and "+state+name"
	db.compoundTest:find({["+state+name+company"] = {"china", gt="name50", limit=5, skip=3}}, function(err, users)  
		assert(#users == 5)
	end);

	-- compound keys with descending order. Notice the "-" sign before `name`.
	db.compoundTest:find({["+state-name+company"] = {"china", limit=5, skip=3}}, function(err, users)  
		assert(#users == 5)
	end);

	-- updateOne with compound key
	db.compoundTest:updateOne({["+state-name+company"] = {"usa", "name1", "paraengine"}}, {name="name0_modified"}, function(err, user)  
		assert(user.name == "name0_modified")
	end);

	-- this query is ineffient since it uses intersection of single keys (with many duplications). 
	-- one should consider use "+state+company", instead of this. 
	db.compoundTest:find({state="china", company="tatfook"}, function(err, users)  
		assert(#users == 16)
	end);

	-- this query is ineffient since it uses intersection of single keys. 
	-- one should consider use "+state+company", instead of this. 
	db.compoundTest:count({state="china", company="tatfook"}, function(err, count)  
		assert(count == 16)
	end);
```

### Compound key vs Standard key

Compound key is very different from standard key in terms of internal implementations. They can coexist in the same table even for the same key. For example:
```lua
-- query with standard key "name"
db.User:find({name="1",});
-- query with compound key "+name"
db.User:find({["+name"]="1",});
```
The above code will create two different kinds of internal index tables for the "name" field. 

Compound key index table is more similar to traditional relational database table. For example, if one creates a compound key of "+key1+key2+key3", then an internal index table of `(cid UNIQUE, name1, name2, name3)` is created with composite index of `CREATE INDEX (name1, name2, name3)`. For each row in the collection, there is a matching row in its internal index table. 

However, a standard key in tableDB stores index table differently as `(name UNIQUE, cids)`, where cids is a text array of collection `cid`. There maybe far less rows in the internal index table than the actual collection. 

Both keys have advantages and disadvantages, generally speaking:
- standard key is faster and good for index intersections. Most suitable for keys with 1-1000 duplications. 
- compound key is good for sorting and ranged query. Suitable for keys with any duplicated rows.

The biggest factor that you should use a compound key is that your key's values have many duplications. 
For example, if you want to query with `gender`+`age` with millions of `user` records, compound key is the only way to go. 

For other single key situations, standard key is recommended. For example, with standard keys you can get all `projects` 
that belongs to a given `userid` very efficiently, given that each user has far less than 1000 projects.  

### Database Viewer Tools
[[NPLCodeWiki]] contains a `db manager` in its tools menu, which can be used to view and modify data, as well as examine and removing `table indices`. It is very important to examine and remove unnecessary indices due to coding mistakes. 

![image](https://cloud.githubusercontent.com/assets/94537/19043995/0451b770-89c5-11e6-9eed-12ea7faa4302.png)

### Remove Table Fields
To remove table fields, use `updateOne` with `_unset` query parameter like below.
```lua
    -- remove "password" field from the given row
    db.User:updateOne({name="LXZ2",}, {_unset = {"password"}, updated="with unset"}, 
       function(err, data)        
          assert(data.password==nil and data.updated=="with unset") 
    end)
```

Another way is the use `replaceOne` to replace the entire document. 
```lua
-- replace the entire document
db.User:replaceOne({name="LXZ2",}, {name="LXZ2", email="lixizhi@yeah.net"}, function(err, data)  assert(data.updated==nil) end)
```
### Always Use Models
In typical web applications, we use model/view/controller (or MVC) architecture. 
A model is usually a wrapper of a single schema-less database record in TableDatabase. 
Usually a model should provide CRUD(create/read/update/delete) operations according to user-defined query. 
It is the model's responsibility to ensure input/output are valid, such as whether a string is too long, or whether the caller is querying with a non-indexed key. 

In case, you are using client side controllers, one can use a router page to automatically redirect URL AJAX queries to the model's function calls. 

For a complete example of url redirection and model class, please see `script/apps/WebServer/admin/wp-content/pages/wiki/routes.page` and `script/apps/WebServer/admin/wp-content/pages/models/abstract/base.page`

Following code illustrates the basic ideas.

Router page such as in `routes.page`
```lua
		local model = models and models[modelname];
		if(not model) then
			return response:status(404):send({message="model not found"});
		else
			model = model:new();
		end

		-- redirect CRUB URL to method in model.
		if(request:GetMethod() == "GET") then
			if(model.get) then
				local result = model:get(request:getparams());
				return response:send(result);
			end
		elseif(request:GetMethod() == "PUT") then
			if(params and params:match("^new")) then
				if(model.create) then
					local result = model:create(request:getparams());
					return response:send(result);
				end
			else
				if(model.update) then
					local result = model:update(request:getparams());
					return response:send(result);
				end
			end
		elseif(request:GetMethod() == "DELETE") then
			if(model.delete) then
				local result = model:delete(request:getparams());
				return response:send(result);
			end
		end
```

And in model class, such as `models/site.page`
```lua
<?npl
--[[
Title: a single web site of a user
Author: LiXizhi
Date: 2016/6/28
]]
include_once("base.page");

local site = inherit(models.base, gettable("models.site"));

site.db_name = "site";

function site:ctor()
	-- unique name of the website, usually same as owner name
	self:addfield("name", "string", true, 30);
	-- markdown text description
	self:addfield("desc", "string");
	-- such as "https://github.com/LiXizhi/wiki"
	self:addfield("store", "string", false, 200);
	-- owner name, not unique
	self:addfield("owner", "string", false, 30);
end
```

### Data Replication
TableDB is a fully ACID and fast database (it uses [Sqlite](https://www.sqlite.org/famous.html) as its internal storage engine). We are going to support tableDB very actively as an integral part of NPL. 
NPL language itself is designed for parallelism, the current version of tableDB is still a local SQL engine. 
It is hence not hard to use NPL to leverage TableDB to a fully distributed and fault-tolerant database system. 

### TODO: RAFT implementation coming in 2017
[[https://raft.github.io/]] is a concensus algorithm that we are going to implement using NPL and TableDB itself to leverage TableDB to support `master/slave` data replication for high-availability database systems. 

There are many existing distributed SQL server that are built over similar architecture and use [Sqlite](https://www.sqlite.org/famous.html) as the storage engine, like these
- [[http://www.actordb.com/]] 
- [[http://www.cortex-ag.com/]]
- [[https://github.com/rqlite/rqlite]]


