-- [[ DEOBFUSCATED BY @dark hub]] --

-- // [1] SERVICES //
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local NetworkClient = game:GetService("NetworkClient")
local HttpService = game:GetService("HttpService")

-- // [2] CONFIGURATION //
-- [[ CONFIG - EDIT EVERYTHING HERE ]] --
local CONFIG = {
    Names = {
        ScreenGui = "darkLagger",
        MainFrame = "MainFrame",
        FloatingButton = "FloatingLaggerButton",
        ConfigFile = "darkLagger_Config.json"
    },
    
    Size = {
        MainFrame = UDim2.new(0, 217, 0, 195),
        FloatingButton = UDim2.new(0, 69, 0, 69),
        ConfigButton = UDim2.new(0.31, 0, 1, 0)
    },
    
    Position = {
        MainFrame = UDim2.new(0.5, -108, 0.5, -97),
        FloatingButton = UDim2.new(1, -80, 0.4, 0)
    },
    
    Colors = {
        Background = Color3.fromRGB(0, 0, 0),
        Stroke = Color3.fromRGB(100, 200, 255),
        TextInfo = Color3.fromRGB(180, 210, 255),
        TextLaggerOff = Color3.fromRGB(220, 220, 220),
        TextLaggerOn = Color3.fromRGB(80, 255, 80),
        ButtonLaggerOff = Color3.fromRGB(180, 40, 40),
        ButtonLaggerOn = Color3.fromRGB(40, 180, 40),
        ButtonSave = Color3.fromRGB(20, 100, 60),
        ButtonLoad = Color3.fromRGB(20, 80, 150),
        ButtonDelete = Color3.fromRGB(120, 30, 30),
        DarkContainer = Color3.fromRGB(15, 15, 15)
    },

    Lagger = {
        Keybind = Enum.KeyCode.G,
        Power = 265,
        WaitTime = 0.95
    }
}

-- // [3] OBJECT REFERENCES //
local UI = {}
local LaggerState = false
local LaggerCoroutine = nil
local IsBinding = false
local RobloxReplicatedStorage = game:FindFirstChild("RobloxReplicatedStorage")

-- // [4] UTILITY FUNCTIONS //

-- Smooth Dragging Logic
local function makeDraggable(gui)
    local dragging, dragInput, dragStart, startPos

    local function update(input)
        local delta = input.Position - dragStart
        gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = input.Position
            startPos = gui.Position
            
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    gui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
end

-- Core Lagger Logic
local function toggleLagger()
    LaggerState = not LaggerState
    
    -- Animate Main Switch
    local knobGoal = LaggerState and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
    TweenService:Create(UI.SwitchKnob, TweenInfo.new(0.3), {Position = knobGoal}):Play()
    
    if LaggerState then
        UI.StatusLabel.Text = "LAGGER: ON"
        UI.StatusLabel.TextColor3 = CONFIG.Colors.TextLaggerOn
        UI.FloatingButton.Text = "Toggle Lagger:ON"
        UI.FloatingButton.BackgroundColor3 = CONFIG.Colors.ButtonLaggerOn
        UI.FloatingStroke.Color = CONFIG.Colors.ButtonLaggerOn
        
        if not LaggerCoroutine then
            LaggerCoroutine = task.spawn(function()
                pcall(function() NetworkClient:SetOutgoingKBPSLimit(CONFIG.Lagger.Power) end)
                while LaggerState do
                    if RobloxReplicatedStorage and RobloxReplicatedStorage:FindFirstChild("SetPlayerBlockList") then
                        pcall(function()
                            RobloxReplicatedStorage.SetPlayerBlockList:FireServer()
                        end)
                    end
                    task.wait(CONFIG.Lagger.WaitTime)
                end
            end)
        end
    else
        UI.StatusLabel.Text = "LAGGER: OFF"
        UI.StatusLabel.TextColor3 = CONFIG.Colors.TextLaggerOff
        UI.FloatingButton.Text = "Toggle Lagger:OFF"
        UI.FloatingButton.BackgroundColor3 = CONFIG.Colors.ButtonLaggerOff
        UI.FloatingStroke.Color = CONFIG.Colors.ButtonLaggerOff
        
        pcall(function() NetworkClient:SetOutgoingKBPSLimit(0) end)
        
        if LaggerCoroutine then
            task.cancel(LaggerCoroutine)
            LaggerCoroutine = nil
        end
    end
end

-- Configuration System
local function saveConfig()
    if writefile then
        local data = {
            Keybind = CONFIG.Lagger.Keybind.Name,
            Power = CONFIG.Lagger.Power
        }
        writefile(CONFIG.Names.ConfigFile, HttpService:JSONEncode(data))
    end
end

local function loadConfig()
    if isfile and isfile(CONFIG.Names.ConfigFile) and readfile then
        local success, data = pcall(function()
            return HttpService:JSONDecode(readfile(CONFIG.Names.ConfigFile))
        end)
        
        if success and data then
            if data.Keybind and Enum.KeyCode[data.Keybind] then
                CONFIG.Lagger.Keybind = Enum.KeyCode[data.Keybind]
                UI.KeybindDisplay.Text = data.Keybind
            end
            if data.Power then
                CONFIG.Lagger.Power = data.Power
            end
        end
    end
end

local function deleteConfig()
    if isfile and isfile(CONFIG.Names.ConfigFile) and delfile then
        delfile(CONFIG.Names.ConfigFile)
        CONFIG.Lagger.Keybind = Enum.KeyCode.G
        UI.KeybindDisplay.Text = "G"
    end
end

-- // [5] GUI CREATION //
local function createGUI()
    if CoreGui:FindFirstChild(CONFIG.Names.ScreenGui) then
        CoreGui[CONFIG.Names.ScreenGui]:Destroy()
    end

    UI.ScreenGui = Instance.new("ScreenGui")
    UI.ScreenGui.Name = CONFIG.Names.ScreenGui
    UI.ScreenGui.ResetOnSpawn = false
    UI.ScreenGui.Parent = CoreGui

    UI.MainFrame = Instance.new("Frame")
    UI.MainFrame.Name = CONFIG.Names.MainFrame
    UI.MainFrame.Size = CONFIG.Size.MainFrame
    UI.MainFrame.Position = CONFIG.Position.MainFrame
    UI.MainFrame.BackgroundColor3 = CONFIG.Colors.Background
    UI.MainFrame.BackgroundTransparency = 0.3
    UI.MainFrame.Active = true
    UI.MainFrame.Parent = UI.ScreenGui
    makeDraggable(UI.MainFrame)

    local MainCorner = Instance.new("UICorner")
    MainCorner.CornerRadius = UDim.new(0, 12)
    MainCorner.Parent = UI.MainFrame

    local MainStroke = Instance.new("UIStroke")
    MainStroke.Color = CONFIG.Colors.Stroke
    MainStroke.Thickness = 1.875
    MainStroke.Transparency = 0.1
    MainStroke.Parent = UI.MainFrame

    UI.Title = Instance.new("TextLabel")
    UI.Title.Size = UDim2.new(1, 0, 0, 26)
    UI.Title.Position = UDim2.new(0, 0, 0, 10)
    UI.Title.BackgroundTransparency = 1
    UI.Title.Text = "Pepsi Lagger"
    UI.Title.TextColor3 = CONFIG.Colors.Stroke
    UI.Title.Font = Enum.Font.GothamBold
    UI.Title.TextSize = 18
    UI.Title.Parent = UI.MainFrame

    UI.ConfigContainer = Instance.new("Frame")
    UI.ConfigContainer.Size = UDim2.new(0.9, 0, 0, 26)
    UI.ConfigContainer.Position = UDim2.new(0.05, 0, 0, 45)
    UI.ConfigContainer.BackgroundTransparency = 1
    UI.ConfigContainer.Parent = UI.MainFrame

    UI.ConfigButtons = {}
    local btnData = {
        {Name = "Save Config", Color = CONFIG.Colors.ButtonSave, Pos = UDim2.new(0, 0, 0, 0), Func = saveConfig},
        {Name = "Load Config", Color = CONFIG.Colors.ButtonLoad, Pos = UDim2.new(0.34, 0, 0, 0), Func = loadConfig},
        {Name = "Delete Config", Color = CONFIG.Colors.ButtonDelete, Pos = UDim2.new(0.68, 0, 0, 0), Func = deleteConfig}
    }
    
    for i, data in ipairs(btnData) do
        local btn = Instance.new("TextButton")
        btn.Size = CONFIG.Size.ConfigButton
        btn.Position = data.Pos
        btn.BackgroundColor3 = data.Color
        btn.TextColor3 = Color3.new(1, 1, 1)
        btn.Text = data.Name
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 9.75
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = btn
        
        btn.Parent = UI.ConfigContainer
        UI.ConfigButtons[data.Name] = btn
    end

    UI.DiscordLabel = Instance.new("TextButton")
    UI.DiscordLabel.Size = UDim2.new(1, 0, 0, 15)
    UI.DiscordLabel.Position = UDim2.new(0, 0, 0, 75)
    UI.DiscordLabel.BackgroundTransparency = 1
    UI.DiscordLabel.Text = "discord.gg/pepsihub"
    UI.DiscordLabel.TextColor3 = CONFIG.Colors.TextInfo
    UI.DiscordLabel.Font = Enum.Font.Gotham
    UI.DiscordLabel.TextSize = 10.5
    UI.DiscordLabel.Parent = UI.MainFrame

    UI.StatusContainer = Instance.new("Frame")
    UI.StatusContainer.Size = UDim2.new(0.9, 0, 0, 31)
    UI.StatusContainer.Position = UDim2.new(0.05, 0, 0, 95)
    UI.StatusContainer.BackgroundColor3 = CONFIG.Colors.DarkContainer
    UI.StatusContainer.BackgroundTransparency = 0.5
    UI.StatusContainer.Parent = UI.MainFrame
    
    local StatusCorner = Instance.new("UICorner")
    StatusCorner.CornerRadius = UDim.new(0, 8)
    StatusCorner.Parent = UI.StatusContainer

    UI.StatusLabel = Instance.new("TextLabel")
    UI.StatusLabel.Size = UDim2.new(0.5, 0, 1, 0)
    UI.StatusLabel.Position = UDim2.new(0, 11, 0, 0)
    UI.StatusLabel.BackgroundTransparency = 1
    UI.StatusLabel.Text = "LAGGER: OFF"
    UI.StatusLabel.TextColor3 = CONFIG.Colors.TextLaggerOff
    UI.StatusLabel.TextXAlignment = Enum.TextXAlignment.Left
    UI.StatusLabel.Font = Enum.Font.GothamBold
    UI.StatusLabel.TextSize = 11.25
    UI.StatusLabel.Parent = UI.StatusContainer

    UI.SwitchBtn = Instance.new("TextButton")
    UI.SwitchBtn.Size = UDim2.new(0, 37, 0, 19)
    UI.SwitchBtn.Position = UDim2.new(1, -48, 0.5, -9)
    UI.SwitchBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    UI.SwitchBtn.Text = ""
    UI.SwitchBtn.Parent = UI.StatusContainer
    
    local SwitchCorner = Instance.new("UICorner")
    SwitchCorner.CornerRadius = UDim.new(1, 0)
    SwitchCorner.Parent = UI.SwitchBtn

    UI.SwitchKnob = Instance.new("Frame")
    UI.SwitchKnob.Size = UDim2.new(0, 16, 0, 16)
    UI.SwitchKnob.Position = UDim2.new(0, 2, 0.5, -8)
    UI.SwitchKnob.BackgroundColor3 = CONFIG.Colors.Stroke
    UI.SwitchKnob.Parent = UI.SwitchBtn
    
    local KnobCorner = Instance.new("UICorner")
    KnobCorner.CornerRadius = UDim.new(1, 0)
    KnobCorner.Parent = UI.SwitchKnob

    UI.KeybindContainer = Instance.new("Frame")
    UI.KeybindContainer.Size = UDim2.new(0.9, 0, 0, 30)
    UI.KeybindContainer.Position = UDim2.new(0.05, 0, 0, 135)
    UI.KeybindContainer.BackgroundColor3 = CONFIG.Colors.DarkContainer
    UI.KeybindContainer.BackgroundTransparency = 0.5
    UI.KeybindContainer.Parent = UI.MainFrame
    
    local KeybindCorner = Instance.new("UICorner")
    KeybindCorner.CornerRadius = UDim.new(0, 8)
    KeybindCorner.Parent = UI.KeybindContainer

    UI.KeybindLabel = Instance.new("TextLabel")
    UI.KeybindLabel.Size = UDim2.new(0.5, 0, 1, 0)
    UI.KeybindLabel.Position = UDim2.new(0, 11, 0, 0)
    UI.KeybindLabel.BackgroundTransparency = 1
    UI.KeybindLabel.Text = "Keybind:"
    UI.KeybindLabel.TextColor3 = CONFIG.Colors.TextLaggerOff
    UI.KeybindLabel.TextXAlignment = Enum.TextXAlignment.Left
    UI.KeybindLabel.Font = Enum.Font.Gotham
    UI.KeybindLabel.TextSize = 10.5
    UI.KeybindLabel.Parent = UI.KeybindContainer

    UI.KeybindDisplay = Instance.new("TextButton")
    UI.KeybindDisplay.Size = UDim2.new(0, 63, 0, 21)
    UI.KeybindDisplay.Position = UDim2.new(1, -75, 0.5, -10)
    UI.KeybindDisplay.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
    UI.KeybindDisplay.TextColor3 = Color3.new(1, 1, 1)
    UI.KeybindDisplay.Text = CONFIG.Lagger.Keybind.Name
    UI.KeybindDisplay.Font = Enum.Font.GothamBold
    UI.KeybindDisplay.TextSize = 10
    UI.KeybindDisplay.Parent = UI.KeybindContainer
    
    local KeyDisplayCorner = Instance.new("UICorner")
    KeyDisplayCorner.CornerRadius = UDim.new(0, 4)
    KeyDisplayCorner.Parent = UI.KeybindDisplay

    UI.FloatingButton = Instance.new("TextButton")
    UI.FloatingButton.Name = CONFIG.Names.FloatingButton
    UI.FloatingButton.Size = CONFIG.Size.FloatingButton
    UI.FloatingButton.Position = CONFIG.Position.FloatingButton
    UI.FloatingButton.BackgroundColor3 = CONFIG.Colors.ButtonLaggerOff
    UI.FloatingButton.TextColor3 = Color3.new(1, 1, 1)
    UI.FloatingButton.Text = "Toggle Lagger:OFF"
    UI.FloatingButton.Font = Enum.Font.GothamBold
    UI.FloatingButton.TextSize = 10.875
    UI.FloatingButton.TextWrapped = true
    UI.FloatingButton.Active = true
    UI.FloatingButton.Parent = UI.ScreenGui
    makeDraggable(UI.FloatingButton)

    local FloatCorner = Instance.new("UICorner")
    FloatCorner.CornerRadius = UDim.new(0, 8)
    FloatCorner.Parent = UI.FloatingButton

    UI.FloatingStroke = Instance.new("UIStroke")
    UI.FloatingStroke.Color = CONFIG.Colors.ButtonLaggerOff
    UI.FloatingStroke.Thickness = 3.375
    UI.FloatingStroke.Parent = UI.FloatingButton
end

-- // [6] FUNCTIONALITY //
local function connectEvents()
    -- Click to toggle lagger
    UI.SwitchBtn.MouseButton1Click:Connect(toggleLagger)
    UI.FloatingButton.MouseButton1Click:Connect(toggleLagger)

    -- Config Buttons
    UI.ConfigButtons["Save Config"].MouseButton1Click:Connect(saveConfig)
    UI.ConfigButtons["Load Config"].MouseButton1Click:Connect(loadConfig)
    UI.ConfigButtons["Delete Config"].MouseButton1Click:Connect(deleteConfig)

    -- Keybind Changer Logic
    UI.KeybindDisplay.MouseButton1Click:Connect(function()
        IsBinding = true
        UI.KeybindDisplay.Text = "..."
    end)

    -- Global Input Processing
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        -- Set new keybind
        if IsBinding then
            if input.UserInputType == Enum.UserInputType.Keyboard then
                CONFIG.Lagger.Keybind = input.KeyCode
                UI.KeybindDisplay.Text = input.KeyCode.Name
                IsBinding = false
            end
            return
        end

        -- Toggle Lagger
        if not gameProcessed and input.KeyCode == CONFIG.Lagger.Keybind then
            toggleLagger()
        end
    end)
end

-- // [7] INITIALIZATION //
createGUI()
connectEvents()

print(string.format("Pepsi Lagger Loaded | Keybind: %s (Keyboard) | Size: 75%% | Power: %s | Wait: %s", 
    CONFIG.Lagger.Keybind.Name, 
    tostring(CONFIG.Lagger.Power), 
    tostring(CONFIG.Lagger.WaitTime)
))
