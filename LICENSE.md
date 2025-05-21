local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer
local Mouse = Player:GetMouse()
local Camera = workspace.CurrentCamera
local Char = Player.Character or Player.CharacterAdded:Wait()

-- Criar GUI estilizada e arrastável
local gui = Instance.new("ScreenGui", Player:WaitForChild("PlayerGui"))
gui.Name = "HackMenu"
gui.ResetOnSpawn = false

local shadow = Instance.new("Frame")
shadow.Size = UDim2.new(0, 286, 0, 366)
shadow.Position = UDim2.new(0, 23, 0, 23)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BackgroundTransparency = 0.7
shadow.BorderSizePixel = 0
shadow.ZIndex = 1
shadow.Parent = gui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 280, 0, 360)
mainFrame.Position = UDim2.new(0, 20, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.ZIndex = 2
mainFrame.Parent = gui

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 0, 40)
title.Position = UDim2.new(0, 20, 0, 10)
title.BackgroundTransparency = 1
title.Text = "💻 Hack Menu"
title.TextColor3 = Color3.fromRGB(0, 255, 128)
title.Font = Enum.Font.Code
title.TextSize = 24
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = mainFrame

local underline = Instance.new("Frame")
underline.Size = UDim2.new(0, 240, 0, 2)
underline.Position = UDim2.new(0, 20, 0, 50)
underline.BackgroundColor3 = Color3.fromRGB(0, 255, 128)
underline.BorderSizePixel = 0
underline.Parent = mainFrame

local function criarBotao(nome, posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 240, 0, 36)
    btn.Position = UDim2.new(0, 20, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.BorderSizePixel = 0
    btn.AutoButtonColor = false
    btn.Font = Enum.Font.Code
    btn.Text = nome
    btn.TextColor3 = Color3.fromRGB(180, 180, 180)
    btn.TextSize = 18
    btn.TextWrapped = true
    btn.Parent = mainFrame

    btn.MouseEnter:Connect(function()
        if btn.BackgroundColor3 ~= Color3.fromRGB(0, 170, 0) then
            btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
            btn.TextColor3 = Color3.fromRGB(220, 220, 220)
        end
    end)
    btn.MouseLeave:Connect(function()
        if btn.BackgroundColor3 ~= Color3.fromRGB(0, 170, 0) then
            btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
            btn.TextColor3 = Color3.fromRGB(180, 180, 180)
        end
    end)

    return btn
end

local hacks = {"Aimbot", "ESP", "Speed", "Fly", "Teleport"}
local botoes = {}
local estado = {}

for i, nome in ipairs(hacks) do
    local botao = criarBotao(nome, 70 + (i - 1) * 46)
    estado[nome] = false
    botao.MouseButton1Click:Connect(function()
        estado[nome] = not estado[nome]
        if estado[nome] then
            botao.BackgroundColor3 = Color3.fromRGB(0, 170, 0)
            botao.TextColor3 = Color3.fromRGB(230, 255, 230)
        else
            botao.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
            botao.TextColor3 = Color3.fromRGB(180, 180, 180)
        end
    end)
    botoes[nome] = botao
end

_G.HackEstado = estado

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, -40, 0, 28)
status.Position = UDim2.new(0, 20, 0, 310)
status.Text = "🟢 Hacks Ativos - Menu"
status.TextColor3 = Color3.fromRGB(0, 255, 128)
status.BackgroundTransparency = 1
status.Font = Enum.Font.Code
status.TextSize = 14
status.TextXAlignment = Enum.TextXAlignment.Left
status.Parent = mainFrame

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
local function criarESP(alvo)
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
    if Player.Character then
        Player.Character.Humanoid.WalkSpeed = _G.HackEstado["Speed"] and 50 or 16
    end
end)

-- Fly / Super Jump
RunService.RenderStepped:Connect(function()
    if Player.Character then
        Player.Character.Humanoid.JumpPower = _G.HackEstado["Fly"] and 150 or 50
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
            index = index + 1
            if index > #lugares then index = 1 end
        end
    end
end)
