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
mainFrame.Size = UDim2.new(0, 280, 0, 360)
mainFrame.Position = UDim2.new(0, 20, 0, 20)
mainFrame.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = gui

-- Sombra (frame atrás para profundidade)
local shadow = Instance.new("Frame")
shadow.Size = mainFrame.Size + UDim2.new(0, 6, 0, 6)
shadow.Position = mainFrame.Position + UDim2.new(0, 3, 0, 3)
shadow.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
shadow.BackgroundTransparency = 0.7
shadow.BorderSizePixel = 0
shadow.ZIndex = mainFrame.ZIndex - 1
shadow.Parent = gui

-- Título do menu
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

-- Linha de destaque sob o título
local underline = Instance.new("Frame")
underline.Size = UDim2.new(0, 240, 0, 2)
underline.Position = UDim2.new(0, 20, 0, 50)
underline.BackgroundColor3 = Color3.fromRGB(0, 255, 128)
underline.BorderSizePixel = 0
underline.Parent = mainFrame

-- Função para criar botões estilizados com efeito hover
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

    -- Efeito hover (mouse entra)
    btn.MouseEnter:Connect(function()
        if btn.BackgroundColor3 ~= Color3.fromRGB(0, 170, 0) then -- se não ativo
            btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
            btn.TextColor3 = Color3.fromRGB(220, 220, 220)
        end
    end)
    -- Efeito hover (mouse sai)
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

-- Status simples
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
