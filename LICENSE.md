-- LocalScript (StarterPlayerScripts)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

local aimRadius = 100
local targetPartName = "Head"
local isEnabled = false
local guiVisible = true

-- Aguarda PlayerGui
local playerGui = player:WaitForChild("PlayerGui")

-- Criar GUI principal
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimAssistGui"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Enabled = true
screenGui.Parent = playerGui

-- Criar container (fundo) com estilo moderno
local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 250, 0, 120)
frame.Position = UDim2.new(0, 50, 0, 50)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

-- Cantos arredondados
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = frame

-- Sombra suave
local shadow = Instance.new("ImageLabel")
shadow.Size = frame.Size + UDim2.new(0, 20, 0, 20)
shadow.Position = frame.Position - UDim2.new(0, 10, 0, 10)
shadow.BackgroundTransparency = 1
shadow.Image = "rbxassetid://1316045217"
shadow.ImageTransparency = 0.5
shadow.ScaleType = Enum.ScaleType.Slice
shadow.SliceCenter = Rect.new(10, 10, 118, 118)
shadow.Parent = screenGui
frame:GetPropertyChangedSignal("Position"):Connect(function()
	shadow.Position = frame.Position - UDim2.new(0, 10, 0, 10)
end)

-- Título
local title = Instance.new("TextLabel")
title.Text = "Aim Assist"
title.Font = Enum.Font.GothamBold
title.TextSize = 22
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.BackgroundTransparency = 1
title.Size = UDim2.new(1, 0, 0, 35)
title.Parent = frame

-- Botão estilizado
local button = Instance.new("TextButton")
button.Size = UDim2.new(0.8, 0, 0, 40)
button.Position = UDim2.new(0.1, 0, 0.55, 0)
button.Text = "Ativar Aim Assist"
button.Font = Enum.Font.Gotham
button.TextSize = 18
button.TextColor3 = Color3.new(1, 1, 1)
button.BackgroundColor3 = Color3.fromRGB(60, 150, 255)
button.Parent = frame

local buttonCorner = Instance.new("UICorner")
buttonCorner.CornerRadius = UDim.new(0, 8)
buttonCorner.Parent = button

button.MouseButton1Click:Connect(function()
	isEnabled = not isEnabled
	button.Text = isEnabled and "Desativar Aim Assist" or "Ativar Aim Assist"
	button.BackgroundColor3 = isEnabled and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(60, 150, 255)
end)

-- Tecla G alterna a interface
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.G then
		guiVisible = not guiVisible
		screenGui.Enabled = guiVisible
	end
end)

-- ESP (nome flutuante)
local function createESP(target)
	if not target:FindFirstChild("Head") then return end
	if target:FindFirstChild("ESP") then return end

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ESP"
	billboard.Size = UDim2.new(0, 100, 0, 30)
	billboard.Adornee = target.Head
	billboard.AlwaysOnTop = true
	billboard.StudsOffset = Vector3.new(0, 2, 0)
	billboard.Parent = target

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.Text = target.Name
	label.TextColor3 = Color3.fromRGB(255, 0, 0)
	label.TextStrokeTransparency = 0.5
	label.TextScaled = true
	label.Font = Enum.Font.GothamBold
	label.Parent = billboard
end

-- Atualizar ESP
task.spawn(function()
	while true do
		for _, npc in pairs(workspace:GetChildren()) do
			if npc:IsA("Model") and npc:FindFirstChild("Head") then
				createESP(npc)
			end
		end
		task.wait(2)
	end
end)

-- Função de alvo
local function getClosestTarget()
	local closestTarget = nil
	local closestDistance = aimRadius

	for _, npc in pairs(workspace:GetChildren()) do
		if npc:FindFirstChild(targetPartName) and not npc:IsA("Tool") then
			local part = npc[targetPartName]
			local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
			if onScreen then
				local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mouse.X, mouse.Y)).Magnitude
				if dist < closestDistance then
					closestDistance = dist
					closestTarget = part
				end
			end
		end
	end

	return closestTarget
end

-- Aim assist
RunService.RenderStepped:Connect(function()
	if not isEnabled then return end

	local target = getClosestTarget()
	if target then
		local direction = (target.Position - camera.CFrame.Position).Unit
		camera.CFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + direction)
	end
end)
