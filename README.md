-- [[ PLOYGIM ULTIMATE - V13.9 ULTRAKILL & ESP OVERHAUL ]] --
-- [[ FULL FUNCTIONALITY + INTEGRATED GUI | ESP FIXED | DRAG LOCK FIXED | HOOK BIND: Q ]] --

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local TweenService = game:GetService("TweenService")
local Lighting = game:GetService("Lighting") -- Добавлено для работы со светом
local VIM = game:GetService("VirtualInputManager") -- Добавлено для UltraKill
local lp = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local FILE_NAME = "PloYgim_Config.json"
local toxicClicks = 0 
local lastGrounded = true
local targetUpdateTick = 0
local atmosphereUpdateTick = 0 -- Для оптимизации света
local isSliderActive = false -- Флаг-предохранитель для слайдеров

-- [[ ГЛОБАЛЬНОЕ СОСТОЯНИЕ ]]
local STATE = {
    Visible = true,
    AccentColor = Color3.fromRGB(160, 32, 240),
    ColorR = 160, ColorG = 32, ColorB = 240,
    CurrentTheme = "Toxic",
    
    -- Aim
    AimEnabled = false, AimFOV = 150, AimSmooth = 3, TeamCheck = false, WallCheck = false,
    LockedPart = nil,
    HitboxChance = 50, -- 0 = Body, 100 = Head
    
    -- Friends
    Friends = {}, 
    
    -- Move
    SpeedEnabled = false, SpeedPower = 2, 
    FlyEnabled = false, FlySpeed = 50,
    Noclip = false, SpinBot = false, SpinSpeed = 1000,
    BhopEnabled = false, BhopPower = 16, BhopMax = 240,
    HookEnabled = false, HookSpeed = 150,
    
    -- SkySpin Function
    SkySpinEnabled = false,
    SkySpinActive = false,
    SkySpinPos = nil,
    SkySpinOrbitAngle = 0,
    SkySpinTarget = nil,
    
    -- [[ ULTRAKILL SETTINGS ]] --
    UltraKillEnabled = true,
    UltraKillDelay = 0.01,
    isUltraTeleporting = false,
    UltraCurrentTarget = nil, -- Фиксация цели для UltraKill

    -- [[ SHAHED SETTINGS ]] --
    ShahedEnabled = false,
    
    -- Visuals
    BoxEsp = false, ChamsEsp = false,
    TargetHUDEnabled = true,
    TPBehindEnabled = false, TPBehindTarget = nil,
    
    -- World Visuals (NEW)
    SnowEnabled = false,
    SnowColorR = 255, SnowColorG = 255, SnowColorB = 255,
    FogEnabled = false,
    FogColorR = 192, FogColorG = 192, FogColorB = 192,
    FogEnd = 10000,
    WorldLightR = 255, WorldLightG = 255, WorldLightB = 255,
    
    -- Skybox & Day/Night (NEW)
    SkyEnabled = false,
    SkyColorR = 255, SkyColorG = 255, SkyColorB = 255,
    TimeOfDay = 14,

    -- Sounds
    HookSoundEnabled = false,
    BhopSoundEnabled = true, -- Звук для бхопа

    -- Config
    Bind_Menu = "RightShift"
}

-- Физические объекты
local bAV = Instance.new("BodyAngularVelocity")
bAV.MaxTorque = Vector3.new(0, math.huge, 0)
bAV.P = 15000
bAV.Name = "SpinbotForce"

local bPos = Instance.new("BodyPosition")
bPos.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bPos.P = 10000
bPos.D = 500
bPos.Name = "SkySpinLock"

local hookForce = Instance.new("BodyVelocity")
hookForce.MaxForce = Vector3.new(math.huge, math.huge, math.huge)

-- Объекты для веревки хука
local hookBeam = Instance.new("Beam")
local att0 = Instance.new("Attachment")
local att1 = Instance.new("Attachment", workspace.Terrain)
hookBeam.Width0 = 0.2
hookBeam.Width1 = 0.2
hookBeam.FaceCamera = true
hookBeam.Enabled = false
hookBeam.Attachment1 = att1

-- Объект снега (ТЕКСТУРА ОБНОВЛЕНА ПО ЗАПРОСУ)
local snowPart = Instance.new("Part", workspace.Terrain)
snowPart.Size = Vector3.new(100, 1, 100)
snowPart.Anchored = true
snowPart.CanCollide = false
snowPart.Transparency = 1
local snowEmitter = Instance.new("ParticleEmitter", snowPart)
snowEmitter.Texture = "rbxassetid://73006344746344"
snowEmitter.Enabled = false
snowEmitter.Speed = NumberRange.new(15, 30)
snowEmitter.Lifetime = NumberRange.new(3, 6)
snowEmitter.Rate = 350
snowEmitter.SpreadAngle = Vector2.new(10, 10)
snowEmitter.Acceleration = Vector3.new(0, -10, 0)
snowEmitter.EmissionDirection = Enum.NormalId.Bottom

-- Объект Скайбокса (ID ЗАМЕНЕН НА 137228195735991)
local customSky = Instance.new("Sky")
customSky.SkyboxBk = "rbxassetid://137228195735991"
customSky.SkyboxDn = "rbxassetid://137228195735991"
customSky.SkyboxFt = "rbxassetid://137228195735991"
customSky.SkyboxLf = "rbxassetid://137228195735991"
customSky.SkyboxRt = "rbxassetid://137228195735991"
customSky.SkyboxUp = "rbxassetid://137228195735991"
customSky.SunTextureId = ""
customSky.MoonTextureId = ""
customSky.Parent = nil

-- [[ ТЕМЫ ]]
local Themes = {
    Toxic = {
        Accent = Color3.fromRGB(0, 255, 100),
        Grad1 = Color3.fromRGB(5, 15, 5), Grad2 = Color3.fromRGB(10, 30, 10)
    },
    Vampire = {
        Accent = Color3.fromRGB(255, 0, 50),
        Grad1 = Color3.fromRGB(15, 5, 5), Grad2 = Color3.fromRGB(30, 10, 10)
    },
    Midnight = {
        Accent = Color3.fromRGB(100, 100, 255),
        Grad1 = Color3.fromRGB(5, 5, 15), Grad2 = Color3.fromRGB(10, 10, 30)
    }
}

-- [[ ПРОВЕРКИ ]]
local function IsVisible(part)
    if not STATE.WallCheck then return true end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {lp.Character, part.Parent}
    local result = workspace:Raycast(Camera.CFrame.Position, part.Position - Camera.CFrame.Position, params)
    return not result
end

local function IsFriend(userId)
    for _, id in pairs(STATE.Friends) do
        if tonumber(id) == userId then return true end
    end
    return false
end

local function IsTeammate(player)
    if IsFriend(player.UserId) then return true end
    if not STATE.TeamCheck then return false end
    return player.Team == lp.Team
end

-- [[ КОНФИГИ ]]
local function SaveConfig()
    local data = {}
    for k, v in pairs(STATE) do
        if type(v) ~= "table" and type(v) ~= "userdata" then data[k] = v end
    end
    local success, err = pcall(function()
        writefile(FILE_NAME, HttpService:JSONEncode(data))
    end)
    if success then
        StarterGui:SetCore("SendNotification", {Title = "Config", Text = "Saved Successfully!", Duration = 2})
    end
end

local function LoadConfig()
    if isfile(FILE_NAME) then
        local success, decoded = pcall(function() return HttpService:JSONDecode(readfile(FILE_NAME)) end)
        if success then 
            for k, v in pairs(decoded) do if STATE[k] ~= nil then STATE[k] = v end end
            StarterGui:SetCore("SendNotification", {Title = "Config", Text = "Loaded Successfully!", Duration = 2})
        end
    end
end

-- [[ UI КОРПУС ]]
local sg = Instance.new("ScreenGui", lp.PlayerGui); 
sg.Name = "PloYgim_V13"; 
sg.ResetOnSpawn = false;
sg.DisplayOrder = 999999;
sg.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local main = Instance.new("Frame", sg)
main.Size = UDim2.new(0, 680, 0, 480); main.Position = UDim2.new(0.5, -340, 0.5, -240); main.BorderSizePixel = 0
main.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
local mainCorner = Instance.new("UICorner", main)
mainCorner.CornerRadius = UDim.new(0, 12)
local mainStroke = Instance.new("UIStroke", main); mainStroke.Thickness = 2
local mainGrad = Instance.new("UIGradient", main); mainGrad.Rotation = 45

-- Сайд sidebar
local sidebar = Instance.new("Frame", main)
sidebar.Size = UDim2.new(0, 160, 1, 0); sidebar.BackgroundTransparency = 0.5; sidebar.BackgroundColor3 = Color3.new(0,0,0)
Instance.new("UICorner", sidebar).CornerRadius = UDim.new(0, 12)

-- ДРАГ
local dragging, dragStart, startPos
sidebar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 and not isSliderActive then
        dragging = true; dragStart = input.Position; startPos = main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement and not isSliderActive then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local logo = Instance.new("TextLabel", sidebar)
logo.Size = UDim2.new(1, 0, 0, 60); logo.Text = "PLOYGIM V13"; logo.Font = "GothamBold"; logo.TextSize = 18; logo.TextColor3 = Color3.new(1,1,1); logo.BackgroundTransparency = 1

local tabHold = Instance.new("ScrollingFrame", sidebar)
tabHold.Size = UDim2.new(1, 0, 1, -60); tabHold.Position = UDim2.new(0, 0, 0, 60); tabHold.BackgroundTransparency = 1; tabHold.ScrollBarThickness = 0
Instance.new("UIListLayout", tabHold).Padding = UDim.new(0, 5)

local content = Instance.new("Frame", main)
content.Size = UDim2.new(1, -170, 1, -20); content.Position = UDim2.new(0, 165, 0, 10); content.BackgroundTransparency = 1

-- TARGET HUD
local function MakeHUD(name, size, pos)
    local f = Instance.new("Frame", sg); f.Name = name; f.Size = size; f.Position = pos; f.BackgroundColor3 = Color3.fromRGB(15,15,17)
    Instance.new("UICorner", f); local s = Instance.new("UIStroke", f); s.Thickness = 1; s.Color = Color3.fromRGB(50,50,50)
    local acc = Instance.new("Frame", f); acc.Name = "Accent"; acc.Size = UDim2.new(0, 2, 1, 0); acc.BorderSizePixel = 0
    return f
end

local targetHud = MakeHUD("Target", UDim2.new(0, 180, 0, 50), UDim2.new(0.5, 50, 0.5, 50))
local targetName = Instance.new("TextLabel", targetHud); targetName.Size = UDim2.new(1, -10, 0, 25); targetName.Position = UDim2.new(0, 8, 0, 5); targetName.TextColor3 = Color3.new(1,1,1); targetName.Font = "GothamMedium"; targetName.TextSize = 13; targetName.BackgroundTransparency = 1; targetName.TextXAlignment = "Left"
local targetHPBg = Instance.new("Frame", targetHud); targetHPBg.Size = UDim2.new(1, -16, 0, 4); targetHPBg.Position = UDim2.new(0, 8, 0, 35); targetHPBg.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
local targetHPFill = Instance.new("Frame", targetHPBg); targetHPFill.Name = "Fill"; targetHPFill.Size = UDim2.new(1, 0, 1, 0); targetHPFill.BorderSizePixel = 0

-- Билдер вкладок
local pages = {}
local toggles = {}

local function createTab(name)
    local p = Instance.new("ScrollingFrame", content); p.Size = UDim2.new(1, 0, 1, 0); p.BackgroundTransparency = 1; p.Visible = false; p.ScrollBarThickness = 2
    Instance.new("UIListLayout", p).Padding = UDim.new(0, 10)
    pages[name] = p
    
    local b = Instance.new("TextButton", tabHold); b.Size = UDim2.new(1, -10, 0, 35); b.Text = name; b.Font = "GothamMedium"; b.TextColor3 = Color3.fromRGB(180, 180, 180); b.BackgroundTransparency = 1; b.TextSize = 14
    b.MouseButton1Click:Connect(function()
        for _, v in pairs(pages) do v.Visible = false end
        for _, v in pairs(tabHold:GetChildren()) do if v:IsA("TextButton") then v.TextColor3 = Color3.fromRGB(180, 180, 180) end end
        p.Visible = true; b.TextColor3 = Color3.new(1,1,1)
    end)
    return p
end

local function addToggle(p, text, key)
    local b = Instance.new("TextButton", p); b.Size = UDim2.new(1, -10, 0, 35); b.BackgroundColor3 = Color3.fromRGB(25, 25, 30); b.Text = "  " .. text; b.TextColor3 = Color3.new(0.8, 0.8, 0.8); b.Font = "Gotham"; b.TextSize = 13; b.TextXAlignment = "Left"
    Instance.new("UICorner", b)
    local ind = Instance.new("Frame", b); ind.Size = UDim2.new(0, 15, 0, 15); ind.Position = UDim2.new(1, -25, 0.5, -7); ind.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    Instance.new("UICorner", ind)
    local fill = Instance.new("Frame", ind); fill.Name = "Fill"; fill.Size = UDim2.new(1, -4, 1, -4); fill.Position = UDim2.new(0, 2, 0, 2); fill.BackgroundTransparency = 1; Instance.new("UICorner", fill)
    
    b.MouseButton1Click:Connect(function()
        STATE[key] = not STATE[key]
    end)
    toggles[key] = fill
end

local function addSlider(p, text, key, min, max)
    local f = Instance.new("Frame", p); f.Size = UDim2.new(1, -10, 0, 45); f.BackgroundTransparency = 1
    local t = Instance.new("TextLabel", f); t.Size = UDim2.new(1, 0, 0, 20); t.Text = text .. ": " .. string.format("%.1f", STATE[key]); t.TextColor3 = Color3.new(0.7,0.7,0.7); t.Font = "Gotham"; t.TextSize = 11; t.BackgroundTransparency = 1
    local bg = Instance.new("Frame", f); bg.Size = UDim2.new(1, -20, 0, 4); bg.Position = UDim2.new(0, 10, 0, 30); bg.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    local fill = Instance.new("Frame", bg); fill.Name = "Fill"; fill.Size = UDim2.new((STATE[key]-min)/(max-min), 0, 1, 0); fill.BorderSizePixel = 0
    bg.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            isSliderActive = true
            dragging = false
            local move; move = RunService.RenderStepped:Connect(function()
                local pc = math.clamp((UIS:GetMouseLocation().X - bg.AbsolutePosition.X)/bg.AbsoluteSize.X, 0, 1)
                fill.Size = UDim2.new(pc, 0, 1, 0)
                local val = min + (max-min)*pc
                STATE[key] = val; t.Text = text .. ": " .. string.format("%.1f", val)
            end)
            UIS.InputEnded:Connect(function(i2) 
                if i2.UserInputType == Enum.UserInputType.MouseButton1 then 
                    move:Disconnect()
                    isSliderActive = false
                end 
            end)
        end
    end)
end

local function addBtn(p, text, func, color)
    local b = Instance.new("TextButton", p); b.Size = UDim2.new(1, -10, 0, 35); b.BackgroundColor3 = color or STATE.AccentColor; b.Text = text; b.TextColor3 = Color3.new(1,1,1); b.Font = "GothamBold"; b.TextSize = 13
    Instance.new("UICorner", b); b.MouseButton1Click:Connect(func)
    return b
end

-- Вкладки
local aimTab = createTab("Aim")
addToggle(aimTab, "Enabled Aimbot", "AimEnabled")
addToggle(aimTab, "Team Check", "TeamCheck")
addToggle(aimTab, "Wall Check", "WallCheck")
addSlider(aimTab, "FOV Radius", "AimFOV", 10, 800)
addSlider(aimTab, "Smoothness", "AimSmooth", 1, 20)
addSlider(aimTab, "Hitbox Chance (L:Body | R:Head)", "HitboxChance", 0, 100)

local moveTab = createTab("Movement")
addToggle(moveTab, "SHAHED MODE (FOLLOW UP)", "ShahedEnabled")
addToggle(moveTab, "ULTRAKILL (FIXED V2)", "UltraKillEnabled")
addToggle(moveTab, "SKY SPIN (LOW HP TARGET)", "SkySpinEnabled")
addToggle(moveTab, "OMNI-BHOP (+3)", "BhopEnabled")
addToggle(moveTab, "Bhop Jump Sound", "BhopSoundEnabled")
addSlider(moveTab, "BhopMax", "BhopMax", 16, 500)
addToggle(moveTab, "Speed Hack", "SpeedEnabled")
addSlider(moveTab, "Speed Power", "SpeedPower", 0, 15)
addToggle(moveTab, "Hook Point (Q)", "HookEnabled")
addSlider(moveTab, "Hook Speed", "HookSpeed", 10, 500)
addToggle(moveTab, "TP Behind (E)", "TPBehindEnabled")
addToggle(moveTab, "Fly Mode", "FlyEnabled")
addSlider(moveTab, "Fly Speed", "FlySpeed", 10, 500)
addToggle(moveTab, "Spinbot", "SpinBot")
addSlider(moveTab, "Spin Speed", "SpinSpeed", 0, 5000)
addToggle(moveTab, "Noclip", "Noclip")

local visTab = createTab("Visuals")
addToggle(visTab, "Box ESP", "BoxEsp")
addToggle(visTab, "Chams ESP", "ChamsEsp")
addToggle(visTab, "Target HUD", "TargetHUDEnabled")
addToggle(visTab, "Snowfall", "SnowEnabled")
addSlider(visTab, "Snow Red", "SnowColorR", 0, 255)
addSlider(visTab, "Snow Green", "SnowColorG", 0, 255)
addSlider(visTab, "Snow Blue", "SnowColorB", 0, 255)
addToggle(visTab, "Custom Skybox", "SkyEnabled")
addSlider(visTab, "Sky Red", "SkyColorR", 0, 255)
addSlider(visTab, "Sky Green", "SkyColorG", 0, 255)
addSlider(visTab, "Sky Blue", "SkyColorB", 0, 255)

local worldTab = createTab("Atmosphere")
addSlider(worldTab, "Time of Day", "TimeOfDay", 0, 24)
addToggle(worldTab, "Fog Enabled", "FogEnabled")
addSlider(worldTab, "Fog End (Distance)", "FogEnd", 0, 10000)
addSlider(worldTab, "Fog Color R", "FogColorR", 0, 255)
addSlider(worldTab, "Fog Color G", "FogColorG", 0, 255)
addSlider(worldTab, "Fog Color B", "FogColorB", 0, 255)
addSlider(worldTab, "World Light R", "WorldLightR", 0, 255)
addSlider(worldTab, "World Light G", "WorldLightG", 0, 255)
addSlider(worldTab, "World Light B", "WorldLightB", 0, 255)

local soundsTab = createTab("Sounds")
addToggle(soundsTab, "Hook Hit Sound", "HookSoundEnabled")

local friendsTab = createTab("Friends")
addBtn(friendsTab, "ADD TARGET TO FRIENDS", function()
    if STATE.TPBehindTarget then
        if not IsFriend(STATE.TPBehindTarget.UserId) then
            table.insert(STATE.Friends, STATE.TPBehindTarget.UserId)
            StarterGui:SetCore("SendNotification", {Title = "Friends", Text = STATE.TPBehindTarget.Name.." added!"})
        end
    end
end)
addBtn(friendsTab, "CLEAR FRIENDS", function() STATE.Friends = {} end)

local themesTab = createTab("Themes")
addBtn(themesTab, "TOXIC GREEN", function() 
    STATE.CurrentTheme = "Toxic"; STATE.ColorR = 0; STATE.ColorG = 255; STATE.ColorB = 50;
end, Color3.fromRGB(0, 200, 80))
addBtn(themesTab, "VAMPIRE RED", function() 
    STATE.CurrentTheme = "Vampire"; STATE.ColorR = 255; STATE.ColorG = 0; STATE.ColorB = 0; 
end, Color3.fromRGB(200, 0, 40))

local colorTab = createTab("Colors")
addSlider(colorTab, "Red", "ColorR", 0, 255)
addSlider(colorTab, "Green", "ColorG", 0, 255)
addSlider(colorTab, "Blue", "ColorB", 0, 255)

local setTab = createTab("Config")
addBtn(setTab, "SAVE CONFIG", SaveConfig, Color3.fromRGB(50, 50, 55))
addBtn(setTab, "LOAD CONFIG", LoadConfig, Color3.fromRGB(50, 50, 55))

-- [[ ЛОГИКА ЦИКЛА ]]
local Boxes = {}
local function GetBox(player)
    if Boxes[player] then return Boxes[player] end
    local b = Drawing.new("Square")
    b.Thickness = 1; b.Filled = false; b.Transparency = 1; b.Visible = false
    Boxes[player] = b
    return b
end

-- NEW TARGET ESP (CORNERS)
local TargetCorners = {}
for i = 1, 4 do
    local l = Drawing.new("Line")
    l.Thickness = 2; l.Transparency = 1; l.Visible = false
    table.insert(TargetCorners, l)
end

local FOV_Circle = Drawing.new("Circle"); FOV_Circle.Thickness = 1; FOV_Circle.NumSides = 60

local hooking = false
local hookTarget = nil

-- [[ CLEANUP ON DEATH ]]
lp.CharacterAdded:Connect(function(newChar)
    local hum = newChar:WaitForChild("Humanoid")
    hum.Died:Connect(function()
        STATE.SkySpinActive = false
        STATE.isUltraTeleporting = false
        if lp.Character:FindFirstChild("SkySpinLock") then lp.Character.SkySpinLock:Destroy() end
        if lp.Character:FindFirstChild("SpinbotForce") then lp.Character.SpinbotForce:Destroy() end
    end)
end)

RunService.RenderStepped:Connect(function()
    STATE.AccentColor = Color3.fromRGB(STATE.ColorR, STATE.ColorG, STATE.ColorB)
    local t = Themes[STATE.CurrentTheme] or {Grad1 = Color3.new(0,0,0), Grad2 = Color3.new(0.1,0.1,0.1)}
    
    mainStroke.Color = STATE.AccentColor
    mainGrad.Color = ColorSequence.new(t.Grad1, t.Grad2)
    
    for k, v in pairs(toggles) do
        v.BackgroundTransparency = STATE[k] and 0 or 1
        v.BackgroundColor3 = STATE.AccentColor
    end

    for _, v in pairs(sg:GetDescendants()) do
        if v.Name == "Fill" or v.Name == "Accent" then v.BackgroundColor3 = STATE.AccentColor end
    end

    local char = lp.Character; local hrp = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChild("Humanoid")
    local mouseLoc = UIS:GetMouseLocation()

    -- HOOK LOGIC
    if STATE.HookEnabled and hooking and hookTarget and hrp then
        hookForce.Parent = hrp
        local dir = (hookTarget - hrp.Position).Unit
        att0.Parent = hrp
        hookBeam.Parent = hrp
        hookBeam.Attachment0 = att0
        att1.Position = hookTarget
        hookBeam.Color = ColorSequence.new(STATE.AccentColor)
        hookBeam.Enabled = true
        if (hookTarget - hrp.Position).Magnitude > 7 then
            hookForce.Velocity = dir * STATE.HookSpeed
        else
            hooking = false
            hookForce.Parent = nil
            hookBeam.Enabled = false
        end
    else
        hookForce.Parent = nil
        hookBeam.Enabled = false
    end

    -- SKY SPIN LOGIC
    if STATE.SkySpinEnabled and hrp and hum and hum.Health > 0 then
        hum.AutoRotate = false 
        if not STATE.SkySpinActive then
            STATE.SkySpinActive = true
            hrp.CFrame = hrp.CFrame * CFrame.new(0, 500, 0) 
            STATE.SkySpinPos = hrp.Position
            STATE.SkySpinOrbitAngle = 0
        end
        
        local minHP = math.huge
        local targetPlayer = nil
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= lp and p.Character and p.Character:FindFirstChild("Humanoid") and not IsFriend(p.UserId) then
                local h = p.Character.Humanoid
                if h.Health < minHP and h.Health > 0 then
                    minHP = h.Health
                    targetPlayer = p
                end
            end
        end
        STATE.SkySpinTarget = targetPlayer

        STATE.SkySpinOrbitAngle = STATE.SkySpinOrbitAngle + 0.1
        local radius = 30 
        local offsetX = math.cos(STATE.SkySpinOrbitAngle) * radius
        local offsetZ = math.sin(STATE.SkySpinOrbitAngle) * radius
        
        local basePoint = STATE.SkySpinPos
        local targetOrbitPos = basePoint + Vector3.new(offsetX, 0, offsetZ)

        local lock = hrp:FindFirstChild("SkySpinLock")
        if not lock then
            lock = Instance.new("BodyPosition")
            lock.Name = "SkySpinLock"
            lock.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            lock.P = 15000; lock.D = 600
            lock.Parent = hrp
        end
        lock.Position = targetOrbitPos

        local spin = hrp:FindFirstChild("SpinbotForce")
        if not spin then
            spin = Instance.new("BodyAngularVelocity")
            spin.Name = "SpinbotForce"
            spin.MaxTorque = Vector3.new(0, math.huge, 0)
            spin.Parent = hrp
        end
        spin.AngularVelocity = Vector3.new(0, STATE.SpinSpeed, 0)
        
        for _, v in pairs(char:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = true end end
        
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and not IsFriend(p.UserId) then
                local dist = (p.Character.HumanoidRootPart.Position - hrp.Position).Magnitude
                if dist < 100 then
                    STATE.SkySpinPos = STATE.SkySpinPos + Vector3.new(0, 500, 0) 
                end
            end
        end
    elseif STATE.SkySpinActive or (hum and hum.Health <= 0) then
        STATE.SkySpinActive = false
        if hrp and hrp:FindFirstChild("SkySpinLock") then hrp.SkySpinLock:Destroy() end
        if hum and not STATE.SpinBot then 
            if hrp and hrp:FindFirstChild("SpinbotForce") then hrp.SpinbotForce:Destroy() end
            hum.AutoRotate = true 
        end
    end

    -- WORLD VISUALS
    if STATE.SnowEnabled then
        snowPart.CFrame = Camera.CFrame * CFrame.new(0, 30, -30)
        snowEmitter.Enabled = true
        snowEmitter.Color = ColorSequence.new(Color3.fromRGB(STATE.SnowColorR, STATE.SnowColorG, STATE.SnowColorB))
        if Lighting.FogEnd > 120 then Lighting.FogEnd = 120 end
    else
        snowEmitter.Enabled = false
    end

    if tick() - atmosphereUpdateTick > 0.05 then
        atmosphereUpdateTick = tick()
        Lighting.ClockTime = STATE.TimeOfDay
        if STATE.SkyEnabled then
            customSky.Parent = Lighting
            Lighting.ColorShift_Top = Color3.fromRGB(STATE.SkyColorR, STATE.SkyColorG, STATE.SkyColorB)
        else
            customSky.Parent = nil
        end
        if STATE.FogEnabled then
            Lighting.FogEnd = STATE.SnowEnabled and 120 or STATE.FogEnd
            Lighting.FogColor = Color3.fromRGB(STATE.FogColorR, STATE.FogColorG, STATE.FogColorB)
        else
            Lighting.FogEnd = STATE.SnowEnabled and 120 or 100000
        end
        Lighting.Ambient = Color3.fromRGB(STATE.WorldLightR, STATE.WorldLightG, STATE.WorldLightB)
    end

   -- [[ BHOP ]]
    if STATE.BhopEnabled and hum and hrp and hum.Health > 0 then
        if UIS:IsKeyDown(Enum.KeyCode.Space) then
            hum.Jump = true
            local moveDir = hum.MoveDirection
            if hum.FloorMaterial == Enum.Material.Air then
                lastGrounded = false
                if moveDir.Magnitude > 0 then
                    hrp.Velocity = Vector3.new(moveDir.X * STATE.BhopPower, hrp.Velocity.Y, moveDir.Z * STATE.BhopPower)
                end
            else
                if not lastGrounded then
                    STATE.BhopPower = math.clamp(STATE.BhopPower + 3, 16, STATE.BhopMax)
                    lastGrounded = true
                end
                if moveDir.Magnitude > 0 then
                    hrp.Velocity = Vector3.new(moveDir.X * STATE.BhopPower, hrp.Velocity.Y, moveDir.Z * STATE.BhopPower)
                end
            end
        else
            STATE.BhopPower = 16 
            lastGrounded = true
        end
    end

    -- TARGETING & ESP & ULTRAKILL LOGIC
    local aim_target, hud_p, bestDist = nil, nil, math.huge
    local bestUltraScore = -math.huge
    local potentialUltraTarget = nil

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and not IsFriend(p.UserId) then
            local root = p.Character.HumanoidRootPart
            local head = p.Character:FindFirstChild("Head")
            local body = p.Character:FindFirstChild("HumanoidRootPart")
            local targetHum = p.Character:FindFirstChild("Humanoid")
            local pos, on = Camera:WorldToViewportPoint(root.Position)
            
            local box = GetBox(p)
            if STATE.BoxEsp and on and targetHum and targetHum.Health > 0 then
                local sizeX = 2500 / pos.Z; local sizeY = 3800 / pos.Z
                box.Size = Vector2.new(sizeX, sizeY); box.Position = Vector2.new(pos.X - sizeX / 2, pos.Y - sizeY / 2)
                box.Color = STATE.AccentColor; box.Visible = true
            else box.Visible = false end

            if STATE.ChamsEsp and targetHum and targetHum.Health > 0 then
                local h = p.Character:FindFirstChild("PloY_Highlight") or Instance.new("Highlight", p.Character)
                h.Name = "PloY_Highlight"; h.FillColor = STATE.AccentColor; h.Enabled = true
            elseif p.Character:FindFirstChild("PloY_Highlight") then
                p.Character.PloY_Highlight.Enabled = false
            end

            if head and not IsTeammate(p) and targetHum and targetHum.Health > 0 then
                local hPos, hOn = Camera:WorldToViewportPoint(head.Position)
                local worldDist = (hrp and (root.Position - hrp.Position).Magnitude) or 99999
                local screenDist = (Vector2.new(hPos.X, hPos.Y) - mouseLoc).Magnitude

                if hOn and screenDist < STATE.AimFOV and worldDist < bestDist and IsVisible(head) then 
                    bestDist = worldDist; 
                    local roll = math.random(1, 100)
                    if roll <= STATE.HitboxChance then aim_target = head else aim_target = body end
                    hud_p = p 
                end

                -- ULTRAKILL SCORING (NO FOV)
                local hpAdvantage = (100 - targetHum.Health) * 30
                local currentScore = hpAdvantage - worldDist
                if currentScore > bestUltraScore then
                    bestUltraScore = currentScore
                    potentialUltraTarget = body
                end
            end
        elseif Boxes[p] then Boxes[p].Visible = false end
    end
    
    -- ULTRAKILL STICKY RECHECK
    if STATE.UltraKillEnabled then
        if STATE.UltraCurrentTarget and STATE.UltraCurrentTarget.Parent and STATE.UltraCurrentTarget.Parent:FindFirstChild("Humanoid") and STATE.UltraCurrentTarget.Parent.Humanoid.Health > 0 then
            local dist = (STATE.UltraCurrentTarget.Position - hrp.Position).Magnitude
            if dist > 1500 then STATE.UltraCurrentTarget = potentialUltraTarget end
        else
            STATE.UltraCurrentTarget = potentialUltraTarget
        end
    else
        STATE.UltraCurrentTarget = nil
    end

    if tick() - targetUpdateTick > 0.05 then
        targetUpdateTick = tick()
        STATE.TPBehindTarget = hud_p; STATE.LockedPart = aim_target
    end

    -- [[ TARGET HUD & FANCY TARGET ESP ]]
    if STATE.TPBehindTarget and STATE.TPBehindTarget.Character and STATE.TPBehindTarget.Character:FindFirstChild("Humanoid") then
        local targetChar = STATE.TPBehindTarget.Character
        local targetH = targetChar.Humanoid
        local root = targetChar:FindFirstChild("HumanoidRootPart")
        
        -- HUD
        targetHud.Visible = STATE.TargetHUDEnabled
        targetName.Text = STATE.TPBehindTarget.Name
        targetHPFill.Size = UDim2.new(math.clamp(targetH.Health/targetH.MaxHealth, 0, 1), 0, 1, 0)
        
        -- FANCY ESP
        local pos, on = Camera:WorldToViewportPoint(root.Position)
        if on then
            local sizeX = 2500 / pos.Z; local sizeY = 3800 / pos.Z
            local x1, y1 = pos.X - sizeX/2, pos.Y - sizeY/2
            local x2, y2 = pos.X + sizeX/2, pos.Y + sizeY/2
            
            -- Уголки (Corners)
            for i, l in pairs(TargetCorners) do
                l.Visible = true; l.Color = STATE.AccentColor
            end
            -- Верхний левый
            TargetCorners[1].From = Vector2.new(x1, y1 + 10); TargetCorners[1].To = Vector2.new(x1, y1)
            -- Верхний правый
            TargetCorners[2].From = Vector2.new(x2 - 10, y1); TargetCorners[2].To = Vector2.new(x2, y1)
            -- Нижний левый
            TargetCorners[3].From = Vector2.new(x1, y2 - 10); TargetCorners[3].To = Vector2.new(x1, y2)
            -- Нижний правый
            TargetCorners[4].From = Vector2.new(x2 - 10, y2); TargetCorners[4].To = Vector2.new(x2, y2)
        else
            for _, l in pairs(TargetCorners) do l.Visible = false end
        end
    else 
        targetHud.Visible = false 
        for _, l in pairs(TargetCorners) do l.Visible = false end
    end

    if STATE.AimEnabled and STATE.LockedPart and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) and not UIS:IsKeyDown(Enum.KeyCode.X) then
        local p = Camera:WorldToViewportPoint(STATE.LockedPart.Position)
        mousemoverel((p.X - mouseLoc.X)/STATE.AimSmooth, (p.Y - mouseLoc.Y)/STATE.AimSmooth)
    end

    -- MOVE LOGIC
    if hrp and hum and hum.Health > 0 then
        if STATE.ShahedEnabled and STATE.LockedPart then
            local targetPos = STATE.LockedPart.Position
            hrp.CFrame = CFrame.new(targetPos + Vector3.new(0, 65, 0), targetPos)
            hrp.Velocity = Vector3.new(0,0,0)
        elseif STATE.FlyEnabled then
            local flyDir = Vector3.new(0,0,0)
            if UIS:IsKeyDown(Enum.KeyCode.W) then flyDir += Camera.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.S) then flyDir -= Camera.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.A) then flyDir -= Camera.CFrame.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.D) then flyDir += Camera.CFrame.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.Space) then flyDir += Vector3.new(0,1,0) end
            if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then flyDir -= Vector3.new(0,1,0) end
            hrp.Velocity = flyDir * STATE.FlySpeed
        elseif STATE.SpeedEnabled and hum.MoveDirection.Magnitude > 0 then 
            hrp.CFrame += (hum.MoveDirection * STATE.SpeedPower / 5) 
        end

        if STATE.SpinBot and not STATE.SkySpinEnabled then
            hum.AutoRotate = false 
            local spin = hrp:FindFirstChild("SpinbotForce")
            if not spin then
                spin = Instance.new("BodyAngularVelocity"); spin.Name = "SpinbotForce"; spin.MaxTorque = Vector3.new(0, math.huge, 0); spin.Parent = hrp
            end
            spin.AngularVelocity = Vector3.new(0, STATE.SpinSpeed, 0)
        elseif not STATE.SkySpinEnabled then
            if hrp:FindFirstChild("SpinbotForce") then hrp.SpinbotForce:Destroy() end
            hum.AutoRotate = true
        end

        if STATE.Noclip and not STATE.SkySpinEnabled then for _, v in pairs(char:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end end
    end
    
    FOV_Circle.Visible = (STATE.AimEnabled and STATE.Visible); FOV_Circle.Radius = STATE.AimFOV; FOV_Circle.Position = mouseLoc; FOV_Circle.Color = STATE.AccentColor
end)

-- [[ КЛАВИШИ ]]
UIS.InputBegan:Connect(function(i, g)
    if not g and i.KeyCode == Enum.KeyCode[STATE.Bind_Menu] then STATE.Visible = not STATE.Visible; main.Visible = STATE.Visible end
    
    -- ULTRAKILL FIXED V2
    if STATE.UltraKillEnabled and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        local isRMB = UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
        local isX = UIS:IsKeyDown(Enum.KeyCode.X)
        
        if isRMB and isX then
            if i.UserInputType == Enum.UserInputType.MouseButton1 or i.KeyCode == Enum.KeyCode.X then
                if STATE.UltraCurrentTarget and STATE.UltraCurrentTarget.Parent then
                    local targetChar = STATE.UltraCurrentTarget.Parent
                    local targetHRP = targetChar:FindFirstChild("HumanoidRootPart")
                    if targetHRP and not IsFriend(Players:GetPlayerFromCharacter(targetChar).UserId) then
                        local myHRP = lp.Character.HumanoidRootPart
                        local oldPos = myHRP.CFrame
                        -- ТП И РАЗВОРОТ К ЦЕЛИ
                        myHRP.CFrame = CFrame.new(targetHRP.Position, targetHRP.Position + targetHRP.CFrame.LookVector)
                        task.wait(0.003)
                        VIM:SendMouseButtonEvent(0, 0, 0, true, game, 0)
                        VIM:SendMouseButtonEvent(0, 0, 0, false, game, 0)
                        task.wait(0.007)
                        myHRP.CFrame = oldPos
                    end
                end
            end
        end
    end

    -- FIXED TP BEHIND (FACING TARGET)
    if not g and i.KeyCode == Enum.KeyCode.E and STATE.TPBehindEnabled and STATE.TPBehindTarget and lp.Character then
        local myHRP = lp.Character:FindFirstChild("HumanoidRootPart")
        local targetHRP = STATE.TPBehindTarget.Character:FindFirstChild("HumanoidRootPart")
        if myHRP and targetHRP then
            -- ТП за спину на 3 метра + разворот в сторону движения цели
            myHRP.CFrame = targetHRP.CFrame * CFrame.new(0, 0, 3)
            -- Принудительно разворачиваем камеру туда же, куда смотрит цель
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + targetHRP.CFrame.LookVector)
        end
    end

    if not g and i.KeyCode == Enum.KeyCode.Q and STATE.HookEnabled then 
        local unitRay = Camera:ViewportPointToRay(UIS:GetMouseLocation().X, UIS:GetMouseLocation().Y)
        local params = RaycastParams.new(); params.FilterType = Enum.RaycastFilterType.Exclude; params.FilterDescendantsInstances = {lp.Character}
        local res = workspace:Raycast(unitRay.Origin, unitRay.Direction * 5000, params)
        if res then hookTarget = res.Position; hooking = true end
    end
end)

UIS.InputEnded:Connect(function(i, g) if i.KeyCode == Enum.KeyCode.Q then hooking = false end end)

LoadConfig()
pages.Aim.Visible = true
