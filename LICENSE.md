-- This is a simple aimbot script for Roblox. It will automatically aim at the player with the highest health.
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local function GetClosestPlayer()
    local ClosestPlayer = nil
    local ClosestDistance = math.huge

    for _, Player in ipairs(Players:GetPlayers()) do
        if Player ~= LocalPlayer and Player.Character and Player.Character:FindFirstChild("Humanoid") and Player.Character.Humanoid.Health > 0 then
            local Distance = (Player.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            if Distance < ClosestDistance then
                ClosestPlayer = Player
                ClosestDistance = Distance
            end
        end
    end

    return ClosestPlayer
end

local function OnInputBegan(Input, GameProcessed)
    if Input.UserInputType == Enum.UserInputType.MouseButton2 and not GameProcessed then
        local ClosestPlayer = GetClosestPlayer()
        if ClosestPlayer then
            Mouse.TargetFilter = ClosestPlayer.Character
        end
    end
end

Mouse.InputBegan:Connect(OnInputBegan)
