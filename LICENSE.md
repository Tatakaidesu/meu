local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local maxDistance = 100 -- Distância máxima para detectar alvos
local aimbotActive = false -- Estado do aimbot

-- Função para encontrar o alvo mais próximo
local function findNearestTarget()
    local closestTarget = nil
    local closestDistance = maxDistance

    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local distance = (player.Character.HumanoidRootPart.Position - otherPlayer.Character.HumanoidRootPart.Position).Magnitude
            if distance < closestDistance then
                closestDistance = distance
                closestTarget = otherPlayer.Character.HumanoidRootPart
            end
        end
    end

    return closestTarget
end

-- Função para atualizar a mira
local function updateAimbot()
    if aimbotActive then
        local target = findNearestTarget()
        if target then
            -- Ajusta a câmera para mirar no alvo
            camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
            -- Ajusta a rotação do personagem (opcional)
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = CFrame.new(player.Character.HumanoidRootPart.Position, Vector3.new(target.Position.X, player.Character.HumanoidRootPart.Position.Y, target.Position.Z))
            end
        end
    end
end

-- Ativar/desativar o aimbot com o botão direito do mouse
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.UserInputType == Enum.UserInputType.MouseButton2 then
        aimbotActive = not aimbotActive
        print(aimbotActive and "Aimbot ativado!" or "Aimbot desativado!")
    end
end)

-- Atualizar a cada frame
RunService.RenderStepped:Connect(updateAimbot)
