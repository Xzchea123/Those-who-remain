--[[ 
  Roblox Lua UI script to mimic the "rexcore" UI with:
    - "Main" tab: Silent Aim toggle + hitbox slider (0-100, text smaller), Silent Aim transparency slider (0-1)
    - "Visual" tab: ESP (Items & Players), Fullbright, No Fog toggles, FOV toggle + slider (70-120, looped)
    - Player ESP draws boxes and healthbars on players, with GUI-adorned healthbars above their heads.
    - "Player" tab: CFrame WalkSpeed slider (16-30, looped CFrame movement)
    - "Settings" tab: keybind help/info
    - Tabs are clickable, UI is draggable from the top bar, and RightShift/Quote toggles are supported.
--]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

local gui = Instance.new("ScreenGui")
gui.Name = "Avoid's Those Who remain hub"
gui.Parent = player:WaitForChild("PlayerGui")

-- Main Container
local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 670, 0, 420)
MainFrame.Position = UDim2.new(0.5, -335, 0.5, -210)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = gui
MainFrame.ClipsDescendants = true
MainFrame.AnchorPoint = Vector2.new(0.5,0.5)
MainFrame.Active = true

local mainCorner = Instance.new("UICorner", MainFrame)
mainCorner.CornerRadius = UDim.new(0,14)

-- Title Bar (DRAGGABLE AREA)
local TitleBar = Instance.new("TextLabel")
TitleBar.Parent = MainFrame
TitleBar.Size = UDim2.new(1,0,0,34)
TitleBar.BackgroundTransparency = 1
TitleBar.Font = Enum.Font.GothamBold
TitleBar.TextSize = 18
TitleBar.TextColor3 = Color3.fromRGB(246, 190, 255)
TitleBar.TextStrokeTransparency = 0.7
TitleBar.Text = "REXCORE"
TitleBar.Position = UDim2.new(0,0,0,2)
TitleBar.Active = true

do
    local dragging = false
    local dragStart, startPos

    TitleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position
        end
    end)

    TitleBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            MainFrame.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)
end

local navSections = {
    {icon="💥", label="Main", y=25},
    {icon="️👀", label="Visual", y=60},
    {icon="🧍", label="Player", y=95},
    {icon="⚙️", label="Settings", y=385}
}

local NavBar = Instance.new("Frame")
NavBar.Parent = MainFrame
NavBar.Size = UDim2.new(0,120,1,0)
NavBar.Position = UDim2.new(0,0,0,0)
NavBar.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
NavBar.BorderSizePixel = 0
local navCorner = Instance.new("UICorner", NavBar)
navCorner.CornerRadius = UDim.new(0,14)

local tabContentFrames = {}
local tabButtons = {}

local Content = Instance.new("Frame")
Content.Parent = MainFrame
Content.Position = UDim2.new(0,130,0,40)
Content.Size = UDim2.new(1,-140,1,-50)
Content.BackgroundTransparency = 1

local function selectTab(tabIdx)
    for i,frame in ipairs(tabContentFrames) do
        frame.Visible = (i == tabIdx)
        if tabButtons[i] then
            if i == tabIdx then
                tabButtons[i].BackgroundTransparency = 0
                tabButtons[i].BackgroundColor3 = Color3.fromRGB(30,30,30)
                tabButtons[i].TextColor3 = Color3.fromRGB(255,255,255)
            else
                tabButtons[i].BackgroundTransparency = 1
                tabButtons[i].BackgroundColor3 = Color3.fromRGB(15,15,15)
                tabButtons[i].TextColor3 = Color3.fromRGB(210,210,210)
            end
        end
    end
end

local function makeButton(parent, text, y)
    local button = Instance.new("TextButton")
    button.Parent = parent
    button.Size = UDim2.new(0.8, 0, 0, 36)
    button.Position = UDim2.new(0.1, 0, 0, y)
    button.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 18
    button.Font = Enum.Font.Gotham
    button.Text = text
    local uic = Instance.new("UICorner", button)
    uic.CornerRadius = UDim.new(0, 8)
    return button
end

for idx,sec in ipairs(navSections) do
    local btn = Instance.new("TextButton")
    btn.Parent = NavBar
    btn.BackgroundTransparency = (idx == 1) and 0 or 1
    btn.BackgroundColor3 = (idx == 1) and Color3.fromRGB(30,30,30) or Color3.fromRGB(15,15,15)
    btn.Position = UDim2.new(0,0,0,sec.y)
    btn.Size = UDim2.new(1,0,0,35)
    btn.TextColor3 = (idx == 1) and Color3.fromRGB(255,255,255) or Color3.fromRGB(210,210,210)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 16
    btn.TextXAlignment = Enum.TextXAlignment.Left
    btn.Text = ("  %s  %s"):format(sec.icon, sec.label)
    local c = Instance.new("UICorner", btn)
    c.CornerRadius = UDim.new(0,8)
    btn.AutoButtonColor = true
    tabButtons[idx] = btn

    local frame = Instance.new("Frame")
    frame.Parent = Content
    frame.Size = UDim2.new(1,0,1,0)
    frame.Position = UDim2.new(0,0,0,0)
    frame.BackgroundTransparency = 1
    frame.Visible = (idx == 1)
    tabContentFrames[idx] = frame

    -- MAIN TAB
    if sec.label == "Main" then
        local uis = UserInputService
        local silentBtn = Instance.new("TextButton")
        silentBtn.Size = UDim2.new(0.8, 0, 0, 40)
        silentBtn.Position = UDim2.new(0.1, 0, 0, 28)
        silentBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        silentBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        silentBtn.TextSize = 14
        silentBtn.Font = Enum.Font.Gotham
        silentBtn.Text = "Silent Aim: OFF"
        silentBtn.Parent = frame
        local uic1 = Instance.new("UICorner", silentBtn)
        uic1.CornerRadius = UDim.new(0, 8)

        local sliderBG = Instance.new("Frame")
        sliderBG.Size = UDim2.new(0.8, 0, 0, 16)
        sliderBG.Position = UDim2.new(0.1, 0, 0, 78)
        sliderBG.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        sliderBG.Parent = frame
        local uic2 = Instance.new("UICorner", sliderBG)
        uic2.CornerRadius = UDim.new(0, 8)
        local sliderFill = Instance.new("Frame")
        sliderFill.Size = UDim2.new(0, 0, 1, 0)
        sliderFill.Position = UDim2.new(0, 0, 0, 0)
        sliderFill.BackgroundColor3 = Color3.fromRGB(150, 0, 255)
        sliderFill.Parent = sliderBG
        sliderFill.ZIndex = 2
        local uic3 = Instance.new("UICorner", sliderFill)
        uic3.CornerRadius = UDim.new(0, 8)
        local knob = Instance.new("Frame")
        knob.Size = UDim2.new(0, 18, 0, 18)
        knob.Position = UDim2.new(0, 0, 0.5, -9)
        knob.BackgroundColor3 = Color3.fromRGB(255,255,255)
        knob.Parent = sliderBG
        knob.ZIndex = 3
        local uic4 = Instance.new("UICorner", knob)
        uic4.CornerRadius = UDim.new(1, 0)

        local hitboxLabel = Instance.new("TextLabel")
        hitboxLabel.Size = UDim2.new(0, 70, 0, 20)
        hitboxLabel.Position = UDim2.new(0.8, 12, 0, 70)
        hitboxLabel.BackgroundTransparency = 1
        hitboxLabel.TextColor3 = Color3.fromRGB(200, 200, 255)
        hitboxLabel.Font = Enum.Font.Gotham
        hitboxLabel.TextSize = 13
        hitboxLabel.Text = "Hitbox: 10"
        hitboxLabel.Parent = frame

        -- Transparency slider for Silent Aim
        local transLabel = Instance.new("TextLabel")
        transLabel.Parent = frame
        transLabel.Size = UDim2.new(0, 120, 0, 16)
        transLabel.Position = UDim2.new(0.1, 0, 0, 112)
        transLabel.BackgroundTransparency = 1
        transLabel.Font = Enum.Font.Gotham
        transLabel.TextSize = 13
        transLabel.TextColor3 = Color3.fromRGB(200,200,255)
        transLabel.Text = "Hitbox Transparency"

        local transSliderBG = Instance.new("Frame")
        transSliderBG.Size = UDim2.new(0.8, 0, 0, 14)
        transSliderBG.Position = UDim2.new(0.1, 0, 0, 132)
        transSliderBG.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        transSliderBG.Parent = frame
        local transUIC = Instance.new("UICorner", transSliderBG)
        transUIC.CornerRadius = UDim.new(0, 8)
        local transSliderFill = Instance.new("Frame")
        transSliderFill.Size = UDim2.new(0, 0, 1, 0)
        transSliderFill.Position = UDim2.new(0, 0, 0, 0)
        transSliderFill.BackgroundColor3 = Color3.fromRGB(150, 0, 255)
        transSliderFill.Parent = transSliderBG
        transSliderFill.ZIndex = 2
        local transFillUIC = Instance.new("UICorner", transSliderFill)
        transFillUIC.CornerRadius = UDim.new(0, 8)
        local transKnob = Instance.new("Frame")
        transKnob.Size = UDim2.new(0, 14, 0, 14)
        transKnob.Position = UDim2.new(0, 0, 0.5, -7)
        transKnob.BackgroundColor3 = Color3.fromRGB(255,255,255)
        transKnob.Parent = transSliderBG
        transKnob.ZIndex = 3
        local transKnobUIC = Instance.new("UICorner", transKnob)
        transKnobUIC.CornerRadius = UDim.new(1, 0)
        local transValueLabel = Instance.new("TextLabel")
        transValueLabel.Size = UDim2.new(0, 70, 0, 16)
        transValueLabel.Position = UDim2.new(0.8, 12, 0, 128)
        transValueLabel.BackgroundTransparency = 1
        transValueLabel.TextColor3 = Color3.fromRGB(200, 200, 255)
        transValueLabel.Font = Enum.Font.Gotham
        transValueLabel.TextSize = 13
        transValueLabel.Text = "0.40"
        transValueLabel.Parent = frame

        local hitboxSize = 10
        local hitboxTrans = 0.4
        local dragging = false
        local transDragging = false

        local function updateSlider(val)
            val = math.clamp(val, 0, 100)
            hitboxSize = math.floor(val)
            sliderFill.Size = UDim2.new(val/100, 0, 1, 0)
            local sliderWidth = sliderBG.AbsoluteSize.X
            local knobWidth = knob.AbsoluteSize.X
            local knobPos = (val/100) * (sliderWidth - knobWidth)
            knob.Position = UDim2.new(0, knobPos, 0.5, -knobWidth/2)
            hitboxLabel.Text = "Hitbox: " .. tostring(hitboxSize)
        end

        local function getSliderValueFromMouse()
            local abs = sliderBG.AbsolutePosition.X
            local mouse = uis:GetMouseLocation().X
            local sliderWidth = sliderBG.AbsoluteSize.X
            local knobWidth = knob.AbsoluteSize.X
            local pct = math.clamp((mouse - abs - knobWidth/2) / (sliderWidth - knobWidth), 0, 1)
            return pct * 100
        end

        sliderBG.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = true
                MainFrame.Draggable = false
                updateSlider(getSliderValueFromMouse())
            end
        end)
        sliderBG.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                dragging = false
                MainFrame.Draggable = true
            end
        end)
        uis.InputChanged:Connect(function(input)
            if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                updateSlider(getSliderValueFromMouse())
            end
        end)

        local function updateTransSlider(val)
            val = math.clamp(val, 0, 1)
            hitboxTrans = math.floor(val*100)/100
            transSliderFill.Size = UDim2.new(val, 0, 1, 0)
            local sliderWidth = transSliderBG.AbsoluteSize.X
            local knobWidth = transKnob.AbsoluteSize.X
            local knobPos = val * (sliderWidth - knobWidth)
            transKnob.Position = UDim2.new(0, knobPos, 0.5, -knobWidth/2)
            transValueLabel.Text = string.format("%.2f", hitboxTrans)
        end

        local function getTransSliderValueFromMouse()
            local abs = transSliderBG.AbsolutePosition.X
            local mouse = uis:GetMouseLocation().X
            local sliderWidth = transSliderBG.AbsoluteSize.X
            local knobWidth = transKnob.AbsoluteSize.X
            local pct = math.clamp((mouse - abs - knobWidth/2) / (sliderWidth - knobWidth), 0, 1)
            return pct
        end

        transSliderBG.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                transDragging = true
                MainFrame.Draggable = false
                updateTransSlider(getTransSliderValueFromMouse())
            end
        end)
        transSliderBG.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                transDragging = false
                MainFrame.Draggable = true
            end
        end)
        uis.InputChanged:Connect(function(input)
            if transDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                updateTransSlider(getTransSliderValueFromMouse())
            end
        end)

        updateSlider(hitboxSize)
        updateTransSlider(hitboxTrans)

        -- Silent Aim logic (with slider and transparency)
        local silentEnabled = false
        local silentLoop
        silentBtn.MouseButton1Click:Connect(function()
            silentEnabled = not silentEnabled
            silentBtn.Text = "Silent Aim: " .. (silentEnabled and "ON" or "OFF")
            if silentLoop then
                silentLoop:Disconnect()
                silentLoop = nil
            end
            if silentEnabled then
                local infectedFolder = workspace:FindFirstChild("Entities") and workspace.Entities:FindFirstChild("Infected")
                if not infectedFolder then return end
                silentLoop = RunService.RenderStepped:Connect(function()
                    for _, entity in pairs(infectedFolder:GetChildren()) do
                        if entity:IsA("Model") and entity:FindFirstChild("Head") then
                            local head = entity.Head
                            pcall(function()
                                head.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
                                head.CanCollide = false
                                head.Material = Enum.Material.ForceField
                                head.BrickColor = BrickColor.new("Really red")
                                head.Transparency = hitboxTrans
                                head.Massless = true
                            end)
                        end
                    end
                end)
            end
        end)

    -- VISUAL TAB
    elseif sec.label == "Visual" then
        local visBox = Instance.new("Frame")
        visBox.Parent = frame
        visBox.Size = UDim2.new(0.5,-15,1,-15)
        visBox.Position = UDim2.new(0,0,0,0)
        visBox.BackgroundColor3 = Color3.fromRGB(29,29,29)
        visBox.BorderSizePixel = 0
        local corner = Instance.new("UICorner", visBox)
        corner.CornerRadius = UDim.new(0,12)
        local visLabel = Instance.new("TextLabel")
        visLabel.Parent = visBox
        visLabel.Size = UDim2.new(1,0,0,30)
        visLabel.Position = UDim2.new(0,0,0,0)
        visLabel.BackgroundTransparency = 1
        visLabel.Font = Enum.Font.GothamMedium
        visLabel.Text = "Visuals"
        visLabel.TextColor3 = Color3.fromRGB(200,200,200)
        visLabel.TextSize = 16
        visLabel.TextXAlignment = Enum.TextXAlignment.Left

        -- ESP: Items
        local espBtn = makeButton(visBox, "ESP: OFF", 18)
        local espEnabled = false
        local espCon
        local espTargets = {
            Ammo = Color3.fromRGB(255, 255, 255),
            Medkit = Color3.fromRGB(255, 0, 0),
            ["Body Armor"] = Color3.fromRGB(0, 255, 255),
            ["Gas Mask"] = Color3.fromRGB(150, 255, 150)
        }
        local function clearESP()
            local itemsHolder = workspace:FindFirstChild("Ignore") and workspace.Ignore:FindFirstChild("Items")
            if itemsHolder then
                for _, it in ipairs(itemsHolder:GetChildren()) do
                    local h = it:FindFirstChildWhichIsA("Highlight")
                    if h then h:Destroy() end
                end
            end
            if espCon then espCon:Disconnect() end
            espCon = nil
        end
        local function updateESP()
            local itemsHolder = workspace:WaitForChild("Ignore"):WaitForChild("Items")
            for _, it in ipairs(itemsHolder:GetChildren()) do
                if espTargets[it.Name] and not it:FindFirstChildOfClass("Highlight") then
                    local hl = Instance.new("Highlight")
                    hl.FillColor = espTargets[it.Name]
                    hl.OutlineColor = espTargets[it.Name]
                    hl.FillTransparency = 0.2
                    hl.OutlineTransparency = 0.5
                    hl.Adornee = it
                    hl.Parent = it
                end
            end
            espCon = itemsHolder.ChildAdded:Connect(function(it)
                task.wait(0.2)
                if espTargets[it.Name] then
                    local hl = Instance.new("Highlight")
                    hl.FillColor = espTargets[it.Name]
                    hl.OutlineColor = espTargets[it.Name]
                    hl.FillTransparency = 0.2
                    hl.OutlineTransparency = 0.5
                    hl.Adornee = it
                    hl.Parent = it
                end
            end)
        end
        espBtn.MouseButton1Click:Connect(function()
            espEnabled = not espEnabled
            espBtn.Text = "ESP: " .. (espEnabled and "ON" or "OFF")
            clearESP()
            if espEnabled then updateESP() end
        end)

        -- ESP: Players + Healthbars
        local espPlayersBtn = makeButton(visBox, "ESP Players: OFF", 53)
        local espPlayersEnabled = false
        local playerESPLoop = nil

        -- Helper functions for player ESP
        local function removePlayerESP()
            if playerESPLoop then
                playerESPLoop:Disconnect()
                playerESPLoop = nil
            end
            -- Remove all adorns and billboards
            for _, plr in ipairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character then
                    local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local adorn = hrp:FindFirstChild("RexcoreESPBox")
                        if adorn then adorn:Destroy() end
                    end
                    local head = plr.Character:FindFirstChild("Head")
                    if head then
                        local bb = head:FindFirstChild("RexcoreHealthbar")
                        if bb then bb:Destroy() end
                    end
                end
            end
        end

        local function addPlayerESP()
            local camera = workspace.CurrentCamera
            playerESPLoop = RunService.RenderStepped:Connect(function()
                for _, plr in ipairs(Players:GetPlayers()) do
                    if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Head") and plr.Character:FindFirstChildOfClass("Humanoid") then
                        local hrp = plr.Character.HumanoidRootPart
                        local hum = plr.Character:FindFirstChildOfClass("Humanoid")
                        local head = plr.Character.Head

                        -- Draw Box (BoxHandleAdornment)
                        local adorn = hrp:FindFirstChild("RexcoreESPBox")
                        if not adorn then
                            adorn = Instance.new("BoxHandleAdornment")
                            adorn.Name = "RexcoreESPBox"
                            adorn.Adornee = hrp
                            adorn.AlwaysOnTop = true
                            adorn.ZIndex = 10
                            adorn.Size = Vector3.new(2,3,1)
                            adorn.Color3 = Color3.fromRGB(255, 255, 0)
                            adorn.Transparency = 0.7
                            adorn.Parent = hrp
                        end

                        -- Draw Healthbar (BillboardGui)
                        local bb = head:FindFirstChild("RexcoreHealthbar")
                        if not bb then
                            bb = Instance.new("BillboardGui")
                            bb.Name = "RexcoreHealthbar"
                            bb.Adornee = head
                            bb.Size = UDim2.new(4,0,0.5,0)
                            bb.StudsOffset = Vector3.new(0,1.7,0)
                            bb.AlwaysOnTop = true
                            bb.Parent = head

                            local bg = Instance.new("Frame")
                            bg.Name = "BG"
                            bg.Size = UDim2.new(1,0,0.4,0)
                            bg.Position = UDim2.new(0,0,0.3,0)
                            bg.BackgroundColor3 = Color3.fromRGB(40,40,40)
                            bg.BorderSizePixel = 0
                            bg.Parent = bb

                            local fill = Instance.new("Frame")
                            fill.Name = "Fill"
                            fill.Size = UDim2.new(1,0,1,0)
                            fill.Position = UDim2.new(0,0,0,0)
                            fill.BackgroundColor3 = Color3.fromRGB(0,255,0)
                            fill.BorderSizePixel = 0
                            fill.Parent = bg

                            local name = Instance.new("TextLabel")
                            name.Name = "Name"
                            name.BackgroundTransparency = 1
                            name.Size = UDim2.new(1,0,0.6,0)
                            name.Position = UDim2.new(0,0,0,0)
                            name.Text = plr.Name
                            name.TextColor3 = Color3.fromRGB(255,255,255)
                            name.TextStrokeTransparency = 0.5
                            name.Font = Enum.Font.GothamSemibold
                            name.TextScaled = true
                            name.Parent = bb
                        end

                        -- Update healthbar fill and color
                        local bg = bb.BG
                        local fill = bg.Fill
                        fill.Size = UDim2.new(math.clamp(hum.Health/hum.MaxHealth,0,1),0,1,0)
                        if hum.Health/hum.MaxHealth > 0.5 then
                            fill.BackgroundColor3 = Color3.fromRGB(0,255,0)
                        elseif hum.Health/hum.MaxHealth > 0.25 then
                            fill.BackgroundColor3 = Color3.fromRGB(255,255,0)
                        else
                            fill.BackgroundColor3 = Color3.fromRGB(255,0,0)
                        end
                        bb.Name.Text = plr.Name
                    end
                end
            end)
        end

        espPlayersBtn.MouseButton1Click:Connect(function()
            espPlayersEnabled = not espPlayersEnabled
            espPlayersBtn.Text = "ESP Players: " .. (espPlayersEnabled and "ON" or "OFF")
            removePlayerESP()
            if espPlayersEnabled then addPlayerESP() end
        end)

        -- Remove player ESP on tab hide
        frame:GetPropertyChangedSignal("Visible"):Connect(function()
            if not frame.Visible then
                removePlayerESP()
            end
        end)

        -- Fullbright
        local fullbrightBtn = makeButton(visBox, "Fullbright: OFF", 88)
        local fullbrightEnabled = false
        local fullbrightLoop
        fullbrightBtn.MouseButton1Click:Connect(function()
            fullbrightEnabled = not fullbrightEnabled
            fullbrightBtn.Text = "Fullbright: " .. (fullbrightEnabled and "ON" or "OFF")
            if fullbrightLoop then
                fullbrightLoop:Disconnect()
                fullbrightLoop = nil
            end
            if fullbrightEnabled then
                local Lighting = game:GetService("Lighting")
                fullbrightLoop = RunService.RenderStepped:Connect(function()
                    Lighting.Brightness = 2
                    Lighting.ClockTime = 12
                    for _, v in pairs(Lighting:GetChildren()) do
                        if v:IsA("Atmosphere") then
                            v:Destroy()
                        end
                    end
                end)
            end
        end)

        -- No Fog
        local noFogBtn = makeButton(visBox, "No Fog: OFF", 158)
        local noFogEnabled = false
        local noFogLoop
        noFogBtn.MouseButton1Click:Connect(function()
            noFogEnabled = not noFogEnabled
            noFogBtn.Text = "No Fog: " .. (noFogEnabled and "ON" or "OFF")
            if noFogLoop then
                noFogLoop:Disconnect()
                noFogLoop = nil
            end
            if noFogEnabled then
                local Lighting = game:GetService("Lighting")
                noFogLoop = RunService.RenderStepped:Connect(function()
                    Lighting.FogEnd = 100000
                    Lighting.GlobalShadows = false
                end)
            end
        end)

        -- Camera FOV Toggle and Slider
        local fovToggleBtn = makeButton(visBox, "FOV: OFF", 228)
        fovToggleBtn.TextSize = 14
        local fovSliderBG = Instance.new("Frame")
        fovSliderBG.Size = UDim2.new(0.8, 0, 0, 18)
        fovSliderBG.Position = UDim2.new(0.1, 0, 0, 268)
        fovSliderBG.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        fovSliderBG.Parent = visBox
        local fovUIC = Instance.new("UICorner", fovSliderBG)
        fovUIC.CornerRadius = UDim.new(0, 8)
        local fovSliderFill = Instance.new("Frame")
        fovSliderFill.Size = UDim2.new(0, 0, 1, 0)
        fovSliderFill.Position = UDim2.new(0, 0, 0, 0)
        fovSliderFill.BackgroundColor3 = Color3.fromRGB(100, 180, 255)
        fovSliderFill.Parent = fovSliderBG
        fovSliderFill.ZIndex = 2
        local fovFillUIC = Instance.new("UICorner", fovSliderFill)
        fovFillUIC.CornerRadius = UDim.new(0, 8)
        local fovKnob = Instance.new("Frame")
        fovKnob.Size = UDim2.new(0, 18, 0, 18)
        fovKnob.Position = UDim2.new(0, 0, 0.5, -9)
        fovKnob.BackgroundColor3 = Color3.fromRGB(255,255,255)
        fovKnob.Parent = fovSliderBG
        fovKnob.ZIndex = 3
        local fovKnobUIC = Instance.new("UICorner", fovKnob)
        fovKnobUIC.CornerRadius = UDim.new(1, 0)
        local fovValueLabel = Instance.new("TextLabel")
        fovValueLabel.Size = UDim2.new(0, 60, 0, 20)
        fovValueLabel.Position = UDim2.new(0.9, 10, 0, 263)
        fovValueLabel.BackgroundTransparency = 1
        fovValueLabel.TextColor3 = Color3.fromRGB(200, 220, 255)
        fovValueLabel.Font = Enum.Font.Gotham
        fovValueLabel.TextSize = 13
        fovValueLabel.Text = "70"
        fovValueLabel.Parent = visBox

        local fovDragging = false
        local fovValue = 70
        local fovActive = false
        local fovLoop

        local function updateFOVSlider(val)
            val = math.clamp(val, 70, 120)
            fovValue = math.floor(val)
            fovSliderFill.Size = UDim2.new((val-70)/(120-70), 0, 1, 0)
            local sliderWidth = fovSliderBG.AbsoluteSize.X
            local knobWidth = fovKnob.AbsoluteSize.X
            local knobPos = ((val-70)/(120-70)) * (sliderWidth - knobWidth)
            fovKnob.Position = UDim2.new(0, knobPos, 0.5, -knobWidth/2)
            fovValueLabel.Text = tostring(fovValue)
        end

        local function getFOVSliderValueFromMouse()
            local abs = fovSliderBG.AbsolutePosition.X
            local mouse = UserInputService:GetMouseLocation().X
            local sliderWidth = fovSliderBG.AbsoluteSize.X
            local knobWidth = fovKnob.AbsoluteSize.X
            local pct = math.clamp((mouse - abs - knobWidth/2) / (sliderWidth - knobWidth), 0, 1)
            return 70 + pct * (120-70)
        end

        fovSliderBG.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                fovDragging = true
                MainFrame.Draggable = false
                updateFOVSlider(getFOVSliderValueFromMouse())
            end
        end)
        fovSliderBG.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                fovDragging = false
                MainFrame.Draggable = true
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if fovDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                updateFOVSlider(getFOVSliderValueFromMouse())
            end
        end)

        updateFOVSlider(fovValue)

        local function updateFOVLoop()
            if fovLoop then fovLoop:Disconnect() end
            if fovActive then
                fovLoop = RunService.RenderStepped:Connect(function()
                    local cam = workspace.CurrentCamera
                    if cam then
                        cam.FieldOfView = fovValue
                    end
                end)
            end
        end

        fovToggleBtn.MouseButton1Click:Connect(function()
            fovActive = not fovActive
            fovToggleBtn.Text = "FOV: " .. (fovActive and "ON" or "OFF")
            updateFOVLoop()
        end)

        frame:GetPropertyChangedSignal("Visible"):Connect(function()
            if not frame.Visible and fovLoop then
                fovLoop:Disconnect()
            end
        end)

    -- PLAYER TAB
    elseif sec.label == "Player" then
        local wsLabel = Instance.new("TextLabel")
        wsLabel.Parent = frame
        wsLabel.Size = UDim2.new(0, 120, 0, 22)
        wsLabel.Position = UDim2.new(0.1, 0, 0, 32)
        wsLabel.BackgroundTransparency = 1
        wsLabel.Font = Enum.Font.Gotham
        wsLabel.TextSize = 14
        wsLabel.TextColor3 = Color3.fromRGB(200,200,255)
        wsLabel.Text = "CFrame WalkSpeed"

        local wsSliderBG = Instance.new("Frame")
        wsSliderBG.Size = UDim2.new(0.8, 0, 0, 18)
        wsSliderBG.Position = UDim2.new(0.1, 0, 0, 60)
        wsSliderBG.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        wsSliderBG.Parent = frame
        local wsUIC = Instance.new("UICorner", wsSliderBG)
        wsUIC.CornerRadius = UDim.new(0, 8)

        local wsSliderFill = Instance.new("Frame")
        wsSliderFill.Size = UDim2.new(0, 0, 1, 0)
        wsSliderFill.Position = UDim2.new(0, 0, 0, 0)
        wsSliderFill.BackgroundColor3 = Color3.fromRGB(150, 255, 100)
        wsSliderFill.Parent = wsSliderBG
        wsSliderFill.ZIndex = 2
        local wsFillUIC = Instance.new("UICorner", wsSliderFill)
        wsFillUIC.CornerRadius = UDim.new(0, 8)

        local wsKnob = Instance.new("Frame")
        wsKnob.Size = UDim2.new(0, 18, 0, 18)
        wsKnob.Position = UDim2.new(0, 0, 0.5, -9)
        wsKnob.BackgroundColor3 = Color3.fromRGB(255,255,255)
        wsKnob.Parent = wsSliderBG
        wsKnob.ZIndex = 3
        local wsKnobUIC = Instance.new("UICorner", wsKnob)
        wsKnobUIC.CornerRadius = UDim.new(1, 0)

        local wsValueLabel = Instance.new("TextLabel")
        wsValueLabel.Size = UDim2.new(0, 50, 0, 20)
        wsValueLabel.Position = UDim2.new(0.9, 10, 0, 55)
        wsValueLabel.BackgroundTransparency = 1
        wsValueLabel.TextColor3 = Color3.fromRGB(200, 255, 200)
        wsValueLabel.Font = Enum.Font.Gotham
        wsValueLabel.TextSize = 13
        wsValueLabel.Text = "16"
        wsValueLabel.Parent = frame

        local wsDragging = false
        local wspeed = 16

        local function updateWSSlider(val)
            val = math.clamp(val, 16, 30)
            wspeed = math.floor(val)
            wsSliderFill.Size = UDim2.new((val-16)/(30-16), 0, 1, 0)
            local sliderWidth = wsSliderBG.AbsoluteSize.X
            local knobWidth = wsKnob.AbsoluteSize.X
            local knobPos = ((val-16)/(30-16)) * (sliderWidth - knobWidth)
            wsKnob.Position = UDim2.new(0, knobPos, 0.5, -knobWidth/2)
            wsValueLabel.Text = tostring(wspeed)
        end

        local function getWSSliderValueFromMouse()
            local abs = wsSliderBG.AbsolutePosition.X
            local mouse = UserInputService:GetMouseLocation().X
            local sliderWidth = wsSliderBG.AbsoluteSize.X
            local knobWidth = wsKnob.AbsoluteSize.X
            local pct = math.clamp((mouse - abs - knobWidth/2) / (sliderWidth - knobWidth), 0, 1)
            return 16 + pct * (30-16)
        end

        wsSliderBG.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                wsDragging = true
                MainFrame.Draggable = false
                updateWSSlider(getWSSliderValueFromMouse())
            end
        end)
        wsSliderBG.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                wsDragging = false
                MainFrame.Draggable = true
            end
        end)
        UserInputService.InputChanged:Connect(function(input)
            if wsDragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                updateWSSlider(getWSSliderValueFromMouse())
            end
        end)

        updateWSSlider(wspeed)

        -- CFrame walkspeed logic (looped)
        local cframeWSLoop
        local cframeWSActive = false
        local enableCFWSBtn = makeButton(frame, "CFrame WS: OFF", 100)
        enableCFWSBtn.TextSize = 14

        enableCFWSBtn.MouseButton1Click:Connect(function()
            cframeWSActive = not cframeWSActive
            enableCFWSBtn.Text = "CFrame WS: " .. (cframeWSActive and "ON" or "OFF")
        end)

        if cframeWSLoop then cframeWSLoop:Disconnect() end
        cframeWSLoop = RunService.RenderStepped:Connect(function(dt)
            if not cframeWSActive then return end
            local char = player.Character
            if not char then return end
            local root = char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not (root and hum) then return end
            local move = Vector3.zero
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then move = move + workspace.CurrentCamera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then move = move - workspace.CurrentCamera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then move = move + workspace.CurrentCamera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then move = move - workspace.CurrentCamera.CFrame.RightVector end
            if move.Magnitude > 0 then
                move = move.Unit * wspeed * dt
                root.CFrame = root.CFrame + Vector3.new(move.X, 0, move.Z)
                hum.PlatformStand = false
            end
        end)

    -- SETTINGS TAB
    elseif sec.label == "Settings" then
        local keybindBox = Instance.new("Frame")
        keybindBox.Parent = frame
        keybindBox.Size = UDim2.new(0.7,-15,1,-15)
        keybindBox.Position = UDim2.new(0,0,0,0)
        keybindBox.BackgroundColor3 = Color3.fromRGB(29,29,29)
        keybindBox.BorderSizePixel = 0
        local corner = Instance.new("UICorner", keybindBox)
        corner.CornerRadius = UDim.new(0,12)
        local keybindLabel = Instance.new("TextLabel")
        keybindLabel.Parent = keybindBox
        keybindLabel.Size = UDim2.new(1,0,0,30)
        keybindLabel.Position = UDim2.new(0,0,0,0)
        keybindLabel.BackgroundTransparency = 1
        keybindLabel.Font = Enum.Font.GothamMedium
        keybindLabel.Text = "Keybinds"
        keybindLabel.TextColor3 = Color3.fromRGB(200,200,200)
        keybindLabel.TextSize = 16
        keybindLabel.TextXAlignment = Enum.TextXAlignment.Left

        local info = Instance.new("TextLabel")
        info.Parent = keybindBox
        info.Size = UDim2.new(1,-20,1,-40)
        info.Position = UDim2.new(0,10,0,35)
        info.BackgroundTransparency = 1
        info.Font = Enum.Font.Gotham
        info.TextSize = 16
        info.TextColor3 = Color3.fromRGB(210,210,210)
        info.TextXAlignment = Enum.TextXAlignment.Left
        info.TextYAlignment = Enum.TextYAlignment.Top
        info.TextWrapped = true
        info.Text = "[RightShift] - Open/Close UI\n['] (Apostrophe) - Unlock/Lock Mouse\n\nHow to use:\n- Press [RightShift] to show/hide the UI.\n- Press ['] to unlock your mouse for interacting with the UI.\n- Press ['] again to lock mouse back into the game."
    end

    btn.MouseButton1Click:Connect(function()
        selectTab(idx)
    end)
end

--// Toggle UI (RightShift)
local visible = true
UserInputService.InputBegan:Connect(function(input, gpe)
    if input.KeyCode == Enum.KeyCode.RightShift then
        visible = not visible
        MainFrame.Visible = visible
    end
end)

--// Mouse Lock/Unlock Keybind (')
local mouseFree = false
local mouseLoop

UserInputService.InputBegan:Connect(function(input, gpe)
    if input.KeyCode == Enum.KeyCode.Quote and not gpe then
        mouseFree = not mouseFree
        if mouseFree then
            if mouseLoop then mouseLoop:Disconnect() end
            mouseLoop = RunService.RenderStepped:Connect(function()
                UserInputService.MouseBehavior = Enum.MouseBehavior.Default
                UserInputService.MouseIconEnabled = true -- SHOW CURSOR
            end)
        else
            if mouseLoop then mouseLoop:Disconnect() end
            UserInputService.MouseBehavior = Enum.MouseBehavior.LockCurrentPosition
            UserInputService.MouseIconEnabled = false -- HIDE CURSOR
        end
    end
end)

gui.AncestryChanged:Connect(function()
    if not gui:IsDescendantOf(game) then
        if mouseLoop then mouseLoop:Disconnect() end
        UserInputService.MouseBehavior = Enum.MouseBehavior.Default
        UserInputService.MouseIconEnabled = true
    end
end)
