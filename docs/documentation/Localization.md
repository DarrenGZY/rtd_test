## Localization

There are some helper class for you to implement localization. For best practices, please see example in paracraft package, which uses `poedit` to edit and store localization strings with multi-languages.

- Helper class for translation table:
  - [script/ide/Locale.lua](https://github.com/NPLPackages/main/tree/master/script/ide/Locale.lua)
- paracraft examples:
  - [Translation](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries/Creator/Game/Common/Translation.lua)
  - [TranslationGetText](https://github.com/NPLPackages/paracraft/tree/master/script/apps/Aries/Creator/Game/Common/TranslationGetText.lua)

### Overview
UTF8 encoding should be used where ever possible in your source code or mcml files. Everything returned from NPL is also UTF8 encoded, except for native file path. 

Follow following steps:
- Use [this class](https://github.com/NPLPackages/main/tree/master/script/ide/Locale.lua) to create a global table `L` to look up for localized text with a primary key. 
- When your application start, populate `table L` with data from your localization files according to current language setting. 
- In your script code or mcml UI page files, replace any text "XXX" with `L"XXX"`
- Re-Run `Poedit` to scan for all source files and update the localization database file.
- Inform your translator to translate the `*.po` files into multiple languages and generate `*.mo` files. Or you may just use google translate or other services.
- Iterate above three steps when you insert new text in your source code. 

> Always use 
```lua
-- Right way
local text = format(L"Some text %d times", times)
``` 
instead of 
```lua
-- BAD & Wrong Way!!!
local text = L"Some text "..times..L" times"
```
for better translation because different language have different syntax. 


### Localization GetText Tools
To store your language strings, you can use plain table in script or use a third-party tool like Poedit. We have created `Poedit` plugin to support NPL code, download the [NPLgettext](https://github.com/LiXizhi/NPLgettext) tool here. 

In paracraft, there is a command called `/poedit` which does the `gettext` job automatically. 

![image](https://cloud.githubusercontent.com/assets/94537/19267027/0b8fed48-8fdf-11e6-8eb2-727e17ecb614.png)
