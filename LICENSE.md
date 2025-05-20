local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local maxDistance = 100 -- Distância máxima para detectar alvos
local aimbotActive = false -- Estado do aimbot

-- Função para aguardar o personagem carregar
local function waitForCharacter()
    print("[Aimbot] Aguardando personagem carregar...")
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        player.CharacterAdded:Wait()
    end
    print("[Aimbot] Personagem carregado!")
    return player.Character
end

-- Função para encontrar o alvo mais próximo
local function findNearestTarget()
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        print("[Aimbot] Erro: Personagem ou HumanoidRootPart não encontrado!")
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
                    print("[Aimbot] Alvo encontrado: " .. otherPlayer.Name .. " a " .. math.floor(distance) .. " studs")
                end
            end
        end
    end

    if not closestTarget then
        print("[Aimbot] Nenhum alvo válido encontrado dentro de " .. maxDistance .. " studs")
    end
    return closestTarget
end

-- Função para atualizar a mira
local function updateAimbot()
    if aimbotActive then
        local target = findNearestTarget()
        if target then
            print("[Aimbot] Mirando em: " .. tostring(target.Parent.Name))
            camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
        end
    end
end

-- Detectar clique do botão direito do mouse
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    print("[Aimbot] Input detectado: " .. tostring(input.UserInputType) .. ", gameProcessed: " .. tostring(gameProcessed))
    if input.UserInputType == Enum.UserInputType.MouseButton2 and not gameProcessed then
        aimbotActive = not aimbotActive
        print(aimbotActive and "[Aimbot] Aimbot ativado!" or "[Aimbot] Aimbot desativado!")
    end
end)

-- Iniciar o script após o personagem carregar
waitForCharacter()

-- Atualizar a cada frame
RunService.RenderStepped:Connect(updateAimbot)
print("[Aimbot] Script iniciado!")
