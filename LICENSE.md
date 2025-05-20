-- LocalScript (coloque dentro de StarterPlayerScripts)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

local aimRadius = 100
local targetPartName = "Head"
local isEnabled = false
local guiVisible = true -- visibilidade da interface

-- Aguarda PlayerGui para garantir que está pronto
local playerGui = player:WaitForChild("PlayerGui")

-- Criar a interface (ScreenGui + Botão)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AimAssistGui"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true -- evita sobreposição com top bar
screenGui.Enabled = true
screenGui.Parent = playerGui

local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Size = UDim2.new(0, 180, 0, 50)
toggleButton.Position = UDim2.new(0, 20, 0, 20)
toggleButton.BackgroundColor3 = Color3.fromRGB(60, 150, 255)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 20
toggleButton.Text = "Ativar Aim Assist"
toggleButton.Parent = screenGui

-- Alternar Aim Assist via botão
toggleButton.MouseButton1Click:Connect(function()
	isEnabled = not isEnabled
	toggleButton.Text = isEnabled and "Desativar Aim Assist" or "Ativar Aim Assist"
	toggleButton.BackgroundColor3 = isEnabled and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(60, 150, 255)
end)

-- Tecla "G" mostra/esconde a interface
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.G then
		guiVisible = not guiVisible
		screenGui.Enabled = guiVisible
	end
end)

-- Mira no inimigo mais próximo da mira
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

-- Mira automática ativada
RunService.RenderStepped:Connect(function()
	if not isEnabled then return end

	local target = getClosestTarget()
	if target then
		local direction = (target.Position - camera.CFrame.Position).Unit
		camera.CFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + direction)
	end
end)
