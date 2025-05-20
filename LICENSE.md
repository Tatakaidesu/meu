-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Configurações
local SETTINGS = {
    AIMBOT = {
        ACTIVATION_KEY = Enum.UserInputType.MouseButton2,  -- Botão direito
        STRENGTH = 0.35,  -- Força do aimbot (0.1-1.0)
        MAX_DISTANCE = 1200,  -- Alcance máximo
        AIM_AT_HEAD = true,  -- Mirar na cabeça
        ENABLED = true
    },
    ESP = {
        ENABLED = true,
        BOX_COLOR = Color3.fromRGB(255, 70, 70),
        TEXT_COLOR = Color3.fromRGB(255, 255, 255)
    }
}

-- Variáveis
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local espCache = {}
local aimbotActive = false

-- Função para criar ESP
local function createESP(target)
    if not target.Character or espCache[target] then return end
    
    local character = target.Character
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.FillTransparency = 0.8
    highlight.OutlineColor = SETTINGS.ESP.BOX_COLOR
    highlight.OutlineTransparency = 0
    highlight.Parent = character

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Billboard"
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Adornee = character:WaitForChild("Head")

    local textLabel = Instance.new("TextLabel")
    textLabel.Text = target.Name
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.TextColor3 = SETTINGS.ESP.TEXT_COLOR
    textLabel.BackgroundTransparency = 1
    textLabel.TextStrokeTransparency = 0.5
    textLabel.Parent = billboard
    billboard.Parent = character

    espCache[target] = {Highlight = highlight, Billboard = billboard}
end

-- Função para remover ESP
local function removeESP(target)
    if espCache[target] then
        espCache[target].Highlight:Destroy()
        espCache[target].Billboard:Destroy()
        espCache[target] = nil
    end
end

-- Atualizar ESP para todos
local function updateAllESP()
    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player then
            if SETTINGS.ESP.ENABLED then
                createESP(target)
            else
                removeESP(target)
            end
        end
    end
end

-- Encontrar melhor alvo (agora prioriza cabeça)
local function findBestTarget()
    local closestTarget = nil
    local closestDistance = SETTINGS.AIMBOT.MAX_DISTANCE
    local localChar = player.Character
    
    if not localChar then return nil end
    
    local localRoot = localChar:FindFirstChild("HumanoidRootPart")
    if not localRoot then return nil end
    
    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player and target.Character then
            local targetHead = target.Character:FindFirstChild("Head")
            local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
            
            if targetHead and targetRoot then
                local distance = (localRoot.Position - targetRoot.Position).Magnitude
                
                if distance < closestDistance then
                    closestDistance = distance
                    closestTarget = SETTINGS.AIMBOT.AIM_AT_HEAD and targetHead or targetRoot
                end
            end
        end
    end
    
    return closestTarget
end

-- Aimbot preciso (com interpolação suave)
local function preciseAimbot()
    if not aimbotActive or not SETTINGS.AIMBOT.ENABLED then return end
    
    local target = findBestTarget()
    if not target then return end
    
    local currentCF = camera.CFrame
    local targetPos = target.Position
    
    -- Cálculo preciso da direção
    local direction = (targetPos - currentCF.Position).Unit
    local newLook = currentCF.LookVector:Lerp(direction, SETTINGS.AIMBOT.STRENGTH)
    
    -- Aplica a mira
    camera.CFrame = CFrame.lookAt(currentCF.Position, currentCF.Position + newLook)
end

-- Controles
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == SETTINGS.AIMBOT.ACTIVATION_KEY then
        aimbotActive = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == SETTINGS.AIMBOT.ACTIVATION_KEY then
        aimbotActive = false
    end
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
    preciseAimbot()
end)

-- Gerenciamento de jogadores
Players.PlayerAdded:Connect(function(newPlayer)
    newPlayer.CharacterAdded:Connect(function()
        if SETTINGS.ESP.ENABLED then
            createESP(newPlayer)
        end
    end)
end)

Players.PlayerRemoving:Connect(removeESP)

-- Inicialização
player.CharacterAdded:Connect(function()
    task.wait(2)  -- Espera o personagem carregar completamente
    updateAllESP()
end)

if player.Character then
    task.wait(2)
    updateAllESP()
end

print("Sistema Aimbot+ESP carregado! Segure o botão direito para mirar na cabeça")
