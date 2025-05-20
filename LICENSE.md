local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local maxDistance = 100 -- Distância máxima para detectar alvos
local aimbotActive = false -- Estado do aimbot

-- Função para aguardar o personagem carregar
local function waitForCharacter()
    print("Aguardando personagem carregar...")
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        player.CharacterAdded:Wait()
    end
    print("Personagem carregado!")
    return player.Character
end

-- Função para encontrar o alvo mais próximo
local function findNearestTarget()
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        print("Erro: Personagem ou HumanoidRootPart não encontrado!")
        return nil
    end

    local closestTarget = nil
    local closestDistance = maxDistance

    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local targetHumanoid = otherPlayer.Character:FindFirstChild("Humanoid")
            if targetHumanoid and targetHumanoid.Health > 0 then
                local distance = (character.HumanoidRootPart.Position - otherPlayer.Character.HumanoidRootPart.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestTarget = otherPlayer.Character.HumanoidRootPart
                    print("Alvo encontrado: " .. otherPlayer.Name .. " a " .. math.floor(distance) .. " studs")
                end
            end
        end
    end

    if not closestTarget then
        print("Nenhum alvo válido encontrado dentro de " .. maxDistance .. " studs")
    end
    return closestTarget
end

-- Função para atualizar a mira
local function updateAimbot()
    if aimbotActive then
        local target = findNearestTarget()
        if target then
            print("Mirando em: " .. tostring(target.Parent.Name))
            camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = CFrame.new(player.Character.HumanoidRootPart.Position, Vector3.new(target.Position.X, player.Character.HumanoidRootPart.Position.Y, target.Position.Z))
            end
        end
    end
end

-- Detectar clique do botão direito do mouse
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    print("Input detectado: " .. tostring(input.UserInputType) .. ", gameProcessed: " .. tostring(gameProcessed))
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        aimbotActive = not aimbotActive
        print(aimbotActive and "Aimbot ativado!" or "Aimbot desativado!")
    end
end)

-- Iniciar o script após o personagem carregar
waitForCharacter()

-- Atualizar a cada frame
RunService.RenderStepped:Connect(updateAimbot)
print("Aimbot script iniciado!")
