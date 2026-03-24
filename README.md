-- CONFIG
local AimbotAtivo = false
local ESPAtivo = false
local TeamCheck = true

-- SERVIÇOS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

-- GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 220, 0, 150)
Frame.Position = UDim2.new(0, 20, 0, 200)
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BorderSizePixel = 0

-- borda bonita
local UIStroke = Instance.new("UIStroke", Frame)
UIStroke.Color = Color3.fromRGB(0,170,255)
UIStroke.Thickness = 2

-- titulo
local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1,0,0,30)
Title.Text = "🔥 Painel Hack"
Title.TextColor3 = Color3.new(1,1,1)
Title.BackgroundTransparency = 1
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18

-- função criar botão
local function criarBotao(nome, posY, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(1, -20, 0, 30)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
    btn.TextColor3 = Color3.new(1,1,1)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 16
    
    btn.Text = nome .. " [OFF]"

    btn.MouseButton1Click:Connect(function()
        callback()
        if btn.Text:find("OFF") then
            btn.Text = nome .. " [ON]"
            btn.BackgroundColor3 = Color3.fromRGB(0,170,0)
        else
            btn.Text = nome .. " [OFF]"
            btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
        end
    end)
end

-- BOTÕES
criarBotao("Aimbot", 35, function()
    AimbotAtivo = not AimbotAtivo
end)

criarBotao("ESP", 70, function()
    ESPAtivo = not ESPAtivo
end)

criarBotao("TeamCheck", 105, function()
    TeamCheck = not TeamCheck
end)

-- BOTÃO EXPANDIR
local expandido = false

local ExpandBtn = Instance.new("TextButton", Frame)
ExpandBtn.Size = UDim2.new(0, 30, 0, 30)
ExpandBtn.Position = UDim2.new(1, -35, 0, 0)
ExpandBtn.Text = "+"
ExpandBtn.BackgroundColor3 = Color3.fromRGB(0,170,255)
ExpandBtn.TextColor3 = Color3.new(1,1,1)

ExpandBtn.MouseButton1Click:Connect(function()
    expandido = not expandido

    if expandido then
        Frame:TweenSize(UDim2.new(0, 350, 0, 250), "Out", "Quad", 0.3, true)
        ExpandBtn.Text = "-"
    else
        Frame:TweenSize(UDim2.new(0, 220, 0, 150), "Out", "Quad", 0.3, true)
        ExpandBtn.Text = "+"
    end
end)

-- DRAG (arrastar GUI)
local dragging, dragInput, dragStart, startPos

Frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = Frame.Position
    end
end)

Frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UIS.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - dragStart
        Frame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- ESP + AIMBOT (MESMO DO ANTERIOR)

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

    if AimbotAtivo then
        local target = getClosest()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end)
