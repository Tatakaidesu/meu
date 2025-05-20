-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- Configurações
local SETTINGS = {
    AIMBOT = {
        ACTIVATION_KEY = Enum.UserInputType.MouseButton2,  -- Botão direito do mouse
        STRENGTH = 0.25,  -- Força da assistência (0.1-0.9)
        MAX_DISTANCE = 1000,  -- Alcance máximo
        ENABLED = true
    },
    ESP = {
        ENABLED = true,
        BOX_COLOR = Color3.fromRGB(255, 50, 50),
        TEXT_COLOR = Color3.fromRGB(255, 255, 255),
        SHOW_NAMES = true
    }
}

-- Variáveis
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local espCache = {}
local aimbotActive = false

-- Funções do ESP
local function createESP(target)
    if not target.Character or espCache[target] then return end
    
    local character = target.Character
    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
    if not humanoidRootPart then return end

    -- Highlight (caixa)
    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.FillTransparency = 0.85
    highlight.OutlineColor = SETTINGS.ESP.BOX_COLOR
    highlight.OutlineTransparency = 0
    highlight.Parent = character

    -- Billboard (nome)
    if SETTINGS.ESP.SHOW_NAMES then
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESP_Billboard"
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.AlwaysOnTop = true
        billboard.Adornee = humanoidRootPart

        local textLabel = Instance.new("TextLabel")
        textLabel.Text = target.Name
        textLabel.Size = UDim2.new(1, 0, 1, 0)
        textLabel.TextColor3 = SETTINGS.ESP.TEXT_COLOR
        textLabel.BackgroundTransparency = 1
        textLabel.TextStrokeTransparency = 0.7
        textLabel.Font = Enum.Font.SciFi
        textLabel.TextSize = 18
        textLabel.Parent = billboard
        billboard.Parent = character

        espCache[target] = {
            Highlight = highlight,
            Billboard = billboard
        }
    else
        espCache[target] = {
            Highlight = highlight
        }
    end
end

local function removeESP(target)
    if not espCache[target] then return end
    
    for _, item in pairs(espCache[target]) do
        if item then
            item:Destroy()
        end
    end
    espCache[target] = nil
end

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

-- Funções do Aimbot
local function findNearestTarget()
    local closestTarget = nil
    local closestDistance = SETTINGS.AIMBOT.MAX_DISTANCE
    local localChar = player.Character
    
    if not localChar then return nil end
    
    local localRoot = localChar:FindFirstChild("HumanoidRootPart")
    if not localRoot then return nil end
    
    for _, target in ipairs(Players:GetPlayers()) do
        if target ~= player and target.Character then
            local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
            if targetRoot then
                local distance = (localRoot.Position - targetRoot.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestTarget = targetRoot
                end
            end
        end
    end
    
    return closestTarget
end

local function applyAimbot()
    if not aimbotActive or not SETTINGS.AIMBOT.ENABLED then return end
    
    local target = findNearestTarget()
    if not target then return end
    
    local currentCF = camera.CFrame
    local targetPos = target.Position
    local direction = (targetPos - currentCF.Position).Unit
    
    local newLook = currentCF.LookVector:Lerp(direction, SETTINGS.AIMBOT.STRENGTH)
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
    applyAimbot()
end)

-- Gerenciamento de jogadores
Players.PlayerAdded:Connect(function(newPlayer)
    newPlayer.CharacterAdded:Connect(function()
        if SETTINGS.ESP.ENABLED then
            createESP(newPlayer)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(leavingPlayer)
    removeESP(leavingPlayer)
end)

-- Inicialização
player.CharacterAdded:Connect(function()
    task.wait(1)  -- Espera o personagem carregar
    updateAllESP()
end)

if player.Character then
    task.wait(1)
    updateAllESP()
end

-- Função para atualizar configurações (opcional)
local function updateSettings(newSettings)
    SETTINGS = newSettings
    updateAllESP()
end

print("Sistema Aimbot+ESP carregado com sucesso!")
print("Configurações atuais:", SETTINGS)
