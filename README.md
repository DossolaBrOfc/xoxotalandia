local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Kick imediato com mensagem personalizada
player:Kick("Banido, não use scripts de Knware 🙄")

local pos1 = Vector3.new(-75528, 10, 50244)
local pos2 = Vector3.new(-75608, 96, 50212)

local toggled = false
local farmThread = nil
local statusLabel = nil

local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local function onCharacterAdded(newChar)
    character = newChar
    humanoidRootPart = character:WaitForChild("HumanoidRootPart", 10)
    if statusLabel then
        statusLabel.Text = "Respawn detectado - Personagem atualizado"
        statusLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    end
    if toggled then
        task.wait(1)
        if farmThread then coroutine.close(farmThread) end
        farmThread = task.spawn(farmLoop)
    end
end
player.CharacterAdded:Connect(onCharacterAdded)

local function updateStatus(text, color)
    if statusLabel then
        statusLabel.Text = text
        statusLabel.TextColor3 = color or Color3.fromRGB(255, 255, 255)
    end
end

local function pressE()
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.E, false, game)
    task.wait(0.1)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.E, false, game)
end

local function farmLoop()
    updateStatus("Farm ATIVO", Color3.fromRGB(50, 200, 50))
    
    while toggled do
        if not character or not character.Parent or not humanoidRootPart or not humanoidRootPart.Parent then
            updateStatus("Aguardando personagem...", Color3.fromRGB(255, 165, 0))
            task.wait(2)
            continue
        end
        
        humanoidRootPart.CFrame = CFrame.new(pos1)
        task.wait(1)
        
        updateStatus("Interagindo (tecla E)", Color3.fromRGB(100, 200, 255))
        pressE()
        task.wait(1.5)
        
        humanoidRootPart.CFrame = CFrame.new(pos2)
        task.wait(2)
    end
    
    updateStatus("Farm DESATIVADO", Color3.fromRGB(255, 100, 100))
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoFarmGUI_V3"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 240, 0, 160)
mainFrame.Position = UDim2.new(0.5, -120, 0.5, -80)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(80, 180, 255)
stroke.Thickness = 2
stroke.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -20, 0, 40)
title.Position = UDim2.new(0, 10, 0, 10)
title.BackgroundTransparency = 1
title.Text = "Auto Farm V3"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0.9, 0, 0, 50)
toggleBtn.Position = UDim2.new(0.05, 0, 0.35, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
toggleBtn.Text = "OFF"
toggleBtn.TextColor3 = Color3.new(1, 1, 1)
toggleBtn.TextScaled = true
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.Parent = mainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 10)
btnCorner.Parent = toggleBtn

statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -20, 0, 40)
statusLabel.Position = UDim2.new(0, 10, 0.7, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "Pronto"
statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
statusLabel.TextScaled = true
statusLabel.TextWrapped = true
statusLabel.Font = Enum.Font.Gotham
statusLabel.Parent = mainFrame

local dragging = false
local dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

toggleBtn.MouseButton1Click:Connect(function()
    toggled = not toggled
    if toggled then
        toggleBtn.Text = "ON"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
        updateStatus("Farm iniciado", Color3.fromRGB(50, 255, 50))
        farmThread = task.spawn(farmLoop)
    else
        toggleBtn.Text = "OFF"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
        updateStatus("Farm parado", Color3.fromRGB(255, 100, 100))
        if farmThread then
            coroutine.close(farmThread)
            farmThread = nil
        end
    end
end)

updateStatus("Script carregado", Color3.fromRGB(100, 255, 100))
