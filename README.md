--// DADA Client UI
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")

-- Remove old UI
if CoreGui:FindFirstChild("DADA_Client") then
    CoreGui:FindFirstChild("DADA_Client"):Destroy()
end

-- Variables
local savedPoints = {}
local currentTab = "Main"

-- Create UI
local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "DADA_Client"
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 450, 0, 350)
MainFrame.Position = UDim2.new(0.5, -225, 0.5, -175)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.Active = true
MainFrame.Draggable = false
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 12)

local TopBar = Instance.new("Frame", MainFrame)
TopBar.Size = UDim2.new(1, 0, 0, 40)
TopBar.BackgroundTransparency = 1

-- Drag logic
local dragging, dragInput, dragStart, startPos
TopBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)
TopBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)
UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
                                       startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Collapsing
local CollapseButton = Instance.new("TextButton", MainFrame)
CollapseButton.Text = "-"
CollapseButton.Font = Enum.Font.GothamBold
CollapseButton.TextSize = 20
CollapseButton.TextColor3 = Color3.fromRGB(255, 255, 120)
CollapseButton.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
CollapseButton.Size = UDim2.new(0, 30, 0, 30)
CollapseButton.Position = UDim2.new(1, -40, 0, 5)
Instance.new("UICorner", CollapseButton).CornerRadius = UDim.new(0, 6)

local Collapsed = false
CollapseButton.MouseButton1Click:Connect(function()
    Collapsed = not Collapsed
    for _, child in pairs(MainFrame:GetChildren()) do
        if child ~= TopBar and child ~= CollapseButton then
            child.Visible = not Collapsed
        end
    end
    CollapseButton.Text = Collapsed and "+" or "-"
end)

-- Tabs
local Tabs = {"Main", "ESP", "Combat", "TP"}
local TabButtons = {}
for i, tabName in pairs(Tabs) do
    local btn = Instance.new("TextButton", MainFrame)
    btn.Size = UDim2.new(0, 100, 0, 30)
    btn.Position = UDim2.new(0, 10 + (i - 1) * 110, 0, 45)
    btn.Text = tabName
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.TextColor3 = Color3.fromRGB(255, 255, 120)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

    TabButtons[tabName] = btn
end

-- Tab Frames
local TabFrames = {}
for _, name in pairs(Tabs) do
    local frame = Instance.new("Frame", MainFrame)
    frame.Name = name .. "_Tab"
    frame.Size = UDim2.new(1, -20, 1, -90)
    frame.Position = UDim2.new(0, 10, 0, 80)
    frame.BackgroundTransparency = 1
    frame.Visible = (name == "Main")
    TabFrames[name] = frame
end

-- Tab switching
for name, btn in pairs(TabButtons) do
    btn.MouseButton1Click:Connect(function()
        for _, f in pairs(TabFrames) do f.Visible = false end
        TabFrames[name].Visible = true
    end)
end

-- MAIN TAB CONTENT
local Header = Instance.new("TextLabel", TabFrames["Main"])
Header.Text = "DADA Inject"
Header.Size = UDim2.new(1, 0, 0, 30)
Header.Font = Enum.Font.GothamBold
Header.TextSize = 20
Header.TextColor3 = Color3.fromRGB(255, 255, 120)
Header.BackgroundTransparency = 1

local SubHeader = Instance.new("TextLabel", TabFrames["Main"])
SubHeader.Text = "Game detect: " .. game.Name
SubHeader.Size = UDim2.new(1, -20, 0, 20)
SubHeader.Position = UDim2.new(0, 10, 0, 35)
SubHeader.Font = Enum.Font.Gotham
SubHeader.TextSize = 14
SubHeader.TextColor3 = Color3.fromRGB(200, 200, 200)
SubHeader.BackgroundTransparency = 1

local BetaTag = Instance.new("TextLabel", TabFrames["Main"])
BetaTag.Text = "DADA Client Beta — могут быть ошибки"
BetaTag.Size = UDim2.new(1, -20, 0, 20)
BetaTag.Position = UDim2.new(0, 10, 0, 60)
BetaTag.Font = Enum.Font.Gotham
BetaTag.TextSize = 12
BetaTag.TextColor3 = Color3.fromRGB(255, 255, 120)
BetaTag.BackgroundTransparency = 1

local AntiAFK = Instance.new("TextButton", TabFrames["Main"])
AntiAFK.Text = "Anti-AFK"
AntiAFK.Position = UDim2.new(0, 10, 0, 90)
AntiAFK.Size = UDim2.new(0, 120, 0, 30)
AntiAFK.Font = Enum.Font.GothamBold
AntiAFK.TextSize = 14
AntiAFK.TextColor3 = Color3.fromRGB(0, 0, 0)
AntiAFK.BackgroundColor3 = Color3.fromRGB(255, 255, 120)
Instance.new("UICorner", AntiAFK).CornerRadius = UDim.new(0, 6)
AntiAFK.MouseButton1Click:Connect(function()
    local conn = LocalPlayer.Idled:Connect(function()
        game:GetService("VirtualUser"):Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
        task.wait(1)
        game:GetService("VirtualUser"):Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
    end)
    AntiAFK.Text = "Anti-AFK: ON"
    AntiAFK.AutoButtonColor = false
    AntiAFK.BackgroundColor3 = Color3.fromRGB(0, 255, 120)
end)

-- COMBAT
local HitboxBtn = Instance.new("TextButton", TabFrames["Combat"])
HitboxBtn.Text = "Hitbox Legit"
HitboxBtn.Size = UDim2.new(0, 150, 0, 30)
HitboxBtn.Position = UDim2.new(0, 10, 0, 10)
HitboxBtn.Font = Enum.Font.GothamBold
HitboxBtn.TextSize = 14
HitboxBtn.TextColor3 = Color3.fromRGB(255, 255, 120)
HitboxBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Instance.new("UICorner", HitboxBtn).CornerRadius = UDim.new(0, 6)
HitboxBtn.MouseButton1Click:Connect(function()
    loadstring(game:HttpGet("https://pastefy.app/ItfO0tdg/raw"))()
end)

-- ESP заглушка
local ESPLabel = Instance.new("TextLabel", TabFrames["ESP"])
ESPLabel.Text = "Здесь будет улучшенный ESP"
ESPLabel.Font = Enum.Font.GothamBold
ESPLabel.TextSize = 14
ESPLabel.TextColor3 = Color3.fromRGB(255, 255, 120)
ESPLabel.BackgroundTransparency = 1
ESPLabel.Size = UDim2.new(1, 0, 0, 30)
ESPLabel.Position = UDim2.new(0, 10, 0, 10)

-- TP СИСТЕМА
local SaveBtn = Instance.new("TextButton", TabFrames["TP"])
SaveBtn.Text = "Сохранить точку"
SaveBtn.Size = UDim2.new(0, 150, 0, 30)
SaveBtn.Position = UDim2.new(0, 10, 0, 10)
SaveBtn.Font = Enum.Font.GothamBold
SaveBtn.TextSize = 14
SaveBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
SaveBtn.BackgroundColor3 = Color3.fromRGB(255, 255, 120)
Instance.new("UICorner", SaveBtn).CornerRadius = UDim.new(0, 6)

local TPScroll = Instance.new("ScrollingFrame", TabFrames["TP"])
TPScroll.Position = UDim2.new(0, 10, 0, 50)
TPScroll.Size = UDim2.new(1, -20, 1, -60)
TPScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
TPScroll.ScrollBarThickness = 6
TPScroll.BackgroundTransparency = 1

local function refreshTPList()
    TPScroll:ClearAllChildren()
    for i, pos in pairs(savedPoints) do
        local btn = Instance.new("TextButton", TPScroll)
        btn.Text = "Точка " .. i
        btn.Size = UDim2.new(1, -10, 0, 30)
        btn.Position = UDim2.new(0, 5, 0, (i - 1) * 35)
        btn.Font = Enum.Font.Gotham
        btn.TextSize = 14
        btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
        btn.TextColor3 = Color3.fromRGB(255, 255, 120)
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
        btn.MouseButton1Click:Connect(function()
            LocalPlayer.Character:SetPrimaryPartCFrame(CFrame.new(pos))
        end)
    end
    TPScroll.CanvasSize = UDim2.new(0, 0, 0, #savedPoints * 35)
end

SaveBtn.MouseButton1Click:Connect(function()
    if LocalPlayer.Character and LocalPlayer.Character.PrimaryPart then
        table.insert(savedPoints, LocalPlayer.Character.PrimaryPart.Position)
        refreshTPList()
    end
end)
# DADAInjectGame
Script roblox
