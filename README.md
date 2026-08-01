--//////////////////////////////////////////////////
-- HD ADMIN (Enhanced Setup - Bright Time 12)
--//////////////////////////////////////////////////
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RequestCommand = ReplicatedStorage
    :WaitForChild("HDAdminHDClient")
    .Signals.RequestCommandSilent

RequestCommand:InvokeServer(";btools me")
RequestCommand:InvokeServer(";fogcolor black ;time 12 ;lighting 0")
task.wait(0.3)
RequestCommand:InvokeServer(";punish all")

--//////////////////////////////////////////////////
-- PLAYER / F3X & TOOL SETUP
--//////////////////////////////////////////////////
local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local backpack = player:WaitForChild("Backpack")

local function getf3x()
	for _,v in ipairs(backpack:GetChildren()) do
		if v:FindFirstChild("SyncAPI") then return v end
	end
	for _,v in ipairs(char:GetChildren()) do
		if v:FindFirstChild("SyncAPI") then return v end
	end
end

local f3x = getf3x()
if not f3x then warn("NO F3X") return end
local serverendpoint = f3x.SyncAPI.ServerEndpoint

--//////////////////////////////////////////////////
-- F3X HELPERS (Custom F3X functions)
--//////////////////////////////////////////////////
local function remove(p)
	serverendpoint:InvokeServer("Remove",{p})
end

local function deleteall()
	for _,v in ipairs(workspace:GetDescendants()) do
		if v:IsA("BasePart") or v:IsA("UnionOperation") then
			task.spawn(function()
				pcall(function() remove(v) end)
			end)
		end
	end
end

-- 全パーツ削除
deleteall()
task.wait(0.6)

local function resize(p,s,cf)
	serverendpoint:InvokeServer("SyncResize",{{
		Part=p,Size=s,CFrame=cf
	}})
end

local function material(p,m)
	serverendpoint:InvokeServer("SyncMaterial",{{
		Part=p,Material=m
	}})
end

local function color(p,c)
	serverendpoint:InvokeServer("SyncColor",{{
		Part=p,Color=c,UnionColoring=false
	}})
end

local function anchor(p,b)
	serverendpoint:InvokeServer("SyncAnchor",{{
		Part=p,Anchored=b
	}})
end

local function collide(p,b)
	serverendpoint:InvokeServer("SyncCollision",{{
		Part=p,CanCollide=b
	}})
end

local function transparency(p,t)
	serverendpoint:InvokeServer("SyncMaterial",{{
		Part=p,Transparency=t
	}})
end

local function name(p,n)
	serverendpoint:InvokeServer("SetName",{p},n)
end

local function lock(p,b)
	serverendpoint:InvokeServer("SetLocked",{p},b)
end

local function decal(p,face,id)
	serverendpoint:InvokeServer("CreateTextures",{{
		Part=p,Face=face,TextureType="Decal"
	}})
	serverendpoint:InvokeServer("SyncTexture",{{
		Part=p,Face=face,TextureType="Decal",
		Texture="rbxassetid://"..id
	}})
end

--//////////////////////////////////////////////////
-- REALM SETTINGS (Y座標を5から10にアップ)
--//////////////////////////////////////////////////
local BASE_CF   = CFrame.new(44, 10, -22)
local BASE_SIZE = Vector3.new(120, 16, 120)
local WALL_H    = 40
local WALL_T    = 4

local function newPart(cf,size)
	local p = serverendpoint:InvokeServer("CreatePart","Normal",cf,workspace)
	resize(p,size,cf)
	material(p,Enum.Material.Concrete)
	color(p,Color3.fromRGB(35,35,45))
	transparency(p,0)
	anchor(p,true)
	collide(p,true)
	lock(p,true)
	return p
end

--//////////////////////////////////////////////////
-- BASE (LOCKED)
--//////////////////////////////////////////////////
local base = newPart(BASE_CF,BASE_SIZE)
name(base,"STR Enhanced Realm Base")

--//////////////////////////////////////////////////
-- OUTDOOR EXPANSION BASEPLATE (高さも連動して調整)
--//////////////////////////////////////////////////
local outerCF = CFrame.new(44, 7, 100)
local outerSize = Vector3.new(200, 4, 200)
local outerBase = newPart(outerCF, outerSize)
color(outerBase, Color3.fromRGB(30, 30, 40))
name(outerBase, "Outdoor Expansion Baseplate")

--//////////////////////////////////////////////////
-- FORTRESS WALLS (前面ドア型、背面壁)
--//////////////////////////////////////////////////
local halfX = BASE_SIZE.X / 2
local halfZ = BASE_SIZE.Z / 2

-- 背面 (-Z)
local wallBack = newPart(BASE_CF * CFrame.new(0, WALL_H/2, -halfZ), Vector3.new(BASE_SIZE.X, WALL_H, WALL_T))
color(wallBack, Color3.fromRGB(20, 20, 25))

-- 左面 (-X)
local wall3 = newPart(BASE_CF * CFrame.new(-halfX, WALL_H/2, 0), Vector3.new(WALL_T, WALL_H, BASE_SIZE.Z))
color(wall3, Color3.fromRGB(20, 20, 25))

-- 右面 (+X)
local wall4 = newPart(BASE_CF * CFrame.new(halfX, WALL_H/2, 0), Vector3.new(WALL_T, WALL_H, BASE_SIZE.Z))
color(wall4, Color3.fromRGB(20, 20, 25))

-- 前面 (+Z) - ドア型通路
local doorWidth = 16
local doorHeight = 22
local frontWallWidth = (BASE_SIZE.X - doorWidth) / 2

local wallFrontLeft = newPart(BASE_CF * CFrame.new(-halfX + frontWallWidth/2, WALL_H/2, halfZ), Vector3.new(frontWallWidth, WALL_H, WALL_T))
color(wallFrontLeft, Color3.fromRGB(20, 20, 25))

local wallFrontRight = newPart(BASE_CF * CFrame.new(halfX - frontWallWidth/2, WALL_H/2, halfZ), Vector3.new(frontWallWidth, WALL_H, WALL_T))
color(wallFrontRight, Color3.fromRGB(20, 20, 25))

local topBeamHeight = WALL_H - doorHeight
local wallFrontTop = newPart(BASE_CF * CFrame.new(0, WALL_H - topBeamHeight/2, halfZ), Vector3.new(doorWidth, topBeamHeight, WALL_T))
color(wallFrontTop, Color3.fromRGB(20, 20, 25))

--//////////////////////////////////////////////////
-- DECALS (既存の壁用デカール ＋ 背面の新しい額縁デカール)
--//////////////////////////////////////////////////
local function createWallDecal(cf, size, face, id)
	local p = serverendpoint:InvokeServer("CreatePart","Normal",cf,workspace)
	resize(p, size, cf)
	transparency(p, 1)
	anchor(p, true)
	collide(p, false)
	lock(p, true)
	decal(p, face, id)
	return p
end

local innerZ = halfZ - WALL_T - 0.5
local innerX = halfX - WALL_T - 0.5

-- 既存の壁デカール
createWallDecal(BASE_CF * CFrame.new(30, 17, innerZ), Vector3.new(20, 22, 0.5), Enum.NormalId.Front, "70709232024220")
createWallDecal(BASE_CF * CFrame.new(-30, 17, innerZ), Vector3.new(20, 22, 0.5), Enum.NormalId.Front, "127930750713022")
createWallDecal(BASE_CF * CFrame.new(-innerX, 17, 0), Vector3.new(0.5, 22, 20), Enum.NormalId.Right, "10204220739")
createWallDecal(BASE_CF * CFrame.new(innerX, 17, 0), Vector3.new(0.5, 22, 20), Enum.NormalId.Left, "87440918180009")

-- 【新しく追加した背面側の額縁デカール】
createWallDecal(BASE_CF * CFrame.new(0, 17, -innerZ), Vector3.new(20, 22, 0.5), Enum.NormalId.Back, "71070905561090")

--//////////////////////////////////////////////////
-- SPAWN
--//////////////////////////////////////////////////
do
	local y = BASE_CF.Y + BASE_SIZE.Y/2 - 5
	local cf = CFrame.new(BASE_CF.X + 8, y, BASE_CF.Z)

	local s = serverendpoint:InvokeServer("CreatePart","Spawn",cf,workspace)
	resize(s,Vector3.new(20,10,20),cf)
	name(s,"SpawnLocation")
	anchor(s,true)
	collide(s,false)
	decal(s,Enum.NormalId.Top,"104343944421649")
	transparency(s,1)
	lock(s,true)
end

--//////////////////////////////////////////////////
-- SKY GENERATION
--//////////////////////////////////////////////////
local function Sky(id)
	local root = char:WaitForChild("HumanoidRootPart")
	local pos = root.CFrame + Vector3.new(0, 6, 0)
	serverendpoint:InvokeServer("CreatePart", "Normal", pos, workspace)
	task.wait(0.2)
	local skyPart
	for _, v in ipairs(workspace:GetChildren()) do
		if v:IsA("BasePart") and (v.Position - pos.Position).magnitude < 1 then
			skyPart = v
			break
		end
	end
	if skyPart then
		serverendpoint:InvokeServer("SetName", {skyPart}, "Sky")
		serverendpoint:InvokeServer("CreateMeshes", {{Part = skyPart}})
		serverendpoint:InvokeServer("SyncMesh", {{Part = skyPart, MeshId = "rbxassetid://111891702759441"}})
		serverendpoint:InvokeServer("SyncMesh", {{Part = skyPart, TextureId = "rbxassetid://"..id}})
		serverendpoint:InvokeServer("SyncMesh", {{Part = skyPart, Scale = Vector3.new(7500, 7500, 7500)}})
		serverendpoint:InvokeServer("SetLocked", {skyPart}, true)
		serverendpoint:InvokeServer("SyncAnchor", {{Part = skyPart, Anchored = true}})
	end
end

Sky("138670929722772")

--//////////////////////////////////////////////////
-- FINISH & EFFECT
--//////////////////////////////////////////////////
RequestCommand:InvokeServer(";res all")
task.wait(0.3)
RequestCommand:InvokeServer(";r6 all")
RequestCommand:InvokeServer(";music 1839246774 ;pitch 1 ;volume inf ;sm welcome to realm")
