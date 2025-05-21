local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local Mouse = Player:GetMouse()
local Camera = workspace.CurrentCamera
local Char = Player.Character or Player.CharacterAdded:Wait()
local gui = Instance.new("ScreenGui", Player:WaitForChild("PlayerGui"))
gui.Name = "HackMenu"
gui.ResetOnSpawn = false

-- Base arrastável
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 240, 0, 320)
mainFrame.Position = UDim2.new(0, 20, 0, 20)
mainFrame.BackgroundTransparency = 1
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = gui

-- Criação de botões
local function criarBotao(nome, posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 200, 0, 30)
    btn.Position = UDim2.new(0, 20, 0, posY)
    btn.Text = nome
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Font = Enum.Font.Code
    btn.TextSize = 16
    btn.Parent = mainFrame
    return btn
end

local hacks = {"Aimbot", "ESP", "Speed", "Fly", "Teleport"}
local botoes = {}
local estado = {}

for i, nome in ipairs(hacks) do
    local botao = criarBotao(nome, 20 + (i - 1) * 40)
    estado[nome] = false
    botao.MouseButton1Click:Connect(function()
        estado[nome] = not estado[nome]
        botao.BackgroundColor3 = estado[nome] and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(30, 30, 30)
    end)
    botoes[nome] = botao
end

_G.HackEstado = estado

-- Status do sistema
local status = Instance.new("TextLabel")
status.Size = UDim2.new(0, 300, 0, 25)
status.Position = UDim2.new(0, 20, 0, #hacks * 40 + 30)
status.Text = "🔐 Sistema Seguro - Indetectável"
status.TextColor3 = Color3.new(0, 1, 0)
status.BackgroundTransparency = 1
status.Font = Enum.Font.Code
status.TextSize = 14
status.Parent = mainFrame

-- Proteção fake
task.spawn(function()
    while true do
        task.wait(math.random(10, 20))
        status.Text = "⚠️ Detecção de Risco - Ocultando Cheats..."
        status.TextColor3 = Color3.new(1, 1, 0)
        for nome, _ in pairs(_G.HackEstado) do
            _G.HackEstado[nome] = false
            botoes[nome].BackgroundColor3 = Color3.fromRGB(100, 100, 100)
        end
        task.wait(5)
        status.Text = "✅ Sistema Reestabilizado - Ative manualmente"
        status.TextColor3 = Color3.new(0, 1, 0)
        for nome, _ in pairs(_G.HackEstado) do
            botoes[nome].BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        end
    end
end)

-- Aimbot
local corpoParaMirar = "Head"
local fov = 120
local function encontrarAlvo()
    local menorDist = fov
    local alvo = nil
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= Player and p.Character and p.Character:FindFirstChild(corpoParaMirar) then
            local parte = p.Character[corpoParaMirar]
            local tela, visivel = Camera:WorldToViewportPoint(parte.Position)
            if visivel then
                local dist = (Vector2.new(tela.X, tela.Y) - Vector2.new(Mouse.X, Mouse.Y)).Magnitude
                if dist < menorDist then
                    menorDist = dist
                    alvo = parte
                end
            end
        end
    end
    return alvo
end

RunService.RenderStepped:Connect(function()
    if _G.HackEstado["Aimbot"] then
        local alvo = encontrarAlvo()
        if alvo then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, alvo.Position)
        end
    end
end)

-- ESP
function criarESP(alvo)
    local box = Instance.new("BoxHandleAdornment")
    box.Size = Vector3.new(2, 3, 1)
    box.Transparency = 0.5
    box.Color3 = Color3.new(1, 0, 0)
    box.AlwaysOnTop = true
    box.ZIndex = 5
    box.Adornee = alvo
    box.Name = "ESPBox"
    box.Parent = alvo
    return box
end

RunService.RenderStepped:Connect(function()
    if _G.HackEstado["ESP"] then
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= Player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local root = p.Character.HumanoidRootPart
                if not root:FindFirstChild("ESPBox") then
                    criarESP(root)
                end
            end
        end
    end
end)

-- Speed Hack
RunService.RenderStepped:Connect(function()
    if _G.HackEstado["Speed"] then
        if Player.Character then
            Player.Character.Humanoid.WalkSpeed = 50
        end
    else
        if Player.Character then
            Player.Character.Humanoid.WalkSpeed = 16
        end
    end
end)

-- Fly / Super Jump
RunService.RenderStepped:Connect(function()
    if _G.HackEstado["Fly"] then
        if Player.Character then
            Player.Character.Humanoid.JumpPower = 150
        end
    else
        if Player.Character then
            Player.Character.Humanoid.JumpPower = 50
        end
    end
end)

-- Teleport Hack (com tecla T)
local lugares = {
    Vector3.new(0, 10, 0),
    Vector3.new(100, 30, 100),
    Vector3.new(0, 100, 0),
}
local index = 1
UserInputService.InputBegan:Connect(function(input, gp)
    if input.KeyCode == Enum.KeyCode.T and _G.HackEstado["Teleport"] then
        if Player.Character then
            Player.Character:MoveTo(lugares[index])
            index += 1
            if index > #lugares then index = 1 end
        end
    end
end)
