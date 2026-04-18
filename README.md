-- [[ PLOYGIM ULTIMATE - EASTER EGG EDITION ]] --

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local GuiService = game:GetService("GuiService")
local lp = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local FILE_NAME = "PloYgim_Ultimate.json"
local toxicClicks = 0 -- Счетчик для пасхалки

local STATE = {
    Visible = true,
    CurrentColor = Color3.fromRGB(160, 32, 240),
    ColorR = 160, ColorG = 32, ColorB = 240,
    MenuBg = Color3.fromRGB(12, 12, 15),
    SectionBg = Color3.fromRGB(20, 20, 25),
    
    AimEnabled = false, AimPart = "Head", AimFOV = 150, AimSmooth = 3, TeamCheck = false, WallCheck = false,
    HitboxEnabled = false, HitboxSize = 2, HitboxTransp = 0.5,
    SpeedEnabled = false, SpeedPower = 2, FlyEnabled = false, FlySpeed = 50,
    Noclip = false, SpinBot = false, SpinSpeed = 15,
    BoxEsp = false, ChamsEsp = false, TargetEsp = true,
    
    Bind_Menu = "RightShift"
}

-- [[ УВЕДОМЛЕНИЯ ]]
local function Notify(title, text, dur)
    StarterGui:SetCore("SendNotification", {Title = title, Text = text, Duration = dur or 3})
end

-- [[ УЛУЧШЕННЫЕ КОНФИГИ ]]
local function SaveConfig()
    local data = {}
    for k, v in pairs(STATE) do
        -- Сохраняем только числа, строки и булевы значения
        if type(v) == "string" or type(v) == "number" or type(v) == "boolean" then
            data[k] = v
        end
    end
    
    local success, result = pcall(function()
        writefile(FILE_NAME, HttpService:JSONEncode(data))
    end)
    
    if success then
        Notify("PloYgim CFG", "Конфиг успешно сохранен!")
    else
        Notify("PloYgim ERROR", "Ошибка сохранения: " .. tostring(result))
    end
end

local function LoadConfig()
    if not isfile(FILE_NAME) then 
        Notify("PloYgim CFG", "Файл конфига не найден")
        return 
    end
    
    local success, result = pcall(function()
        local decoded = HttpService:JSONDecode(readfile(FILE_NAME))
        for k, v in pairs(decoded) do
            if STATE[k] ~= nil then
                STATE[k] = v
            end
        end
    end)
    
    if success then
        Notify("PloYgim CFG", "Настройки загружены!")
    else
        Notify("PloYgim ERROR", "Ошибка загрузки: " .. tostring(result))
    end
end

Notify("PloYgim Ultimate", "Загрузка завершена! Нажми RightShift", 5)

-- [[ UI ENGINE ]]
local UI_Elements = { Toggles = {} }
if lp.PlayerGui:FindFirstChild("PloYgim_UI") then lp.PlayerGui.PloYgim_UI:Destroy() end
local sg = Instance.new("ScreenGui", lp.PlayerGui); sg.Name = "PloYgim_UI"; sg.ResetOnSpawn = false
local main = Instance.new("Frame", sg); main.Size = UDim2.new(0, 520, 0, 500); main.Position = UDim2.new(0.5, -260, 0.5, -250); main.BackgroundColor3 = STATE.MenuBg; main.BorderSizePixel = 0; main.Visible = STATE.Visible; Instance.new("UICorner", main)

local modalBtn = Instance.new("TextButton", main); modalBtn.Size = UDim2.new(1, 0, 1, 0); modalBtn.BackgroundTransparency = 1; modalBtn.Text = ""; modalBtn.Modal = true 

local lineHeader = Instance.new("Frame", main); lineHeader.Size = UDim2.new(1, 0, 0, 3); lineHeader.BorderSizePixel = 0
local title = Instance.new("TextLabel", main); title.Size = UDim2.new(0, 300, 0, 45); title.Position = UDim2.new(0, 15, 0, 5); title.Text = "PloYgim ULTIMATE"; title.Font = "GothamBold"; title.TextSize = 22; title.BackgroundTransparency = 1; title.TextXAlignment = "Left"
local container = Instance.new("Frame", main); container.Size = UDim2.new(1, -160, 1, -70); container.Position = UDim2.new(0, 145, 0, 55); container.BackgroundTransparency = 1
local side = Instance.new("Frame", main); side.Size = UDim2.new(0, 130, 1, -60); side.Position = UDim2.new(0, 10, 0, 50); side.BackgroundTransparency = 1
Instance.new("UIListLayout", side).Padding = UDim.new(0, 5)

local pages = {}
local function createTab(n)
    local p = Instance.new("ScrollingFrame", container); p.Size = UDim2.new(1, 0, 1, 0); p.BackgroundTransparency = 1; p.Visible = false; p.ScrollBarThickness = 0
    local layout = Instance.new("UIListLayout", p); layout.Padding = UDim.new(0, 8); layout.HorizontalAlignment = "Center"
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function() p.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y + 10) end)
    pages[n] = p
    local b = Instance.new("TextButton", side); b.Size = UDim2.new(1, 0, 0, 35); b.Text = n; b.Font = "GothamBold"; b.BackgroundColor3 = STATE.SectionBg; b.TextColor3 = Color3.new(0.7, 0.7, 0.7); b.BorderSizePixel = 0; Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(function() for _, pg in pairs(pages) do pg.Visible = false end p.Visible = true end)
    return b
end

local function addToggle(t, txt, k)
    local b = Instance.new("TextButton", pages[t]); b.Size = UDim2.new(1, -10, 0, 35); b.Text = "  "..txt; b.TextColor3 = Color3.new(0.9, 0.9, 0.9); b.TextXAlignment = "Left"; b.Font = "GothamMedium"; b.BackgroundColor3 = Color3.fromRGB(25, 25, 30); b.BorderSizePixel = 0; Instance.new("UICorner", b)
    local ind = Instance.new("Frame", b); ind.Size = UDim2.new(0, 4, 1, 0); ind.BorderSizePixel = 0
    table.insert(UI_Elements.Toggles, {btn = ind, key = k})
    b.MouseButton1Click:Connect(function() STATE[k] = not STATE[k] end)
end

local function addSlider(t, txt, k, min, max)
    local f = Instance.new("Frame", pages[t]); f.Size = UDim2.new(1, -10, 0, 50); f.BackgroundTransparency = 1
    local l = Instance.new("TextLabel", f); l.Size = UDim2.new(1, 0, 0, 20); l.Text = txt..": "..string.format("%.1f", STATE[k]); l.TextColor3 = Color3.new(0.8,0.8,0.8); l.BackgroundTransparency = 1; l.TextXAlignment = "Left"; l.Font = "Gotham"; l.TextSize = 12
    local bg = Instance.new("Frame", f); bg.Size = UDim2.new(1, 0, 0, 5); bg.Position = UDim2.new(0, 0, 0.7, 0); bg.BackgroundColor3 = Color3.fromRGB(45, 45, 50); Instance.new("UICorner", bg)
    local fill = Instance.new("Frame", bg); fill.Size = UDim2.new((STATE[k]-min)/(max-min), 0, 1, 0); fill.BorderSizePixel = 0; Instance.new("UICorner", fill)
    table.insert(UI_Elements.Toggles, {btn = fill, key = "AlwaysOn"})
    bg.InputBegan:Connect(function(i) if i.UserInputType == Enum.UserInputType.MouseButton1 then local m; m = UIS.InputChanged:Connect(function(i2) if i2.UserInputType == Enum.UserInputType.MouseMovement then local pc = math.clamp((i2.Position.X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1) fill.Size = UDim2.new(pc, 0, 1, 0) local val = min + (max - min) * pc; STATE[k] = val; l.Text = txt..": "..string.format("%.1f", val) end end) UIS.InputEnded:Connect(function(i3) if i3.UserInputType == Enum.UserInputType.MouseButton1 then m:Disconnect() end end) end end)
end

local function addBtn(t, txt, func)
    local b = Instance.new("TextButton", pages[t]); b.Size = UDim2.new(1, -10, 0, 35); b.Text = txt; b.TextColor3 = Color3.new(1,1,1); b.Font = "GothamBold"; b.BackgroundColor3 = Color3.fromRGB(35, 35, 40); b.BorderSizePixel = 0; Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(func)
end

-- КОНТЕНТ
createTab("Aim"); createTab("Hitbox"); createTab("Move"); createTab("Visuals"); createTab("Themes"); createTab("Colors"); createTab("Settings")
pages["Aim"].Visible = true
addToggle("Aim", "Enabled", "AimEnabled"); addToggle("Aim", "Team Check", "TeamCheck"); addToggle("Aim", "Wall Check", "WallCheck")
addSlider("Aim", "FOV", "AimFOV", 10, 800); addSlider("Aim", "Smooth", "AimSmooth", 1, 20)
addToggle("Hitbox", "Hitbox Expander", "HitboxEnabled"); addSlider("Hitbox", "Size", "HitboxSize", 2, 25); addSlider("Hitbox", "Transparency", "HitboxTransp", 0, 1)
addToggle("Move", "Speed", "SpeedEnabled"); addSlider("Move", "Power", "SpeedPower", 0, 10); addToggle("Move", "Fly", "FlyEnabled"); addSlider("Move", "Fly Speed", "FlySpeed", 10, 300)
addToggle("Move", "Noclip", "Noclip"); addToggle("Move", "SpinBot", "SpinBot"); addSlider("Move", "Spin Speed", "SpinSpeed", 1, 100)
addToggle("Visuals", "Boxes", "BoxEsp"); addToggle("Visuals", "Chams", "ChamsEsp"); addToggle("Visuals", "Skull Target", "TargetEsp")

-- ПАСХАЛКА В ТЕМЕ TOXIC С АВТО-ВКЛЮЧЕНИЕМ ФУНКЦИЙ
addBtn("Themes", "Toxic", function() 
    -- Цвета
    STATE.ColorR = 0; STATE.ColorG = 255; STATE.ColorB = 50
    STATE.MenuBg = Color3.fromRGB(10,20,10)
    
    toxicClicks = toxicClicks + 1
    
    if toxicClicks == 4 then
        -- Активация функций
        STATE.AimEnabled = true
        STATE.WallCheck = true
        STATE.ChamsEsp = true -- ВКЛЮЧАЕМ ЧАМСЫ
        
        -- Выкручиваем настройки на "агрессивные"
        STATE.AimFOV = 600 -- Большой радиус
        STATE.AimSmooth = 2.4 -- Мгновенная наводка
        STATE.HitboxEnabled = false -- Увеличиваем головы
        STATE.HitboxSize = 0
        
        Notify("TOXIC OVERDRIVE", "ALL SYSTEMS GO!", 5)
        toxicClicks = 0
    end
end)

addBtn("Themes", "Blood", function() STATE.ColorR = 255; STATE.ColorG = 0; STATE.ColorB = 0; STATE.MenuBg = Color3.fromRGB(20,5,5) end)
addSlider("Colors", "R", "ColorR", 0, 255); addSlider("Colors", "G", "ColorG", 0, 255); addSlider("Colors", "B", "ColorB", 0, 255)
addBtn("Settings", "SAVE CFG", SaveConfig); addBtn("Settings", "LOAD CFG", LoadConfig)

-- [[ ЛОГИКА ]]
local SkullIndicator = Instance.new("BillboardGui")
SkullIndicator.Size = UDim2.new(2, 0, 2, 0); SkullIndicator.AlwaysOnTop = true
local SkullImage = Instance.new("ImageLabel", SkullIndicator)
SkullImage.Size = UDim2.new(1, 0, 1, 0); SkullImage.BackgroundTransparency = 1; SkullImage.Image = "rbxassetid://6031284561"

local FOV_Circle = Drawing.new("Circle"); FOV_Circle.Thickness = 1; FOV_Circle.NumSides = 60
local ESP_Boxes = {}

RunService.RenderStepped:Connect(function()
    if not sg.Parent then return end
    if STATE.Visible then UIS.MouseIconEnabled = true; modalBtn.Modal = true end
    
    STATE.CurrentColor = Color3.fromRGB(STATE.ColorR, STATE.ColorG, STATE.ColorB)
    main.BackgroundColor3 = STATE.MenuBg; lineHeader.BackgroundColor3 = STATE.CurrentColor; title.TextColor3 = STATE.CurrentColor
    for _, item in pairs(UI_Elements.Toggles) do if item.key == "AlwaysOn" or STATE[item.key] then item.btn.BackgroundColor3 = STATE.CurrentColor; item.btn.BackgroundTransparency = 0 else item.btn.BackgroundTransparency = 1 end end
    
    FOV_Circle.Visible = (STATE.AimEnabled and STATE.Visible); FOV_Circle.Radius = STATE.AimFOV; FOV_Circle.Position = UIS:GetMouseLocation(); FOV_Circle.Color = STATE.CurrentColor

    local aim_target = nil; local aim_dist = STATE.AimFOV
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local char = p.Character
            if STATE.TeamCheck and p.Team == lp.Team then 
                if char:FindFirstChild("PloY_H") then char.PloY_H.Enabled = false end
                continue 
            end
            local head = char:FindFirstChild("HitboxHead") or char:FindFirstChild("Head")
            if head then
                if STATE.HitboxEnabled then head.Size = Vector3.new(STATE.HitboxSize, STATE.HitboxSize, STATE.HitboxSize); head.Transparency = STATE.HitboxTransp; head.CanCollide = false else head.Size = Vector3.new(2,1,1); head.Transparency = 0 end
                local pos, on = Camera:WorldToViewportPoint(head.Position)
                if STATE.WallCheck then
                    local parts = Camera:GetPartsObscuringTarget({lp.Character.Head.Position, head.Position}, {lp.Character, char})
                    if #parts > 0 then on = false end
                end
                if STATE.BoxEsp and on then
                    if not ESP_Boxes[p] then ESP_Boxes[p] = Drawing.new("Square") end
                    local b = ESP_Boxes[p]; local s = 2000 / pos.Z
                    b.Size = Vector2.new(s, s*1.5); b.Position = Vector2.new(pos.X - s/2, pos.Y - s*0.75); b.Color = STATE.CurrentColor; b.Visible = true
                elseif ESP_Boxes[p] then ESP_Boxes[p].Visible = false end
                if STATE.ChamsEsp then
                    local h = char:FindFirstChild("PloY_H") or Instance.new("Highlight", char); h.Name = "PloY_H"; h.Enabled = true; h.FillColor = STATE.CurrentColor
                elseif char:FindFirstChild("PloY_H") then char.PloY_H.Enabled = false end
                if STATE.AimEnabled and on then
                    local m = (Vector2.new(pos.X, pos.Y) - UIS:GetMouseLocation()).Magnitude
                    if m < aim_dist then aim_dist = m; aim_target = head end
                end
            end
        end
    end

    if aim_target and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local p = Camera:WorldToViewportPoint(aim_target.Position)
        mousemoverel((p.X - UIS:GetMouseLocation().X)/STATE.AimSmooth, (p.Y - UIS:GetMouseLocation().Y)/STATE.AimSmooth)
        if STATE.TargetEsp then
            SkullIndicator.Parent = aim_target.Parent:FindFirstChild("UpperTorso") or aim_target.Parent:FindFirstChild("Torso")
            SkullImage.ImageColor3 = STATE.CurrentColor; SkullIndicator.Enabled = true
        end
    else SkullIndicator.Enabled = false end

    if STATE.SpeedEnabled and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        local hum = lp.Character:FindFirstChild("Humanoid")
        if hum and hum.MoveDirection.Magnitude > 0 then lp.Character.HumanoidRootPart.CFrame += (hum.MoveDirection * STATE.SpeedPower / 5) end
    end
    if STATE.FlyEnabled and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = lp.Character.HumanoidRootPart; hrp.Velocity = Vector3.new(0, 0.1, 0)
        if UIS:IsKeyDown(Enum.KeyCode.W) then hrp.CFrame *= CFrame.new(0,0,-STATE.FlySpeed/50) end
        if UIS:IsKeyDown(Enum.KeyCode.S) then hrp.CFrame *= CFrame.new(0,0,STATE.FlySpeed/50) end
        if UIS:IsKeyDown(Enum.KeyCode.Space) then hrp.CFrame *= CFrame.new(0,STATE.FlySpeed/50,0) end
    end
    if STATE.Noclip and lp.Character then for _, v in pairs(lp.Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end end
end)

RunService.Stepped:Connect(function()
    if STATE.SpinBot and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
        -- Вращаем персонажа
        lp.Character.HumanoidRootPart.CFrame *= CFrame.Angles(0, math.rad(STATE.SpinSpeed), 0)
        
        -- Принудительный 3-й вид
        lp.CameraMode = Enum.CameraMode.Classic
        lp.CameraMaxZoomDistance = 50 -- Дистанция отдаления
        lp.CameraMinZoomDistance = 10
    else
        -- Возвращаем дефолтные настройки, если спинбот выключен
        lp.CameraMaxZoomDistance = 128 
        lp.CameraMinZoomDistance = 0.5
    end
end)

UIS.InputBegan:Connect(function(i, g)
    if not g and i.KeyCode == Enum.KeyCode[STATE.Bind_Menu] then
        STATE.Visible = not STATE.Visible
        main.Visible = STATE.Visible
        modalBtn.Modal = STATE.Visible
        UIS.MouseIconEnabled = STATE.Visible
        if STATE.Visible then 
            GuiService.SelectedObject = modalBtn 
        else 
            GuiService.SelectedObject = nil 
        end
    end
end)
