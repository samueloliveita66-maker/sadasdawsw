-- CONFIG
local Aimbot = false
local ESP = false
local TeamCheck = true

-- SERVIÇOS
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

-- GUI
local gui = Instance.new("ScreenGui")
gui.Parent = game.CoreGui

local frame = Instance.new("Frame")
frame.Parent = gui
frame.Size = UDim2.new(0,200,0,140)
frame.Position = UDim2.new(0,50,0,200)
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)

-- TITULO
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1,0,0,30)
title.Text = "MENU HACK"
title.BackgroundColor3 = Color3.fromRGB(0,170,255)
title.TextColor3 = Color3.new(1,1,1)

-- BOTÕES
local function criar(texto, y, func)
    local b = Instance.new("TextButton", frame)
    b.Size = UDim2.new(1,-20,0,30)
    b.Position = UDim2.new(0,10,0,y)
    b.Text = texto.." OFF"
    b.BackgroundColor3 = Color3.fromRGB(40,40,40)
    b.TextColor3 = Color3.new(1,1,1)

    b.MouseButton1Click:Connect(function()
        func()
        if b.Text:find("OFF") then
            b.Text = texto.." ON"
            b.BackgroundColor3 = Color3.fromRGB(0,170,0)
        else
            b.Text = texto.." OFF"
            b.BackgroundColor3 = Color3.fromRGB(40,40,40)
        end
    end)
end

criar("Aimbot",40,function() Aimbot = not Aimbot end)
criar("ESP",75,function() ESP = not ESP end)
criar("Team",110,function() TeamCheck = not TeamCheck end)

-- EXPANDIR
local big = false
local expand = Instance.new("TextButton", frame)
expand.Size = UDim2.new(0,30,0,30)
expand.Position = UDim2.new(1,-30,0,0)
expand.Text = "+"
expand.BackgroundColor3 = Color3.fromRGB(0,170,255)

expand.MouseButton1Click:Connect(function()
    big = not big
    if big then
        frame.Size = UDim2.new(0,300,0,220)
        expand.Text = "-"
    else
        frame.Size = UDim2.new(0,200,0,140)
        expand.Text = "+"
    end
end)

-- AIMBOT
local function getTarget()
    local target = nil
    local dist = math.huge

    for _,p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
            if TeamCheck and p.Team == LocalPlayer.Team then continue end

            local pos,vis = Camera:WorldToViewportPoint(p.Character.Head.Position)
            if vis then
                local diff = (Vector2.new(pos.X,pos.Y) - Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude
                if diff < dist then
                    dist = diff
                    target = p
                end
            end
        end
    end

    return target
end

-- LOOP
RunService.RenderStepped:Connect(function()
    if Aimbot then
        local t = getTarget()
        if t and t.Character then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, t.Character.Head.Position)
        end
    end
end)
