local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")

-- VARIÁVEIS
local ESP_ENABLED = false
local BOX_ENABLED = false
local RAINBOW_ESP_ENABLED = false
local CIRCLE_ENABLED = false
local RAINBOW_CIRCLE_ENABLED = false
local WALL_CHECK_ENABLED = false
local TEAM_CHECK_ENABLED = false
local FOV_SIZE = 100
local AIM_VELOCITY = 1.5
local ESP_COLOR = Color3.new(1, 0, 0)
local espData = {}
local IS_MINIMIZED = false
local MAIN_LOOP_CONNECTION = nil
local SPIN_CONNECTION = nil
local TOGGLE_SPIN_CONNECTION = nil
local GUI_IS_FOCUSED = false

-- ScreenGui - ACIMA DE TUDO
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.IgnoreGuiInset = true
ScreenGui.ResetOnSpawn = false
ScreenGui.Name = "JzinnInterfaceV2"
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.DisplayOrder = 999999 -- ACIMA DE TUDO

-- CIRCLE FOV
local CircleFrame = Instance.new("Frame")
CircleFrame.Size = UDim2.new(0, FOV_SIZE*2, 0, FOV_SIZE*2)
CircleFrame.Position = UDim2.new(0.5, -FOV_SIZE, 0.5, -FOV_SIZE)
CircleFrame.BackgroundTransparency = 1
CircleFrame.Visible = false
CircleFrame.ZIndex = 1000
CircleFrame.Parent = ScreenGui

local CircleStroke = Instance.new("UIStroke")
CircleStroke.Color = ESP_COLOR
CircleStroke.Thickness = 3
CircleStroke.Parent = CircleFrame

local CircleCorner = Instance.new("UICorner")
CircleCorner.CornerRadius = UDim.new(1, 0)
CircleCorner.Parent = CircleFrame

-- TOGGLE BUTTON MELHORADO (MOBILE FRIENDLY)
local ToggleButton = Instance.new("TextButton")
ToggleButton.Parent = ScreenGui
ToggleButton.Size = UDim2.new(0, 85, 0, 35) -- MAIOR PARA MOBILE
ToggleButton.Position = UDim2.new(0, 15, 0, 60)
ToggleButton.BackgroundColor3 = Color3.fromRGB(74, 74, 74)
ToggleButton.BackgroundTransparency = 0.2
ToggleButton.Text = "Toggle"
ToggleButton.TextColor3 = Color3.new(1, 1, 1)
ToggleButton.TextScaled = false
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 11 -- FONTE MENOR
ToggleButton.Visible = false
ToggleButton.ZIndex = 1001

local ToggleCorner = Instance.new("UICorner")
ToggleCorner.CornerRadius = UDim.new(0, 10)
ToggleCorner.Parent = ToggleButton

local ToggleStroke = Instance.new("UIStroke")
ToggleStroke.Color = Color3.fromRGB(0, 162, 255) -- AZUL
ToggleStroke.Thickness = 3
ToggleStroke.Parent = ToggleButton

-- GRADIENTE GIRATÓRIO NO TOGGLE
local ToggleGradient = Instance.new("UIGradient")
ToggleGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 200, 255)),
    ColorSequenceKeypoint.new(0.2, Color3.fromRGB(0, 162, 255)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 120, 200)),
    ColorSequenceKeypoint.new(0.8, Color3.fromRGB(50, 180, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 220, 255))
}
ToggleGradient.Rotation = 0
ToggleGradient.Parent = ToggleStroke

-- EFEITO PULSANTE NO TOGGLE
local fadeInfo = TweenInfo.new(1.2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
local toggleFadeTween = TweenService:Create(ToggleStroke, fadeInfo, {Transparency = 0.85})
toggleFadeTween:Play()

-- FUNÇÕES ESP
local function clearESP()
    for playerName, data in pairs(espData) do
        if data.highlight then data.highlight:Destroy() end
        if data.boxGui then data.boxGui:Destroy() end
    end
    espData = {}
end

local function createFullESP(player)
    if player == LocalPlayer then return end
    local character = player.Character
    if not character then return end
    
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end
    
    local playerName = player.Name
    espData[playerName] = {}
    
    if ESP_ENABLED then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESPHighlight"
        highlight.Parent = character
        highlight.FillColor = ESP_COLOR
        highlight.OutlineColor = ESP_COLOR
        highlight.FillTransparency = 0.4
        highlight.OutlineTransparency = 0
        espData[playerName].highlight = highlight
    end
    
    if BOX_ENABLED then
        local boxGui = Instance.new("BillboardGui")
        boxGui.Name = "BoxESP"
        boxGui.Adornee = humanoidRootPart
        boxGui.Size = UDim2.new(0, 12, 0, 18)
        boxGui.StudsOffset = Vector3.new(0, 0, 0)
        boxGui.Parent = CoreGui
        boxGui.AlwaysOnTop = true
        boxGui.ZIndex = 999
        
        local boxFrame = Instance.new("Frame")
        boxFrame.Size = UDim2.new(1, 0, 1, 0)
        boxFrame.BackgroundTransparency = 1
        boxFrame.ZIndex = 1000
        boxFrame.Parent = boxGui
        
        local boxStroke = Instance.new("UIStroke")
        boxStroke.Color = ESP_COLOR
        boxStroke.Thickness = 2
        boxStroke.Parent = boxFrame
        
        espData[playerName].boxGui = boxGui
    end
end

local function isBehindWall(targetPart)
    if not WALL_CHECK_ENABLED then return false end
    local rayOrigin = Camera.CFrame.Position
    local rayDirection = (targetPart.Position - rayOrigin).Unit * 1000
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {LocalPlayer.Character}
    local raycastResult = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
    return raycastResult and raycastResult.Instance.Parent ~= targetPart.Parent
end

local function isEnemy(player)
    if not TEAM_CHECK_ENABLED then return true end
    if not LocalPlayer.Team or not player.Team then return true end
    return player.Team ~= LocalPlayer.Team
end

local bodyParts = {
    "Head", "UpperTorso", "LowerTorso", "Torso", "HumanoidRootPart",
    "LeftUpperArm", "LeftLowerArm", "LeftHand", "Left Arm",
    "RightUpperArm", "RightLowerArm", "RightHand", "Right Arm",
    "LeftUpperLeg", "LeftLowerLeg", "LeftFoot", "Left Leg",
    "RightUpperLeg", "RightLowerLeg", "RightFoot", "Right Leg"
}

local function findTargetInFOV()
    if not CIRCLE_ENABLED then return nil end
    local closestPlayer = nil
    local closestDistance = FOV_SIZE
    local screenCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and isEnemy(player) and player.Character then
            for _, partName in pairs(bodyParts) do
                local part = player.Character:FindFirstChild(partName)
                if part then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
                    if onScreen and screenPos.Z > 0 and not isBehindWall(part) then
                        local distance = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                        if distance <= FOV_SIZE and distance < closestDistance then
                            closestDistance = distance
                            closestPlayer = player
                            break
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- INTERFACE PRINCIPAL
local Main = Instance.new("Frame")
Main.Parent = ScreenGui
Main.Size = UDim2.new(0.9,0,0.7,0)
Main.Position = UDim2.new(0.5,0,0.55,0)
Main.AnchorPoint = Vector2.new(0.5,0.5)
Main.BackgroundColor3 = Color3.fromRGB(74,74,74)
Main.BackgroundTransparency = 0.25
Main.BorderSizePixel = 0
Main.Visible = true
Main.ZIndex = 1002

local Corner = Instance.new("UICorner")
Corner.CornerRadius = UDim.new(0,25)
Corner.Parent = Main

local Aspect = Instance.new("UIAspectRatioConstraint")
Aspect.Parent = Main
Aspect.AspectRatio = 1.8

-- BORDA GIRATÓRIA MAIN
local BorderStroke = Instance.new("UIStroke")
BorderStroke.Thickness = 3
BorderStroke.Color = Color3.fromRGB(0, 162, 255)
BorderStroke.Transparency = 0.5
BorderStroke.ZIndex = 1003
BorderStroke.Parent = Main

local MainGradient = Instance.new("UIGradient")
MainGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 200, 255)),
    ColorSequenceKeypoint.new(0.2, Color3.fromRGB(0, 162, 255)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 120, 200)),
    ColorSequenceKeypoint.new(0.8, Color3.fromRGB(50, 180, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 220, 255))
}
MainGradient.Rotation = 0
MainGradient.Parent = BorderStroke

-- ANIMAÇÕES MAIN
SPIN_CONNECTION = RunService.Heartbeat:Connect(function()
    MainGradient.Rotation = (MainGradient.Rotation + 3) % 360
end)

local fadeTween = TweenService:Create(BorderStroke, fadeInfo, {Transparency = 0.85})
fadeTween:Play()

-- TOPBAR
local Topbar = Instance.new("Frame")
Topbar.Parent = Main
Topbar.Size = UDim2.new(1,0,0.16,0)
Topbar.Position = UDim2.new(0,0,0,0)
Topbar.BackgroundColor3 = Color3.fromRGB(90,90,90)
Topbar.BackgroundTransparency = 0.2
Topbar.ZIndex = 1004

local TopCorner = Instance.new("UICorner")
TopCorner.CornerRadius = UDim.new(0,25)
TopCorner.Parent = Topbar

local JLogo = Instance.new("ImageLabel")
JLogo.Parent = Topbar
JLogo.Size = UDim2.new(0.08,0,0.7,0)
JLogo.Position = UDim2.new(0.05,0,0.15,0)
JLogo.BackgroundTransparency = 1
JLogo.Image = "rbxassetid://114066337075437"
JLogo.ScaleType = Enum.ScaleType.Fit
JLogo.ZIndex = 1005

local TopLine = Instance.new("Frame")
TopLine.Parent = Topbar
TopLine.Size = UDim2.new(0.006,0,0.7,0)
TopLine.Position = UDim2.new(0.165,0,0.15,0)
TopLine.BackgroundColor3 = Color3.fromRGB(130,130,130)
TopLine.BackgroundTransparency = 0.25
TopLine.ZIndex = 1005

local TopLineCorner = Instance.new("UICorner")
TopLineCorner.CornerRadius = UDim.new(1,0)
TopLineCorner.Parent = TopLine

local Name = Instance.new("TextLabel")
Name.Parent = Topbar
Name.Size = UDim2.new(0.3,0,1,0)
Name.Position = UDim2.new(0.19,0,0.08,0)
Name.BackgroundTransparency = 1
Name.Text = "by jzzin"
Name.TextScaled = false
Name.TextSize = 14
Name.Font = Enum.Font.GothamMedium
Name.TextColor3 = Color3.new(1,1,1)
Name.TextTransparency = 0.7
Name.TextXAlignment = Enum.TextXAlignment.Left
Name.TextYAlignment = Enum.TextYAlignment.Center
Name.ZIndex = 1005

local CloseBtn = Instance.new("TextButton")
CloseBtn.Parent = Topbar
CloseBtn.Size = UDim2.new(0.08,0,0.8,0)
CloseBtn.Position = UDim2.new(0.95,0,0.5,0)
CloseBtn.AnchorPoint = Vector2.new(1,0.5)
CloseBtn.BackgroundTransparency = 1
CloseBtn.Text = "×"
CloseBtn.TextScaled = true
CloseBtn.Font = Enum.Font.GothamBold
CloseBtn.TextSize = 16
CloseBtn.TextColor3 = Color3.new(1,1,1)
CloseBtn.ZIndex = 1006
CloseBtn.MouseButton1Click:Connect(function()
    if MAIN_LOOP_CONNECTION then MAIN_LOOP_CONNECTION:Disconnect() end
    if SPIN_CONNECTION then SPIN_CONNECTION:Disconnect() end
    if TOGGLE_SPIN_CONNECTION then TOGGLE_SPIN_CONNECTION:Disconnect() end
    if fadeTween then fadeTween:Cancel() end
    if toggleFadeTween then toggleFadeTween:Cancel() end
    clearESP()
    CircleFrame.Visible = false
    ScreenGui:Destroy()
end)

local MinimizeBtn = Instance.new("TextButton")
MinimizeBtn.Parent = Topbar
MinimizeBtn.Size = UDim2.new(0.08,0,0.8,0)
MinimizeBtn.Position = UDim2.new(0.87,0,0.5,0)
MinimizeBtn.AnchorPoint = Vector2.new(1,0.5)
MinimizeBtn.BackgroundTransparency = 1
MinimizeBtn.Text = " --"
MinimizeBtn.TextScaled = true
MinimizeBtn.Font = Enum.Font.GothamBold
MinimizeBtn.TextSize = 14
MinimizeBtn.TextColor3 = Color3.new(1,1,1)
MinimizeBtn.ZIndex = 1006
MinimizeBtn.MouseButton1Click:Connect(function()
    IS_MINIMIZED = not IS_MINIMIZED
    Main.Visible = not IS_MINIMIZED
    ToggleButton.Visible = IS_MINIMIZED
end)

-- TOGGLE BUTTON EVENT
ToggleButton.MouseButton1Click:Connect(function()
    IS_MINIMIZED = not IS_MINIMIZED
    Main.Visible = not IS_MINIMIZED
    ToggleButton.Visible = IS_MINIMIZED
end)

-- Sidebar + Avatar
local SidebarLine = Instance.new("Frame")
SidebarLine.Parent = Main
SidebarLine.Size = UDim2.new(0.006,0,0.65,0)
SidebarLine.Position = UDim2.new(0.165,0,0.25,0)
SidebarLine.BackgroundColor3 = Color3.fromRGB(130,130,130)
SidebarLine.BackgroundTransparency = 0.25
SidebarLine.ZIndex = 1005

local LineCorner = Instance.new("UICorner")
LineCorner.CornerRadius = UDim.new(1,0)
LineCorner.Parent = SidebarLine

local Ball = Instance.new("Frame")
Ball.Parent = Main
Ball.Size = UDim2.new(0,55,0,55)
Ball.Position = UDim2.new(0.03,0,0.85,0)
Ball.AnchorPoint = Vector2.new(0,0.5)
Ball.BackgroundColor3 = Color3.fromRGB(0,0,0)
Ball.BackgroundTransparency = 0.6
Ball.ZIndex = 1005

local BallCorner = Instance.new("UICorner")
BallCorner.CornerRadius = UDim.new(1,0)
BallCorner.Parent = Ball

local BallStroke = Instance.new("UIStroke")
BallStroke.Thickness = 3
BallStroke.Color = Color3.fromRGB(0, 162, 255)
BallStroke.Transparency = 0.5
BallStroke.ZIndex = 1006
BallStroke.Parent = Ball

local BallGradient = Instance.new("UIGradient")
BallGradient.Color = ColorSequence.new{
    ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 200, 255)),
    ColorSequenceKeypoint.new(0.2, Color3.fromRGB(0, 162, 255)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 120, 200)),
    ColorSequenceKeypoint.new(0.8, Color3.fromRGB(50, 180, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 220, 255))
}
BallGradient.Rotation = 0
BallGradient.Parent = BallStroke

local ballFadeTween = TweenService:Create(BallStroke, fadeInfo, {Transparency = 0.85})
ballFadeTween:Play()

RunService.Heartbeat:Connect(function()
    BallGradient.Rotation = (BallGradient.Rotation + 3) % 360
end)

local Avatar = Instance.new("ImageLabel")
Avatar.Parent = Ball
Avatar.Size = UDim2.new(0.9,0,0.9,0)
Avatar.Position = UDim2.new(0.05,0,0.05,0)
Avatar.BackgroundTransparency = 1
Avatar.ScaleType = Enum.ScaleType.Crop
Avatar.ClipsDescendants = true
Avatar.ZIndex = 1007

local AvatarCorner = Instance.new("UICorner")
AvatarCorner.CornerRadius = UDim.new(1,0)
AvatarCorner.Parent = Avatar

local UserId = LocalPlayer.UserId
local thumbnailUrl = Players:GetUserThumbnailAsync(UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420)
Avatar.Image = thumbnailUrl

-- LABELS CLICÁVEIS
local AimbotLabel = Instance.new("TextButton")
AimbotLabel.Parent = Main
AimbotLabel.Size = UDim2.new(0.14, 0, 0.06, 0)
AimbotLabel.Position = UDim2.new(0.035, 0, 0.28, 0)
AimbotLabel.BackgroundTransparency = 1
AimbotLabel.Text = "Aimbot"
AimbotLabel.TextScaled = true
AimbotLabel.Font = Enum.Font.GothamBold
AimbotLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
AimbotLabel.TextSize = 12
AimbotLabel.TextXAlignment = Enum.TextXAlignment.Left
AimbotLabel.ZIndex = 1005

local EspsLabel = Instance.new("TextButton")
EspsLabel.Parent = Main
EspsLabel.Size = UDim2.new(0.14, 0, 0.06, 0)
EspsLabel.Position = UDim2.new(0.035, 0, 0.38, 0)
EspsLabel.BackgroundTransparency = 1
EspsLabel.Text = "Esp's"
EspsLabel.TextScaled = true
EspsLabel.Font = Enum.Font.GothamBold
EspsLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
EspsLabel.TextSize = 12
EspsLabel.TextXAlignment = Enum.TextXAlignment.Left
EspsLabel.ZIndex = 1005

local CheckAllLabel = Instance.new("TextButton")
CheckAllLabel.Parent = Main
CheckAllLabel.Size = UDim2.new(0.14, 0, 0.06, 0)
CheckAllLabel.Position = UDim2.new(0.022, 0, 0.48, 0)
CheckAllLabel.BackgroundTransparency = 1
CheckAllLabel.Text = "Check All"
CheckAllLabel.TextScaled = true
CheckAllLabel.Font = Enum.Font.GothamBold
CheckAllLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
CheckAllLabel.TextSize = 11
CheckAllLabel.TextXAlignment = Enum.TextXAlignment.Left
CheckAllLabel.ZIndex = 1005

-- SEÇÕES
local AimbotSection = Instance.new("Frame")
AimbotSection.Parent = Main
AimbotSection.Size = UDim2.new(0.6,0,0.35,0)
AimbotSection.Position = UDim2.new(0.22,0,0.27,0)
AimbotSection.BackgroundTransparency = 1
AimbotSection.Visible = false
AimbotSection.ZIndex = 1005

local AimbotListLayout = Instance.new("UIListLayout")
AimbotListLayout.Parent = AimbotSection
AimbotListLayout.SortOrder = Enum.SortOrder.LayoutOrder
AimbotListLayout.Padding = UDim.new(0, 5)

local EspSection = Instance.new("Frame")
EspSection.Parent = Main
EspSection.Size = UDim2.new(0.6,0,0.20,0)
EspSection.Position = UDim2.new(0.22,0,0.25,0)
EspSection.BackgroundTransparency = 1
EspSection.Visible = false
EspSection.ClipsDescendants = false
EspSection.ZIndex = 1005

local EspListLayout = Instance.new("UIListLayout")
EspListLayout.Parent = EspSection
EspListLayout.SortOrder = Enum.SortOrder.LayoutOrder
EspListLayout.Padding = UDim.new(0, 2)
EspListLayout.VerticalAlignment = Enum.VerticalAlignment.Top
EspListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left

local CheckSection = Instance.new("Frame")
CheckSection.Parent = Main
CheckSection.Size = UDim2.new(0.6,0,0.15,0)
CheckSection.Position = UDim2.new(0.22,0,0.25,0)
CheckSection.BackgroundTransparency = 1
CheckSection.Visible = false
CheckSection.ZIndex = 1005

local CheckListLayout = Instance.new("UIListLayout")
CheckListLayout.Parent = CheckSection
CheckListLayout.SortOrder = Enum.SortOrder.LayoutOrder
CheckListLayout.Padding = UDim.new(0, 5)

-- FUNÇÃO PARA CONTROLAR FOCO DA GUI
local function setGuiFocus(focused)
    GUI_IS_FOCUSED = focused
end

-- FUNÇÕES DE INTERFACE
local function createToggle(parent, labelText, callback)
    local Option = Instance.new("Frame")
    Option.Size = UDim2.new(1,0,0,28)
    Option.BackgroundTransparency = 1
    Option.ZIndex = 1006
    Option.Parent = parent
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.65,0,1,0)
    Label.BackgroundTransparency = 1
    Label.Text = labelText
    Label.TextColor3 = Color3.new(1,1,1)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 11
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.ZIndex = 1007
    Label.Parent = Option
    
    local Switch = Instance.new("TextButton")
    Switch.Size = UDim2.new(0,40,0,20)
    Switch.Position = UDim2.new(1,-50,0.5,-10)
    Switch.BackgroundColor3 = Color3.fromRGB(85,85,85)
    Switch.Text = ""
    Switch.ZIndex = 1008
    Switch.Parent = Option
    
    local sc = Instance.new("UICorner")
    sc.CornerRadius = UDim.new(0,18)
    sc.Parent = Switch
    
    local Knob = Instance.new("Frame")
    Knob.Size = UDim2.new(0,14,0,14)
    Knob.Position = UDim2.new(0,3,0,3)
    Knob.BackgroundColor3 = Color3.new(1,1,1)
    Knob.ZIndex = 1009
    Knob.Parent = Switch
    
    local kc = Instance.new("UICorner")
    kc.CornerRadius = UDim.new(0.5,0)
    kc.Parent = Knob
    
    local toggled = false
    Switch.MouseButton1Down:Connect(function()
        setGuiFocus(true)
    end)
    Switch.MouseButton1Up:Connect(function()
        setGuiFocus(false)
    end)
    Switch.MouseButton1Click:Connect(function()
        toggled = not toggled
        TweenService:Create(Switch, TweenInfo.new(0.2), {
            BackgroundColor3 = toggled and Color3.new(0,1,0) or Color3.fromRGB(85,85,85)
        }):Play()
        TweenService:Create(Knob, TweenInfo.new(0.2), {
            Position = toggled and UDim2.new(1,-17,0,3) or UDim2.new(0,3,0,3)
        }):Play()
        callback(toggled)
    end)
end

local function createInputBox(parent, labelText, defaultValue, minVal, maxVal, callback)
    local Option = Instance.new("Frame")
    Option.Size = UDim2.new(1,0,0,35)
    Option.BackgroundTransparency = 1
    Option.ZIndex = 1006
    Option.Parent = parent
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.65,0,1,0)
    Label.BackgroundTransparency = 1
    Label.Text = labelText
    Label.TextColor3 = Color3.new(1,1,1)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 11
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.ZIndex = 1007
    Label.Parent = Option
    
    local InputBox = Instance.new("TextBox")
    InputBox.Size = UDim2.new(0,60,0,25)
    InputBox.Position = UDim2.new(1,-70,0.5,-12.5)
    InputBox.BackgroundColor3 = Color3.fromRGB(58,58,58)
    InputBox.Text = tostring(defaultValue)
    InputBox.TextColor3 = Color3.new(1,1,1)
    InputBox.Font = Enum.Font.Gotham
    InputBox.TextSize = 12
    InputBox.ZIndex = 1008
    InputBox.Parent = Option
    
    local InputCorner = Instance.new("UICorner")
    InputCorner.CornerRadius = UDim.new(0,8)
    InputCorner.Parent = InputBox
    
    InputBox.Focused:Connect(function()
        setGuiFocus(true)
    end)
    InputBox.FocusLost:Connect(function()
        setGuiFocus(false)
        local num = tonumber(InputBox.Text)
        if num and num >= minVal and num <= maxVal then
            callback(num)
        else
            InputBox.Text = tostring(defaultValue)
        end
    end)
end

local function createSlider(parent, labelText, defaultValue, callback)
    local Option = Instance.new("Frame")
    Option.Size = UDim2.new(1,0,0,40)
    Option.BackgroundTransparency = 1
    Option.ZIndex = 1006
    Option.Parent = parent
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.65,0,1,0)
    Label.BackgroundTransparency = 1
    Label.Text = labelText
    Label.TextColor3 = Color3.new(1,1,1)
    Label.Font = Enum.Font.Gotham
    Label.TextSize = 11
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.ZIndex = 1007
    Label.Parent = Option
    
    local SliderContainer = Instance.new("Frame")
    SliderContainer.Size = UDim2.new(0,260,0,25)
    SliderContainer.Position = UDim2.new(1,-270,0.5,-12.5)
    SliderContainer.BackgroundTransparency = 1
    SliderContainer.ZIndex = 1008
    SliderContainer.Parent = Option
    
    local SliderBg = Instance.new("Frame")
    SliderBg.Size = UDim2.new(0,240,0,8)
    SliderBg.Position = UDim2.new(0.08,0,0.5,-4)
    SliderBg.BackgroundColor3 = Color3.fromRGB(58,58,58)
    SliderBg.ZIndex = 1009
    SliderBg.Parent = SliderContainer
    
    local SliderCorner = Instance.new("UICorner")
    SliderCorner.CornerRadius = UDim.new(0,4)
    SliderCorner.Parent = SliderBg
    
    local initialRelativePos = (defaultValue - 0.5) / 6.5
    local SliderFill = Instance.new("Frame")
    SliderFill.Size = UDim2.new(math.clamp(initialRelativePos, 0, 1),0,1,0)
    SliderFill.BackgroundColor3 = Color3.new(0,1,0)
    SliderFill.BorderSizePixel = 0
    SliderFill.ZIndex = 1010
    SliderFill.Parent = SliderBg
    
    local SliderFillCorner = Instance.new("UICorner")
    SliderFillCorner.CornerRadius = UDim.new(0,4)
    SliderFillCorner.Parent = SliderFill
    
    local ValueLabel = Instance.new("TextLabel")
    ValueLabel.Size = UDim2.new(0,45,1,0)
    ValueLabel.Position = UDim2.new(1,-50,0,0)
    ValueLabel.BackgroundTransparency = 1
    ValueLabel.Text = string.format("%.1f", defaultValue)
    ValueLabel.TextColor3 = Color3.new(1,1,1)
    ValueLabel.Font = Enum.Font.GothamBold
    ValueLabel.TextSize = 12
    ValueLabel.ZIndex = 1011
    ValueLabel.Parent = SliderContainer
    
    local dragging = false
    local connection
    
    local function updateSlider(relativePos)
        local value = 0.5 + (relativePos * 6.5)
        value = math.clamp(value, 0.5, 7.0)
        SliderFill.Size = UDim2.new(relativePos,0,1,0)
        ValueLabel.Text = string.format("%.1f", value)
        callback(value)
    end
    
    SliderContainer.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            setGuiFocus(true)
            dragging = true
            connection = RunService.Heartbeat:Connect(function()
                if dragging then
                    local mouse = UserInputService:GetMouseLocation()
                    local sliderPos = SliderBg.AbsolutePosition
                    local sliderSize = SliderBg.AbsoluteSize.X
                    local relativePos = math.clamp((mouse.X - sliderPos.X) / sliderSize, 0, 1)
                    updateSlider(relativePos)
                end
            end)
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            if dragging then
                dragging = false
                setGuiFocus(false)
                if connection then
                    connection:Disconnect()
                    connection = nil
                end
            end
        end
    end)
end

-- CRIAÇÃO DOS ELEMENTOS
createToggle(EspSection, "Enable ESP", function(toggled)
    ESP_ENABLED = toggled
    clearESP()
    for _, player in pairs(Players:GetPlayers()) do
        createFullESP(player)
    end
end)

createToggle(EspSection, "Enable Boxes", function(toggled)
    BOX_ENABLED = toggled
    clearESP()
    for _, player in pairs(Players:GetPlayers()) do
        createFullESP(player)
    end
end)

createToggle(EspSection, "Rainbow All", function(toggled)
    RAINBOW_ESP_ENABLED = toggled
end)

createToggle(AimbotSection, "Enable Circle", function(toggled)
    CIRCLE_ENABLED = toggled
    CircleFrame.Visible = toggled
end)

createToggle(AimbotSection, "Rainbow Circle", function(toggled)
    RAINBOW_CIRCLE_ENABLED = toggled
end)

createInputBox(AimbotSection, "FOV Size (50-300)", FOV_SIZE, 50, 300, function(value)
    FOV_SIZE = value
    CircleFrame.Size = UDim2.new(0, FOV_SIZE*2, 0, FOV_SIZE*2)
    CircleFrame.Position = UDim2.new(0.5, -FOV_SIZE, 0.5, -FOV_SIZE)
end)

createSlider(AimbotSection, "Velocity Aim", AIM_VELOCITY, function(value)
    AIM_VELOCITY = value
end)

createToggle(CheckSection, "Wall Check", function(toggled)
    WALL_CHECK_ENABLED = toggled
end)

createToggle(CheckSection, "Teams Check", function(toggled)
    TEAM_CHECK_ENABLED = toggled
end)

-- CLICK NOS LABELS
local currentSection = nil
AimbotLabel.MouseButton1Down:Connect(function()
    setGuiFocus(true)
end)
AimbotLabel.MouseButton1Up:Connect(function()
    setGuiFocus(false)
end)
AimbotLabel.MouseButton1Click:Connect(function()
    if currentSection == AimbotSection then
        AimbotSection.Visible = false
        currentSection = nil
    else
        if currentSection then currentSection.Visible = false end
        AimbotSection.Visible = true
        currentSection = AimbotSection
    end
end)

EspsLabel.MouseButton1Down:Connect(function()
    setGuiFocus(true)
end)
EspsLabel.MouseButton1Up:Connect(function()
    setGuiFocus(false)
end)
EspsLabel.MouseButton1Click:Connect(function()
    if currentSection == EspSection then
        EspSection.Visible = false
        currentSection = nil
    else
        if currentSection then currentSection.Visible = false end
        EspSection.Visible = true
        currentSection = EspSection
    end
end)

CheckAllLabel.MouseButton1Down:Connect(function()
    setGuiFocus(true)
end)
CheckAllLabel.MouseButton1Up:Connect(function()
    setGuiFocus(false)
end)
CheckAllLabel.MouseButton1Click:Connect(function()
    if currentSection == CheckSection then
        CheckSection.Visible = false
        currentSection = nil
    else
        if currentSection then currentSection.Visible = false end
        CheckSection.Visible = true
        currentSection = CheckSection
    end
end)

-- LOOP PRINCIPAL
local function startMainLoop()
    if MAIN_LOOP_CONNECTION then
        MAIN_LOOP_CONNECTION:Disconnect()
    end
    
    MAIN_LOOP_CONNECTION = RunService.Heartbeat:Connect(function()
        if not GUI_IS_FOCUSED then
            if RAINBOW_ESP_ENABLED then
                ESP_COLOR = Color3.fromHSV(tick() % 5 / 5, 1, 1)
                for playerName, data in pairs(espData) do
                    if data.highlight then
                        data.highlight.FillColor = ESP_COLOR
                        data.highlight.OutlineColor = ESP_COLOR
                    end
                    if data.boxGui and data.boxGui:FindFirstChild("Frame") then
                        local stroke = data.boxGui.Frame:FindFirstChild("UIStroke")
                        if stroke then stroke.Color = ESP_COLOR end
                    end
                end
            end
            
            if RAINBOW_CIRCLE_ENABLED then
                CircleStroke.Color = Color3.fromHSV(tick() % 5 / 5, 1, 1)
            end
            
            if CIRCLE_ENABLED then
                local target = findTargetInFOV()
                if target and target.Character then
                    local head = target.Character:FindFirstChild("Head")
                    if head then
                        local targetCFrame = CFrame.lookAt(Camera.CFrame.Position, head.Position)
                        if AIM_VELOCITY >= 6.9 then
                            Camera.CFrame = targetCFrame
                        else
                            Camera.CFrame = Camera.CFrame:lerp(targetCFrame, AIM_VELOCITY * 0.1)
                        end
                    end
                end
            end
        end
    end)
end

-- ANIMAÇÃO GIRATÓRIA DO TOGGLE
TOGGLE_SPIN_CONNECTION = RunService.Heartbeat:Connect(function()
    if ToggleButton.Visible then
        ToggleGradient.Rotation = (ToggleGradient.Rotation + 3) % 360
    end
end)

startMainLoop()

-- PLAYER TRACKING
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        wait(0.5)
        createFullESP(player)
    end)
end)

for _, player in pairs(Players:GetPlayers()) do
    if player.Character then createFullESP(player) end
    player.CharacterAdded:Connect(function()
        wait(0.5)
        createFullESP(player)
    end)
end

Players.PlayerRemoving:Connect(function(player)
    if espData[player.Name] then
        if espData[player.Name].highlight then espData[player.Name].highlight:Destroy() end
        if espData[player.Name].boxGui then espData[player.Name].boxGui:Destroy() end
        espData[player.Name] = nil
    end
end)

print("✅ JZINN INTERFACE V2 - COMPLETO MOBILE + ZINDEX MÁXIMO!")
print("✅ Toggle: Fonte 11px, Borda AZUL GIRATÓRIA, ACIMA DE TUDO!")
