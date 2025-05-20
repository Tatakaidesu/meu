-- LocalScript (StarterPlayerScripts)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = workspace.CurrentCamera

-- CONFIGURAÇÃO
local AIM_RADIUS = 120
local TARGET_PART = "HumanoidRootPart" -- ou "Head" se preferir

-- ESTADOS
local aimAssistEnabled = false
local espEnabled = false

-- GUI
local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.Name = "AssistDebugUI"
screenGui.ResetOnSpawn = false

-- Painel principal
local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 250, 0, 150)
frame.Position = UDim2.new(0, 50, 0, 50)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
frame.Active = true
frame.Draggable = true
local corner = Instance.new("UICorner", frame)
corner.CornerRadius = UDim.new(0, 12)

-- Título
local title = Instance.new("TextLabel", frame)
title.Text = "Sistema de Assistência"
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundTransparency = 1
title.TextColor3 = Color3.new(1, 1, 1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

-- Botão Aim Assist
local aimButton = Instance.new("TextButton", frame)
aimButton.Size = UDim2.new(0.8, 0, 0, 30)
aimButton.Position = UDim2.new(0.1, 0, 0.35, 0)
aimButton.Text = "Ativar Aim Assist"
aimButton.BackgroundColor3 = Color3.fromRGB(50, 120, 255)
aimButton.TextColor3 = Color3.new(1, 1, 1)
aimButton.Font = Enum.Font.Gotham
aimButton.TextSize = 16
Instance.new("UICorner", aimButton)

-- Botão ESP
local espButton = Instance.new("TextButton", frame)
espButton.Size = UDim2.new(0.8, 0, 0, 30)
espButton.Position = UDim2.new(0.1, 0, 0.65, 0)
espButton.Text = "Ativar ESP"
espButton.BackgroundColor3 = Color3.fromRGB(100, 200, 100)
espButton.TextColor3 = Color3.new(1, 1, 1)
espButton.Font = Enum.Font.Gotham
espButton.TextSize = 16
Instance.new("UICorner", espButton)

-- Função: Alvo mais próximo
local function getClosestTarget()
	local closest = nil
	local shortestDist = AIM_RADIUS

	for _, model in pairs(workspace:GetChildren()) do
		if model:IsA("Model") and model ~= LocalPlayer.Character and model:FindFirstChild(TARGET_PART) then
			local part = model[TARGET_PART]
			local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
			if onScreen then
				local dist = (Vector2.new(Mouse.X, Mouse.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
				if dist < shortestDist then
					shortestDist = dist
					closest = part
				end
			end
		end
	end

	return closest
end

-- Mira assistida suave
RunService.RenderStepped:Connect(function()
	if aimAssistEnabled then
		local targetPart = getClosestTarget()
		if targetPart and Camera then
			local camPos = Camera.CFrame.Position
			local dir = (targetPart.Position - camPos).Unit
			local newLook = camPos + dir
			Camera.CFrame = Camera.CFrame:Lerp(CFrame.new(camPos, newLook), 0.15)
		end
	end
end)

-- ESP (marcadores flutuantes)
local function applyESP(model)
	if model:FindFirstChild("ESP") or not model:FindFirstChild(TARGET_PART) then return end

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ESP"
	billboard.Size = UDim2.new(0, 100, 0, 20)
	billboard.Adornee = model[TARGET_PART]
	billboard.AlwaysOnTop = true
	billboard.StudsOffset = Vector3.new(0, 2, 0)

	local label = Instance.new("TextLabel", billboard)
	label.Size = UDim2.new(1, 0, 1, 0)
	label.BackgroundTransparency = 1
	label.Text = model.Name
	label.TextColor3 = Color3.new(1, 0, 0)
	label.Font = Enum.Font.GothamBold
	label.TextScaled = true

	billboard.Parent = model
end

-- Atualiza ESP a cada 2s
task.spawn(function()
	while true do
		if espEnabled then
			for _, model in pairs(workspace:GetChildren()) do
				if model:IsA("Model") and model:FindFirstChild(TARGET_PART) and not model:FindFirstChild("ESP") then
					applyESP(model)
				end
			end
		else
			for _, model in pairs(workspace:GetChildren()) do
				if model:FindFirstChild("ESP") then
					model:FindFirstChild("ESP"):Destroy()
				end
			end
		end
		task.wait(2)
	end
end)

-- Botões
aimButton.MouseButton1Click:Connect(function()
	aimAssistEnabled = not aimAssistEnabled
	aimButton.Text = aimAssistEnabled and "Desativar Aim Assist" or "Ativar Aim Assist"
	aimButton.BackgroundColor3 = aimAssistEnabled and Color3.fromRGB(200, 80, 80) or Color3.fromRGB(50, 120, 255)
end)

espButton.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	espButton.Text = espEnabled and "Desativar ESP" or "Ativar ESP"
	espButton.BackgroundColor3 = espEnabled and Color3.fromRGB(200, 80, 80) or Color3.fromRGB(100, 200, 100)
end)

-- Atalho G para mostrar/ocultar UI
UIS.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.KeyCode == Enum.KeyCode.G then
		screenGui.Enabled = not screenGui.Enabled
	end
end)
