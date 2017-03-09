# NPL Admin Site Framework

`WebServer/admin` is a open source NPL-based web site framework. It comes with the [`main` package](https://github.com/NPLPackages/main/tree/master/script/apps/WebServer/admin/) and contains all source code of [[NPLCodeWiki]]. It is served as a demo and debugger for your own web application. It is also by default run side by side on `127.0.0.1:8099/` with your website on `localhost:8099`. See [default.webserver.config.xml](https://github.com/NPLPackages/main/blob/master/script/apps/WebServer/default.webserver.config.xml).


## How to run NPL Admin Site
```
npl script/apps/WebServer/WebServer.lua
```
or you can run with more options like
```
npl bootstrapper="script/apps/WebServer/WebServer.lua"  port="8099"
```
Once started, you can visit the admin site from [[http://localhost:8099]]

## Introduction
I have implemented it according to the famous `WordPress.org` blog site. The default theme template is based on "sensitive" which is a free theme of wordpress. "sensitive" theme has no dependency on media and can adjust display and layout according to screen width, which is suitable for showing on mobile phone device. 

## How to use
### Usage 1: Use by reference (Recommended)
Basically, if you want to use admin framework without cloning the file, simply add following line to your rule file
```xml
    <!--wp framework related js, css, files-->
    <rule match="^/?wp%-" with="WebServer.filehandler" params='{baseDir = "script/apps/WebServer/admin/"}'></rule>
```
In your page `index.page` file, you can include the main file of the wp framework, such as this: 
```lua
<?npl

-- we will not load complete framework, but only ajax and helper functions
WP_USE_MINI_LOADER = true;
include_once("script/apps/WebServer/admin/wp-main.page");

-- call your router, such as   
include_once("./myproj/routes.page");
```

### Usage 2: Use by cloning
   * copy everything in this folder to your own web site root, say /www/MySite/
   * modify wp-content/database/*.xml for site description and menus. 
   * add your own web pages to wp-content/pages/, which are accessed by their filename in the url.
   * If you want more customization to the look, modify the wp-content/themes/sensitive or create your own theme folder. Remember to set your theme in `wp-content/database/table_sitemeta.xml`, which contains all options for the site. 

### Usage 3: Adding new features to it
   * Simply put your own files in `script/apps/WebServer/admin/` in your dev or working directory, your files will be loaded first before the one in `npl_package/main`. 

## Architecture
	The architecture is based on Wordpress.org (4.0.1). Although everything is rewritten in NPL, I have kept all functions, filters, and file names identical to wordpress. 
	See `wp-includes/` for the framework source code.

Code locations:
	* framework loader is in `wp-settings.page`
	* site options: `wp-content/database/table_sitemeta.xml`: such as theme, default menu, etc.
	* menus: `wp-content/database/table_nav_menu.xml`

## Ajax framework
	Any request url begins with `ajax/xxx` use the `wp-admin/admin-ajax.page`. it will automatically load the page xxx.  and invoke `do_action('wp_ajax_xxx')`. 
	If the request url begins with `ajax/xxx?action=yyy`, then page xxx is loaded, and `do_action('wp_ajax_xxx')` is invoked. 
	A page that handles ajax request needs to call `add_action('wp_ajax_xxx'， function_name)` to register a handler for ajax actions. 
	see `wp-content/pages/aboutus.page` for an example. 