local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "AIM HUB V2",
    LoadingTitle = "Carregando...",
    LoadingSubtitle = "by Kaio",
    ConfigurationSaving = { Enabled = true },
    Discord = { Enabled = false },
    KeySystem = false
})

-----------------------------------------------------
-- Aba AimBot
-----------------------------------------------------
local AimTab = Window:CreateTab("AimBot", 4483362458)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local AimAssistEnabled = false
local FOV_Radius = 100 -- padrão atualizado
local AimSmooth = 0.07 -- padrão atualizado
local MaxDistance = 200 -- padrão atualizado
local TeamCheck = false

-- Desenho do FOV
local Circle = Drawing.new("Circle")
Circle.Visible = false
Circle.Thickness = 2
Circle.Color = Color3.fromRGB(0, 255, 0)
Circle.NumSides = 100
Circle.Radius = FOV_Radius
Circle.Filled = false

-- Função para achar o player mais próximo dentro do FOV
local function GetClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = FOV_Radius
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer 
        and player.Character 
        and player.Character:FindFirstChild("HumanoidRootPart")
        and player.Character:FindFirstChild("Humanoid") 
        and player.Character.Humanoid.Health > 0 then

            if not TeamCheck or (player.Team ~= LocalPlayer.Team) then
                local targetPart = player.Character.HumanoidRootPart
                local direction = targetPart.Position - Camera.CFrame.Position

                local rayParams = RaycastParams.new()
                rayParams.FilterType = Enum.RaycastFilterType.Blacklist
                rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
                local ray = workspace:Raycast(Camera.CFrame.Position, direction, rayParams)
                local visible = ray and ray.Instance:IsDescendantOf(player.Character)

                if visible then
                    local pos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                    if onScreen then
                        local mousePos = UserInputService:GetMouseLocation()
                        local distance = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
                        local distanceFromPlayer = (LocalPlayer.Character.HumanoidRootPart.Position - targetPart.Position).Magnitude
                        if distance < shortestDistance and distanceFromPlayer <= MaxDistance then
                            shortestDistance = distance
                            closestPlayer = player
                        end
                    end
                end
            end
        end
    end
    return closestPlayer
end

-- Atualizar FOV e mira suave (Aim Assist)
RunService.RenderStepped:Connect(function()
    if Circle.Visible then
        local mouse = UserInputService:GetMouseLocation()
        Circle.Position = Vector2.new(mouse.X, mouse.Y)
    end

    if AimAssistEnabled then
        local target = GetClosestPlayer()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = target.Character.HumanoidRootPart.Position
            local desiredCFrame = CFrame.new(Camera.CFrame.Position, targetPos)
            Camera.CFrame = Camera.CFrame:Lerp(desiredCFrame, math.clamp(AimSmooth, 0.01, 1))
            Circle.Color = Color3.fromRGB(255, 0, 0)
        else
            Circle.Color = Color3.fromRGB(0, 255, 0)
        end
    end
end)

-- Toggle do Aim Assist
AimTab:CreateToggle({
    Name = "Aim Assist",
    CurrentValue = false,
    Flag = "AimAssist",
    Callback = function(Value)
        AimAssistEnabled = Value
        Circle.Visible = Value
    end,
})

-- Slider do FOV
AimTab:CreateSlider({
    Name = "Tamanho do FOV",
    Range = {50, 400},
    Increment = 10,
    Suffix = " px",
    CurrentValue = FOV_Radius, -- padrão 100
    Flag = "FOVSlider",
    Callback = function(Value)
        FOV_Radius = Value
        Circle.Radius = Value
    end,
})

-- Slider de suavidade
AimTab:CreateSlider({
    Name = "Suavidade (0.01 = muito suave, 1 = instantâneo)",
    Range = {0.01, 1},
    Increment = 0.01,
    CurrentValue = AimSmooth, -- padrão 0.07
    Flag = "AimSmooth",
    Callback = function(Value)
        AimSmooth = Value
    end,
})

-- Slider de distância máxima
AimTab:CreateSlider({
    Name = "Distância Máxima",
    Range = {50, 2000},
    Increment = 50,
    Suffix = " studs",
    CurrentValue = MaxDistance, -- padrão 200
    Flag = "MaxDistance",
    Callback = function(Value)
        MaxDistance = Value
    end,
})

-- Toggle do Team Check
AimTab:CreateToggle({
    Name = "Team Check",
    CurrentValue = false,
    Flag = "TeamCheck",
    Callback = function(Value)
        TeamCheck = Value
    end,
})

-----------------------------------------------------
-- Aba Visual
-----------------------------------------------------
local Lighting = game:GetService("Lighting")

-- Guardar valores originais
local OriginalFogEnd = Lighting.FogEnd
local OriginalFogStart = Lighting.FogStart

local VisualTab = Window:CreateTab("Visual", 4483362458)

-- Função NoFog
VisualTab:CreateToggle({
    Name = "NoFog",
    CurrentValue = false,
    Flag = "NoFog",
    Callback = function(Value)
        if Value then
            Lighting.FogEnd = 100000 -- remove neblina
            Lighting.FogStart = 0
            if Lighting:FindFirstChild("Atmosphere") then
                Lighting.Atmosphere.Density = 0
            end
        else
            Lighting.FogEnd = OriginalFogEnd
            Lighting.FogStart = OriginalFogStart
            if Lighting:FindFirstChild("Atmosphere") then
                Lighting.Atmosphere.Density = 0.3
            end
        end
    end,
})
