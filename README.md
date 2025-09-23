local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- キャラ取得
local function getCharacter()
	local char = player.Character or player.CharacterAdded:Wait()
	return char, char:WaitForChild("HumanoidRootPart")
end

-- UIドラッグ機能
local function makeDraggable(guiObject)
	local dragging = false
	local dragStart, startPos

	local function update(input)
		local delta = input.Position - dragStart
		guiObject.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end

	guiObject.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
			dragging = true
			dragStart = input.Position
			startPos = guiObject.Position

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	guiObject.InputChanged:Connect(function(input)
		if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
			update(input)
		end
	end)
end

-- UI作成
local function createUI()
	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "FloatUI"
	screenGui.ResetOnSpawn = false
	screenGui.Parent = player:WaitForChild("PlayerGui")

	local button = Instance.new("TextButton")
	button.Name = "FloatToggleButton"
	button.Size = UDim2.new(0, 160, 0, 60)
	button.Position = UDim2.new(0.5, -80, 1, -120)
	button.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
	button.TextColor3 = Color3.new(1, 1, 1)
	button.TextScaled = true
	button.Font = Enum.Font.GothamBold
	button.Text = "🚀 Float ON"
	button.Parent = screenGui

	-- 丸み
	local corner = Instance.new("UICorner", button)
	corner.CornerRadius = UDim.new(0, 16)

	-- 枠線
	local stroke = Instance.new("UIStroke", button)
	stroke.Color = Color3.fromRGB(255, 255, 255)
	stroke.Thickness = 2
	stroke.Transparency = 0.3

	-- ドラッグできるように
	makeDraggable(button)

	return button
end

-- フロート作成
local function createFROAT(char)
	local root = char:WaitForChild("HumanoidRootPart")

	local froat = Instance.new("Part")
	froat.Name = "FROAT"
	froat.Size = Vector3.new(2, 0.4, 2)
	froat.Position = root.Position - Vector3.new(0, 2.5, 0)
	froat.Anchored = false
	froat.CanCollide = false
	froat.BrickColor = BrickColor.new("Bright blue")
	froat.Material = Enum.Material.Neon
	froat.Parent = char

	local weld = Instance.new("WeldConstraint")
	weld.Part0 = froat
	weld.Part1 = root
	weld.Parent = froat

	return froat
end

-- 浮遊機能
local function setupFloat(button, char, root)
	local isFloating = false
	local bodyPos = nil
	local froat = nil

	local function startFloat()
		isFloating = true
		button.Text = "❌ Float OFF"
		button.BackgroundColor3 = Color3.fromRGB(255, 80, 80)

		bodyPos = Instance.new("BodyPosition")
		bodyPos.MaxForce = Vector3.new(0, math.huge, 0)
		bodyPos.Position = root.Position
		bodyPos.D = 1000
		bodyPos.P = 4000
		bodyPos.Parent = root

		froat = createFROAT(char)
	end

	local function stopFloat()
		isFloating = false
		button.Text = "🚀 Float ON"
		button.BackgroundColor3 = Color3.fromRGB(0, 200, 255)

		if bodyPos then bodyPos:Destroy() end
		if froat then froat:Destroy() end
	end

	button.MouseButton1Click:Connect(function()
		if isFloating then
			stopFloat()
		else
			startFloat()
		end
	end)

	-- ジャンプで上昇
	UserInputService.JumpRequest:Connect(function()
		if isFloating and bodyPos then
			bodyPos.Position += Vector3.new(0, 5, 0)
		end
	end)
end

-- 実行
local function main()
	local char, root = getCharacter()
	local button = createUI()
	setupFloat(button, char, root)

	player.CharacterAdded:Connect(function(newChar)
		local newRoot = newChar:WaitForChild("HumanoidRootPart")
		setupFloat(button, newChar, newRoot)
	end)
end

main()
