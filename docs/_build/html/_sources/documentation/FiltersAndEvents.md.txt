# Overview
To write modular code, a system may expose its interface in the form of virtual functions, events or filters.
- `virtual functions`: allows a derived class to hook the input and output of a single function in the base class. 
- `events`: feeds input to external functions when certain things happened. Events only hooks the input without providing an output.
- `filters`: allows external functions to form input-output chains to modify data at any point of execution. 

Use whatever patterns to allow external modifications to the input-output of your modular code. 

## Filters
Filters is a design pattern of input-output chains. Please see [script/ide/System/Core/Filters.lua](https://github.com/NPLPackages/main/blob/master/script/ide/System/Core/Filters.lua) for detailed usage.

You can simply apply a named filters at any point of execution in your code to allow other users to modify your data. 

For example, one can turn `data = data;` into an equivalent filter, like below
```lua
data = GameLogic.GetFilters():apply_filters("my_data", data, some_parameters);
```
The above code does nothing until some other modules `add_filters` to `"my_data"`. 

Each filter function takes the output of the previous filter function as its first parameter, all other input parameters are shared by all filter functions, like below
```
F1(input, ...)-->F2(F1_output, ...)-->F3(F2_output, ...)-->F3_output
```

Filters is the recommended way for plugin/app developers to extend or modify paracraft. 

> [Click here to see all Paracraft Filters](https://github.com/NPLPackages/paracraft/blob/master/script/apps/Aries/Creator/Game/ParacraftFilters.md)
