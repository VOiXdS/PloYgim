-- [[ PLOYGIM ULTIMATE - ПОЛНОСТЬЮ НА РУССКОМ + ФИКС ЧАМСОВ ]] --

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local lp = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local FILE_NAME = "PloYgim_Ultimate.json"
local toxicClicks = 0 

-- Объект физического вращения (BodyAngularVelocity)
local bAV = Instance.new("BodyAngularVelocity")
bAV.MaxTorque = Vector3.new(0, math.huge, 0)
bAV.P = 15000

local STATE = {
    Visible = true,
    CurrentColor = Color3.fromRGB(160, 32, 240),
    ColorR = 160, ColorG = 32, ColorB = 240,
    MenuBg = Color3.fromRGB(12, 12, 15),
    SectionBg = Color3.fromRGB(20, 20, 25),
    
    -- Aim
    AimEnabled = false, AimFOV = 150, AimSmooth = 3, TeamCheck = false, WallCheck = false,
    -- Move
    SpeedEnabled = false, SpeedPower = 2, 
    FlyEnabled = false, FlySpeed = 50,
    Noclip = false, SpinBot = false, SpinSpeed = 120,
    -- Visuals
    BoxEsp = false, ChamsEsp = false, TargetEsp = true, TargetHUDEnabled = true,
    -- TP
    TPBehindEnabled = false, TPBehindTarget = nil, TPBehindBind = "E", TPColor = Color3.fromRGB(0, 191, 255),
    
    Bind_Menu = "RightShift"
}

-- [[ ПРОВЕРКИ (СТЕНЫ/КОМАНДА) ]]
local function IsVisible(part)
    if not STATE.WallCheck then return true end
    local castPoints = {Camera.CFrame.Position, part.Position}
    local ignoreList = {lp.Character, part.Parent}
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = ignoreList
    local result = workspace:Raycast(castPoints[1], castPoints[2] - castPoints[1], params)
    return not result
end

local function IsTeammate(player)
    if not STATE.TeamCheck then return false end
    return player.Team == lp.Team
end

-- [[ СИСТЕМА КОНФИГОВ ]]
local function SaveConfig()
    local data = {}
    for k, v in pairs(STATE) do
        if type(v) == "string" or type(v) == "number" or type(v) == "boolean" then data[k] = v end
    end
    writefile(FILE_NAME, HttpService:JSONEncode(data))
    StarterGui:SetCore("SendNotification", {Title = "КОНФИГ", Text = "Настройки успешно сохранены!"})
end

local function LoadConfig()
    if isfile(FILE_NAME) then
        local decoded = HttpService:JSONDecode(readfile(FILE_NAME))
        for k, v in pairs(decoded) do if STATE[k] ~= nil then STATE[k] = v end end
        StarterGui:SetCore("SendNotification", {Title = "КОНФИГ", Text = "Настройки загружены!"})
    end
end

-- [[ UI ДВИЖОК ]]
local UI_Elements = { Toggles = {} }
if lp.PlayerGui:FindFirstChild("PloYgim_UI") then lp.PlayerGui.PloYgim_UI:Destroy() end
local sg = Instance.new("ScreenGui", lp.PlayerGui); sg.Name = "PloYgim_UI"; sg.ResetOnSpawn = false
local main = Instance.new("Frame", sg); main.Size = UDim2.new(0, 520, 0, 500); main.Position = UDim2.new(0.5, -260, 0.5, -250); main.BackgroundColor3 = STATE.MenuBg; main.BorderSizePixel = 0; main.Visible = STATE.Visible; Instance.new("UICorner", main)

-- [[ ТАРГЕТ ХАД (ИНФО О ЦЕЛИ) ]]
local thud = Instance.new("Frame", sg); thud.Size = UDim2.new(0, 200, 0, 60); thud.Position = UDim2.new(0.5, 100, 0.5, 50); thud.BackgroundColor3 = Color3.fromRGB(15, 15, 20); thud.Visible = false; thud.BorderSizePixel = 0; Instance.new("UICorner", thud)
local thud_line = Instance.new("Frame", thud); thud_line.Size = UDim2.new(1, 0, 0, 2); thud_line.BorderSizePixel = 0
local thud_name = Instance.new("TextLabel", thud); thud_name.Size = UDim2.new(1, -10, 0, 30); thud_name.Position = UDim2.new(0, 10, 0, 5); thud_name.BackgroundTransparency = 1; thud_name.TextColor3 = Color3.new(1,1,1); thud_name.TextXAlignment = "Left"; thud_name.Font = "GothamBold"; thud_name.TextSize = 14; thud_name.Text = "Цель"
local thud_hp_bg = Instance.new("Frame", thud); thud_hp_bg.Size = UDim2.new(1, -20, 0, 6); thud_hp_bg.Position = UDim2.new(0, 10, 0, 40); thud_hp_bg.BackgroundColor3 = Color3.fromRGB(30, 30, 35); Instance.new("UICorner", thud_hp_bg)
local thud_hp_fill = Instance.new("Frame", thud_hp_bg); thud_hp_fill.Size = UDim2.new(1, 0, 1, 0); thud_hp_fill.BorderSizePixel = 0; Instance.new("UICorner", thud_hp_fill)

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
    bg.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            local m; m = RunService.RenderStepped:Connect(function()
                local pc = math.clamp((UIS:GetMouseLocation().X - bg.AbsolutePosition.X) / bg.AbsoluteSize.X, 0, 1)
                fill.Size = UDim2.new(pc, 0, 1, 0); local val = min + (max - min) * pc; STATE[k] = val; l.Text = txt..": "..string.format("%.1f", val)
            end)
            local up; up = UIS.InputEnded:Connect(function(i2) if i2.UserInputType == Enum.UserInputType.MouseButton1 then m:Disconnect() up:Disconnect() end end)
        end
    end)
end

local function addBtn(t, txt, func)
    local b = Instance.new("TextButton", pages[t]); b.Size = UDim2.new(1, -10, 0, 35); b.Text = txt; b.TextColor3 = Color3.new(1,1,1); b.Font = "GothamBold"; b.BackgroundColor3 = Color3.fromRGB(35, 35, 40); b.BorderSizePixel = 0; Instance.new("UICorner", b)
    b.MouseButton1Click:Connect(func)
end

-- ВКЛАДКИ (НА РУССКОМ)
createTab("Аим"); createTab("Движение"); createTab("Визуалы"); createTab("Темы"); createTab("Цвета"); createTab("Настройки")
pages["Аим"].Visible = true

addToggle("Аим", "Включить", "AimEnabled"); addToggle("Аим", "Проверка тимы", "TeamCheck"); addToggle("Аим", "Проверка стен", "WallCheck"); addSlider("Аим", "FOV (Радиус)", "AimFOV", 10, 800); addSlider("Аим", "Плавность", "AimSmooth", 1, 20)
addToggle("Движение", "ТП за спину (E)", "TPBehindEnabled"); addToggle("Движение", "Спидхак", "SpeedEnabled"); addSlider("Движение", "Сила скорости", "SpeedPower", 0, 15); addToggle("Движение", "Полет (Fly)", "FlyEnabled"); addSlider("Движение", "Скорость полета", "FlySpeed", 10, 300); addToggle("Движение", "Проход сквозь стены", "Noclip"); addToggle("Движение", "СпинБот (Физика)", "SpinBot"); addSlider("Движение", "Скорость вращения", "SpinSpeed", 10, 1500)
addToggle("Визуалы", "Боксы", "BoxEsp"); addToggle("Визуалы", "Чамсы (Сквозь стены)", "ChamsEsp"); addToggle("Визуалы", "Инфо о цели", "TargetHUDEnabled")

addBtn("Темы", "Токсик Мод", function() STATE.ColorR = 0; STATE.ColorG = 255; STATE.ColorB = 50; STATE.MenuBg = Color3.fromRGB(5, 15, 5); toxicClicks = toxicClicks + 1; if toxicClicks >= 4 then STATE.AimEnabled = true; STATE.AimFOV = 700; STATE.SpeedEnabled = true; toxicClicks = 0; StarterGui:SetCore("SendNotification", {Title = "ТОКСИК", Text = "РЕЖИМ УНИЧТОЖЕНИЯ ВКЛЮЧЕН!"}) end end)
addBtn("Темы", "Кровавая Луна", function() STATE.ColorR = 255; STATE.ColorG = 0; STATE.ColorB = 0; STATE.MenuBg = Color3.fromRGB(20,5,5) end)

addSlider("Цвета", "Красный (R)", "ColorR", 0, 255); addSlider("Цвета", "Зеленый (G)", "ColorG", 0, 255); addSlider("Цвета", "Синий (B)", "ColorB", 0, 255)
addBtn("Настройки", "СОХРАНИТЬ КФГ", SaveConfig); addBtn("Настройки", "ЗАГРУЗИТЬ КФГ", LoadConfig)

-- [[ ЛОГИКА ]]
local FOV_Circle = Drawing.new("Circle"); FOV_Circle.Thickness = 1; FOV_Circle.NumSides = 60

RunService.RenderStepped:Connect(function()
    STATE.CurrentColor = Color3.fromRGB(STATE.ColorR, STATE.ColorG, STATE.ColorB)
    main.BackgroundColor3 = STATE.MenuBg; lineHeader.BackgroundColor3 = STATE.CurrentColor; title.TextColor3 = STATE.CurrentColor
    thud_line.BackgroundColor3 = STATE.CurrentColor; thud_hp_fill.BackgroundColor3 = STATE.CurrentColor
    
    for _, item in pairs(UI_Elements.Toggles) do if item.key == "AlwaysOn" or STATE[item.key] then item.btn.BackgroundColor3 = STATE.CurrentColor; item.btn.BackgroundTransparency = 0 else item.btn.BackgroundTransparency = 1 end end
    FOV_Circle.Visible = (STATE.AimEnabled and STATE.Visible); FOV_Circle.Radius = STATE.AimFOV; FOV_Circle.Position = UIS:GetMouseLocation(); FOV_Circle.Color = STATE.CurrentColor

    local char = lp.Character; local hrp = char and char:FindFirstChild("HumanoidRootPart"); local hum = char and char:FindFirstChild("Humanoid")
    local mouseLoc = UIS:GetMouseLocation()

    -- SILENT SPIN (ФИЗИЧЕСКИЙ)
    if STATE.SpinBot and hrp and hum then
        bAV.Parent = hrp
        bAV.AngularVelocity = Vector3.new(0, STATE.SpinSpeed, 0)
        hum.AutoRotate = false
        hum.PlatformStand = true 
        if (Camera.Focus.p - Camera.CoordinateFrame.p).Magnitude < 1 then lp.CameraMode = Enum.CameraMode.Classic end
    elseif hum then
        bAV.Parent = nil
        hum.AutoRotate = true
        hum.PlatformStand = false
    end

    -- NOCLIP
    if STATE.Noclip and char then
        for _, v in pairs(char:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide = false end end
    end

    -- ПОИСК ЦЕЛИ
    local aim_target = nil; local hud_p = nil; local shortest = STATE.AimFOV
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("Head") and not IsTeammate(p) then
            local head = p.Character.Head
            if IsVisible(head) then
                local pos, on = Camera:WorldToViewportPoint(head.Position)
                if on then
                    local d = (Vector2.new(pos.X, pos.Y) - mouseLoc).Magnitude
                    if d < shortest then shortest = d; aim_target = head; hud_p = p end
                end
            end
        end
    end
    STATE.TPBehindTarget = (STATE.TPBehindEnabled and hud_p) or nil

    -- ТАРГЕТ ХАД
    if STATE.TargetHUDEnabled and hud_p and hud_p.Character:FindFirstChild("Humanoid") then
        local h = hud_p.Character.Humanoid
        thud_name.Text = hud_p.Name; thud_hp_fill.Size = UDim2.new(math.clamp(h.Health/h.MaxHealth, 0, 1), 0, 1, 0); thud.Visible = true
    else thud.Visible = false end

    -- ЧАМСЫ (ФИКС: ОБНОВЛЕНИЕ КАЖДЫЙ КАДР)
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local highlight = p.Character:FindFirstChild("PloY_Visual")
            if STATE.ChamsEsp or (STATE.TPBehindEnabled and p == STATE.TPBehindTarget) then
                if not highlight then 
                    highlight = Instance.new("Highlight", p.Character)
                    highlight.Name = "PloY_Visual"
                end
                highlight.Enabled = true
                highlight.FillColor = (p == STATE.TPBehindTarget) and STATE.TPColor or STATE.CurrentColor
                highlight.OutlineColor = Color3.new(1,1,1)
                highlight.FillTransparency = 0.5
                highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            else
                if highlight then highlight:Destroy() end
            end
        end
    end

    -- СПИДХАК И ПОЛЕТ
    if hrp and hum then
        if STATE.SpeedEnabled and hum.MoveDirection.Magnitude > 0 then hrp.CFrame += (hum.MoveDirection * STATE.SpeedPower / 5) end
        if STATE.FlyEnabled then
            hrp.Velocity = Vector3.new(0, 0.1, 0)
            local fDir = Vector3.new(0,0,0)
            if UIS:IsKeyDown(Enum.KeyCode.W) then fDir += Camera.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.S) then fDir -= Camera.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.A) then fDir -= Camera.CFrame.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.D) then fDir += Camera.CFrame.RightVector end
            if fDir.Magnitude > 0 then hrp.CFrame += (fDir.Unit * (STATE.FlySpeed / 50)) end
        end
    end

    -- АИМБОТ
    if STATE.AimEnabled and aim_target and UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
        local p = Camera:WorldToViewportPoint(aim_target.Position)
        mousemoverel((p.X - mouseLoc.X)/STATE.AimSmooth, (p.Y - mouseLoc.Y)/STATE.AimSmooth)
    end
end)

-- [[ КНОПКИ ]]
UIS.InputBegan:Connect(function(i, g)
    if not g then
        if i.KeyCode == Enum.KeyCode[STATE.Bind_Menu] then 
            STATE.Visible = not STATE.Visible; main.Visible = STATE.Visible
        elseif i.KeyCode == Enum.KeyCode[STATE.TPBehindBind] and STATE.TPBehindEnabled and STATE.TPBehindTarget then
            local tChar = STATE.TPBehindTarget.Character
            if tChar and tChar:FindFirstChild("HumanoidRootPart") and lp.Character:FindFirstChild("HumanoidRootPart") then
                lp.Character.HumanoidRootPart.CFrame = tChar.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
            end
        end
    end
end)
