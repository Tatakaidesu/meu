local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local SoundService = game:GetService("SoundService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local maxDistance = 100 -- Distância máxima para detectar alvos
local aimbotActive = false -- Estado do aimbot
local toolName = "Gun" -- Nome da ferramenta necessária para ativar o aimbot (ajuste conforme necessário)

-- Criar UI para feedback visual
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = player:WaitForChild("PlayerGui")
ScreenGui.Name = "AimbotUI"
local TextLabel = Instance.new("TextLabel")
TextLabel.Size = UDim2.new(0, 200, 0, 50)
TextLabel.Position = UDim2.new(0.5, -100, 0, 10)
TextLabel.BackgroundTransparency = 0.5
TextLabel.BackgroundColor3 = Color3.new(0, 0, 0)
TextLabel.TextColor3 = Color3.new(1, 1, 1)
TextLabel.Text = "Aimbot: OFF"
TextLabel.Parent = ScreenGui

-- Criar sons para ativação/desativação
local activateSound = Instance.new("Sound")
activateSound.SoundId = "rbxasset://sounds/click.wav" -- Som padrão do Roblox (pode mudar)
activateSound.Volume = 1
activateSound.Parent = SoundService

local deactivateSound = Instance.new("Sound")
deactivateSound.SoundId = "rbxasset://sounds/switch.wav" -- Som padrão do Roblox (pode mudar)
deactivateSound.Volume = 1
deactivateSound.Parent = SoundService

-- Função para aguardar o personagem carregar
local function waitForCharacter()
    print("[Aimbot] Aguardando personagem carregar...")
    if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then
        player.CharacterAdded:Wait()
    end
    print("[Aimbot] Personagem carregado!")
    return player.Character
end

-- Função para verificar se a ferramenta correta está equipada
local function isToolEquipped()
    local character = player.Character
    if character then
        local tool = character:FindFirstChildOfClass("Tool")
        return tool and tool.Name == toolName
    end
    return false
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

    -- Verificar NPCs (opcional, inclui alvos não-jogadores com Humanoid)
    for _, npc in pairs(workspace:GetDescendants()) do
        if npc:IsA("Model") and npc:FindFirstChild("Humanoid") and npc:FindFirstChild("HumanoidRootPart") then
            local targetHumanoid = npc.Humanoid
            if targetHumanoid.Health > 0 then
                local distance = (character.HumanoidRootPart.Position - npc.HumanoidRootPart.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestTarget = npc.HumanoidRootPart
                    print("[Aimbot] Alvo NPC encontrado: " .. npc.Name .. " a " .. math.floor(distance) .. " studs")
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
    if aimbotActive and isToolEquipped() then
        local target = findNearestTarget()
        if target then
            print("[Aimbot] Mirando em: " .. tostring(target.Parent.Name))
            camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = CFrame.new(player.Character.HumanoidRootPart.Position, Vector3.new(target.Position.X,
