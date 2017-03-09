> We highly recommend the native database system in NPL: [[TableDatabase|UsingTableDatabase]]. It is way faster and built-in with NPL runtime. 

# Using MySql Client

### Install Guide
* On linux platform, mysql client is by default installed when you build NPLRuntime from source code. You should find a `libluasql.so` in NPL runtime directory.
* On windows platform, you must manually install the mysql client connector plugin.
> [[https://github.com/LiXizhi/luasql]]

Once installed, you can use it like below:
```lua
NPL.load("(gl)script/ide/mysql/mysql.lua");
local MySql = commonlib.gettable("System.Database.MySql");
local mysql_db = MySql:new():init(db_user, db_password, db_name, db_host, db_port);
```

### A Better Way to Code
It is recommended that you write a wrapper file like `my_db.page` which exposes a global object such as `my_db` that  contains functions to access to your actual database. 

An example in the Admin web site framework is here. 
 * `script/apps/WebServer/admin/wp-includes/wp-db.page`

It will manage connection strings (passwords, ips, pools) from global configuration files, etc. 
In all other server pages, you simply include the file, and call its functions like this
```lua
local query = string.format("select * from wp_users where %s = '%s' ",db_field,value);
local user = wpdb:get_row(query);
```

