## Timer
- Encoding: [script/ide/timer.lua](https://github.com/NPLPackages/main/tree/master/script/ide/timer.lua)

```lua
NPL.load("(gl)script/ide/timer.lua");

local mytimer = commonlib.Timer:new({callbackFunc = function(timer)
	commonlib.log({"ontimer", timer.id, timer.delta, timer.lastTick})
end})

-- start the timer after 0 milliseconds, and signal every 1000 millisecond
mytimer:Change(0, 1000)

-- start the timer after 1000 milliseconds, and stop it immediately.
mytimer:Change(1000, nil)

-- now kill the timer. 
mytimer:Change()

-- kill all timers in the pool
commonlib.TimerManager.Clear()

-- dump timer info
commonlib.TimerManager.DumpTimerCount()

-- get the current time in millisecond. This may be faster than ParaGlobal_timeGetTime() since it is updated only at rendering frame rate. 
commonlib.TimerManager.GetCurrentTime();

-- one time timer
commonlib.TimerManager.SetTimeout(function()  end, 1000)
```