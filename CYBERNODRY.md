-- FERRAMENTA DE BLOCOS V17 - VOCÊ VAI JUNTO COM O BLOCO!
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera

local grabEnabled = false
local heldBlock = nil
local holdDistance = 10
local grabConnection = nil
local alignPosition = nil
local alignOrientation = nil
local attachment2 = nil
local holdingMinus = false
local holdingPlus = false
local hitOffset = Vector3.new(0, 0, 0)
local smoothTarget = Vector3.new(0, 0, 0)
local lastBlockPos = nil -- 🔥 POSIÇÃO ANTERIOR DO BLOCO

-- 🔥 CONTROLE INFINITO
local function enableInfiniteControl()
    pcall(function()
        sethiddenproperty(player, "SimulationRadius", math.huge)
        player.ReplicationFocus = workspace
    end)
end

enableInfiniteControl()

RunService.Heartbeat:Connect(function()
    pcall(function()
        sethiddenproperty(player, "SimulationRadius", math.huge)
    end)
end)

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "UnanchoredBlocksGUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 240, 0, 180)
mainFrame.Position = UDim2.new(0.5, -120, 0.3, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)

local mainStroke = Instance.new("UIStroke")
mainStroke.Color = Color3.fromRGB(100, 200, 255)
mainStroke.Thickness = 2
mainStroke.Parent = mainFrame

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 35)
titleBar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
titleBar.BorderSizePixel = 0
titleBar.Parent = mainFrame
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -70, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🧲 Move Blocos BY CYBER"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 14
title.Font = Enum.Font.GothamBold
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = titleBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
minimizeBtn.Position = UDim2.new(1, -65, 0, 2.5)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(255, 165, 0)
minimizeBtn.Text = "➖"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.TextSize = 14
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.Parent = titleBar
Instance.new("UICorner", minimizeBtn).CornerRadius = UDim.new(1, 0)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -33, 0, 2.5)
closeBtn.BackgroundColor3 = Color3.fromRGB(220, 53, 69)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextSize = 16
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = titleBar
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(1, 0)

local miniButton = Instance.new("TextButton")
miniButton.Size = UDim2.new(0, 50, 0, 50)
miniButton.Position = UDim2.new(0, 10, 0, 150)
miniButton.BackgroundColor3 = Color3.fromRGB(100, 200, 255)
miniButton.Text = "🧲"
miniButton.TextColor3 = Color3.fromRGB(255, 255, 255)
miniButton.TextSize = 26
miniButton.Font = Enum.Font.GothamBold
miniButton.Active = true
miniButton.Draggable = true
miniButton.Visible = false
miniButton.Parent = screenGui
Instance.new("UICorner", miniButton).CornerRadius = UDim.new(1, 0)

local grabBtn = Instance.new("TextButton")
grabBtn.Size = UDim2.new(1, -20, 0, 40)
grabBtn.Position = UDim2.new(0, 10, 0, 45)
grabBtn.BackgroundColor3 = Color3.fromRGB(220, 53, 69)
grabBtn.Text = "✋ Pegar Blocos: OFF"
grabBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
grabBtn.TextSize = 12
grabBtn.Font = Enum.Font.GothamBold
grabBtn.Parent = mainFrame
Instance.new("UICorner", grabBtn).CornerRadius = UDim.new(0, 8)

local freezeBtn = Instance.new("TextButton")
freezeBtn.Size = UDim2.new(1, -20, 0, 40)
freezeBtn.Position = UDim2.new(0, 10, 0, 92)
freezeBtn.BackgroundColor3 = Color3.fromRGB(100, 150, 255)
freezeBtn.Text = "🔒 Congelar Bloco"
freezeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
freezeBtn.TextSize = 12
freezeBtn.Font = Enum.Font.GothamBold
freezeBtn.Parent = mainFrame
Instance.new("UICorner", freezeBtn).CornerRadius = UDim.new(0, 8)

local distanceFrame = Instance.new("Frame")
distanceFrame.Size = UDim2.new(1, -20, 0, 50)
distanceFrame.Position = UDim2.new(0, 10, 0, 120)
distanceFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
distanceFrame.BorderSizePixel = 0
distanceFrame.Visible = false
distanceFrame.Parent = mainFrame
Instance.new("UICorner", distanceFrame).CornerRadius = UDim.new(0, 8)

local distanceLabel = Instance.new("TextLabel")
distanceLabel.Size = UDim2.new(1, 0, 0, 20)
distanceLabel.Position = UDim2.new(0, 0, 0, 2)
distanceLabel.BackgroundTransparency = 1
distanceLabel.Text = "📏 Distância: 10"
distanceLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
distanceLabel.TextSize = 11
distanceLabel.Font = Enum.Font.GothamBold
distanceLabel.Parent = distanceFrame

local minusBtn = Instance.new("TextButton")
minusBtn.Size = UDim2.new(0, 50, 0, 25)
minusBtn.Position = UDim2.new(0, 10, 0, 22)
minusBtn.BackgroundColor3 = Color3.fromRGB(255, 100, 100)
minusBtn.Text = "-"
minusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minusBtn.TextSize = 18
minusBtn.Font = Enum.Font.GothamBold
minusBtn.Parent = distanceFrame
Instance.new("UICorner", minusBtn).CornerRadius = UDim.new(0, 6)

local plusBtn = Instance.new("TextButton")
plusBtn.Size = UDim2.new(0, 50, 0, 25)
plusBtn.Position = UDim2.new(1, -60, 0, 22)
plusBtn.BackgroundColor3 = Color3.fromRGB(100, 255, 100)
plusBtn.Text = "+"
plusBtn.TextColor3 = Color3.fromRGB(0, 0, 0)
plusBtn.TextSize = 18
plusBtn.Font = Enum.Font.GothamBold
plusBtn.Parent = distanceFrame
Instance.new("UICorner", plusBtn).CornerRadius = UDim.new(0, 6)

-- Mira
for _, data in pairs({
    {UDim2.new(0.5,-2,0.48,-2), UDim2.new(0,4,0,4)},
    {UDim2.new(0.5,-8,0.48,-1), UDim2.new(0,16,0,2)},
    {UDim2.new(0.5,-1,0.48,-8), UDim2.new(0,2,0,16)}
}) do
    local f = Instance.new("Frame")
    f.Position = data[1]
    f.Size = data[2]
    f.BackgroundColor3 = Color3.fromRGB(255,255,255)
    f.BorderSizePixel = 0
    f.ZIndex = 10
    f.Parent = screenGui
end

-- ATTACHMENT PRINCIPAL
local mainAttachment = Instance.new("Attachment")
mainAttachment.Name = "MainAttachment"
mainAttachment.Parent = workspace.Terrain

-- Raycast com offset exato
local function getTargetBlock()
    local ray = camera:ScreenPointToRay(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {player.Character}
    local result = workspace:Raycast(ray.Origin, ray.Direction * 1000, raycastParams)
    if result and result.Instance:IsA("BasePart") and not result.Instance.Anchored then
        return result.Instance, result.Position
    end
    return nil, nil
end

-- 🔥 DETECTA SE VOCÊ TÁ EM CIMA DO BLOCO
local function isStandingOnBlock(block)
    local character = player.Character
    if not character then return false end
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    
    -- Checa se tem o bloco logo abaixo do personagem
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Whitelist
    raycastParams.FilterDescendantsInstances = {block}
    
    local result = workspace:Raycast(hrp.Position, Vector3.new(0, -4, 0), raycastParams)
    return result ~= nil
end

-- Soltar bloco
local function releaseBlock()
    if alignPosition then alignPosition:Destroy() alignPosition = nil end
    if alignOrientation then alignOrientation:Destroy() alignOrientation = nil end
    if attachment2 then attachment2:Destroy() attachment2 = nil end
    
    if heldBlock and heldBlock.Parent then
        heldBlock.CustomPhysicalProperties = nil
        heldBlock.CanCollide = true
    end
    
    heldBlock = nil
    hitOffset = Vector3.new(0,0,0)
    lastBlockPos = nil
    distanceFrame.Visible = false
    mainFrame.Size = UDim2.new(0, 240, 0, 180)
end

-- Botão: Pegar
grabBtn.MouseButton1Click:Connect(function()
    grabEnabled = not grabEnabled
    
    if grabEnabled then
        grabBtn.Text = "✋ Pegar Blocos: ON"
        grabBtn.BackgroundColor3 = Color3.fromRGB(40, 167, 69)
        distanceFrame.Visible = true
        mainFrame.Size = UDim2.new(0, 240, 0, 240)
        
        grabConnection = RunService.RenderStepped:Connect(function()
            if not grabEnabled then return end
            
            if not heldBlock then
                local targetBlock, hitPos = getTargetBlock()
                
                if targetBlock and hitPos then
                    heldBlock = targetBlock
                    hitOffset = heldBlock.CFrame:PointToObjectSpace(hitPos)
                    
                    local originalCFrame = heldBlock.CFrame
                    
                    heldBlock.CustomPhysicalProperties = PhysicalProperties.new(0.01, 0.3, 0.5, 1, 1)
                    heldBlock.CanCollide = true
                    
                    pcall(function()
                        if heldBlock:CanSetNetworkOwnership() then
                            heldBlock:SetNetworkOwner(player)
                        end
                    end)
                    
                    for _, obj in pairs(heldBlock:GetChildren()) do
                        if obj:IsA("BodyAngularVelocity") or obj:IsA("BodyForce") or
                           obj:IsA("BodyGyro") or obj:IsA("BodyPosition") or
                           obj:IsA("BodyThrust") or obj:IsA("BodyVelocity") or
                           obj:IsA("RocketPropulsion") or obj:IsA("Torque") or
                           obj.Name:match("Frozen") or obj.Name:match("Freeze") then
                            obj:Destroy()
                        end
                    end
                    
                    local charPos = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                    if charPos then
                        holdDistance = (hitPos - charPos.Position).Magnitude
                        distanceLabel.Text = "📏 Distância: " .. math.floor(holdDistance)
                    end
                    
                    attachment2 = Instance.new("Attachment")
                    attachment2.Name = "BlockAttachment"
                    attachment2.Position = hitOffset
                    attachment2.Parent = heldBlock
                    
                    alignPosition = Instance.new("AlignPosition")
                    alignPosition.MaxForce = math.huge
                    alignPosition.MaxVelocity = 150
                    alignPosition.Responsiveness = 35
                    alignPosition.Attachment0 = attachment2
                    alignPosition.Attachment1 = mainAttachment
                    alignPosition.Parent = heldBlock
                    
                    alignOrientation = Instance.new("AlignOrientation")
                    alignOrientation.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
                    alignOrientation.MaxAngularVelocity = 0
                    alignOrientation.Responsiveness = 200
                    alignOrientation.RigidityEnabled = true
                    alignOrientation.Attachment0 = attachment2
                    alignOrientation.Attachment1 = mainAttachment
                    alignOrientation.Parent = heldBlock
                    
                    mainAttachment.CFrame = originalCFrame - originalCFrame.Position
                    smoothTarget = hitPos
                    lastBlockPos = heldBlock.Position -- 🔥 SALVA POSIÇÃO INICIAL
                end
            end
            
            if heldBlock and heldBlock.Parent and not heldBlock.Anchored and alignPosition then
                local rawTarget = camera.CFrame.Position + (camera.CFrame.LookVector * holdDistance)
                smoothTarget = smoothTarget:Lerp(rawTarget, 0.12)
                mainAttachment.WorldPosition = smoothTarget
                
                -- 🔥 DETECTA MOVIMENTO DO BLOCO E MOVE VOCÊ JUNTO
                local currentBlockPos = heldBlock.Position
                
                if lastBlockPos and isStandingOnBlock(heldBlock) then
                    local blockDelta = currentBlockPos - lastBlockPos
                    
                    -- Só move junto se o bloco se moveu bastante (evita tremor)
                    if blockDelta.Magnitude > 0.01 then
                        local character = player.Character
                        local hrp = character and character:FindFirstChild("HumanoidRootPart")
                        
                        if hrp then
                            -- 🔥 MOVE VOCÊ NA MESMA DIREÇÃO QUE O BLOCO MOVEU!
                            hrp.CFrame = hrp.CFrame + blockDelta
                        end
                    end
                end
                
                lastBlockPos = currentBlockPos -- 🔥 ATUALIZA POSIÇÃO ANTERIOR
                
                pcall(function()
                    if heldBlock:CanSetNetworkOwnership() then
                        heldBlock:SetNetworkOwner(player)
                    end
                end)
                
                heldBlock.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                
            elseif heldBlock and (not heldBlock.Parent or heldBlock.Anchored) then
                releaseBlock()
            end
        end)
    else
        grabBtn.Text = "✋ Pegar Blocos: OFF"
        grabBtn.BackgroundColor3 = Color3.fromRGB(220, 53, 69)
        releaseBlock()
        
        if grabConnection then
            grabConnection:Disconnect()
            grabConnection = nil
        end
    end
end)

-- Congelar
freezeBtn.MouseButton1Click:Connect(function()
    if heldBlock and heldBlock.Parent then
        local currentPos = heldBlock.Position
        local currentCFrame = heldBlock.CFrame
        
        if alignPosition then alignPosition:Destroy() alignPosition = nil end
        if alignOrientation then alignOrientation:Destroy() alignOrientation = nil end
        if attachment2 then attachment2:Destroy() attachment2 = nil end
        
        heldBlock.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        heldBlock.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
        
        local bodyPosition = Instance.new("BodyPosition")
        bodyPosition.Name = "FrozenPosition"
        bodyPosition.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        bodyPosition.D = 99999
        bodyPosition.P = 999999
        bodyPosition.Position = currentPos
        bodyPosition.Parent = heldBlock
        
        local bodyGyro = Instance.new("BodyGyro")
        bodyGyro.Name = "FrozenGyro"
        bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        bodyGyro.D = 999999
        bodyGyro.P = 9999999
        bodyGyro.CFrame = currentCFrame
        bodyGyro.Parent = heldBlock
        
        heldBlock.CanCollide = true
        heldBlock.CustomPhysicalProperties = nil
        heldBlock = nil
        hitOffset = Vector3.new(0,0,0)
        lastBlockPos = nil
        
        grabEnabled = false
        grabBtn.Text = "✋ Pegar Blocos: OFF"
        grabBtn.BackgroundColor3 = Color3.fromRGB(220, 53, 69)
        distanceFrame.Visible = false
        mainFrame.Size = UDim2.new(0, 240, 0, 180)
        
        if grabConnection then
            grabConnection:Disconnect()
            grabConnection = nil
        end
        
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "🔒 Bloco Congelado!";
            Text = "Travado no ar!";
            Duration = 2;
        })
    else
        game:GetService("StarterGui"):SetCore("SendNotification", {
            Title = "⚠️ Aviso";
            Text = "Segure um bloco primeiro!";
            Duration = 2;
        })
    end
end)

-- Botões distância
minusBtn.MouseButton1Down:Connect(function() holdingMinus = true end)
minusBtn.MouseButton1Up:Connect(function() holdingMinus = false end)
plusBtn.MouseButton1Down:Connect(function() holdingPlus = true end)
plusBtn.MouseButton1Up:Connect(function() holdingPlus = false end)

RunService.RenderStepped:Connect(function()
    if holdingMinus then
        holdDistance = math.max(3, holdDistance - 0.3)
        distanceLabel.Text = "📏 Distância: " .. math.floor(holdDistance)
    end
    if holdingPlus then
        holdDistance = holdDistance + 0.3
        distanceLabel.Text = "📏 Distância: " .. math.floor(holdDistance)
    end
end)

minimizeBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    miniButton.Visible = true
end)

miniButton.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    miniButton.Visible = false
end)

closeBtn.MouseButton1Click:Connect(function()
    grabEnabled = false
    releaseBlock()
    if grabConnection then grabConnection:Disconnect() end
    screenGui:Destroy()
end)

player.CharacterAdded:Connect(function()
    grabEnabled = false
    releaseBlock()
    if grabConnection then
        grabConnection:Disconnect()
        grabConnection = nil
    end
end)

print("✅ V17 - VOCÊ VAI JUNTO COM O BLOCO!")
print("🔥 isStandingOnBlock() detecta quando você tá em cima")
print("🔥 blockDelta move você na mesma direção do bloco")
print("💪 AGORA SIM PERFEITO!")
