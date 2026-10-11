--========================================================
-- WATERBALLOON + GUI ON/OFF
--========================================================

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

--========================================================
-- CONFIGURAÇÕES
--========================================================

local WATERBALLOON_IMAGE_ID = "rbxassetid://89108586792149"
local THROW_ANIMATION_ID = "rbxassetid://18324269091"

local COOLDOWN_TIME = 2.5
local HIDE_DELAY = 0.6

local canThrowBalloon = true
local waterballoonConnections = {}
local waterballoonThread

local enabled = false

--========================================================
-- REMOVER GUI ANTIGO
--========================================================

local oldGui = playerGui:FindFirstChild("WaterBalloonGUI")

if oldGui then
    oldGui:Destroy()
end

--========================================================
-- GUI
--========================================================

local gui = Instance.new("ScreenGui")
gui.Name = "WaterBalloonGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

local main = Instance.new("Frame")
main.Size = UDim2.fromOffset(180, 65)
main.Position = UDim2.new(0.5, -90, 0.2, 0)
main.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
main.BorderSizePixel = 0
main.Active = true
main.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = main

local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(255, 255, 255)
stroke.Thickness = 1
stroke.Transparency = 0.75
stroke.Parent = main

--========================================================
-- TÍTULO
--========================================================

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 25)
title.Position = UDim2.fromOffset(0, 3)
title.BackgroundTransparency = 1
title.Text = "WATERBALLOON"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Center
title.Parent = main

--========================================================
-- BOTÃO ON/OFF
--========================================================

local toggle = Instance.new("TextButton")
toggle.Size = UDim2.fromOffset(145, 27)
toggle.Position = UDim2.new(0.5, -72, 0, 32)
toggle.BackgroundColor3 = Color3.fromRGB(80, 30, 30)
toggle.BorderSizePixel = 0
toggle.AutoButtonColor = false
toggle.Text = "OFF"
toggle.TextColor3 = Color3.fromRGB(255, 220, 220)
toggle.Font = Enum.Font.GothamBold
toggle.TextSize = 12
toggle.Parent = main

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 7)
toggleCorner.Parent = toggle

--========================================================
-- ATUALIZAR BOTÃO
--========================================================

local function updateButton()
    if enabled then
        toggle.Text = "ON"
        
        TweenService:Create(
            toggle,
            TweenInfo.new(0.15),
            {
                BackgroundColor3 = Color3.fromRGB(25, 130, 55)
            }
        ):Play()
        
        toggle.TextColor3 = Color3.fromRGB(220, 255, 225)
    else
        toggle.Text = "OFF"
        
        TweenService:Create(
            toggle,
            TweenInfo.new(0.15),
            {
                BackgroundColor3 = Color3.fromRGB(80, 30, 30)
            }
        ):Play()
        
        toggle.TextColor3 = Color3.fromRGB(255, 220, 220)
    end
end

--========================================================
-- CRIAR WATERBALLOON
--========================================================

local function createWaterballoonTool()

    if not enabled then
        return
    end

    local backpack = player:WaitForChild("Backpack", 5)

    if not backpack then
        return
    end

    -- Evita duplicar
    if backpack:FindFirstChild("waterballoon")
        or (player.Character and player.Character:FindFirstChild("waterballoon")) then
        return
    end

    local tool = Instance.new("Tool")

    tool.Name = "waterballoon"
    tool.RequiresHandle = true
    tool.CanBeDropped = false
    tool.TextureId = WATERBALLOON_IMAGE_ID

    tool.Grip =
        CFrame.new(0.2, 0, 0)
        * CFrame.Angles(0, 0, math.rad(90))

    --====================================================
    -- HANDLE
    --====================================================

    local handle = Instance.new("Part")

    handle.Name = "Handle"
    handle.Size = Vector3.new(1.2, 1.2, 1.2)
    handle.CanCollide = false
    handle.CanTouch = false
    handle.CanQuery = false
    handle.Massless = true
    handle.Color = Color3.fromRGB(235, 64, 88)
    handle.Material = Enum.Material.SmoothPlastic
    handle.Parent = tool

    --====================================================
    -- MESH
    --====================================================

    local mesh = Instance.new("SpecialMesh")

    mesh.MeshId = "rbxassetid://137510221029430"
    mesh.Scale = Vector3.new(0.45, 0.45, 0.45)
    mesh.Parent = handle

    --====================================================
    -- ANIMAÇÃO
    --====================================================

    local throwTrack

    local function loadAnimation()

        local character = player.Character

        if not character then
            return
        end

        local humanoid = character:FindFirstChildOfClass("Humanoid")

        if not humanoid then
            return
        end

        local animator =
            humanoid:FindFirstChildOfClass("Animator")
            or Instance.new("Animator", humanoid)

        local animation = Instance.new("Animation")
        animation.AnimationId = THROW_ANIMATION_ID

        pcall(function()
            throwTrack = animator:LoadAnimation(animation)
        end)
    end

    loadAnimation()

    --====================================================
    -- ARREMESSAR
    --====================================================

    tool.Activated:Connect(function()

        if not enabled then
            return
        end

        if not canThrowBalloon then
            return
        end

        if not tool.Parent then
            return
        end

        canThrowBalloon = false

        if not throwTrack then
            loadAnimation()
        end

        if throwTrack then
            pcall(function()
                throwTrack:Play()
            end)
        end

        task.wait(HIDE_DELAY)

        if handle and handle.Parent then
            handle.Transparency = 1
        end

        task.wait(
            math.max(
                0,
                COOLDOWN_TIME - HIDE_DELAY
            )
        )

        if handle and handle.Parent then
            handle.Transparency = 0
        end

        canThrowBalloon = true
    end)

    tool.Parent = backpack
end

--========================================================
-- REMOVER WATERBALLOON
--========================================================

local function removeWaterballoonTool()

    local backpack = player:FindFirstChild("Backpack")

    if backpack then

        local item = backpack:FindFirstChild("waterballoon")

        if item then
            item:Destroy()
        end
    end

    if player.Character then

        local item =
            player.Character:FindFirstChild("waterballoon")

        if item then
            item:Destroy()
        end
    end
end

--========================================================
-- WATERBALLOON ON
--========================================================

local function waterballoonOn()

    enabled = true
    canThrowBalloon = true

    createWaterballoonTool()

    -- Recria depois do respawn
    table.insert(
        waterballoonConnections,
        player.CharacterAdded:Connect(function()

            task.wait(1)

            if not enabled then
                return
            end

            canThrowBalloon = true

            createWaterballoonTool()
        end)
    )

    -- Verifica se a Tool foi removida
    waterballoonThread = task.spawn(function()

        while enabled do

            task.wait(2)

            if not enabled then
                break
            end

            local backpack = player:FindFirstChild("Backpack")

            if backpack then

                local hasTool =
                    backpack:FindFirstChild("waterballoon")
                    or (
                        player.Character
                        and player.Character:FindFirstChild("waterballoon")
                    )

                if not hasTool then
                    createWaterballoonTool()
                end
            end
        end
    end)

    print("✅ WATERBALLOON ON")
end

--========================================================
-- WATERBALLOON OFF
--========================================================

local function waterballoonOff()

    enabled = false
    canThrowBalloon = true

    for _, connection in ipairs(waterballoonConnections) do
        pcall(function()
            connection:Disconnect()
        end)
    end

    waterballoonConnections = {}

    if waterballoonThread then

        task.cancel(waterballoonThread)

        waterballoonThread = nil
    end

    removeWaterballoonTool()

    print("❌ WATERBALLOON OFF")
end

--========================================================
-- BOTÃO ON/OFF
--========================================================

toggle.MouseButton1Click:Connect(function()

    enabled = not enabled

    updateButton()

    if enabled then
        task.spawn(waterballoonOn)
    else
        task.spawn(waterballoonOff)
    end
end)

--========================================================
-- ARRASTAR NO MOBILE E PC
--========================================================

local UserInputService = game:GetService("UserInputService")

local dragging = false
local dragStart
local startPosition
local dragInput

local function updateDrag(input)

    local delta = input.Position - dragStart

    main.Position = UDim2.new(
        startPosition.X.Scale,
        startPosition.X.Offset + delta.X,
        startPosition.Y.Scale,
        startPosition.Y.Offset + delta.Y
    )
end

main.InputBegan:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPosition = main.Position

        input.Changed:Connect(function()

            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

main.InputChanged:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then

        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)

    if input == dragInput and dragging then
        updateDrag(input)
    end
end)

--========================================================
-- INICIAL
--========================================================

updateButton()

print("💧 WATERBALLOON GUI carregado!")
