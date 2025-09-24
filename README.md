local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")
local RunService = game:GetService("RunService")
local workspace = game:GetService("Workspace")
local Players = game:GetService("Players")

-- =====================
-- ScreenGui 作成
-- =====================
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "CyberFootESPUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

-- メインフレーム（半透明ネオン風）
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 80)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 40)
mainFrame.BackgroundTransparency = 0.2
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

-- UI光の縁（Neonフレーム的）
local uicorner = Instance.new("UICorner")
uicorner.CornerRadius = UDim.new(0,12)
uicorner.Parent = mainFrame

local frameGradient = Instance.new("UIGradient")
frameGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0,255,255)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255,0,255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0,255,255))
})
frameGradient.Rotation = 45
frameGradient.Parent = mainFrame

-- ボタンの共通作成関数
local function createButton(name, text, pos, color)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 120, 0, 40)
    btn.Position = pos
    btn.Text = text
    btn.BackgroundColor3 = color
    btn.TextColor3 = Color3.fromRGB(0,0,0)
    btn.Font = Enum.Font.GothamBlack
    btn.TextScaled = true
    btn.AutoButtonColor = false
    btn.Parent = mainFrame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,10)
    corner.Parent = btn

    local gradient = Instance.new("UIGradient")
    gradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255,0,255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(0,255,255))
    })
    gradient.Rotation = 45
    gradient.Parent = btn

    return btn
end

-- 足場ボタン
local createPartButton = createButton("FootButton", "Hold: Rainbow Foot", UDim2.new(0,10,0,20), Color3.fromRGB(0,255,0))

-- =====================
-- 足場生成処理
-- =====================
local creating = false
local footParts = {}

local function createFootPart()
    local footPos = hrp.Position - Vector3.new(0, hrp.Size.Y/2 + 1, 0)
    local part = Instance.new("Part")
    part.Size = Vector3.new(4,1,4)
    part.Position = footPos
    part.Anchored = true
    part.CanCollide = true
    part.Material = Enum.Material.Neon
    part.Parent = workspace

    table.insert(footParts, part)

    delay(3, function()
        for i,p in pairs(footParts) do
            if p==part then table.remove(footParts,i) break end
        end
        if part then part:Destroy() end
    end)
end

createPartButton.MouseButton1Down:Connect(function()
    creating = true
    while creating do
        createFootPart()
        wait(0.1)
    end
end)

createPartButton.MouseButton1Up:Connect(function()
    creating = false
end)

-- 虹色発光
RunService.RenderStepped:Connect(function()
    for _, part in pairs(footParts) do
        if part then
            part.Color = Color3.fromHSV(tick()%1,1,1)
        end
    end
end)

-- =====================
-- ESP機能
-- =====================
local espFolder = Instance.new("Folder")
espFolder.Name = "ESP_Folder"
espFolder.Parent = workspace

local function createESP(playerTarget)
    if playerTarget == player then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP"
    billboard.Size = UDim2.new(0,100,0,50)
    billboard.AlwaysOnTop = true
    billboard.Adornee = playerTarget.Character and playerTarget.Character:FindFirstChild("HumanoidRootPart")
    billboard.Parent = espFolder

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1,0,1,0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255,0,0)
    label.TextStrokeTransparency = 0
    label.TextScaled = true
    label.Font = Enum.Font.GothamBold
    label.Text = playerTarget.Name
    label.Parent = billboard

    playerTarget.CharacterRemoving:Connect(function()
        billboard:Destroy()
    end)
end

for _,p in pairs(Players:GetPlayers()) do
    if p.Character then createESP(p) end
end

Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function() createESP(p) end)
end)
