-- CONFIG
local AimbotAtivo = false
local ESPAtivo = false
local TeamCheck = true

-- SERVIÇOS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 200, 0, 120)
Frame.Position = UDim2.new(0, 20, 0, 200)
Frame.BackgroundColor3 = Color3.fromRGB(30,30,30)

local function criarBotao(texto, posY, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.Position = UDim2.new(0, 0, 0, posY)
    btn.Text = texto
    btn.BackgroundColor3 = Color3.fromRGB(50,50,50)
    
    btn.MouseButton1Click:Connect(callback)
end

criarBotao("Aimbot ON/OFF", 0, function()
    AimbotAtivo = not AimbotAtivo
end)

criarBotao("ESP ON/OFF", 30, function()
    ESPAtivo = not ESPAtivo
end)

criarBotao("Team Check ON/OFF", 60, function()
    TeamCheck = not TeamCheck
end)

-- ESP
local ESPs = {}

function criarESP(player)
    if player == LocalPlayer then return end

    local Box = Drawing.new("Square")
    Box.Color = Color3.new(1,0,0)
    Box.Thickness = 2
    Box.Filled = false

    local Line = Drawing.new("Line")
    Line.Color = Color3.new(1,1,1)
    Line.Thickness = 1

    ESPs[player] = {Box = Box, Line = Line}
end

for _, p in pairs(Players:GetPlayers()) do
    criarESP(p)
end

Players.PlayerAdded:Connect(criarESP)

-- Aimbot
function getClosest()
    local closest = nil
    local dist = math.huge

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            
            if TeamCheck and p.Team == LocalPlayer.Team then
                continue
            end

            local pos, onScreen = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if onScreen then
                local diff = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
                if diff < dist then
                    dist = diff
                    closest = p
                end
            end
        end
    end

    return closest
end

-- LOOP
RunService.RenderStepped:Connect(function()
    for player, esp in pairs(ESPs) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and ESPAtivo then
            
            if TeamCheck and player.Team == LocalPlayer.Team then
                esp.Box.Visible = false
                esp.Line.Visible = false
                continue
            end

            local pos, onScreen = Camera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)

            if onScreen then
                esp.Box.Size = Vector2.new(50, 80)
                esp.Box.Position = Vector2.new(pos.X - 25, pos.Y - 40)
                esp.Box.Visible = true

                esp.Line.From = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y)
                esp.Line.To = Vector2.new(pos.X, pos.Y)
                esp.Line.Visible = true
            else
                esp.Box.Visible = false
                esp.Line.Visible = false
            end
        else
            esp.Box.Visible = false
            esp.Line.Visible = false
        end
    end

    -- AIMBOT
    if AimbotAtivo then
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)
