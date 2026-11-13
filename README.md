--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local movingPart = nil
local followDistance = 10
local followSpeed = 200
local useCamera = false
local lastPosition = nil

local gui, frame, nameLabel, collapseButton, decreaseButton, numberBox, increaseButton, cameraButton
local collapsed = false

local selectionBox = Instance.new("SelectionBox")
selectionBox.Color3 = Color3.fromRGB(0, 255, 0)
selectionBox.LineThickness = 0.08
selectionBox.SurfaceTransparency = 1
selectionBox.Adornee = nil
selectionBox.Parent = game:GetService("CoreGui") 
selectionBox.Name = "TelSelBox"

local function stopMoving()
    movingPart = nil
    selectionBox.Adornee = nil
end

local function movePartVelocity(part, targetPos, deltaTime)
    if not part or part.Anchored then return end
    local direction = (targetPos - part.Position)
    local velocity = direction * 15
    if velocity.Magnitude > followSpeed then
        velocity = velocity.Unit * followSpeed
    end
    part.AssemblyLinearVelocity = velocity
end

local function startMoving(part)
    movingPart = part
    lastPosition = part.Position
    selectionBox.Adornee = part
end

local function togglePartMove(part)
    if movingPart == part then
        stopMoving()
    elseif not part.Anchored then
        stopMoving()
        startMoving(part)
    end
end

local function createTool()
    if player.Backpack:FindFirstChild("TelTool") then return end
    local tool = Instance.new("Tool")
    tool.Name = "Telekinesis"
    tool.RequiresHandle = false
    tool.CanBeDropped = true
    tool.Parent = player.Backpack

    tool.Equipped:Connect(function(mouse)
        if mouse then
            mouse.Button1Down:Connect(function()
                local target = mouse.Target
                if target and target:IsA("BasePart") and not target.Anchored then
                    togglePartMove(target)
                end
            end)
        end

        local touchConnection
        touchConnection = UserInputService.TouchTapInWorld:Connect(function(pos, processed)
            if processed then return end
            local ray = camera:ScreenPointToRay(pos.X, pos.Y)
            local params = RaycastParams.new()
            params.FilterDescendantsInstances = {player.Character}
            params.FilterType = Enum.RaycastFilterType.Blacklist
            local result = workspace:Raycast(ray.Origin, ray.Direction * 1000, params)
            if result and result.Instance and result.Instance:IsA("BasePart") and not result.Instance.Anchored then
                togglePartMove(result.Instance)
            end
        end)

        tool.Unequipped:Connect(function()
            stopMoving()
            if touchConnection then
                touchConnection:Disconnect()
                touchConnection = nil
            end
        end)
    end)
end

RunService.Heartbeat:Connect(function(deltaTime)
    if movingPart then
        local char = player.Character
        local root = char and char:FindFirstChild("HumanoidRootPart")
        if not (root and movingPart and movingPart.Parent and not movingPart.Anchored) then
            stopMoving()
            return
        end

        local targetPosition
        if useCamera then
            targetPosition = camera.CFrame.Position + camera.CFrame.LookVector * followDistance
        else
            targetPosition = root.Position + root.CFrame.LookVector * followDistance
        end

        movePartVelocity(movingPart, targetPosition, deltaTime)

        if (movingPart.Position - lastPosition).Magnitude > 100 then
            stopMoving()
            return
        end
        lastPosition = movingPart.Position
    end
end)

local function createGUI()
    if gui then gui:Destroy() end

    gui = Instance.new("ScreenGui")
    gui.Name = "TelGUI"
    gui.ResetOnSpawn = false
    gui.DisplayOrder = 50000
    gui.ZIndexBehavior = Enum.ZIndexBehavior.Global
    gui.Parent = game:GetService("CoreGui") 
    

    frame = Instance.new("Frame")
    frame.Size = UDim2.new(0.24, 0, 0.3, 0)
    frame.Position = UDim2.new(0.02, 0, 0.1, 0)
    frame.BackgroundColor3 = Color3.fromRGB(127, 127, 127)
    frame.BackgroundTransparency = 0.05
    frame.BorderColor3 = Color3.fromRGB(27, 42, 53)
    frame.BorderSizePixel = 1
    frame.Active = true
    frame.Draggable = true
    frame.Parent = gui

    nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.2, 0)
    nameLabel.Position = UDim2.new(0, 0, 0, 0)
    nameLabel.BackgroundColor3 = Color3.fromRGB(162, 162, 162)
    nameLabel.Text = "Telekinesis By Ano"
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.Legacy
    nameLabel.TextColor3 = Color3.fromRGB(27, 42, 53)
    nameLabel.Parent = frame

    collapseButton = Instance.new("TextButton")
    collapseButton.Size = UDim2.new(0.12, 0, 1, 0)
    collapseButton.Position = UDim2.new(0.88, 0, 0, 0)
    collapseButton.Text = "V"
    collapseButton.TextScaled = true
    collapseButton.Font = Enum.Font.Legacy
    collapseButton.TextColor3 = Color3.fromRGB(27, 42, 53)
    collapseButton.BackgroundColor3 = Color3.fromRGB(150, 150, 150)
    collapseButton.Parent = nameLabel

    collapseButton.MouseButton1Click:Connect(function()
        collapsed = not collapsed
        for _, child in ipairs(frame:GetChildren()) do
            if child ~= nameLabel then
                child.Visible = not collapsed
            end
        end
        if collapsed then
            frame.Size = UDim2.new(0.24, 0, 0.06, 0)
            nameLabel.Size = UDim2.new(1, 0, 1, 0)
            frame.BackgroundTransparency = 1
            collapseButton.Text = "<"
        else
            frame.Size = UDim2.new(0.24, 0, 0.3, 0)
            nameLabel.Size = UDim2.new(1, 0, 0.2, 0)
            frame.BackgroundTransparency = 0.05
            collapseButton.Text = "V"
        end
    end)

    decreaseButton = Instance.new("TextButton")
    decreaseButton.Size = UDim2.new(0.18, 0, 0.3, 0)
    decreaseButton.Position = UDim2.new(0.08, 0, 0.25, 0)
    decreaseButton.Text = "-"
    decreaseButton.TextScaled = true
    decreaseButton.Font = Enum.Font.Legacy
    decreaseButton.TextColor3 = Color3.fromRGB(27, 42, 53)
    decreaseButton.BackgroundColor3 = Color3.fromRGB(160, 160, 160)
    decreaseButton.Parent = frame

    numberBox = Instance.new("TextBox")
    numberBox.Size = UDim2.new(0.45, 0, 0.3, 0)
    numberBox.Position = UDim2.new(0.28, 0, 0.25, 0)
    numberBox.Text = tostring(followDistance)
    numberBox.Font = Enum.Font.Legacy
    numberBox.TextColor3 = Color3.fromRGB(27, 42, 53)
    numberBox.BackgroundColor3 = Color3.fromRGB(159, 159, 159)
    numberBox.ClearTextOnFocus = true
    numberBox.Parent = frame

    increaseButton = Instance.new("TextButton")
    increaseButton.Size = UDim2.new(0.18, 0, 0.3, 0)
    increaseButton.Position = UDim2.new(0.75, 0, 0.25, 0)
    increaseButton.Text = "+"
    increaseButton.TextScaled = true
    increaseButton.Font = Enum.Font.Legacy
    increaseButton.TextColor3 = Color3.fromRGB(27, 42, 53)
    increaseButton.BackgroundColor3 = Color3.fromRGB(160, 160, 160)
    increaseButton.Parent = frame

    cameraButton = Instance.new("TextButton")
    cameraButton.Size = UDim2.new(0.85, 0, 0.3, 0)
    cameraButton.Position = UDim2.new(0.08, 0, 0.6, 0)
    cameraButton.Text = "Camera Follow: OFF"
    cameraButton.TextScaled = true
    cameraButton.Font = Enum.Font.Legacy
    cameraButton.TextColor3 = Color3.fromRGB(27, 42, 53)
    cameraButton.BackgroundColor3 = Color3.fromRGB(160, 160, 160)
    cameraButton.Parent = frame

    local function updateNumberBox()
        numberBox.Text = tostring(followDistance)
    end

    decreaseButton.MouseButton1Click:Connect(function()
        followDistance = math.clamp(followDistance - 1, 1, 500000)
        updateNumberBox()
    end)

    increaseButton.MouseButton1Click:Connect(function()
        followDistance = math.clamp(followDistance + 1, 1, 500000)
        updateNumberBox()
    end)

    numberBox.FocusLost:Connect(function()
        local num = tonumber(numberBox.Text)
        if num then
            followDistance = math.clamp(num, 0.001, 500000)
        end
        updateNumberBox()
    end)

    cameraButton.MouseButton1Click:Connect(function()
        useCamera = not useCamera
        cameraButton.Text = "Camera Follow: " .. (useCamera and "ON" or "OFF")
    end)
end

createTool()
createGUI()
player.CharacterAdded:Connect(function()
    createTool()
    createGUI()
end)
