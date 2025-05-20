-- =============================================
-- AIMBOT HACKER (SEM COOLDOWN) - LOCALSCRIPT
-- =============================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- Configurações ajustáveis
local AIM_ASSIST_STRENGTH = 0.25  -- Força da assistência (0.1 = fraco, 0.9 = forte)
local MAX_DISTANCE = 150  -- Distância máxima de detecção
local HIGHLIGHT_COLOR = Color3.fromRGB(255, 50, 50)  -- Cor do highlight

-- Variáveis de estado
local active = false
local currentTarget = nil
local highlightInstances = {}

-- Função para destacar inimigos
local function highlightEnemies()
    -- Remove highlights antigos
    for _, highlight in ipairs(highlightInstances) do
        highlight:Destroy()
    end
    highlightInstances = {}
    
    -- Adiciona novos highlights
    local enemies = workspace:FindFirstChild("Enemies") or workspace
    for _, enemy in ipairs(enemies:GetChildren()) do
        if enemy:FindFirstChild("Humanoid") and enemy:FindFirstChild("Head") then
            local highlight = Instance.new("Highlight")
            highlight.FillTransparency = 0.8
            highlight.OutlineColor = HIGHLIGHT_COLOR
            highlight.Parent = enemy
            table.insert(highlightInstances, highlight)
        end
    end
end

-- Encontra o inimigo mais próximo
local function findNearestEnemy()
    local closest = nil
    local closestDistance = MAX_DISTANCE
    local char = player.Character
    
    if not char or not char:FindFirstChild("HumanoidRootPart") then 
        return nil 
    end
    
    local rootPos = char.HumanoidRootPart.Position
    
    for _, enemy in ipairs(workspace:GetChildren()) do
        if enemy:FindFirstChild("Humanoid") and enemy:FindFirstChild("HumanoidRootPart") then
            local enemyRoot = enemy.HumanoidRootPart
            local distance = (rootPos - enemyRoot.Position).Magnitude
            
            if distance < closestDistance then
                closestDistance = distance
                closest = enemyRoot
            end
        end
    end
    
    return closest
end

-- Suaviza a mira em direção ao alvo
local function smoothAim(target)
    if not target then return end
    
    local currentCF = camera.CFrame
    local targetPos = target.Position
    local direction = (targetPos - currentCF.Position).Unit
    
    -- Aplica a interpolação suave
    local newLook = currentCF.LookVector:Lerp(direction, AIM_ASSIST_STRENGTH)
    camera.CFrame = CFrame.lookAt(currentCF.Position, currentCF.Position + newLook)
end

-- Ativa/desativa com a tecla Q
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.Q and not gameProcessed then
        active = not active
        
        if active then
            highlightEnemies()
            print("Aimbot Hacker ATIVADO")
        else
            for _, highlight in ipairs(highlightInstances) do
                highlight:Destroy()
            end
            highlightInstances = {}
            print("Aimbot Hacker DESATIVADO")
        end
    end
end)

-- Loop principal de assistência
RunService.RenderStepped:Connect(function()
    if active then
        currentTarget = findNearestEnemy()
        if currentTarget then
            smoothAim(currentTarget)
        end
    end
end)

-- Limpeza quando o script é destruído
script.Destroying:Connect(function()
    for _, highlight in ipairs(highlightInstances) do
        highlight:Destroy()
    end
end)

print("Aimbot Hacker pronto! Pressione Q para ativar/desativar")
