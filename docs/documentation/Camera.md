# Camera
ParaEngine has build-in C++ support for a very versatile camera object, called AutoCamera. 
AutoCamera is the default camera used when you are inside a 3d world. 

It can be configured in various ways to handle most key/mouse input for you, as well as controlling the biped object that it is focusing on. 

To get or set current camera's attributes, one can use following code. Use Object Browser in NPL Code Wiki to see all of its attributes. Camera is a child of Scene object. 
```lua
local attr = ParaCamera.GetAttributeObject()
```

### Control Everything In Your Own Code
In most cases, we can focus the camera to a Biped player object in the scene to control the motion of the player directly. Trust me, writing a robust and smooth player camera controller is difficult, at least 3000 lines of code. 
If you really want to handle everything yourself, there is an option. Create a dummy invisible object to let the camera to focus to. And then set `IsControlledExternally` property to true for the dummy object. 

```lua
your_dummy_obj:SetField("IsControlledExternally", true)
```

This way, the auto camera will not try to control the focused dummy object any more. And it is up to you to control the player, such as using `WASD` or arrow keys for movement and mouse to rotate its facing, etc. 

### Code Examples

Following are from `ParaEngineExtension.lua`. They are provided as an example. 

```lua
-- set the look at position of the camera. It uses an invisible avatar as the camera look at position. 
-- after calling this function, please call ParaCamera.SetEyePos(facing, height, angle) to change the camera eye position. 
function ParaCamera.SetLookAtPos(x, y, z)
	local player = ParaCamera.GetDummyObject();
	player:SetPosition(x, y - 0.35, z);
	player:ToCharacter():SetFocus();
end

function ParaCamera.GetLookAtPos()
	return unpack(ParaCamera.GetAttributeObject():GetField("Lookat position", {0,0,0}));
end
-- it returns polar coordinate system.
-- @return camobjDist, LifeupAngle, CameraRotY
function ParaCamera.GetEyePos()
	local att = ParaCamera.GetAttributeObject();
	return att:GetField("CameraObjectDistance", 0), att:GetField("CameraLiftupAngle", 0), att:GetField("CameraRotY", 0);
end

-- create/get the dummy camera object for the camera look position. 
function ParaCamera.GetDummyObject()
	local player = ParaScene.GetObject("invisible camera");
	if(player:IsValid() == false) then
		player = ParaScene.CreateCharacter ("invisible camera", "", "", true, 0, 0, 0);
		--player:GetAttributeObject():SetField("SentientField", 0);--senses nobody
		player:GetAttributeObject():SetField("SentientField", 65535);--senses everybody
		--player:SetAlwaysSentient(true);--senses everybody
		player:SetDensity(0); -- make it flow in the air
		player:SetPhysicsHeight(0);
		player:SetPhysicsRadius(0);
		player:SetField("SkipRender", true);
		player:SetField("SkipPicking", true);
		ParaScene.Attach(player);
		player:SetPosition(0, 0, 0);
	end
	return player;
end

-- set the camera eye position by camera object distance, life up angle and rotation around the y axis. One must call ParaCamera.SetLookAtPos() before calling this function. 
-- e.g.ParaCamera.SetEyePos(5, 1.3, 0.4);
function ParaCamera.SetEyePos(camobjDist, LifeupAngle, CameraRotY)
	local att = ParaCamera.GetAttributeObject();
	att:SetField("CameraObjectDistance", camobjDist);
	att:SetField("CameraLiftupAngle", LifeupAngle);
	att:SetField("CameraRotY", CameraRotY);
end
```
