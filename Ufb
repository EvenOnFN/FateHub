--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
local UserInputService = game:GetService("UserInputService")
local isMobile = UserInputService.TouchEnabled and not UserInputService.KeyboardEnabled

if isMobile then
    local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
    
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Workspace = game:GetService("Workspace")
    local plr = Players.LocalPlayer

    getgenv().SecureMode = true

    local mechMod = ReplicatedStorage:FindFirstChild("Assets") 
        and ReplicatedStorage.Assets:FindFirstChild("Modules") 
        and ReplicatedStorage.Assets.Modules:FindFirstChild("Client") 
        and ReplicatedStorage.Assets.Modules.Client:FindFirstChild("Mechanics")
    
    if mechMod then
        mechMod = require(mechMod)
    end

    local ConnectionManager = {}
    ConnectionManager.connections = {}

    function ConnectionManager:Add(name, connection)
        if self.connections[name] then
            self.connections[name]:Disconnect()
        end
        self.connections[name] = connection
    end

    function ConnectionManager:Remove(name)
        if self.connections[name] then
            self.connections[name]:Disconnect()
            self.connections[name] = nil
        end
    end

    function ConnectionManager:CleanupAll()
        for name, conn in pairs(self.connections) do
            if conn then
                conn:Disconnect()
            end
        end
        self.connections = {}
    end

    local pullVectorEnabled = false
    local smoothPullEnabled = false
    local isPullingBall = false
    local isSmoothPulling = false
    local walkSpeedEnabled = false
    local jumpPowerEnabled = false
    local bigheadEnabled = false
    local tackleReachEnabled = false
    local playerHitboxEnabled = false
    local jumpBoostEnabled = false
    local jumpBoostTradeMode = false
    local diveBoostEnabled = false
    local autoFollowBallCarrierEnabled = false
    local pullButtonActive = false
    local legPullButtonActive = false
    local dragButtonsEnabled = false
    local CanDiveBoost = true
    local CanBoost = true
    local isSprinting = false
    

    local offsetDistance = 15
    local magnetSmoothness = 0.20
    local customWalkSpeed = 25
    local customJumpPower = 50
    local bigheadSize = 1
    local bigheadTransparency = 0.5
    local tackleReachDistance = 5
    local playerHitboxSize = 5
    local maxPullDistance = 35
    local autoFollowBlatancy = 0.5
    local BOOST_FORCE_Y = 32
    local BALL_DETECTION_RADIUS = 10
    local BOOST_COOLDOWN = 1
    local DIVE_BOOST_POWER = 15
    local DIVE_BOOST_COOLDOWN = 2

    local jumpConnection = nil
    local bigheadConnection = nil
    local tackleReachConnection = nil
    local playerHitboxConnection = nil
    local walkSpeedConnection = nil
    local autoFollowConnection = nil
    local mobileInputMethod = "Buttons" 
    local isParkMatch = Workspace:FindFirstChild("ParkMatchMap") ~= nil

    local character = plr.Character or plr.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    local humanoid = character:WaitForChild("Humanoid")
    local head = character:WaitForChild("Head")
    local defaultHeadSize = head.Size
    local defaultHeadTransparency = head.Transparency

    local function onCharacterAdded(char)
        character = char
        humanoidRootPart = char:WaitForChild("HumanoidRootPart")
        humanoid = char:WaitForChild("Humanoid")
        head = char:WaitForChild("Head")
        defaultHeadSize = head.Size
        defaultHeadTransparency = head.Transparency
    end

    ConnectionManager:Add("CharacterAdded", plr.CharacterAdded:Connect(onCharacterAdded))

    local function getFootball()
        local parkMap = Workspace:FindFirstChild("ParkMap")
        if parkMap and parkMap:FindFirstChild("Replicated") then
            local fields = parkMap.Replicated:FindFirstChild("Fields")
            if fields then
                local parkFields = {
                    fields:FindFirstChild("LeftField"),
                    fields:FindFirstChild("RightField"),
                    fields:FindFirstChild("BLeftField"),
                    fields:FindFirstChild("BRightField"),
                    fields:FindFirstChild("HighField"),
                    fields:FindFirstChild("TLeftField"),
                    fields:FindFirstChild("TRightField")
                }
                
                for _, field in ipairs(parkFields) do
                    if field and field:FindFirstChild("Replicated") then
                        local football = field.Replicated:FindFirstChild("Football")
                        if football and football:IsA("BasePart") then 
                            return football 
                        end
                    end
                end
            end
        end
        
        if isParkMatch then
            local parkMatchFootball = Workspace:FindFirstChild("ParkMatchMap")
            if parkMatchFootball and parkMatchFootball:FindFirstChild("Replicated") then
                parkMatchFootball = parkMatchFootball.Replicated:FindFirstChild("Fields")
                if parkMatchFootball and parkMatchFootball:FindFirstChild("MatchField") then
                    parkMatchFootball = parkMatchFootball.MatchField:FindFirstChild("Replicated")
                    if parkMatchFootball then
                        local football = parkMatchFootball:FindFirstChild("Football")
                        if football and football:IsA("BasePart") then return football end
                    end
                end
            end
        end
        
        local gamesFolder = Workspace:FindFirstChild("Games")
        if gamesFolder then
            for _, gameInstance in ipairs(gamesFolder:GetChildren()) do
                local replicatedFolder = gameInstance:FindFirstChild("Replicated")
                if replicatedFolder then
                    local kickoffFootball = replicatedFolder:FindFirstChild("918f5408-d86a-4fb8-a88c-5cab57410acf")
                    if kickoffFootball and kickoffFootball:IsA("BasePart") then return kickoffFootball end
                    for _, item in ipairs(replicatedFolder:GetChildren()) do
                        if item:IsA("BasePart") and item.Name == "Football" then return item end
                    end
                end
            end
        end
        return nil
    end

    local function getBallCarrier()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= plr and player.Character then
                local football = player.Character:FindFirstChild("Football")
                if football then
                    return player
                end
            end
        end
        return nil
    end

    local function teleportToBall()
        local ball = getFootball()
        if ball and humanoidRootPart then
            if Workspace:FindFirstChild("ParkMap") then
                local distance = (ball.Position - humanoidRootPart.Position).Magnitude
                
                if distance > maxPullDistance then
                    return
                end
            end
            
            local ballVelocity = ball.Velocity
            local ballPosition = ball.Position
            local direction = ballVelocity.Unit
            local targetPosition = ballPosition + (direction * 12) - Vector3.new(0, 1.5, 0) + Vector3.new(0, 5.197499752044678 / 6, 0)
            local lookDirection = (ballPosition - humanoidRootPart.Position).Unit
            humanoidRootPart.CFrame = CFrame.new(targetPosition, targetPosition + lookDirection)
        end
    end

    local function smoothTeleportToBall()
        local ball = getFootball()
        if ball and humanoidRootPart then
            if Workspace:FindFirstChild("ParkMap") then
                local distance = (ball.Position - humanoidRootPart.Position).Magnitude
                if distance > maxPullDistance then return end
            end
            
            local ballVelocity = ball.Velocity
            local ballSpeed = ballVelocity.Magnitude
            local offset = (ballSpeed > 0) and (ballVelocity.Unit * offsetDistance) or Vector3.new(0, 0, 0)
            local targetPosition = ball.Position + offset + Vector3.new(0, 3, 0)
            local lookDirection = (ball.Position - humanoidRootPart.Position).Unit
            humanoidRootPart.CFrame = humanoidRootPart.CFrame:Lerp(CFrame.new(targetPosition, targetPosition + lookDirection), magnetSmoothness)
        end
    end

    local function applyJumpBoost(rootPart)
        local bv = Instance.new("BodyVelocity")
        bv.Velocity = Vector3.new(0, BOOST_FORCE_Y, 0)
        bv.MaxForce = Vector3.new(0, math.huge, 0)
        bv.P = 5000
        bv.Parent = rootPart
        game:GetService("Debris"):AddItem(bv, 0.2)
    end

    local Players = game:GetService("Players")
    local lp = Players.LocalPlayer

local TweenService = game:GetService("TweenService")
local lp = game:GetService("Players").LocalPlayer

getgenv().turkeyFarmEnabled = getgenv().turkeyFarmEnabled or false
getgenv().turkeySpeed = getgenv().turkeySpeed or 0.3

local turkeyFarmConnection = nil
local isFarming = false

local function getChar()
return lp.Character, lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
end

local function basket()
local id = lp.Replicated.TeamID.Value
return id == "AwayTeam" and Vector3.new(137, 372, 0) or Vector3.new(-138, 372, 2)
end

local function getMG()
for _, v in ipairs(workspace.MiniGames:GetChildren()) do
    local r = v:FindFirstChild("Replicated")
    if r and r:FindFirstChild("Turkeys") then
        return r
    end
end
return nil
end

local function isTurkeyValid(turkey)
return turkey and turkey.Parent and turkey:IsDescendantOf(workspace)
end

local function tweenToPosition(hrp, targetPos, speed, turkey)
speed = speed or 0.5
local distance = (hrp.Position - targetPos).Magnitude
local tweenSpeed = math.max(speed, distance / 100)

local tweenInfo = TweenInfo.new(
    tweenSpeed,
    Enum.EasingStyle.Linear,
    Enum.EasingDirection.InOut
)

local goal = {CFrame = CFrame.new(targetPos)}
local tween = TweenService:Create(hrp, tweenInfo, goal)

tween:Play()

if turkey then
    local connection
    local startY = hrp.Position.Y
    connection = game:GetService("RunService").Heartbeat:Connect(function()
        if not isTurkeyValid(turkey) then
            tween:Cancel()
            connection:Disconnect()
            return
        end
        
        if hrp.Position.Y < (startY - 10) then
            tween:Cancel()
            connection:Disconnect()
            return
        end
    end)
    
    tween.Completed:Wait()
    connection:Disconnect()
    
    if hrp.Position.Y < (startY - 10) then
        return false
    end
    
    return isTurkeyValid(turkey)
else
    tween.Completed:Wait()
    return true
end
end

game:GetService("RunService").Heartbeat:Connect(function()
if getgenv().turkeyFarmEnabled then
    if not turkeyFarmConnection then
        turkeyFarmConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not getgenv().turkeyFarmEnabled then
                if turkeyFarmConnection then
                    turkeyFarmConnection:Disconnect()
                    turkeyFarmConnection = nil
                end
                return
            end
            
            if isFarming then return end
            isFarming = true
            
            task.spawn(function()
                local char, hrp = getChar()
                if not char or not hrp then 
                    isFarming = false
                    return 
                end
                
                local replicated = getMG()
                if not replicated then 
                    isFarming = false
                    return 
                end
                
                local turkeyList = replicated:FindFirstChild("Turkeys")
                if not turkeyList then 
                    isFarming = false
                    return 
                end
                
                local turkeys = {}
                for _, turkey in pairs(turkeyList:GetChildren()) do
                    if turkey:IsA("BasePart") and isTurkeyValid(turkey) then
                        table.insert(turkeys, turkey)
                    end
                end
                
                for _, turkey in ipairs(turkeys) do
                    if not getgenv().turkeyFarmEnabled then break end
                    
                    if isTurkeyValid(turkey) then
                        pcall(function()
                            -- Check if turkey is too far below character
                            local char, hrp = getChar()
                            if not hrp then return end
                            
                            local yDifference = hrp.Position.Y - turkey.Position.Y
                            if yDifference > 15 then
                                -- Turkey is more than 15 studs below, skip it
                                return
                            end
                            
                            local stillValid = tweenToPosition(hrp, turkey.Position, getgenv().turkeySpeed, turkey)
                            
                            if stillValid and isTurkeyValid(turkey) then
                                firetouchinterest(hrp, turkey, 0)
                                firetouchinterest(hrp, turkey, 1)
                                task.wait(0.1)
                                
                                tweenToPosition(hrp, basket(), getgenv().turkeySpeed, nil)
                                task.wait(0.1)
                            end
                        end)
                    end
                end
                
                isFarming = false
            end)
        end)
    end
else
    if turkeyFarmConnection then
        turkeyFarmConnection:Disconnect()
        turkeyFarmConnection = nil
    end
end
end)

    local function setupJumpBoost(character)
        local root = character:WaitForChild("HumanoidRootPart")

        ConnectionManager:Add("JumpBoostTouch", root.Touched:Connect(function(hit)
            if not jumpBoostEnabled or not CanBoost then return end
            if root.Velocity.Y >= -2 then return end

            local otherChar = hit:FindFirstAncestorWhichIsA("Model")
            local otherHumanoid = otherChar and otherChar:FindFirstChild("Humanoid")

            if otherChar and otherChar ~= character and otherHumanoid then
                if jumpBoostTradeMode then
                    CanBoost = false
                    applyJumpBoost(root)
                    task.delay(BOOST_COOLDOWN, function()
                        CanBoost = true
                    end)
                else
                    local football = getFootball()
                    if football then
                        local distance = (football.Position - root.Position).Magnitude
                        if distance <= BALL_DETECTION_RADIUS then
                            CanBoost = false
                            applyJumpBoost(root)
                            task.delay(BOOST_COOLDOWN, function()
                                CanBoost = true
                            end)
                        end
                    end
                end
            end
        end))
    end

    local function updateDivePower()
        if not diveBoostEnabled then return end
        
        local gameId = plr:FindFirstChild("Replicated") and plr.Replicated:FindFirstChild("GameID")
        if not gameId then return end
        
        local gid = gameId.Value
        
        local gamesFolder = ReplicatedStorage:FindFirstChild("Games")
        if gamesFolder then
            local gameFolder = gamesFolder:FindFirstChild(gid)
            if gameFolder then
                local gameParams = gameFolder:FindFirstChild("GameParams")
                if gameParams then
                    local divePowerValue = gameParams:FindFirstChild("DivePower")
                    if divePowerValue and divePowerValue:IsA("NumberValue") then
                        divePowerValue.Value = diveBoostPower
                    end
                end
            end
        end
        
        local miniGamesFolder = ReplicatedStorage:FindFirstChild("MiniGames")
        if miniGamesFolder then
            local gameFolder = miniGamesFolder:FindFirstChild(gid)
            if gameFolder then
                local gameParams = gameFolder:FindFirstChild("GameParams")
                if gameParams then
                    local divePowerValue = gameParams:FindFirstChild("DivePower")
                    if divePowerValue and divePowerValue:IsA("NumberValue") then
                        divePowerValue.Value = diveBoostPower
                    end
                end
            end
        end
    end

    local pullButtonGui = Instance.new("ScreenGui")
    pullButtonGui.Name = "PullButtonGui"
    pullButtonGui.ResetOnSpawn = false
    pullButtonGui.Parent = plr:WaitForChild("PlayerGui")
    
    local pullContainer = Instance.new("Frame")
    pullContainer.Name = "PullContainer"
    pullContainer.Size = UDim2.new(0, 100, 0, 100)
    pullContainer.Position = UDim2.new(0.85, 0, 0.7, 0)
    pullContainer.BackgroundTransparency = 1
    pullContainer.Active = true
    pullContainer.Draggable = false
    pullContainer.Parent = pullButtonGui
    
    local pullButton = Instance.new("TextButton")
    pullButton.Size = UDim2.new(0, 70, 0, 70)
    pullButton.Position = UDim2.new(0, 15, 0, 15)
    pullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    pullButton.BorderSizePixel = 0
    pullButton.Text = "Pull"
    pullButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    pullButton.Font = Enum.Font.GothamBold
    pullButton.TextSize = 16
    pullButton.Parent = pullContainer
    
    local pullCorner = Instance.new("UICorner")
    pullCorner.CornerRadius = UDim.new(1, 0)
    pullCorner.Parent = pullButton
    
    local legPullContainer = Instance.new("Frame")
    legPullContainer.Size = UDim2.new(0, 100, 0, 100)
    legPullContainer.Position = UDim2.new(0.85, 0, 0.55, 0)
    legPullContainer.BackgroundTransparency = 1
    legPullContainer.Active = true
    legPullContainer.Draggable = false
    legPullContainer.Parent = pullButtonGui
    
    local legPullButton = Instance.new("TextButton")
    legPullButton.Size = UDim2.new(0, 70, 0, 70)
    legPullButton.Position = UDim2.new(0, 15, 0, 15)
    legPullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    legPullButton.BorderSizePixel = 0
    legPullButton.Text = "Legit"
    legPullButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    legPullButton.Font = Enum.Font.GothamBold
    legPullButton.TextSize = 16
    legPullButton.Parent = legPullContainer
    
    local legPullCorner = Instance.new("UICorner")
    legPullCorner.CornerRadius = UDim.new(1, 0)
    legPullCorner.Parent = legPullButton
    
    pullButton.MouseButton1Down:Connect(function()
        if pullVectorEnabled then
            isPullingBall = true
            pullButtonActive = true
            pullButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
            spawn(function()
                while isPullingBall and pullVectorEnabled do
                    teleportToBall()
                    wait(0.05)
                end
            end)
        end
    end)
    
    pullButton.MouseButton1Up:Connect(function()
        isPullingBall = false
        pullButtonActive = false
        pullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end)
    
    legPullButton.MouseButton1Down:Connect(function()
        if smoothPullEnabled then
            isSmoothPulling = true
            legPullButtonActive = true
            legPullButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
            spawn(function()
                while legPullButtonActive and smoothPullEnabled do
                    smoothTeleportToBall()
                    wait(0.01)
                end
            end)
        end
    end)
    
    legPullButton.MouseButton1Up:Connect(function()
        isSmoothPulling = false
        legPullButtonActive = false
        legPullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end)
    
    UserInputService.TouchEnded:Connect(function(touch, gameProcessed)
        if pullButtonActive then
            isPullingBall = false
            pullButtonActive = false
            pullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        end
        if legPullButtonActive then
            isSmoothPulling = false
            legPullButtonActive = false
            legPullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        end
    end)
    
    RunService.RenderStepped:Connect(function()
        if pullVectorEnabled then
            pullButton.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
            pullButton.Text = "Pull ✓"
        elseif not pullButtonActive then
            pullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            pullButton.Text = "Pull"
        end
        
        if smoothPullEnabled then
            legPullButton.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
            legPullButton.Text = "Legit ✓"
        elseif not legPullButtonActive then
            legPullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            legPullButton.Text = "Legit"
        end
        
        if mobileInputMethod == "Tapping" then
            pullButton.Visible = false
            legPullButton.Visible = false
        elseif mobileInputMethod == "Buttons" or mobileInputMethod == "Both" then
            pullButton.Visible = pullVectorEnabled
            legPullButton.Visible = smoothPullEnabled
        end
    end)
    
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == Enum.KeyCode.ButtonR2 then
            if pullVectorEnabled then
                isPullingBall = true
                pullButtonActive = true
                pullButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
                spawn(function()
                    while isPullingBall and pullVectorEnabled do
                        teleportToBall()
                        wait(0.05)
                    end
                end)
            end
            if smoothPullEnabled then
                isSmoothPulling = true
                legPullButtonActive = true
                legPullButton.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
                spawn(function()
                    while legPullButtonActive and smoothPullEnabled do
                        smoothTeleportToBall()
                        wait(0.01)
                    end
                end)
            end
        end
    end)
    
    UserInputService.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == Enum.KeyCode.ButtonR2 then
            if pullButtonActive then
                isPullingBall = false
                pullButtonActive = false
                pullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            end
            if legPullButtonActive then
                isSmoothPulling = false
                legPullButtonActive = false
                legPullButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            end
        end
    end)
    
    local function getPlayerTeam(player)
        local menuGui = plr:FindFirstChild("PlayerGui") -- Use local player's GUI
        if menuGui then
            local menu = menuGui:FindFirstChild("Menu")
            if menu then
                local basis = menu:FindFirstChild("Basis")
                if basis then
                    local window = basis:FindFirstChild("Window")
                    if window then
                        local addFriends = window:FindFirstChild("AddFriends")
                        if addFriends then
                            local frame = addFriends:FindFirstChild("Basis")
                            if frame then
                                frame = frame:FindFirstChild("Frame")
                                if frame then
                                    local homeTeam = frame:FindFirstChild("HomeTeam")
                                    local awayTeam = frame:FindFirstChild("AwayTeam")
                                    
                                    -- Check if the player is in HomeTeam
                                    if homeTeam and homeTeam:FindFirstChild("Frame") then
                                        if homeTeam.Frame:FindFirstChild(player.Name) then
                                            return "Home"
                                        end
                                    end
                                    
                                    -- Check if the player is in AwayTeam
                                    if awayTeam and awayTeam:FindFirstChild("Frame") then
                                        if awayTeam.Frame:FindFirstChild(player.Name) then
                                            return "Away"
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
        return nil
    end

    local Window = Rayfield:CreateWindow({
        Name = "Kali Hub | NFL Universe [Mobile]",
        LoadingTitle = "Loading NFL Universe Script",
        LoadingSubtitle = "by Kali Hub",
        ConfigurationSaving = {
            Enabled = true,
            FolderName = "NFLUniverse",
            FileName = "KaliHub"
        }
    })
    
    local MainTab = Window:CreateTab("⚡ Main", nil)
    
    local ButtonSection = MainTab:CreateSection("Button Controls")
    
    local DragButtonsToggle = MainTab:CreateToggle({
        Name = "Draggable Buttons",
        CurrentValue = false,
        Flag = "DragButtons",
        Callback = function(Value)
            dragButtonsEnabled = Value
            
            pullContainer.Draggable = Value
            legPullContainer.Draggable = Value
            
            if Value then
                pullContainer.BackgroundTransparency = 0.8
                pullContainer.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
                legPullContainer.BackgroundTransparency = 0.8
                legPullContainer.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
            else
                pullContainer.BackgroundTransparency = 1
                legPullContainer.BackgroundTransparency = 1
            end
        end,
    })
    
    local InputMethodDropdown = MainTab:CreateDropdown({
        Name = "Mobile Input Method",
        Options = {"Buttons", "Tapping", "Both"},
        CurrentOption = {"Buttons"},
        MultipleOptions = false,
        Flag = "InputMethod",
        Callback = function(Option)
            mobileInputMethod = Option[1]
            
            if mobileInputMethod == "Tapping" then
                pullButton.Visible = false
                legPullButton.Visible = false
            elseif mobileInputMethod == "Buttons" then
                pullButton.Visible = pullVectorEnabled
                legPullButton.Visible = smoothPullEnabled
            elseif mobileInputMethod == "Both" then
                pullButton.Visible = pullVectorEnabled
                legPullButton.Visible = smoothPullEnabled
            end
        end,
    })
    
if string.split(identifyexecutor() or "None", " ")[1] ~= "Xeno" then

            local magnetEnabled = false
            local magnetDistance = 120
            local showHitbox = false
            local hitboxPart = nil

            local plr = game.Players.LocalPlayer
            local char = plr.Character or plr.CharacterAdded:Wait()
            local hrp = char:WaitForChild('HumanoidRootPart')

            local og1 = CFrame.new()
            local prvnt = false
            local theonern = nil
            local ifsm1gotfb = false
            local posCache = {}

            local validNames = {
                ['Football'] = true,
                ['Football MeshPart'] = true
            }

            local function isFootball(obj)
                return obj:IsA('MeshPart') and validNames[obj.Name]
            end

            local function createHitbox()
                if hitboxPart then
                    hitboxPart:Destroy()
                end
                
                hitboxPart = Instance.new("Part")
                hitboxPart.Name = "MagnetHitbox"
                hitboxPart.Size = Vector3.new(magnetDistance * 2, magnetDistance * 2, magnetDistance * 2)
                hitboxPart.Anchored = true
                hitboxPart.CanCollide = false
                hitboxPart.Transparency = 0.7
                hitboxPart.Material = Enum.Material.ForceField
                hitboxPart.Color = Color3.fromRGB(138, 43, 226)
                hitboxPart.CastShadow = false
                hitboxPart.Shape = Enum.PartType.Ball
                hitboxPart.Parent = workspace
                
                return hitboxPart
            end

            local function removeHitbox()
                if hitboxPart then
                    hitboxPart:Destroy()
                    hitboxPart = nil
                end
            end

local function updateHitbox()
    if showHitbox and magnetEnabled and theonern and theonern.Parent then
        if not hitboxPart then
            createHitbox()
        end
        hitboxPart.CFrame = theonern.CFrame
        hitboxPart.Size = Vector3.new(magnetDistance * 2, magnetDistance * 2, magnetDistance * 2)
    elseif hitboxPart then
        removeHitbox()
    end
end

            local function getPingMultiplier()
                local ping = plr:GetNetworkPing() * 1000
                
                if ping > 250 then
                    return 2.5
                elseif ping > 200 then
                    return 2.0
                elseif ping > 150 then
                    return 1.7
                elseif ping > 100 then
                    return 1.4
                elseif ping > 50 then
                    return 1.2
                else
                    return 1.0
                end
            end

            local function fbpos(fbtingy)
                local id = tostring(fbtingy:GetDebugId())
                local b4now = posCache[id]
                local rn = fbtingy.Position
                posCache[id] = rn
                return rn, b4now or rn
            end

            local function ifsm1gotit()
                if theonern and theonern.Parent then
                    local parent = theonern.Parent
                    if parent:IsA('Model') and game.Players:GetPlayerFromCharacter(parent) then
                        return true
                    end
                    for _, player in next, game.Players:GetPlayers() do
                        if player.Character and theonern:IsDescendantOf(player.Character) then
                            return true
                        end
                    end
                end
                return false
            end

            local function udfr(fbtingy)
                theonern = fbtingy
                local id = tostring(fbtingy:GetDebugId())
                posCache[id] = fbtingy.Position
            end

            workspace.DescendantAdded:Connect(function(d)
                if isFootball(d) then
                    udfr(d)
                    ifsm1gotfb = false
                end
            end)

            workspace.DescendantAdded:Connect(function(d)
                if isFootball(d) then
                    d.AncestryChanged:Connect(function()
                        if d.Parent and d.Parent:IsA('Model') and game.Players:GetPlayerFromCharacter(d.Parent) then
                            ifsm1gotfb = true
                        elseif d.Parent == workspace or d.Parent == nil then
                            ifsm1gotfb = false
                        end
                    end)
                end
            end)

            workspace.DescendantRemoving:Connect(function(d)
                if d == theonern then
                    theonern = nil
                    ifsm1gotfb = false
                end
            end)

            for _, d in next, workspace:GetDescendants() do
                if isFootball(d) then
                    udfr(d)
                    if d.Parent and d.Parent:IsA('Model') and game.Players:GetPlayerFromCharacter(d.Parent) then
                        ifsm1gotfb = true
                    end
                end
            end

            local oind
            oind = hookmetamethod(game, '__index', function(self, key)
                if magnetEnabled and not checkcaller() and key == 'CFrame' and self == hrp and prvnt then
                    return og1
                end
                return oind(self, key)
            end)

            game:GetService('RunService').Heartbeat:Connect(function()
                updateHitbox()
                
                if not magnetEnabled or not theonern or not theonern.Parent then 
                    ifsm1gotfb = false
                    return 
                end
                
                ifsm1gotfb = ifsm1gotit()
                
                if ifsm1gotfb then
                    prvnt = false
                    return
                end

                local pos, old = fbpos(theonern)
                local d0 = (hrp.Position - pos).Magnitude
                if d0 > magnetDistance then return end

                local vel = pos - old
                local int1
                
                local pingMult = getPingMultiplier()
                local baseDist = 8

                if vel.Magnitude > 0.1 then
                    int1 = pos + (vel.Unit * baseDist * pingMult)
                else
                    int1 = pos + Vector3.new(5, 0, 5) * pingMult
                end

                int1 = Vector3.new(int1.X, math.max(int1.Y, pos.Y), int1.Z)

                og1 = hrp.CFrame
                prvnt = true
                hrp.CFrame = CFrame.new(int1)
                game:GetService('RunService').RenderStepped:Wait()
                hrp.CFrame = og1
                prvnt = false
            end)

            plr.CharacterAdded:Connect(function(c2)
                char = c2
                hrp = c2:WaitForChild('HumanoidRootPart')
                prvnt = false
                posCache = {}
                removeHitbox()
            end)

                local MagnetSection = MainTab:CreateSection("Magnets")

            local MagnetToggle = MainTab:CreateToggle({
                Name = "Desync Mags",
                CurrentValue = false,
                Flag = "FootballMagnet",
                Callback = function(Value)
                    magnetEnabled = Value
                    if not Value then
                        prvnt = false
                        removeHitbox()
                    end
                end,
            })
            
            local MagnetDistSlider = MainTab:CreateSlider({
                Name = "Magnet Distance",
                Range = {0, 120},
                Increment = 1,
                CurrentValue = 120,
                Flag = "FootballDistance",
                Callback = function(Value)
                    magnetDistance = Value
                end,
            })
            
            local ShowHitboxToggle = MainTab:CreateToggle({
                Name = "Show Hitbox",
                CurrentValue = false,
                Flag = "ShowHitbox",
                Callback = function(Value)
                    showHitbox = Value
                    if not Value then
                        removeHitbox()
                    end
                end,
            })
        end

    local LegitSection = MainTab:CreateSection("Legit Pull Vector")
    
    local LegitPullToggle = MainTab:CreateToggle({
        Name = "Legit Pull Vector",
        CurrentValue = false,
        Flag = "LegitPull",
        Callback = function(Value)
            smoothPullEnabled = Value
            
            if mobileInputMethod == "Buttons" or mobileInputMethod == "Both" then
                legPullButton.Visible = Value
            end
        end,
    })
    
    local SmoothSlider = MainTab:CreateSlider({
        Name = "Vector Smoothing",
        Range = {0.01, 1},
        Increment = 0.01,
        CurrentValue = 0.20,
        Flag = "Smoothness",
        Callback = function(Value)
            magnetSmoothness = Value
        end,
    })
    
    local PullSection = MainTab:CreateSection("Pull Vector")
    
    local PullToggle = MainTab:CreateToggle({
        Name = "Pull Vector",
        CurrentValue = false,
        Flag = "PullVector",
        Callback = function(Value)
            pullVectorEnabled = Value
            
            if mobileInputMethod == "Buttons" or mobileInputMethod == "Both" then
                pullButton.Visible = Value
            end
        end,
    })
    
    local OffsetSlider = MainTab:CreateSlider({
        Name = "Offset Distance",
        Range = {0, 30},
        Increment = 1,
        CurrentValue = 15,
        Flag = "Offset",
        Callback = function(Value)
            offsetDistance = Value
        end,
    })
    
    local MaxDistSlider = MainTab:CreateSlider({
        Name = "Max Pull Distance",
        Range = {1, 100},
        Increment = 1,
        CurrentValue = 35,
        Flag = "MaxDist",
        Callback = function(Value)
            maxPullDistance = Value
        end,
    })
    
    local function getSprintingValue()
        local ReplicatedStorage = game:GetService("ReplicatedStorage")

        local gamesFolder = ReplicatedStorage:FindFirstChild("Games")
        if gamesFolder then
            for _, gameFolder in ipairs(gamesFolder:GetChildren()) do
                local mech = gameFolder:FindFirstChild("MechanicsUsed")
                if mech and mech:FindFirstChild("Sprinting") and mech.Sprinting:IsA("BoolValue") then
                    return mech.Sprinting
                end
            end
        end

        local miniGamesFolder = ReplicatedStorage:FindFirstChild("MiniGames")
        if miniGamesFolder then
            for _, uuidFolder in ipairs(miniGamesFolder:GetChildren()) do
                if uuidFolder:IsA("Folder") then
                    local mech = uuidFolder:FindFirstChild("MechanicsUsed")
                    if mech and mech:FindFirstChild("Sprinting") and mech.Sprinting:IsA("BoolValue") then
                        return mech.Sprinting
                    end
                end
            end
        end

        return nil
    end

    local PlayerTab = Window:CreateTab("👤 Player", nil)
    
    local StaminaSection = PlayerTab:CreateSection("Stamina")

local StaminaDepletion = PlayerTab:CreateToggle({
Name = "Infinite Stamina",
CurrentValue = false,
Flag = "StaminaDepletion",
Callback = function(enabled)
    staminaDepletionEnabled = enabled
    
    if enabled then
        spawn(function()
            while staminaDepletionEnabled do
                task.wait()
                if mechMod then
                    mechMod.Stamina = 100
                end
            end
        end)
    end
end,
})

    local SpeedSection = PlayerTab:CreateSection("WalkSpeed")
    
    local WalkSpeedToggle = PlayerTab:CreateToggle({
        Name = "WalkSpeed",
        CurrentValue = false,
        Flag = "WalkSpeed",
        Callback = function(value)
            walkSpeedEnabled = value
            
            if walkSpeedConnection then
                walkSpeedConnection:Disconnect()
                walkSpeedConnection = nil
            end
            
            local function setSpeed()
                local humanoid = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = walkSpeedEnabled and customWalkSpeed or 16
                end
            end
            
            if value then
                setSpeed()
                local humanoid = plr.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    walkSpeedConnection = humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(setSpeed)
                end
            else
                setSpeed()
            end
        end,
    })
    
    local WalkSpeedSlider = PlayerTab:CreateSlider({
        Name = "Custom WalkSpeed",
        Range = {16, 35},
        Increment = 1,
        CurrentValue = 25,
        Flag = "WSValue",
        Callback = function(Value)
            customWalkSpeed = Value
            if walkSpeedEnabled then
                local humanoid = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = Value
                end
            end
        end,
    })
    
    local JumpSection = PlayerTab:CreateSection("JumpPower")
    
    local JumpToggle = PlayerTab:CreateToggle({
        Name = "JumpPower",
        CurrentValue = false,
        Flag = "JumpPower",
        Callback = function(value)
            jumpPowerEnabled = value
            if value then
                if jumpConnection then jumpConnection:Disconnect() end
                jumpConnection = humanoid.Jumping:Connect(function()
                    if jumpPowerEnabled and humanoidRootPart then
                        local jumpVelocity = Vector3.new(0, customJumpPower, 0)
                        humanoidRootPart.Velocity = Vector3.new(humanoidRootPart.Velocity.X, 0, humanoidRootPart.Velocity.Z) + jumpVelocity
                    end
                end)
            else
                if jumpConnection then jumpConnection:Disconnect() end
                jumpConnection = nil
            end
        end,
    })
    
    local JumpSlider = PlayerTab:CreateSlider({
        Name = "Custom JumpPower",
        Range = {10, 200},
        Increment = 5,
        CurrentValue = 50,
        Flag = "JPValue",
        Callback = function(Value)
            customJumpPower = Value
        end,
    })
    
    local BoostSection = PlayerTab:CreateSection("Jump Boost")
    
    local JumpBoostToggle = PlayerTab:CreateToggle({
        Name = "Jump Boost",
        CurrentValue = false,
        Flag = "JumpBoost",
        Callback = function(value)
            jumpBoostEnabled = value
            if value then
                if plr.Character then
                    setupJumpBoost(plr.Character)
                end
            else
                ConnectionManager:Remove("JumpBoostTouch")
            end
        end
    })
    
    local JumpBoostModeToggle = PlayerTab:CreateToggle({
        Name = "Always Boost Mode",
        CurrentValue = false,
        Flag = "BoostMode",
        Callback = function(Value)
            jumpBoostTradeMode = Value
        end
    })
    
    local BoostForceSlider = PlayerTab:CreateSlider({
        Name = "Boost Force",
        Range = {10, 100},
        Increment = 2,
        CurrentValue = 32,
        Flag = "BoostForce",
        Callback = function(Value)
            BOOST_FORCE_Y = Value
        end,
    })
    
    local DiveSection = PlayerTab:CreateSection("Dive Boost")
    
    local DiveBoostToggle = PlayerTab:CreateToggle({
        Name = "Dive Boost",
        CurrentValue = false,
        Flag = "DiveBoost",
        Callback = function(value)
            diveBoostEnabled = value
            
            if value then
                updateDivePower()
            else
                local gameId = plr:FindFirstChild("Replicated") and plr.Replicated:FindFirstChild("GameID")
                if gameId then
                    local gid = gameId.Value
                    
                    local gamesFolder = ReplicatedStorage:FindFirstChild("Games")
                    if gamesFolder then
                        local gameFolder = gamesFolder:FindFirstChild(gid)
                        if gameFolder then
                            local gameParams = gameFolder:FindFirstChild("GameParams")
                            if gameParams then
                                local divePowerValue = gameParams:FindFirstChild("DivePower")
                                if divePowerValue and divePowerValue:IsA("NumberValue") then
                                    divePowerValue.Value = 2.2
                                end
                            end
                        end
                    end
                    
                    local miniGamesFolder = ReplicatedStorage:FindFirstChild("MiniGames")
                    if miniGamesFolder then
                        local gameFolder = miniGamesFolder:FindFirstChild(gid)
                        if gameFolder then
                            local gameParams = gameFolder:FindFirstChild("GameParams")
                            if gameParams then
                                local divePowerValue = gameParams:FindFirstChild("DivePower")
                                if divePowerValue and divePowerValue:IsA("NumberValue") then
                                    divePowerValue.Value = 2.2
                                end
                            end
                        end
                    end
                end
            end
        end,
    })
    
    local DivePowerSlider = PlayerTab:CreateSlider({
        Name = "Dive Boost Power",
        Range = {2.2, 10},
        Increment = 0.1,
        CurrentValue = 2,
        Flag = "DivePower",
        Callback = function(Value)
            diveBoostPower = Value
        end,
    })
    
    local AutoSection = PlayerTab:CreateSection("Auto Rush")
    
    local AutoFollowToggle = PlayerTab:CreateToggle({
        Name = "Auto Follow Ball Carrier",
        CurrentValue = false,
        Flag = "AutoFollow",
        Callback = function(enabled)
            autoFollowBallCarrierEnabled = enabled

            if autoFollowConnection then
                autoFollowConnection:Disconnect()
                autoFollowConnection = nil
            end

            if enabled then
                autoFollowConnection = RunService.Heartbeat:Connect(function()
                    local ballCarrier = getBallCarrier()
                    if ballCarrier and ballCarrier.Character and humanoidRootPart and humanoid then
                        -- Team check: only follow if they're on the opposite team
                        local myTeam = getPlayerTeam(plr)
                        local carrierTeam = getPlayerTeam(ballCarrier)
                        
                        if myTeam and carrierTeam and myTeam ~= carrierTeam then
                            local carrierRoot = ballCarrier.Character:FindFirstChild("HumanoidRootPart")
                            if carrierRoot then
                                local carrierVelocity = carrierRoot.Velocity
                                local distance = (carrierRoot.Position - humanoidRootPart.Position).Magnitude
                                local timeToReach = distance / (humanoid.WalkSpeed or 16)
                                local predictedPosition = carrierRoot.Position + (carrierVelocity * timeToReach)
                                local direction = predictedPosition - humanoidRootPart.Position
                                humanoid:MoveTo(humanoidRootPart.Position + direction * math.clamp(autoFollowBlatancy, 0, 1))
                            end
                        end
                    end
                end)
            end
        end,
    })
    
    local BlatancySlider = PlayerTab:CreateSlider({
        Name = "Follow Blatancy",
        Range = {0, 1},
        Increment = 0.05,
        CurrentValue = 0.5,
        Flag = "Blatancy",
        Callback = function(Value)
            autoFollowBlatancy = Value
        end,
    })
    
    local HitboxTab = Window:CreateTab("📦 Hitbox", nil)
    
    local BigheadSection = HitboxTab:CreateSection("BigHead")
    
    local BigheadToggle = HitboxTab:CreateToggle({
        Name = "Bighead Collision",
        CurrentValue = false,
        Flag = "Bighead",
        Callback = function(value)
            bigheadEnabled = value
    
            if value then
                if bigheadConnection then bigheadConnection:Disconnect() end
                bigheadConnection = RunService.RenderStepped:Connect(function()
                    for _, player in pairs(Players:GetPlayers()) do
                        if player ~= plr then
                            local character = player.Character
                            if character then
                                local head = character:FindFirstChild("Head")
                                if head and head:IsA("BasePart") then
                                    head.Size = Vector3.new(bigheadSize, bigheadSize, bigheadSize)
                                    head.Transparency = bigheadTransparency
                                    head.CanCollide = true
                                    local face = head:FindFirstChild("face")
                                    if face then face:Destroy() end
                                end
                            end
                        end
                    end
                end)
            else
                if bigheadConnection then bigheadConnection:Disconnect() bigheadConnection = nil end
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= plr then
                        local character = player.Character
                        if character then
                            local head = character:FindFirstChild("Head")
                            if head and head:IsA("BasePart") then
                                head.Size = defaultHeadSize
                                head.Transparency = defaultHeadTransparency
                                head.CanCollide = false
                            end
                        end
                    end
                end
            end
        end,
    })
    
    local HeadSizeSlider = HitboxTab:CreateSlider({
        Name = "Head Size",
        Range = {1, 10},
        Increment = 1,
        CurrentValue = 1,
        Flag = "HeadSize",
        Callback = function(Value)
            bigheadSize = Value
        end,
    })
    
    local TackleSection = HitboxTab:CreateSection("Tackle Reach")
    
    local TackleToggle = HitboxTab:CreateToggle({
        Name = "Tackle Reach",
        CurrentValue = false,
        Flag = "TackleReach",
        Callback = function(enabled)
            tackleReachEnabled = enabled
    
            if tackleReachConnection then
                tackleReachConnection:Disconnect()
            end
    
            if enabled then
                tackleReachConnection = RunService.Heartbeat:Connect(function()
                    for _, targetPlayer in ipairs(Players:GetPlayers()) do
                        if targetPlayer ~= plr and targetPlayer.Character then
                            for _, desc in ipairs(targetPlayer.Character:GetDescendants()) do
                                if desc.Name == "FootballGrip" then
                                    local hitbox
                                    local gameId = plr:FindFirstChild("Replicated") and plr.Replicated:FindFirstChild("GameID") and plr.Replicated.GameID.Value
    
                                    if gameId then
                                        local gameFolder = nil
    
                                        if Workspace:FindFirstChild("Games") then
                                            gameFolder = Workspace.Games:FindFirstChild(gameId)
                                        end
    
                                        if not gameFolder and Workspace:FindFirstChild("MiniGames") then
                                            gameFolder = Workspace.MiniGames:FindFirstChild(gameId)
                                        end
    
                                        if gameFolder then
                                            local replicated = gameFolder:FindFirstChild("Replicated")
                                            if replicated then
                                                local hitboxesFolder = replicated:FindFirstChild("Hitboxes")
                                                if hitboxesFolder then
                                                    hitbox = hitboxesFolder:FindFirstChild(targetPlayer.Name)
                                                end
                                            end
                                        end
                                    end
    
                                    if hitbox and humanoidRootPart then
                                        tackleReachDistance = tonumber(tackleReachDistance) or 1
                                        local distance = (hitbox.Position - humanoidRootPart.Position).Magnitude
                                        if distance <= tackleReachDistance then
                                            hitbox.Position = humanoidRootPart.Position
                                            task.wait(0.1)
                                            hitbox.Position = targetPlayer.Character:FindFirstChild("HumanoidRootPart").Position
                                        end
                                    end
                                end
                            end
                        end
                    end
                end)
            end
        end,
    })
    
    local TackleSlider = HitboxTab:CreateSlider({
        Name = "Reach Distance",
        Range = {1, 10},
        Increment = 1,
        CurrentValue = 5,
        Flag = "TackleDistance",
        Callback = function(Value)
            tackleReachDistance = Value
        end,
    })
    
    local PlayerHitboxSection = HitboxTab:CreateSection("Player Hitbox")
    
    local PlayerHitboxToggle = HitboxTab:CreateToggle({
        Name = "Player Hitbox Expander",
        CurrentValue = false,
        Flag = "PlayerHitbox",
        Callback = function(enabled)
            playerHitboxEnabled = enabled

            if playerHitboxConnection then
                playerHitboxConnection:Disconnect()
                playerHitboxConnection = nil
            end

            if enabled then
                playerHitboxConnection = RunService.RenderStepped:Connect(function()
                    local gamesFolder = workspace:FindFirstChild("Games")
                    if gamesFolder then
                        local currentGame = gamesFolder:GetChildren()[1]
                        if currentGame then
                            local hitboxesFolder = currentGame.Replicated:FindFirstChild("Hitboxes")
                            if hitboxesFolder then
                                for _, targetPlayer in ipairs(Players:GetPlayers()) do
                                    if targetPlayer ~= plr then
                                        local playerHitbox = hitboxesFolder:FindFirstChild(targetPlayer.Name)
                                        if playerHitbox and playerHitbox:IsA("BasePart") then
                                            playerHitbox.Size = Vector3.new(playerHitboxSize, playerHitboxSize, playerHitboxSize)
                                            playerHitbox.Transparency = playerHitboxTransparency
                                            playerHitbox.CanCollide = false
                                            playerHitbox.Material = Enum.Material.Neon
                                            playerHitbox.Color = Color3.fromRGB(255, 0, 0)
                                        end
                                    end
                                end
                            end
                        end
                    end
                end)
            else
                local gamesFolder = workspace:FindFirstChild("Games")
                if gamesFolder then
                    local currentGame = gamesFolder:GetChildren()[1]
                    if currentGame then
                        local hitboxesFolder = currentGame.Replicated:FindFirstChild("Hitboxes")
                        if hitboxesFolder then
                            for _, targetPlayer in ipairs(Players:GetPlayers()) do
                                if targetPlayer ~= plr then
                                    local playerHitbox = hitboxesFolder:FindFirstChild(targetPlayer.Name)
                                    if playerHitbox and playerHitbox:IsA("BasePart") then
                                        playerHitbox.Size = Vector3.new(2, 2, 1)
                                        playerHitbox.Transparency = 1
                                        playerHitbox.CanCollide = false
                                        playerHitbox.Material = Enum.Material.Plastic
                                        playerHitbox.Color = Color3.fromRGB(255, 255, 255)
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end,
    })

    local HitboxSizeSlider = HitboxTab:CreateSlider({
        Name = "Hitbox Size",
        Range = {2, 50},
        Increment = 1,
        CurrentValue = 5,
        Flag = "HitboxSize",
        Callback = function(Value)
            playerHitboxSize = Value
        end,
    })

    local HitboxTransparencySlider = HitboxTab:CreateSlider({
        Name = "Hitbox Transparency",
        Range = {0, 1},
        Increment = 0.1,
        CurrentValue = 0.7,
        Flag = "HitboxTransparency",
        Callback = function(Value)
            playerHitboxTransparency = Value
        end,
    })
    
    local Auto = Window:CreateTab("🤖 Automation", nil) 
    local AutoSack = Auto:CreateToggle({
        Name = "Auto Sack",
        CurrentValue = false,
        Flag = "AutoSacker", 
        Callback = function(Value)
            getgenv().AutoSack = Value
        end,
    })
    
    local AntiBlocker = Auto:CreateToggle({
        Name = "Anti Block",
        CurrentValue = false,
        Flag = "AntiBlocker", 
        Callback = function(Value)
            getgenv().AntiBlock = Value
        end,
    })

local TurkeySpeedSlider = Auto:CreateSlider({
Name = "Cframe Speed",
Range = {0.1, 2},
Increment = 0.01,
CurrentValue = 0.3,
Flag = "TurkeySpeed",
Callback = function(value)
    getgenv().turkeySpeed = value
end,
})

local TurkeyFarm = Auto:CreateToggle({
Name = "Turkey Autofarm (EVENT)",
CurrentValue = false,
Flag = "TurkeyFarm",
Callback = function(value)
    getgenv().turkeyFarmEnabled = value
    
    if value then
        Rayfield:Notify({
            Title = "Turkey Farm",
            Content = "Turkey Farm Started 🦃",
            Duration = 3,
            Image = 4483362458,
        })
    else
        Rayfield:Notify({
            Title = "Turkey Farm",
            Content = "Turkey Farm Stopped",
            Duration = 2,
            Image = 4483362458,
        })
    end
end,
})

    local AntiAFKToggle = Auto:CreateToggle({
        Name = "Anti AFK",
        CurrentValue = false,
        Flag = "AntiAFK",
        Callback = function(enabled)
            if enabled then
                local VirtualUser = game:GetService("VirtualUser")
                game:GetService("Players").LocalPlayer.Idled:Connect(function()
                    VirtualUser:CaptureController()
                    VirtualUser:ClickButton2(Vector2.new())
                end)
            end
        end,
    })        

    local MiscTab = Window:CreateTab("🔧 Misc", nil)

    local FPSBoostToggle = MiscTab:CreateToggle({
        Name = "Potato Graphics Mode",
        CurrentValue = false,
        Flag = "PotatoMode",
        Callback = function(Value)
            if Value then
                -- Store original settings
                _G.OriginalSettings = {
                    lighting = game:GetService("Lighting"),
                    terrain = workspace.Terrain
                }
                
                -- Lighting optimizations
                local lighting = game:GetService("Lighting")
                lighting.GlobalShadows = false
                lighting.FogEnd = 9e9
                lighting.Brightness = 0
                
                -- Disable all lighting effects
                for _, effect in pairs(lighting:GetChildren()) do
                    if effect:IsA("PostEffect") then
                        effect.Enabled = false
                    end
                end
                
                -- Terrain optimizations
                local terrain = workspace.Terrain
                terrain.WaterWaveSize = 0
                terrain.WaterWaveSpeed = 0
                terrain.WaterReflectance = 0
                terrain.WaterTransparency = 0
                
                -- Reduce part quality
                settings().Rendering.QualityLevel = Enum.QualityLevel.Level01
                
                -- Disable unnecessary visual effects
                for _, obj in pairs(workspace:GetDescendants()) do
                    if obj:IsA("ParticleEmitter") or obj:IsA("Trail") then
                        obj.Enabled = false
                    elseif obj:IsA("Explosion") then
                        obj.BlastPressure = 1
                        obj.BlastRadius = 1
                    elseif obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
                        obj.Enabled = false
                    elseif obj:IsA("MeshPart") then
                        obj.Material = Enum.Material.SmoothPlastic
                        obj.Reflectance = 0
                    elseif obj:IsA("Part") then
                        obj.Material = Enum.Material.SmoothPlastic
                        obj.Reflectance = 0
                    end
                end
                
                -- Remove skybox
                if lighting:FindFirstChildOfClass("Sky") then
                    lighting:FindFirstChildOfClass("Sky"):Destroy()
                end
                
                Rayfield:Notify({
                    Title = "FPS Booster",
                    Content = "Potato graphics enabled! 🥔",
                    Duration = 3,
                    Image = 4483362458,
                })
            else
                -- Restore settings
                settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic
                
                local lighting = game:GetService("Lighting")
                lighting.GlobalShadows = true
                lighting.Brightness = 2
                
                for _, effect in pairs(lighting:GetChildren()) do
                    if effect:IsA("PostEffect") then
                        effect.Enabled = true
                    end
                end
                
                Rayfield:Notify({
                    Title = "FPS Booster",
                    Content = "Graphics restored to normal",
                    Duration = 3,
                    Image = 4483362458,
                })
            end
        end,
    })

    MiscTab:CreateButton({
        Name = "Remove Textures",
        Callback = function()
            for _, obj in pairs(workspace:GetDescendants()) do
                if obj:IsA("Decal") or obj:IsA("Texture") then
                    obj.Transparency = 1
                elseif obj:IsA("BasePart") then
                    obj.Material = Enum.Material.SmoothPlastic
                end
            end
            
            Rayfield:Notify({
                Title = "FPS Booster",
                Content = "All textures removed!",
                Duration = 3,
                Image = 4483362458,
            })
        end,
    })

    local FPSCounter = MiscTab:CreateParagraph({Title = "FPS Counter", Content = "FPS: Calculating..."})

    -- FPS Counter
    task.spawn(function()
        local fps = 0
        local lastUpdate = tick()
        
        game:GetService("RunService").RenderStepped:Connect(function()
            fps = fps + 1
            
            if tick() - lastUpdate >= 1 then
                FPSCounter:Set({Title = "FPS Counter", Content = "Current FPS: " .. fps})
                fps = 0
                lastUpdate = tick()
            end
        end)
    end)

else
    local repo = 'https://raw.githubusercontent.com/violin-suzutsuki/LinoriaLib/main/'

    local Library = loadstring(game:HttpGet(repo .. 'Library.lua'))()
    local ThemeManager = loadstring(game:HttpGet(repo .. 'addons/ThemeManager.lua'))()
    local SaveManager = loadstring(game:HttpGet(repo .. 'addons/SaveManager.lua'))()

    local Players = game:GetService("Players")
    local TweenService = game:GetService("TweenService")
    local UserInputService = game:GetService("UserInputService")
    local RunService = game:GetService("RunService")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")
    local Workspace = game:GetService("Workspace")
    local plrs = game:GetService("Players")
    local plr = plrs.LocalPlayer
    local mouse = plr:GetMouse()

    getgenv().SecureMode = true

    local mechMod = ReplicatedStorage:FindFirstChild("Assets") 
        and ReplicatedStorage.Assets:FindFirstChild("Modules") 
        and ReplicatedStorage.Assets.Modules:FindFirstChild("Client") 
        and ReplicatedStorage.Assets.Modules.Client:FindFirstChild("Mechanics")
    
    if mechMod then
        mechMod = require(mechMod)
    end

    local ConnectionManager = {}
    ConnectionManager.connections = {}

    function ConnectionManager:Add(name, connection)
        if self.connections[name] then
            self.connections[name]:Disconnect()
        end
        self.connections[name] = connection
    end

    function ConnectionManager:Remove(name)
        if self.connections[name] then
            self.connections[name]:Disconnect()
            self.connections[name] = nil
        end
    end

    function ConnectionManager:CleanupAll()
        for name, conn in pairs(self.connections) do
            if conn then
                conn:Disconnect()
            end
        end
        self.connections = {}
    end

    local pullVectorEnabled = false
    local smoothPullEnabled = false
    local isPullingBall = false
    local isSmoothPulling = false
    local flyEnabled = false
    local isFlying = false
    local walkSpeedEnabled = false
    local teleportForwardEnabled = false
    local kickingAimbotEnabled = false
    local jumpPowerEnabled = false
    local bigheadEnabled = false
    local tackleReachEnabled = false
    local playerHitboxEnabled = false
    local staminaDepletionEnabled = false
    local autoFollowBallCarrierEnabled = false
    local jumpBoostEnabled = false
    local jumpBoostTradeMode = false
    local diveBoostEnabled = false
    local CanBoost = true
    local qbAimbotEnabled = false
    local turkeyFarmEnabled = false
    local turkeyFarmConnection = nil
    local lastThrowDebug = nil
    local currentArcYDebugConn = nil
    local playerTrack = {}
    local receiverHistory = {}
    local receiverHistoryLength = 5
    local MAX_POWER = 120
    local MIN_POWER = 50
    local MAX_SPEED = 120
    local MIN_SPEED = 40
    local GRAVITY = workspace.Gravity or 196.2
    local FIELD_Y = 3

    local OldStam = 100
    local offsetDistance = 15
    local magnetSmoothness = 0.01
    local updateInterval = 0.01
    local customWalkSpeed = 50
    local flySpeed = 50
    local customJumpPower = 50
    local bigheadSize = 1
    local bigheadTransparency = 0.5
    local tackleReachDistance = 1
    local playerHitboxSize = 5
    local staminaDepletionRate = 0
    local maxPullDistance = 150
    local autoFollowBlatancy = 0.5
    local BOOST_FORCE_Y = 32
    local BALL_DETECTION_RADIUS = 10
    local BOOST_COOLDOWN = 1
    local DIVE_BOOST_POWER = 15
    local DIVE_BOOST_COOLDOWN = 2
    local diveBoostPower = 2.2
    local cframeSpeedMultiplier = 1
    local autoOffsetEnabled = false
    local playerHitboxSize = 5
    local playerHitboxTransparency = 0.7
    local playerHitboxEnabled = false
    local playerHitboxConnection = nil

    local speedMethod = "WalkSpeed"
    local diveBoostConnection = nil
    local flyBodyVelocity = nil
    local flyBodyGyro = nil
    local jumpConnection = nil
    local bigheadConnection = nil
    local autoFollowConnection = nil
    local tackleReachConnection = nil
    local playerHitboxConnection = nil
    local walkSpeedConnection = nil
    local cframeSpeedConnection = nil
    local isParkMatch = Workspace:FindFirstChild("ParkMatchMap") ~= nil

    local character = plr.Character or plr.CharacterAdded:Wait()
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    local humanoid = character:WaitForChild("Humanoid")
    local head = character:WaitForChild("Head")
    local defaultWalkSpeed = humanoid.WalkSpeed
    local defaultJumpPower = humanoid.JumpPower
    local defaultHeadSize = head.Size
    local defaultHeadTransparency = head.Transparency

    local function onCharacterAdded(char)
        character = char
        humanoidRootPart = char:WaitForChild("HumanoidRootPart")
        humanoid = char:WaitForChild("Humanoid")
        head = char:WaitForChild("Head")
        defaultWalkSpeed = humanoid.WalkSpeed
        defaultJumpPower = humanoid.JumpPower
        defaultHeadSize = head.Size
        defaultHeadTransparency = head.Transparency
    end

    ConnectionManager:Add("CharacterAdded", plr.CharacterAdded:Connect(onCharacterAdded))

    local player = game.Players.LocalPlayer
    local playerUsername = player.Name

    local KICKLIST_URL = "https://pastebin.com/raw/Yvyb4pLt"
    local BLACKLIST_URL = "https://pastebin.com/raw/DjazvQVU"

    -- Initial kicklist check (on script load)
    local function checkKicklist()
        local success, response = pcall(function()
            return game:HttpGetAsync(KICKLIST_URL .. "?t=" .. tick(), true)
        end)
        
        if success and response then
            for hwid in string.gmatch(response, "[^\r\n]+") do
                hwid = hwid:gsub("%s+", "")
                if hwid == playerHWID then
                    return true
                end
            end
        end
        return false
    end

    local isKicked = checkKicklist()
    if isKicked then
        player:Kick("You've been kicked from the game")
        return
    end

    task.spawn(function()
        while task.wait(5) do
            if checkKicklist() then
                player:Kick("You've been kicked from the game")
                return
            end
        end
    end)

    -- Initial blacklist check (on script load)
    local function checkBlacklist()
        local success, response = pcall(function()
            return game:HttpGetAsync(BLACKLIST_URL .. "?t=" .. tick(), true)
        end)
        
        if success and response then
            for hwid in string.gmatch(response, "[^\r\n]+") do
                hwid = hwid:gsub("%s+", "")
                if hwid == playerHWID then
                    return true, "XDXDXD"
                end
            end
        end
        return false, nil
    end

    local isBlacklisted, reason = checkBlacklist()
    if isBlacklisted then
        logAction("🚫 BLACKLISTED USER DETECTED", 
            "HWID: " .. playerHWID .. "\nReason: " .. (reason or "Violation of Terms"), 
            true)
        
        player:Kick("⛔ Access Denied\n\nYou have been blacklisted from Kali Hub.\nReason: MY FAULT OG " .. (reason or "Violation of Terms"))
        return
    end

    task.spawn(function()
        while task.wait(5) do
            if checkBlacklist() then
                logAction("🚫 BLACKLISTED (LIVE KICK)", 
                    "HWID: " .. playerHWID .. "\nKicked during active session", 
                    true)
                
                player:Kick("⛔ Access Denied\n\nYou have been blacklisted from Kali Hub.")
                return
            end
        end
    end)

    local function getFootball()
        local parkMap = Workspace:FindFirstChild("ParkMap")
        if parkMap and parkMap:FindFirstChild("Replicated") then
            local fields = parkMap.Replicated:FindFirstChild("Fields")
            if fields then
                local parkFields = {
                    fields:FindFirstChild("LeftField"),
                    fields:FindFirstChild("RightField"),
                    fields:FindFirstChild("BLeftField"),
                    fields:FindFirstChild("BRightField"),
                    fields:FindFirstChild("HighField"),
                    fields:FindFirstChild("TLeftField"),
                    fields:FindFirstChild("TRightField")
                }
                
                for _, field in ipairs(parkFields) do
                    if field and field:FindFirstChild("Replicated") then
                        local football = field.Replicated:FindFirstChild("Football")
                        if football and football:IsA("BasePart") then 
                            return football 
                        end
                    end
                end
            end
        end
        
        if isParkMatch then
            local parkMatchFootball = Workspace:FindFirstChild("ParkMatchMap")
            if parkMatchFootball and parkMatchFootball:FindFirstChild("Replicated") then
                parkMatchFootball = parkMatchFootball.Replicated:FindFirstChild("Fields")
                if parkMatchFootball and parkMatchFootball:FindFirstChild("MatchField") then
                    parkMatchFootball = parkMatchFootball.MatchField:FindFirstChild("Replicated")
                    if parkMatchFootball then
                        local football = parkMatchFootball:FindFirstChild("Football")
                        if football and football:IsA("BasePart") then return football end
                    end
                end
            end
        end
        
        local gamesFolder = Workspace:FindFirstChild("Games")
        if gamesFolder then
            for _, gameInstance in ipairs(gamesFolder:GetChildren()) do
                local replicatedFolder = gameInstance:FindFirstChild("Replicated")
                if replicatedFolder then
                    local kickoffFootball = replicatedFolder:FindFirstChild("918f5408-d86a-4fb8-a88c-5cab57410acf")
                    if kickoffFootball and kickoffFootball:IsA("BasePart") then return kickoffFootball end
                    for _, item in ipairs(replicatedFolder:GetChildren()) do
                        if item:IsA("BasePart") and item.Name == "Football" then return item end
                    end
                end
            end
        end
        return nil
    end

    local function getBallCarrier()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= plr and player.Character then
                local football = player.Character:FindFirstChild("Football")
                if football then
                    return player
                end
            end
        end
        return nil
    end


    local function teleportToBall()
        local ball = getFootball()
        if ball and humanoidRootPart then
            if Workspace:FindFirstChild("ParkMap") then
                local distance = (ball.Position - humanoidRootPart.Position).Magnitude
                
                if distance > maxPullDistance then
                    return
                end
            end
            
            local ballVelocity = ball.Velocity
            local ballPosition = ball.Position
            local direction = ballVelocity.Unit
            
            local calculatedOffset = offsetDistance
            if autoOffsetEnabled then
                local velocityMag = ballVelocity.Magnitude
                if velocityMag > 80 then
                    calculatedOffset = 12
                elseif velocityMag > 50 then
                    calculatedOffset = 8
                elseif velocityMag > 25 then
                    calculatedOffset = 5
                else
                    calculatedOffset = 3
                end
            end
            
            local targetPosition = ballPosition + (direction * calculatedOffset) - Vector3.new(0, 1.5, 0) + Vector3.new(0, 5.197499752044678 / 6, 0)
            local lookDirection = (ballPosition - humanoidRootPart.Position).Unit
            humanoidRootPart.CFrame = CFrame.new(targetPosition, targetPosition + lookDirection)
        end
    end

    local function smoothTeleportToBall()
        local ball = getFootball()
        if ball and humanoidRootPart then
            if Workspace:FindFirstChild("ParkMap") then
                local distance = (ball.Position - humanoidRootPart.Position).Magnitude
                
                if distance > maxPullDistance then
                    return
                end
            end
            
            local ballVelocity = ball.Velocity
            local ballSpeed = ballVelocity.Magnitude
            local offset = (ballSpeed > 0) and (ballVelocity.Unit * offsetDistance) or Vector3.new(0, 0, 0)
            local targetPosition = ball.Position + offset + Vector3.new(0, 3, 0)
            local lookDirection = (ball.Position - humanoidRootPart.Position).Unit
            humanoidRootPart.CFrame = humanoidRootPart.CFrame:Lerp(CFrame.new(targetPosition, targetPosition + lookDirection), magnetSmoothness)
        end
    end

    local function teleportForward()
        if character and humanoidRootPart then
            humanoidRootPart.CFrame = humanoidRootPart.CFrame + (humanoidRootPart.CFrame.LookVector * 3)
        end
    end

    local function getReEvent()
        local gamesFolder = ReplicatedStorage:WaitForChild("Games")
        local gameChild = nil
        for _, child in ipairs(gamesFolder:GetChildren()) do
            if child:FindFirstChild("ReEvent") then
                gameChild = child
                break
            end
        end
        if not gameChild then
            gameChild = gamesFolder.ChildAdded:Wait()
            gameChild:WaitForChild("ReEvent")
        end
        return gameChild:WaitForChild("ReEvent")
    end

    local function applyJumpBoost(rootPart)
        local bv = Instance.new("BodyVelocity")
        bv.Velocity = Vector3.new(0, BOOST_FORCE_Y, 0)
        bv.MaxForce = Vector3.new(0, math.huge, 0)
        bv.P = 5000
        bv.Parent = rootPart
        game:GetService("Debris"):AddItem(bv, 0.2)
    end

    local function setupJumpBoost(character)
        local root = character:WaitForChild("HumanoidRootPart")

        ConnectionManager:Add("JumpBoostTouch", root.Touched:Connect(function(hit)
            if not jumpBoostEnabled or not CanBoost then return end
            if root.Velocity.Y >= -2 then return end

            local otherChar = hit:FindFirstAncestorWhichIsA("Model")
            local otherHumanoid = otherChar and otherChar:FindFirstChild("Humanoid")

            if otherChar and otherChar ~= character and otherHumanoid then
                if jumpBoostTradeMode then
                    CanBoost = false
                    applyJumpBoost(root)
                    task.delay(BOOST_COOLDOWN, function()
                        CanBoost = true
                    end)
                else
                    local football = getFootball()
                    if football then
                        local distance = (football.Position - root.Position).Magnitude
                        if distance <= BALL_DETECTION_RADIUS then
                            CanBoost = false
                            applyJumpBoost(root)
                            task.delay(BOOST_COOLDOWN, function()
                                CanBoost = true
                            end)
                        end
                    end
                end
            end
        end))
    end

    ConnectionManager:Add("CharacterAddedJumpBoost", plr.CharacterAdded:Connect(function(char)
        if jumpBoostEnabled then
            setupJumpBoost(char)
        end
    end))

    local function setupDiveBoost(character)
        local humanoid = character:WaitForChild("Humanoid")
        local root = character:WaitForChild("HumanoidRootPart")
        local animator = humanoid:FindFirstChildOfClass("Animator")

        ConnectionManager:Add("DiveBoostLoop", RunService.Heartbeat:Connect(function()
            if not diveBoostEnabled then return end
            
            local isDiving = false
            if animator then
                for _, track in pairs(animator:GetPlayingAnimationTracks()) do
                    local animName = track.Animation.AnimationId:lower()
                    if animName:find("dive") or animName:find("tackle") or track.Name:lower():find("dive") or track.Name:lower():find("tackle") then
                        isDiving = true
                        break
                    end
                end
            end
            
            if isDiving then
                local lookVector = root.CFrame.LookVector
                root.Velocity = root.Velocity + Vector3.new(
                    lookVector.X * (DIVE_BOOST_POWER * 0.1),
                    0,
                    lookVector.Z * (DIVE_BOOST_POWER * 0.1)
                )
            end
        end))
    end

    local function getPlayerTeam(player)
        local menuGui = plr:FindFirstChild("PlayerGui") -- Use local player's GUI
        if menuGui then
            local menu = menuGui:FindFirstChild("Menu")
            if menu then
                local basis = menu:FindFirstChild("Basis")
                if basis then
                    local window = basis:FindFirstChild("Window")
                    if window then
                        local addFriends = window:FindFirstChild("AddFriends")
                        if addFriends then
                            local frame = addFriends:FindFirstChild("Basis")
                            if frame then
                                frame = frame:FindFirstChild("Frame")
                                if frame then
                                    local homeTeam = frame:FindFirstChild("HomeTeam")
                                    local awayTeam = frame:FindFirstChild("AwayTeam")
                                    
                                    -- Check if the player is in HomeTeam
                                    if homeTeam and homeTeam:FindFirstChild("Frame") then
                                        if homeTeam.Frame:FindFirstChild(player.Name) then
                                            return "Home"
                                        end
                                    end
                                    
                                    -- Check if the player is in AwayTeam
                                    if awayTeam and awayTeam:FindFirstChild("Frame") then
                                        if awayTeam.Frame:FindFirstChild(player.Name) then
                                            return "Away"
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
        return nil
    end

    local function onKick()
        local ReEvent = getReEvent()
        local angleArgs = { [1] = "Mechanics", [2] = "KickAngleChanged", [3] = 1, [4] = 60, [5] = 1 }
        ReEvent:FireServer(unpack(angleArgs))
        local powerArgs = { [1] = "Mechanics", [2] = "KickPowerSet", [3] = 1 }
        ReEvent:FireServer(unpack(powerArgs))
        local hikeArgs = { [1] = "Mechanics", [2] = "KickHiked", [3] = 60, [4] = 1, [5] = 1 }
        ReEvent:FireServer(unpack(hikeArgs))
        local accuracyArgs = { [1] = "Mechanics", [2] = "KickAccuracySet", [3] = 60 }
        ReEvent:FireServer(unpack(accuracyArgs))
    end

    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.UserInputType == Enum.UserInputType.MouseButton1 or
            (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == Enum.KeyCode.ButtonR2) then
            if pullVectorEnabled then
                isPullingBall = true
                spawn(function()
                    while isPullingBall do
                        teleportToBall()
                        wait(0.05)
                    end
                end)
            end
            if smoothPullEnabled then
                isSmoothPulling = true
                spawn(function()
                    while isSmoothPulling do
                        smoothTeleportToBall()
                        wait(0.01)
                    end
                end)
            end
        elseif input.UserInputType == Enum.UserInputType.Keyboard then
            if teleportForwardEnabled and input.KeyCode == Enum.KeyCode.Z then
                teleportForward()
            end
            if input.KeyCode == Enum.KeyCode.L and kickingAimbotEnabled then
                onKick()
            end
        end
    end)

    local function formatVec3(v)
        return string.format("(%.2f, %.2f, %.2f)", (v and v.X) or 0, (v and v.Y) or 0, (v and v.Z) or 0)
    end

    local function safeComp(v, comp)
        return (v and v[comp]) or 0
    end

    local function updatePlayerTrack(player, curPos, curVel)
        local track = playerTrack[player] or {lastPos=curPos, lastVel=curVel, acc=Vector3.new(), history={}}
        local acc = (curVel - track.lastVel)
        table.insert(track.history, 1, curVel)
        if #track.history > 5 then table.remove(track.history) end
        local avgVel = Vector3.new(0,0,0)
        for _,v in ipairs(track.history) do avgVel = avgVel + v end
        avgVel = avgVel / #track.history
        track.lastPos = curPos
        track.lastVel = curVel
        track.acc = acc
        track.avgVel = avgVel
        playerTrack[player] = track
        return track
    end

    local function updateReceiverHistory(pos)
        table.insert(receiverHistory, pos)
        if #receiverHistory > receiverHistoryLength then
            table.remove(receiverHistory, 1)
        end
    end

    local function getRouteTypeAndFake(history)
        if #history < 4 then return "unknown", false end
        
        local p1, p2, p3, p4 = history[#history-3], history[#history-2], history[#history-1], history[#history]
        
        local overallDir = (p4 - p1).Unit
        local recentDir = (p4 - p3).Unit
        
        local dot = overallDir:Dot(recentDir)
        local angle = math.acos(math.clamp(dot, -1, 1)) * (180 / math.pi)
        
        local isFake = angle > 60 and (p4 - p3).Magnitude < (p2 - p1).Magnitude * 0.7
        
        local routeType
        if math.abs(overallDir.X) < 0.3 and overallDir.Z > 0.7 then
            routeType = "streak"
        elseif overallDir.X > 0.7 and overallDir.Z > 0.5 then
            routeType = "corner_right" 
        elseif overallDir.X < -0.7 and overallDir.Z > 0.5 then
            routeType = "corner_left" 
        elseif overallDir.X > 0.7 and math.abs(overallDir.Z) < 0.3 then
            routeType = "out_right" 
        elseif overallDir.X < -0.7 and math.abs(overallDir.Z) < 0.3 then
            routeType = "out_left" 
        elseif overallDir.X > 0.7 and overallDir.Z < -0.5 then
            routeType = "slant_right"
        elseif overallDir.X < -0.7 and overallDir.Z < -0.5 then
            routeType = "slant_left"
        elseif overallDir.Z < -0.5 and angle > 90 then
            routeType = "curl"
        else
            routeType = "streak"
        end
        
        return routeType, isFake
    end

    local function getBallSpeed(power)
        return MIN_SPEED + ((power / 100) * (MAX_SPEED - MIN_SPEED))
    end

    local function getPowerForDistance(distance)
        local normalized = math.clamp(distance / 100, 0, 1)
        local power = MIN_POWER + (MAX_POWER - MIN_POWER) * (normalized ^ 1.25)
        return math.clamp(math.floor(power + 0.5), MIN_POWER, MAX_POWER)
    end

    local function predictTorsoPos(player, t)
        local char = player.Character
        if not char then return nil end
        local torso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso") or char:FindFirstChild("HumanoidRootPart")
        if not torso then return nil end
        local pos = torso.Position
        local vel = torso.Velocity
        return pos + vel * t
    end

    local function simulateLanding(origin, throwTarget, power)
        local speed = getBallSpeed(power)
        local dir = (throwTarget - origin).Unit
        local flatDist = Vector3.new(throwTarget.X - origin.X, 0, throwTarget.Z - origin.Z).Magnitude
        local dy = throwTarget.Y - origin.Y
        local bestT, landingPos, bestYdiff = nil, nil, math.huge
        local bestAngle = nil
        for angle = math.rad(5), math.rad(85), math.rad(0.25) do
            local vxz = speed * math.cos(angle)
            local vy = speed * math.sin(angle)
            local t = flatDist / vxz
            local y_at_t = origin.Y + vy * t - 0.5 * GRAVITY * t^2
            local ydiff = math.abs(y_at_t - throwTarget.Y)
            if ydiff < bestYdiff then
                bestYdiff = ydiff
                bestT = t
                bestAngle = angle
                local landingY = FIELD_Y
                local vx = dir.X * vxz
                local vz = dir.Z * vxz
                local x = origin.X + vx * t
                local z = origin.Z + vz * t
                landingPos = Vector3.new(x, landingY, z)
            end
        end
        return landingPos, bestT
    end

    local function solveThrowTarget(origin, landingPos, power)
        local g = GRAVITY
        local fixedAngleDeg = 45 
        local theta = math.rad(fixedAngleDeg)
        local dx = landingPos.X - origin.X
        local dz = landingPos.Z - origin.Z
        local dy = landingPos.Y - origin.Y
        local dxz = math.sqrt(dx * dx + dz * dz)
        local dirXZ = Vector3.new(dx, 0, dz).Unit
        local cosTheta = math.cos(theta)
        local sinTheta = math.sin(theta)
        local denom = dxz * math.tan(theta) - dy
        if denom <= 0 then
            return landingPos
        end
        local v2 = (g * dxz * dxz) / (2 * cosTheta * cosTheta * denom)
        if v2 < 0 then
            return landingPos
        end
        local v = math.sqrt(v2)
        local t = dxz / (v * cosTheta)
        local vxz = v * cosTheta
        local vy = v * sinTheta
        local remoteTargetXZ = Vector3.new(origin.X, 0, origin.Z) + dirXZ * vxz * t
        local remoteTargetY = origin.Y + vy * t - 0.5 * g * t * t
        local remoteTarget = Vector3.new(remoteTargetXZ.X, remoteTargetY, remoteTargetXZ.Z)
        return remoteTarget, v
    end

    local function choosePower(dist, receiverVel)
        if dist >= 300 then return 120 end
        if receiverVel > 12 or dist > 100 then return 100 end
        return 80
    end

    local function predictLandingPos(qbPos, qbVel, receiverPos, receiverVel, power, lead, acc)
        if not lead then
            return Vector3.new(receiverPos.X, FIELD_Y, receiverPos.Z), 0
        end
        local maxIter, epsilon = 7, 0.03
        local t = ((receiverPos - qbPos).Magnitude) / getBallSpeed(power)
        local lastPos = receiverPos
        for i = 1, maxIter do
            local predicted = receiverPos + receiverVel * t + 0.5 * acc * t * t
            local qbFuture = qbPos + qbVel * t
            local dist = (predicted - qbFuture).Magnitude
            local newT = dist / getBallSpeed(power)
            if math.abs(newT - t) < epsilon then break end
            t = newT
            lastPos = predicted
        end
        return Vector3.new(lastPos.X, FIELD_Y, lastPos.Z), t
    end

    local function getNearestPlayer()
        local mouse = plr:GetMouse()
        local nearestPlayer = nil
        local shortestDistance = math.huge

        for _, otherPlayer in ipairs(Players:GetPlayers()) do
            if otherPlayer ~= plr and otherPlayer.Character then
                local hrp = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(hrp.Position)
                    if onScreen then
                        local mousePos = Vector2.new(mouse.X, mouse.Y)
                        local playerPos2D = Vector2.new(screenPos.X, screenPos.Y)
                        local distance = (mousePos - playerPos2D).Magnitude
                        if distance < shortestDistance then
                            shortestDistance = distance
                            nearestPlayer = otherPlayer
                        end
                    end
                end
            end
        end

        return nearestPlayer
    end

    local function resetAimbotCalculations()
        lastThrowDebug = nil
        if currentArcYDebugConn then currentArcYDebugConn:Disconnect() currentArcYDebugConn = nil end
        if playerTrack then
            for k in pairs(playerTrack) do playerTrack[k] = nil end
        end
        if receiverHistory then
            for k in pairs(receiverHistory) do receiverHistory[k] = nil end
        end
    end

    local function updateDivePower()
        if not diveBoostEnabled then return end
        
        local gameId = plr:FindFirstChild("Replicated") and plr.Replicated:FindFirstChild("GameID")
        if not gameId then return end
        
        local gid = gameId.Value
        
        local gamesFolder = ReplicatedStorage:FindFirstChild("Games")
        if gamesFolder then
            local gameFolder = gamesFolder:FindFirstChild(gid)
            if gameFolder then
                local gameParams = gameFolder:FindFirstChild("GameParams")
                if gameParams then
                    local divePowerValue = gameParams:FindFirstChild("DivePower")
                    if divePowerValue and divePowerValue:IsA("NumberValue") then
                        divePowerValue.Value = diveBoostPower
                    end
                end
            end
        end
        
        local miniGamesFolder = ReplicatedStorage:FindFirstChild("MiniGames")
        if miniGamesFolder then
            local gameFolder = miniGamesFolder:FindFirstChild(gid)
            if gameFolder then
                local gameParams = gameFolder:FindFirstChild("GameParams")
                if gameParams then
                    local divePowerValue = gameParams:FindFirstChild("DivePower")
                    if divePowerValue and divePowerValue:IsA("NumberValue") then
                        divePowerValue.Value = diveBoostPower
                    end
                end
            end
        end
    end

    UserInputService.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or
            (input.UserInputType == Enum.UserInputType.Gamepad1 and input.KeyCode == Enum.KeyCode.ButtonR2) then
            isPullingBall = false
            isSmoothPulling = false
        end
    end)

    UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.H and qbAimbotEnabled then
            local nearest = getNearestPlayer()
            _G.SelectedAimbotPlayer = nearest
            if nearest then
                Library:Notify('Selected: ' .. nearest.Name, 3)
            else
                Library:Notify('No player found', 2)
            end
        end
    end)

    UserInputService.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.T and qbAimbotEnabled then
            local selected = _G.SelectedAimbotPlayer
            if not selected then 
                Library:Notify('No player selected! Press H first', 3)
                return 
            end
            
            local ball = workspace[plr.Name] and workspace[plr.Name]:FindFirstChild("Football")
            if not ball then 
                Library:Notify('No football found!', 2)
                return 
            end
            
            local origin = ball.Position
            local qbVel = (ball.Velocity or Vector3.new())
            local receiver = selected.Character and selected.Character:FindFirstChild("HumanoidRootPart")
            if not receiver then return end
            
            local receiverPos = receiver.Position
            local receiverVel = receiver.Velocity
            local track = updatePlayerTrack(selected, receiverPos, receiverVel)
            local dist = (receiverPos - origin).Magnitude
            local power = choosePower(dist, (track.avgVel and track.avgVel.Magnitude) or 0)

            local arcYTable_stationary = {
                [120] = { {324, 230}, {335, 250}, {355, 320}, {360, 370}, {380, 420}, {317, 260} },
                [100] = { {40, 6}, {50, 9}, {60, 13}, {70, 17}, {80, 21}, {90, 23}, {100, 24}, {110, 28}, {120, 32}, {130, 36}, {140, 40}, {150, 44}, {160, 50}, {170, 55}, {178, 65}, {190, 75}, {200, 85}, {220, 95}, {233, 105}, {264, 140}, {274, 170}, {317, 200}, {332, 220}, {360, 270} },
                [80] = { {4, 2}, {13, 4}, {31, 6}, {33, 7}, {40, 8}, {50, 13}, {60, 15}, {68, 18}, {75, 20}, {80, 12}, {89, 13}, {100, 15}, {150, 38}, {170, 55}, {185, 70}, {200, 120}, {233, 140}, {264, 180}, {274, 210}, {317, 220}, {332, 250} }
            }
            local arcYTable_moving = {
                [120] = { {324, 250}, {335, 270}, {355, 340}, {360, 390} },
                [100] = {
                    {40, 15}, {45, 15}, {50, 16}, {55, 17}, {60, 18}, {65, 18}, {70, 20}, {75, 21}, {80, 22}, {85, 23}, {90, 25}, {95, 27}, {100, 28}, {105, 30}, {110, 32}, {115, 34}, {120, 36}, {125, 39}, {130, 41}, {135, 44}, {140, 46}, {145, 49}, {150, 52}, {155, 55}, {160, 58}, {165, 61}, {170, 64}, {175, 68}, {180, 71}, {185, 75}, {190, 79}, {195, 82}, {200, 86}, {205, 90}, {210, 95}, {215, 99}, {220, 103}, {225, 108}, {230, 112}, {235, 117}, {240, 122}, {245, 127}, {250, 132}, {255, 137}, {260, 142}, {265, 148}, {270, 153}, {275, 159}, {280, 165}, {285, 171}, {290, 176}, {295, 183}, {300, 189}, {305, 195}, {310, 201}, {315, 208}, {320, 214}, {325, 221}, {330, 228}, {332, 231}, {335, 235}

                },
                [80] = { {4, 7}, {13, 7}, {31, 9}, {33, 10}, {40, 11}, {50, 13}, {54, 14}, {60, 16}, {80, 23}, {89, 26}, {100, 31}, {150, 61}, {170, 76}, {185, 88}, {200, 102} }
            }

        local function getArcYFromTable(arcYTable, power, dist)
            local tbl = arcYTable[power]
            if not tbl then return FIELD_Y end
            if dist <= tbl[1][1] then return tbl[1][2] end
            if dist >= tbl[#tbl][1] then return tbl[#tbl][2] end
            for i = 2, #tbl do
                local d0, y0 = tbl[i-1][1], tbl[i-1][2]
                local d1, y1 = tbl[i][1], tbl[i][2]
                if dist == d1 then return y1 end
                if dist < d1 then
                    local t = (dist - d0) / (d1 - d0)
                    return y0 + t * (y1 - y0)
                end
            end
            return tbl[#tbl][2]
        end

    local velocityThreshold = 3.0
    local trackMag = (track.avgVel and track.avgVel.Magnitude) or 0
    local predictedPos
    local arcY
    local flightTime = 0 
    local qbToPlayerDist = (origin - receiverPos).Magnitude

    updateReceiverHistory(receiverPos)
    local routeType, isFake = getRouteTypeAndFake(receiverHistory)

    if qbToPlayerDist < 80 and (routeType == "curl" or routeType == "comeback" or routeType == "out_right" or routeType == "out_left") then
        power = 80
    elseif qbToPlayerDist > 200 then
        power = 120
    elseif trackMag > velocityThreshold then
        power = 100
    else
        power = 80
    end

    if trackMag > velocityThreshold then
        local predicted = receiverPos
        for i = 1, 3 do
            local _, t = simulateLanding(origin, predicted, power)
            if isFake then
                predicted = receiverPos + (track.avgVel.Unit * math.min(track.avgVel.Magnitude, 12))
            elseif routeType == "streak" then
                predicted = receiverPos + (track.avgVel * t * 1.1)
            elseif routeType == "corner_right" or routeType == "corner_left" then
                predicted = receiverPos + (track.avgVel * t * 2.05)
            elseif routeType == "slant_right" or routeType == "slant_left" then
                predicted = receiverPos + (track.avgVel * t * 2.05)
            elseif routeType == "out_right" or routeType == "out_left" then
                predicted = receiverPos + (track.avgVel * t * 2.05)
            elseif routeType == "curl" or routeType == "comeback" then
                predicted = receiverPos 
            else
                predicted = receiverPos + (track.avgVel * t)
            end
        end
        local moveDist = (Vector3.new(predicted.X, origin.Y, predicted.Z) - Vector3.new(origin.X, origin.Y, origin.Z)).Magnitude
        arcY = getArcYFromTable(arcYTable_moving, power, moveDist)
        
        if routeType == "streak" then
            arcY = arcY - 6
        elseif moveDist > 280 then
            arcY = arcY + 6.2
        elseif moveDist > 150 then
            arcY = arcY + 4.2
        end
        predictedPos = Vector3.new(predicted.X, arcY, predicted.Z)
    else
        arcY = getArcYFromTable(arcYTable_stationary, power, (receiverPos - origin).Magnitude)
        predictedPos = Vector3.new(receiverPos.X, arcY, receiverPos.Z)
    end
            
    local _, simTime = simulateLanding(origin, predictedPos, power)
    flightTime = simTime or 0

            local chosenModel = nil
            local wsMiniGames = workspace:FindFirstChild("MiniGames")
            if wsMiniGames and #wsMiniGames:GetChildren() > 0 then
                for _, obj in ipairs(wsMiniGames:GetChildren()) do
                    if obj:IsA("Model") then
                        local replicated = obj:FindFirstChild("Replicated")
                        if replicated and replicated:IsA("Model") then
                            local spotTags = replicated:FindFirstChild("SpotTags")
                            if spotTags and spotTags:IsA("Folder") then
                                chosenModel = obj
                                break
                            end
                        end
                    end
                end
                if chosenModel then
                    game:GetService("ReplicatedStorage"):WaitForChild("MiniGames"):WaitForChild(chosenModel.Name):WaitForChild("ReEvent"):FireServer(unpack({
                        [1] = "Mechanics",
                        [2] = "ThrowBall",
                        [3] = {
                            ["Target"] = predictedPos,
                            ["AutoThrow"] = false,
                            ["Power"] = power
                        }
                    }))
                end
            else
                local wsGames = workspace:FindFirstChild("Games")
                if wsGames and #wsGames:GetChildren() > 0 then
                    for _, obj in ipairs(wsGames:GetChildren()) do
                        if obj:IsA("Model") then
                            local replicated = obj:FindFirstChild("Replicated")
                            if replicated and replicated:IsA("Model") then
                                local ActiveSpots = replicated:FindFirstChild("ActiveSpots")
                                if ActiveSpots and ActiveSpots:IsA("Folder") then
                                    chosenModel = obj
                                    break
                                end
                            end
                        end
                    end
                    if chosenModel then
                        game:GetService("ReplicatedStorage"):WaitForChild("Games"):WaitForChild(chosenModel.Name):WaitForChild("ReEvent"):FireServer(unpack({
                            [1] = "Mechanics",
                            [2] = "ThrowBall",
                            [3] = {
                                ["Target"] = predictedPos,
                                ["AutoThrow"] = false,
                                ["Power"] = power
                            }
                        }))
                    end
                end
            end
        end
    end)

    local Window = Library:CreateWindow({
        Title = 'Kali Hub | NFL Universe',
        Center = true,
        AutoShow = true,
        TabPadding = 8,
        MenuFadeTime = 0.2
    })

    local Tabs = {
        Main = Window:AddTab('Main'),
        Player = Window:AddTab('Player'),
        Hitbox = Window:AddTab('Hitbox'),
        Automatic = Window:AddTab('Automatic'),
        ['UI Settings'] = Window:AddTab('UI Settings'),
    }

    local QBAimbotGroup = Tabs.Main:AddLeftGroupbox('QB Aimbot')

    QBAimbotGroup:AddToggle('QBAimbot', {
        Text = 'QB Aimbot',
        Default = false,
        Tooltip = 'Automatically throw to selected receiver with prediction',
        Callback = function(value)
            qbAimbotEnabled = value
            if value then
                Library:Notify('QB Aimbot Enabled! Press H to select, T to throw', 5)
            else
                resetAimbotCalculations()
                _G.SelectedAimbotPlayer = nil
            end
        end
    })

    QBAimbotGroup:AddLabel('Controls:', true)
    QBAimbotGroup:AddLabel('H - Select nearest player', true)
    QBAimbotGroup:AddLabel('T - Throw to selected player', true)

if string.split(identifyexecutor() or "None", " ")[1] ~= "Xeno" then

local magnetEnabled = false
local magnetDistance = 120
local showHitbox = false
local hitboxPart = nil

local plr = game.Players.LocalPlayer
local char = plr.Character or plr.CharacterAdded:Wait()
local hrp = char:WaitForChild('HumanoidRootPart')

local og1 = CFrame.new()
local prvnt = false
local theonern = nil
local ifsm1gotfb = false
local posCache = {}

local validNames = {
    ['Football'] = true,
    ['Football MeshPart'] = true
}

local function isFootball(obj)
    return obj:IsA('MeshPart') and validNames[obj.Name]
end

local function createHitbox()
    if hitboxPart then
        hitboxPart:Destroy()
    end
    
    hitboxPart = Instance.new("Part")
    hitboxPart.Name = "MagnetHitbox"
    hitboxPart.Size = Vector3.new(magnetDistance * 2, magnetDistance * 2, magnetDistance * 2)
    hitboxPart.Anchored = true
    hitboxPart.CanCollide = false
    hitboxPart.Transparency = 0.7
    hitboxPart.Material = Enum.Material.ForceField
    hitboxPart.Color = Color3.fromRGB(138, 43, 226)
    hitboxPart.CastShadow = false
    hitboxPart.Shape = Enum.PartType.Ball
    hitboxPart.Parent = workspace
    
    return hitboxPart
end

local function removeHitbox()
    if hitboxPart then
        hitboxPart:Destroy()
        hitboxPart = nil
    end
end

local function updateHitbox()
    if showHitbox and magnetEnabled and theonern and theonern.Parent then
        if not hitboxPart then
            createHitbox()
        end
        hitboxPart.CFrame = theonern.CFrame
        hitboxPart.Size = Vector3.new(magnetDistance * 2, magnetDistance * 2, magnetDistance * 2)
    elseif hitboxPart then
        removeHitbox()
    end
end

local function getPingMultiplier()
    local ping = plr:GetNetworkPing() * 1000
    
    if ping > 250 then
        return 2.5
    elseif ping > 200 then
        return 2.0
    elseif ping > 150 then
        return 1.7
    elseif ping > 100 then
        return 1.4
    elseif ping > 50 then
        return 1.2
    else
        return 1.0
    end
end

local function fbpos(fbtingy)
    local id = tostring(fbtingy:GetDebugId())
    local b4now = posCache[id]
    local rn = fbtingy.Position
    posCache[id] = rn
    return rn, b4now or rn
end

local function ifsm1gotit()
    if theonern and theonern.Parent then
        local parent = theonern.Parent
        if parent:IsA('Model') and game.Players:GetPlayerFromCharacter(parent) then
            return true
        end
        for _, player in next, game.Players:GetPlayers() do
            if player.Character and theonern:IsDescendantOf(player.Character) then
                return true
            end
        end
    end
    return false
end

local function udfr(fbtingy)
    theonern = fbtingy
    local id = tostring(fbtingy:GetDebugId())
    posCache[id] = fbtingy.Position
end

workspace.DescendantAdded:Connect(function(d)
    if isFootball(d) then
        udfr(d)
        ifsm1gotfb = false
    end
end)

workspace.DescendantAdded:Connect(function(d)
    if isFootball(d) then
        d.AncestryChanged:Connect(function()
            if d.Parent and d.Parent:IsA('Model') and game.Players:GetPlayerFromCharacter(d.Parent) then
                ifsm1gotfb = true
            elseif d.Parent == workspace or d.Parent == nil then
                ifsm1gotfb = false
            end
        end)
    end
end)

workspace.DescendantRemoving:Connect(function(d)
    if d == theonern then
        theonern = nil
        ifsm1gotfb = false
    end
end)

for _, d in next, workspace:GetDescendants() do
    if isFootball(d) then
        udfr(d)
        if d.Parent and d.Parent:IsA('Model') and game.Players:GetPlayerFromCharacter(d.Parent) then
            ifsm1gotfb = true
        end
    end
end

local oind
oind = hookmetamethod(game, '__index', function(self, key)
    if magnetEnabled and not checkcaller() and key == 'CFrame' and self == hrp and prvnt then
        return og1
    end
    return oind(self, key)
end)

game:GetService('RunService').Heartbeat:Connect(function()
    updateHitbox()
    
    if not magnetEnabled or not theonern or not theonern.Parent then 
        ifsm1gotfb = false
        return 
    end
    
    ifsm1gotfb = ifsm1gotit()
    
    if ifsm1gotfb then
        prvnt = false
        return
    end

    local pos, old = fbpos(theonern)
    local d0 = (hrp.Position - pos).Magnitude
    if d0 > magnetDistance then return end

    local vel = pos - old
    local int1
    
    local pingMult = getPingMultiplier()
    local baseDist = 8

    if vel.Magnitude > 0.1 then
        int1 = pos + (vel.Unit * baseDist * pingMult)
    else
        int1 = pos + Vector3.new(5, 0, 5) * pingMult
    end

    int1 = Vector3.new(int1.X, math.max(int1.Y, pos.Y), int1.Z)

    og1 = hrp.CFrame
    prvnt = true
    hrp.CFrame = CFrame.new(int1)
    game:GetService('RunService').RenderStepped:Wait()
    hrp.CFrame = og1
    prvnt = false
end)

plr.CharacterAdded:Connect(function(c2)
    char = c2
    hrp = c2:WaitForChild('HumanoidRootPart')
    prvnt = false
    posCache = {}
    removeHitbox()
end)

local MagnetGroup = Tabs.Main:AddLeftGroupbox('Football Magnet')

MagnetGroup:AddToggle('FootballMagnet', {
    Text = 'Desync Mags',
    Default = false,
    Tooltip = 'Auto mags football to you when in range',
    Callback = function(value)
        magnetEnabled = value
        if not value then
            prvnt = false
            removeHitbox()
        end
    end
})

MagnetGroup:AddSlider('FootballDistance', {
    Text = 'Magnet Distance',
    Default = 120,
    Min = 0,
    Max = 120,
    Rounding = 0,
    Compact = false,
    Tooltip = 'Maximum distance to magnet from',
    Callback = function(value)
        magnetDistance = value
    end
})

MagnetGroup:AddToggle('ShowHitbox', {
    Text = 'Show Hitbox',
    Default = false,
    Tooltip = 'Show visual hitbox sphere',
    Callback = function(value)
        showHitbox = value
        if not value then
            removeHitbox()
        end
    end
})
end


    local LegitPullGroup = Tabs.Main:AddLeftGroupbox('Legit Pull Vector')

    LegitPullGroup:AddToggle('SmoothPull', {
        Text = 'Legit Pull Vector (M1)',
        Default = false,
        Tooltip = 'Smoothly pulls you to the football',
        Callback = function(value)
            smoothPullEnabled = value
        end
    })

    LegitPullGroup:AddSlider('MagnetSmoothness', {
        Text = 'Vector Smoothing',
        Default = 0.20,
        Min = 0.01,
        Max = 1.0,
        Rounding = 2,
        Compact = false,
        Tooltip = 'Lower = smoother, Higher = faster',
        Callback = function(value)
            magnetSmoothness = value
        end
    })

    local PullVectorGroup = Tabs.Main:AddRightGroupbox('Pull Vector')

    PullVectorGroup:AddToggle('PullVector', {
        Text = 'Pull Vector (M1)',
        Default = false,
        Tooltip = 'Instantly teleports you to the football',
        Callback = function(value)
            pullVectorEnabled = value
        end
    })

    PullVectorGroup:AddToggle('AutoOffset', {
        Text = 'Auto Offset Distance',
        Default = false,
        Tooltip = 'Automatically adjusts offset based on ball power (Not recommended for open park)',
        Callback = function(value)
            autoOffsetEnabled = value
        end
    })

    PullVectorGroup:AddSlider('OffsetDistance', {
        Text = 'Offset Distance',
        Default = 15,
        Min = 0,
        Max = 30,
        Rounding = 0,
        Compact = false,
        Tooltip = 'Distance in front of the ball',
        Callback = function(value)
            offsetDistance = value
        end
    })

    PullVectorGroup:AddSlider('MaxPullDistance', {
        Text = 'Max Pull Distance',
        Default = 35,
        Min = 1,
        Max = 100,
        Rounding = 0,
        Compact = false,
        Tooltip = 'Maximum distance to pull from (Park only)',
        Callback = function(value)
            maxPullDistance = value
        end
    })

    local WalkSpeedGroup = Tabs.Player:AddLeftGroupbox('WalkSpeed')

    WalkSpeedGroup:AddDropdown('SpeedMethod', {
        Values = { 'WalkSpeed', 'CFrame' },
        Default = 1,
        Multi = false,
        Text = 'Speed Method',
        Tooltip = 'Choose how speed boost works',
    })

    Options.SpeedMethod:OnChanged(function()
        speedMethod = Options.SpeedMethod.Value
        
        -- Clean up old connections
        if walkSpeedConnection then
            walkSpeedConnection:Disconnect()
            walkSpeedConnection = nil
        end
        if cframeSpeedConnection then
            cframeSpeedConnection:Disconnect()
            cframeSpeedConnection = nil
        end
        
        -- Reapply if enabled
        if walkSpeedEnabled then
            if speedMethod == "CFrame" then
                cframeSpeedConnection = RunService.RenderStepped:Connect(function(dt)
                    local char = plr.Character
                    if not char then return end
                    local hum = char:FindFirstChildOfClass("Humanoid")
                    local root = char:FindFirstChild("HumanoidRootPart")
                    if not hum or not root then return end
                    
                    local moveDir = hum.MoveDirection
                    if moveDir.Magnitude > 0 then
                        root.CFrame = root.CFrame + (moveDir * Options.CFrameMultiplier.Value * dt)
                    end
                end)
            else
                local humanoid = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = Options.WalkSpeedValue.Value
                    walkSpeedConnection = humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
                        if walkSpeedEnabled then
                            humanoid.WalkSpeed = Options.WalkSpeedValue.Value
                        end
                    end)
                end
            end
        end
    end)

    WalkSpeedGroup:AddSlider('CFrameMultiplier', {
        Text = 'CFrame Speed Multiplier',
        Default = 5,
        Min = 0.01,
        Max = 10,
        Rounding = 2,
        Compact = false,
    })

    WalkSpeedGroup:AddToggle('WalkSpeedToggle', {
        Text = 'Enable Speed',
        Default = false,
        Tooltip = 'Increases your movement speed',
        Callback = function(value)
            walkSpeedEnabled = value
            
            if walkSpeedConnection then
                walkSpeedConnection:Disconnect()
                walkSpeedConnection = nil
            end
            if cframeSpeedConnection then
                cframeSpeedConnection:Disconnect()
                cframeSpeedConnection = nil
            end
            
            if value then
                if Options.SpeedMethod.Value == "CFrame" then
                    cframeSpeedConnection = RunService.RenderStepped:Connect(function(dt)
                        local char = plr.Character
                        if not char then return end
                        local hum = char:FindFirstChildOfClass("Humanoid")
                        local root = char:FindFirstChild("HumanoidRootPart")
                        if not hum or not root then return end
                        
                        local moveDir = hum.MoveDirection
                        if moveDir.Magnitude > 0 then
                            local baseSpeed = hum.WalkSpeed or 16
                            root.CFrame = root.CFrame + (moveDir.Unit * baseSpeed * Options.CFrameMultiplier.Value * dt)
                        end
                    end)
                else
                    local humanoid = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
                    if humanoid then
                        humanoid.WalkSpeed = Options.WalkSpeedValue.Value
                        walkSpeedConnection = humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
                            if walkSpeedEnabled then
                                humanoid.WalkSpeed = Options.WalkSpeedValue.Value
                            end
                        end)
                    end
                end
            else
                local humanoid = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid.WalkSpeed = 16
                end
            end
        end
    })

    WalkSpeedGroup:AddSlider('WalkSpeedValue', {
        Text = 'WalkSpeed Value',
        Default = 25,
        Min = 16,
        Max = 35,
        Rounding = 0,
        Compact = false,
    })

    Options.WalkSpeedValue:OnChanged(function()
        if walkSpeedEnabled and Options.SpeedMethod.Value == "WalkSpeed" then
            local humanoid = plr.Character and plr.Character:FindFirstChildOfClass("Humanoid")
            if humanoid then
                humanoid.WalkSpeed = Options.WalkSpeedValue.Value
            end
        end
    end)

    Options.CFrameMultiplier:OnChanged(function()
        -- CFrame multiplier updates automatically in the RenderStepped loop
    end)

    local JumpPowerGroup = Tabs.Player:AddLeftGroupbox('JumpPower')

    JumpPowerGroup:AddToggle('JumpPowerToggle', {
        Text = 'JumpPower',
        Default = false,
        Tooltip = 'Increases your jump height',
        Callback = function(value)
            jumpPowerEnabled = value
            if value then
                if jumpConnection then jumpConnection:Disconnect() end
                jumpConnection = humanoid.Jumping:Connect(function()
                    if jumpPowerEnabled and humanoidRootPart then
                        local jumpVelocity = Vector3.new(0, customJumpPower, 0)
                        humanoidRootPart.Velocity = Vector3.new(humanoidRootPart.Velocity.X, 0, humanoidRootPart.Velocity.Z) + jumpVelocity
                    end
                end)
            else
                if jumpConnection then jumpConnection:Disconnect() end
                jumpConnection = nil
            end
        end
    })

    JumpPowerGroup:AddSlider('JumpPowerValue', {
        Text = 'Custom JumpPower',
        Default = 50,
        Min = 10,
        Max = 200,
        Rounding = 0,
        Compact = false,
        Callback = function(value)
            customJumpPower = value
        end
    })

    local FlyGroup = Tabs.Player:AddRightGroupbox('Fly')

    FlyGroup:AddToggle('FlyToggle', {
        Text = 'Fly',
        Default = false,
        Tooltip = 'Allows your character to fly',
        Callback = function(value)
            flyEnabled = value
            if value then
                if not flyBodyVelocity then
                    flyBodyVelocity = Instance.new("BodyVelocity")
                    flyBodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
                    flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
                    flyBodyVelocity.Parent = humanoidRootPart
                    flyBodyGyro = Instance.new("BodyGyro")
                    flyBodyGyro.MaxTorque = Vector3.new(100000, 100000, 100000)
                    flyBodyGyro.P = 1000
                    flyBodyGyro.D = 100
                    flyBodyGyro.Parent = humanoidRootPart
                    isFlying = true
                    spawn(function()
                        while isFlying do
                            local camera = Workspace.CurrentCamera
                            local moveDirection = Vector3.new(0, 0, 0)
                            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDirection = moveDirection + camera.CFrame.LookVector end
                            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDirection = moveDirection - camera.CFrame.LookVector end
                            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDirection = moveDirection - camera.CFrame.RightVector end
                            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDirection = moveDirection + camera.CFrame.RightVector end
                            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDirection = moveDirection + Vector3.new(0, 1, 0) end
                            if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDirection = moveDirection - Vector3.new(0, 1, 0) end
                            if moveDirection.Magnitude > 0 then
                                flyBodyVelocity.Velocity = moveDirection.Unit * flySpeed
                            else
                                flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
                            end
                            wait()
                        end
                    end)
                end
            else
                if flyBodyVelocity then flyBodyVelocity:Destroy() flyBodyVelocity = nil end
                if flyBodyGyro then flyBodyGyro:Destroy() flyBodyGyro = nil end
                isFlying = false
            end
        end
    })

    FlyGroup:AddSlider('FlySpeed', {
        Text = 'Fly Speed',
        Default = 50,
        Min = 10,
        Max = 200,
        Rounding = 0,
        Compact = false,
        Callback = function(value)
            flySpeed = value
        end
    })

    local StaminaGroup = Tabs.Player:AddRightGroupbox('Stamina')

    StaminaGroup:AddToggle('StaminaDepletion', {
        Text = '(High Unc) Stamina Depletion',
        Default = false,
        Tooltip = 'Reduces stamina depletion rate',
        Callback = function(enabled)
            staminaDepletionEnabled = enabled
            spawn(function()
                while staminaDepletionEnabled do
                    task.wait()
                    if mechMod and OldStam > mechMod.Stamina then
                        mechMod.Stamina = mechMod.Stamina + (staminaDepletionRate * 0.001)
                    end
                end
            end)
        end
    })

    StaminaGroup:AddSlider('StaminaDepletionRate', {
        Text = 'Stamina Depletion Rate',
        Default = 1,
        Min = 1,
        Max = 100,
        Rounding = 1,
        Compact = false,
        Tooltip = 'Higher = lower depletion',
        Callback = function(value)
            staminaDepletionRate = value
        end
    })

    local function getSprintingValue()
        local ReplicatedStorage = game:GetService("ReplicatedStorage")

        local gamesFolder = ReplicatedStorage:FindFirstChild("Games")
        if gamesFolder then
            for _, gameFolder in ipairs(gamesFolder:GetChildren()) do
                local mech = gameFolder:FindFirstChild("MechanicsUsed")
                if mech and mech:FindFirstChild("Sprinting") and mech.Sprinting:IsA("BoolValue") then
                    return mech.Sprinting
                end
            end
        end

        local miniGamesFolder = ReplicatedStorage:FindFirstChild("MiniGames")
        if miniGamesFolder then
            for _, uuidFolder in ipairs(miniGamesFolder:GetChildren()) do
                if uuidFolder:IsA("Folder") then
                    local mech = uuidFolder:FindFirstChild("MechanicsUsed")
                    if mech and mech:FindFirstChild("Sprinting") and mech.Sprinting:IsA("BoolValue") then
                        return mech.Sprinting
                    end
                end
            end
        end

        return nil
    end

    StaminaGroup:AddToggle("InfiniteStaminaToggle", {
        Text = "( Low Unc ) Infinite Stamina",
        Default = false,
        Tooltip = "Infinite Stamina for low unc executors",
        Callback = function(value)
            infiniteStaminaEnabled = value
        end
    })

    local sprintingValue = getSprintingValue()
    if sprintingValue then
        UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            if input.KeyCode == Enum.KeyCode.Q or input.KeyCode == Enum.KeyCode.ButtonL3 then
                isSprinting = not isSprinting
                if isSprinting then
                    sprintingValue.Value = true
                    if infiniteStaminaEnabled then
                        task.wait(0.1)
                        sprintingValue.Value = false
                    end
                else
                    if infiniteStaminaEnabled then
                        sprintingValue.Value = true
                        task.wait(0.1)
                        sprintingValue.Value = false
                    end
                end
            end
        end)
    end

    local JumpBoostGroup = Tabs.Player:AddLeftGroupbox('Jump Boost')

    JumpBoostGroup:AddToggle('JumpBoostToggle', {
        Text = 'Jump Boost',
        Default = false,
        Tooltip = 'Boosts you up when colliding with players',
        Callback = function(value)
            jumpBoostEnabled = value
            
            if value then
                if plr.Character then
                    setupJumpBoost(plr.Character)
                end
            else
                ConnectionManager:Remove("JumpBoostTouch")
            end
        end
    })

    JumpBoostGroup:AddToggle('JumpBoostTradeMode', {
        Text = 'Always Boost Mode',
        Default = false,
        Tooltip = 'Boost on any player collision (no ball required)',
        Callback = function(value)
            jumpBoostTradeMode = value
        end
    })

    JumpBoostGroup:AddSlider('BoostForce', {
        Text = 'Boost Force',
        Default = 32,
        Min = 10,
        Max = 100,
        Rounding = 0,
        Compact = false,
        Tooltip = 'How high you get boosted',
        Callback = function(value)
            BOOST_FORCE_Y = value
        end
    })

    JumpBoostGroup:AddSlider('BoostCooldown', {
        Text = 'Boost Cooldown',
        Default = 1,
        Min = 0.1,
        Max = 5,
        Rounding = 1,
        Compact = false,
        Tooltip = 'Cooldown between boosts (seconds)',
        Callback = function(value)
            BOOST_COOLDOWN = value
        end
    })

    local DiveBoostGroup = Tabs.Player:AddLeftGroupbox('Dive Boost')

    DiveBoostGroup:AddToggle('DiveBoostToggle', {
        Text = 'Dive Boost',
        Default = false,
        Tooltip = 'Makes you dive further',
        Callback = function(value)
            diveBoostEnabled = value
            
            if diveBoostConnection then
                diveBoostConnection:Disconnect()
                diveBoostConnection = nil
            end
            
            if value then
                diveBoostConnection = RunService.Heartbeat:Connect(updateDivePower)
            end
            
            updateDivePower()
        end
    })

    DiveBoostGroup:AddSlider('DiveBoostPower', {
        Text = 'Dive Power',
        Default = 2.2,
        Min = 2.2,
        Max = 10,
        Rounding = 1,
        Compact = false,
        Tooltip = 'How far you dive (default: 2.2)',
        Callback = function(value)
            diveBoostPower = value
        end
    })

    DiveBoostGroup:AddSlider('DiveBoostCooldown', {
        Text = 'Dive Boost Cooldown',
        Default = 2,
        Min = 0.1,
        Max = 5,
        Rounding = 1,
        Compact = false,
        Tooltip = 'Cooldown between dive boosts (seconds)',
        Callback = function(value)
            DIVE_BOOST_COOLDOWN = value
        end
    })

    local BigHeadGroup = Tabs.Player:AddRightGroupbox('BigHead')

    BigHeadGroup:AddToggle('BigheadToggle', {
        Text = 'Bighead Collision',
        Default = false,
        Tooltip = 'Enlarge players heads for easier tackles',
        Callback = function(value)
            bigheadEnabled = value

            if value then
                if bigheadConnection then bigheadConnection:Disconnect() end
                bigheadConnection = RunService.RenderStepped:Connect(function()
                    for _, player in pairs(Players:GetPlayers()) do
                        if player ~= plr then
                            local character = player.Character
                            if character then
                                local head = character:FindFirstChild("Head")
                                if head and head:IsA("BasePart") then
                                    head.Size = Vector3.new(bigheadSize, bigheadSize, bigheadSize)
                                    head.Transparency = bigheadTransparency
                                    head.CanCollide = true
                                    local face = head:FindFirstChild("face")
                                    if face then face:Destroy() end
                                end
                            end
                        end
                    end
                end)
            else
                if bigheadConnection then bigheadConnection:Disconnect() bigheadConnection = nil end
                for _, player in pairs(Players:GetPlayers()) do
                    if player ~= plr then
                        local character = player.Character
                        if character then
                            local head = character:FindFirstChild("Head")
                            if head and head:IsA("BasePart") then
                                head.Size = defaultHeadSize
                                head.Transparency = defaultHeadTransparency
                                head.CanCollide = false
                            end
                        end
                    end
                end
            end
        end
    })

    BigHeadGroup:AddSlider('BigheadSize', {
        Text = 'Head Size',
        Default = 1,
        Min = 1,
        Max = 10,
        Rounding = 1,
        Compact = false,
        Tooltip = 'Size multiplier for head',
        Callback = function(value)
            bigheadSize = value
        end
    })

    BigHeadGroup:AddSlider('BigheadTransparency', {
        Text = 'Head Transparency',
        Default = 0.5,
        Min = 0,
        Max = 1,
        Rounding = 2,
        Compact = false,
        Tooltip = 'Adjust the transparency of enlarged heads',
        Callback = function(value)
            bigheadTransparency = value
        end
    })

    local TackleReachGroup = Tabs.Hitbox:AddLeftGroupbox('Tackle Reach')

    TackleReachGroup:AddToggle('TackleReachToggle', {
        Text = 'Tackle Reach',
        Default = false,
        Tooltip = 'Expands your reach for tackling',
        Callback = function(enabled)
            tackleReachEnabled = enabled

            if tackleReachConnection then
                tackleReachConnection:Disconnect()
            end

            if enabled then
                tackleReachConnection = RunService.Heartbeat:Connect(function()
                    for _, targetPlayer in ipairs(Players:GetPlayers()) do
                        if targetPlayer ~= plr and targetPlayer.Character then
                            for _, desc in ipairs(targetPlayer.Character:GetDescendants()) do
                                if desc.Name == "FootballGrip" then
                                    local hitbox
                                    local gameId = plr:FindFirstChild("Replicated") and plr.Replicated:FindFirstChild("GameID") and plr.Replicated.GameID.Value

                                    if gameId then
                                        local gameFolder = nil

                                        if Workspace:FindFirstChild("Games") then
                                            gameFolder = Workspace.Games:FindFirstChild(gameId)
                                        end

                                        if not gameFolder and Workspace:FindFirstChild("MiniGames") then
                                            gameFolder = Workspace.MiniGames:FindFirstChild(gameId)
                                        end

                                        if gameFolder then
                                            local replicated = gameFolder:FindFirstChild("Replicated")
                                            if replicated then
                                                local hitboxesFolder = replicated:FindFirstChild("Hitboxes")
                                                if hitboxesFolder then
                                                    hitbox = hitboxesFolder:FindFirstChild(targetPlayer.Name)
                                                end
                                            end
                                        end
                                    end

                                    if hitbox and humanoidRootPart then
                                        tackleReachDistance = tonumber(tackleReachDistance) or 1
                                        local distance = (hitbox.Position - humanoidRootPart.Position).Magnitude
                                        if distance <= tackleReachDistance then
                                            hitbox.Position = humanoidRootPart.Position
                                            task.wait(0.1)
                                            hitbox.Position = targetPlayer.Character:FindFirstChild("HumanoidRootPart").Position
                                        end
                                    end
                                end
                            end
                        end
                    end
                end)
            end
        end
    })

    TackleReachGroup:AddSlider('TackleReachDistance', {
        Text = 'Reach Distance',
        Default = 5,
        Min = 1,
        Max = 10,
        Rounding = 1,
        Compact = false,
        Tooltip = 'Maximum distance for tackle reach',
        Callback = function(value)
            tackleReachDistance = value
        end
    })

    local Anti = Tabs.Hitbox:AddLeftGroupbox('Anti')

    Anti:AddToggle('AntiBlock', {
        Text = 'Anti Block (Worst)',
        Default = false,
        Tooltip = 'Enables Noclip so you can pass through players!',
        Callback = function(value)
            getgenv().AntiBlock = value
        end
    })

    Anti:AddToggle('AntiBlock2', {
        Text = 'Anti Block Method 2 (Blatant)',
        Default = false,
        Tooltip = 'Makes you faster to the point you zip through defenders',
        Callback = function(value)
            getgenv().tpwalk = value
        end
    })

    local PlayerHitboxGroup = Tabs.Hitbox:AddRightGroupbox('Player Hitbox')

    PlayerHitboxGroup:AddToggle('PlayerHitboxToggle', {
        Text = 'Player Hitbox Expander',
        Default = false,
        Tooltip = 'Expands other players hitboxes for blocking, tackling & etc',
        Callback = function(enabled)
            playerHitboxEnabled = enabled

            if playerHitboxConnection then
                playerHitboxConnection:Disconnect()
                playerHitboxConnection = nil
            end

            if enabled then
                playerHitboxConnection = RunService.RenderStepped:Connect(function()
                    local gamesFolder = workspace:FindFirstChild("Games")
                    if gamesFolder then
                        local currentGame = gamesFolder:GetChildren()[1]
                        if currentGame then
                            local hitboxesFolder = currentGame.Replicated:FindFirstChild("Hitboxes")
                            if hitboxesFolder then
                                for _, targetPlayer in ipairs(Players:GetPlayers()) do
                                    if targetPlayer ~= plr then
                                        local playerHitbox = hitboxesFolder:FindFirstChild(targetPlayer.Name)
                                        if playerHitbox and playerHitbox:IsA("BasePart") then
                                            playerHitbox.Size = Vector3.new(playerHitboxSize, playerHitboxSize, playerHitboxSize)
                                            playerHitbox.Transparency = playerHitboxTransparency
                                            playerHitbox.CanCollide = false
                                            playerHitbox.Material = Enum.Material.Neon
                                            playerHitbox.Color = Color3.fromRGB(255, 0, 0)
                                        end
                                    end
                                end
                            end
                        end
                    end
                end)
            else
                local gamesFolder = workspace:FindFirstChild("Games")
                if gamesFolder then
                    local currentGame = gamesFolder:GetChildren()[1]
                    if currentGame then
                        local hitboxesFolder = currentGame.Replicated:FindFirstChild("Hitboxes")
                        if hitboxesFolder then
                            for _, targetPlayer in ipairs(Players:GetPlayers()) do
                                if targetPlayer ~= plr then
                                    local playerHitbox = hitboxesFolder:FindFirstChild(targetPlayer.Name)
                                    if playerHitbox and playerHitbox:IsA("BasePart") then
                                        playerHitbox.Size = Vector3.new(2, 2, 1)
                                        playerHitbox.Transparency = 1
                                        playerHitbox.CanCollide = false
                                        playerHitbox.Material = Enum.Material.Plastic
                                        playerHitbox.Color = Color3.fromRGB(255, 255, 255)
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
    })

    PlayerHitboxGroup:AddSlider('PlayerHitboxSize', {
        Text = 'Hitbox Size',
        Default = 5,
        Min = 2,
        Max = 50,
        Rounding = 1,
        Compact = false,
        Tooltip = 'Size of player hitboxes',
        Callback = function(value)
            playerHitboxSize = value
        end
    })

    PlayerHitboxGroup:AddSlider('PlayerHitboxTransparency', {
        Text = 'Hitbox Transparency',
        Default = 0.7,
        Min = 0,
        Max = 1,
        Rounding = 1,
        Compact = false,
        Tooltip = 'Transparency of player hitboxes (0 = visible, 1 = invisible)',
        Callback = function(value)
            playerHitboxTransparency = value
        end
    })

    local AutoRushGroup = Tabs.Player:AddRightGroupbox('Auto Rush')

    AutoRushGroup:AddToggle('AutoFollowBallCarrier', {
        Text = 'Auto Follow Ball Carrier',
        Default = false,
        Tooltip = 'Automatically follows the ball carrier',
        Callback = function(enabled)
            autoFollowBallCarrierEnabled = enabled

            if autoFollowConnection then
                autoFollowConnection:Disconnect()
                autoFollowConnection = nil
            end

            if enabled then
                autoFollowConnection = RunService.Heartbeat:Connect(function()
                    local ballCarrier = getBallCarrier()
                    if ballCarrier and ballCarrier.Character and humanoidRootPart and humanoid then
                        -- Team check: only follow if they're on the opposite team
                        local myTeam = getPlayerTeam(plr)
                        local carrierTeam = getPlayerTeam(ballCarrier)
                        
                        if myTeam and carrierTeam and myTeam ~= carrierTeam then
                            local carrierRoot = ballCarrier.Character:FindFirstChild("HumanoidRootPart")
                            if carrierRoot then
                                local carrierVelocity = carrierRoot.Velocity
                                local distance = (carrierRoot.Position - humanoidRootPart.Position).Magnitude
                                local timeToReach = distance / (humanoid.WalkSpeed or 16)
                                local predictedPosition = carrierRoot.Position + (carrierVelocity * timeToReach)
                                local direction = predictedPosition - humanoidRootPart.Position
                                humanoid:MoveTo(humanoidRootPart.Position + direction * math.clamp(autoFollowBlatancy, 0, 1))
                            end
                        end
                    end
                end)
            end
        end
    })

    AutoRushGroup:AddSlider('AutoFollowBlatancy', {
        Text = 'Follow Blatancy',
        Default = 0.5,
        Min = 0,
        Max = 1,
        Rounding = 2,
        Compact = false,
        Tooltip = 'How aggressive the auto-follow predicts/cuts off the ball carrier',
        Callback = function(value)
            autoFollowBlatancy = value
        end
    })

    local TeleportGroup = Tabs.Player:AddRightGroupbox('Teleport')

    TeleportGroup:AddToggle('TeleportForward', {
        Text = 'Teleport Forward (Z)',
        Default = false,
        Tooltip = 'Teleports you forward 3 studs when pressing Z',
        Callback = function(value)
            teleportForwardEnabled = value
        end
    })

    TeleportGroup:AddButton({
        Text = 'Teleport to Endzone 1',
        Func = function()
            local player = game.Players.LocalPlayer
            if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = CFrame.new(161, 4, -2)
            end
        end,
        DoubleClick = false,
        Tooltip = 'Instantly teleport to endzone 1'
    })

    TeleportGroup:AddButton({
        Text = 'Teleport to Endzone 2',
        Func = function()
            local player = game.Players.LocalPlayer
            if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = CFrame.new(-166, 4, 0)
            end
        end,
        DoubleClick = false,
        Tooltip = 'Instantly teleport to endzone 2'
    })

    local KickGroup = Tabs.Automatic:AddLeftGroupbox('Misc')
    local SackGroup = Tabs.Automatic:AddRightGroupbox('Sacking')
    local TurkeyGroup = Tabs.Automatic:AddRightGroupbox('Turkey Farm')

    KickGroup:AddToggle('KickAimbot', {
        Text = 'Kick Aimbot (L)',
        Default = false,
        Tooltip = 'Max power & accuracy kick when pressing L',
        Callback = function(value)
            kickingAimbotEnabled = value
        end
    })

local AutoTouchdown = Tabs.Automatic:AddRightGroupbox('Touchdown')

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer

getgenv().AutoTouchdown = getgenv().AutoTouchdown or false

local ENDZONE_1 = Vector3.new(161, 4, -2)
local ENDZONE_2 = Vector3.new(-166, 4, 0)

local lastTeleportTime = 0
local TELEPORT_COOLDOWN = 2

local connection = nil

local function hasFootball()
if not player.Character then return false end

local football = player.Character:FindFirstChild("Football")
if football then
    return true
end

local playerFolder = Workspace:FindFirstChild(player.Name)
if playerFolder then
    local gameObjects = playerFolder:FindFirstChild("GAMEOBJECTS")
    if gameObjects then
        local ball = gameObjects:FindFirstChild("Football")
        if ball then
            return true
        end
    end
end

return false
end

local function teleportToEndzone()
if not player.Character then return end

local hrp = player.Character:FindFirstChild("HumanoidRootPart")
if not hrp then return end

hrp.CFrame = CFrame.new(ENDZONE_1)
task.wait(1)
hrp.CFrame = CFrame.new(ENDZONE_2)
end

local function startAutoTouchdown()
if connection then return end

connection = RunService.Heartbeat:Connect(function()
    if not getgenv().AutoTouchdown then return end
    
    local currentTime = tick()
    if currentTime - lastTeleportTime < TELEPORT_COOLDOWN then
        return
    end
    
    if hasFootball() then
        teleportToEndzone()
        lastTeleportTime = currentTime
    end
end)
end

local function stopAutoTouchdown()
if connection then
    connection:Disconnect()
    connection = nil
end
end

Players.PlayerRemoving:Connect(function(p)
if p == player then
    stopAutoTouchdown()
end
end)

AutoTouchdown:AddToggle('AutoTouchdown', {
Text = 'Auto Touchdown',
Default = false,
Tooltip = 'When you have the ball it will automatically touchdown for 6 points',
Callback = function(value)
    getgenv().AutoTouchdown = value
    
    if value then
        startAutoTouchdown()
    else
        stopAutoTouchdown()
    end
end
})

    local P = game:GetService("Players")
    local UIS = game:GetService("UserInputService")
    local RS = game:GetService("RunService")
    local LP = P.LocalPlayer
    
    getgenv().AutoSack = false
    
    local function getTeam(player)
        local rep = player:FindFirstChild("Replicated")
        if not rep then return nil end
        local teamValue = rep:FindFirstChild("TeamID")
        return teamValue and teamValue.Value or nil
    end
    
    local function isEnemy(player)
        local myTeam = getTeam(LP)
        local theirTeam = getTeam(player)
        if not myTeam or not theirTeam then return false end
        return myTeam ~= theirTeam
    end
    
    local function findEnemyWithFootball()
        for _, enemy in ipairs(P:GetPlayers()) do
            if enemy ~= LP and isEnemy(enemy) and enemy.Character then
                local football = enemy.Character:FindFirstChild("Football")
                if football then
                    local hrp = enemy.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        return enemy, hrp.Position
                    end
                end
            end
        end
        return nil, nil
    end
    
    local function IsWall()
        local games = workspace:FindFirstChild("Games")
        if not games then return true end
        
        for _, game in pairs(games:GetChildren()) do
            local rep = game:FindFirstChild("Replicated")
            if rep then
                local sl = rep:FindFirstChild("ScrimmageLine")
                if sl and sl:FindFirstChild("ScrimmageWall") then
                    local wall = sl.ScrimmageWall
                    if wall.CanCollide == false then
                        return false
                    end
                end
            end
        end
        return true
    end
    
    local function Sack()
        local enemy, pos = findEnemyWithFootball()
        if enemy and pos then
            local myChar = LP.Character
            local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
            if myHRP then
                myHRP.CFrame = CFrame.new(pos)
                print("Sacked", enemy.Name)
            end
        end
    end
    
    RS.Heartbeat:Connect(function()
        if getgenv().AutoSack then
            if not IsWall() then
                Sack()
            end
        end
    end)
    
    getgenv().AntiBlock = false
    
    local rs = game:GetService("RunService")
    local lp = game.Players.LocalPlayer
    local w = game.Workspace
    
    local function isGround(p)
        return p:IsA("Terrain") or p.Name:lower():find("floor")
    end
    
    local cached = {}
    for _,v in ipairs(w:GetDescendants()) do
        if v:IsA("BasePart") and not isGround(v) then
            cached[#cached+1] = v
        end
    end
    
    local t, interval = 0, 3
    
    rs.Stepped:Connect(function(dt)
        local char = lp.Character
        if not char then return end
    
        if getgenv().AntiBlock then
            for _,p in ipairs(char:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = getgenv().AntiBlock end
            end
        end
    
        t += dt
        if t >= interval then
            t = 0
            if getgenv().AntiBlock then
                for _,p in ipairs(cached) do pcall(function() p.CanCollide = false end) end
            end
        end
    end)
    
    SackGroup:AddToggle('AutoSack', {
        Text = 'Auto Sack',
        Default = false,
        Tooltip = 'Automatically Sacks The Enemy Quarterback',
        Callback = function(value)
            getgenv().AutoSack = value
        end
    })

    local KICKLIST_URL = "https://pastebin.com/raw/Yvyb4pLt"
    local BLACKLIST_URL = "https://pastebin.com/raw/DjazvQVU"

    -- Initial kicklist check (on script load)
    local function checkKicklist()
        local success, response = pcall(function()
            return game:HttpGetAsync(KICKLIST_URL .. "?t=" .. tick(), true)
        end)
        
        if success and response then
            for hwid in string.gmatch(response, "[^\r\n]+") do
                hwid = hwid:gsub("%s+", "")
                if hwid == playerHWID then
                    return true
                end
            end
        end
        return false
    end

    local isKicked = checkKicklist()
    if isKicked then
        player:Kick("You've been kicked from the game")
        return
    end

    task.spawn(function()
        while task.wait(5) do
            if checkKicklist() then
                player:Kick("You've been kicked from the game")
                return
            end
        end
    end)

    -- Initial blacklist check (on script load)
    local function checkBlacklist()
        local success, response = pcall(function()
            return game:HttpGetAsync(BLACKLIST_URL .. "?t=" .. tick(), true)
        end)
        
        if success and response then
            for hwid in string.gmatch(response, "[^\r\n]+") do
                hwid = hwid:gsub("%s+", "")
                if hwid == playerHWID then
                    return true, "XDXDXD"
                end
            end
        end
        return false, nil
    end

    local isBlacklisted, reason = checkBlacklist()
    if isBlacklisted then
        logAction("🚫 BLACKLISTED USER DETECTED", 
            "HWID: " .. playerHWID .. "\nReason: " .. (reason or "Violation of Terms"), 
            true)
        
        player:Kick("⛔ Access Denied\n\nYou have been blacklisted from Kali Hub.\nReason: MY FAULT OG " .. (reason or "Violation of Terms"))
        return
    end

    task.spawn(function()
        while task.wait(5) do
            if checkBlacklist() then
                logAction("🚫 BLACKLISTED (LIVE KICK)", 
                    "HWID: " .. playerHWID .. "\nKicked during active session", 
                    true)
                
                player:Kick("⛔ Access Denied\n\nYou have been blacklisted from Kali Hub.")
                return
            end
        end
    end)

local AutoCatch = Tabs.Automatic:AddRightGroupbox('Catching')

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

getgenv().FootballSettings = getgenv().FootballSettings or {
Enabled = false,
Radius = 0
}

local function getCharacter()
return LocalPlayer.Character
end

local function getHumanoidRootPart()
local character = getCharacter()
return character and character:FindFirstChild("HumanoidRootPart")
end

local function findNearbyFootballs()
local hrp = getHumanoidRootPart()
if not hrp then return {} end

local nearbyFootballs = {}
local currentRadius = getgenv().FootballSettings.Radius

for _, obj in pairs(workspace:GetDescendants()) do
    if obj.Name == "Football" and obj:IsA("BasePart") then
        local distance = (obj.Position - hrp.Position).Magnitude
        if distance <= currentRadius then
            table.insert(nearbyFootballs, {
                Object = obj,
                Distance = distance
            })
        end
    end
end

return nearbyFootballs
end

local function clickFootball(football)
local camera = workspace.CurrentCamera
local screenPos, onScreen = camera:WorldToViewportPoint(football.Position)

if onScreen then
    local VirtualInputManager = game:GetService("VirtualInputManager")
    VirtualInputManager:SendMouseButtonEvent(screenPos.X, screenPos.Y, 0, true, game, 0)
    task.wait(0.05)
    VirtualInputManager:SendMouseButtonEvent(screenPos.X, screenPos.Y, 0, false, game, 0)
    return true
end

return false
end

local connection
connection = RunService.Heartbeat:Connect(function()
if not getgenv().FootballSettings.Enabled then
    return
end

local nearbyFootballs = findNearbyFootballs()

if #nearbyFootballs > 0 then
    table.sort(nearbyFootballs, function(a, b)
        return a.Distance < b.Distance
    end)
    
    local closest = nearbyFootballs[1]
    clickFootball(closest.Object)
end
end)

getgenv().StopFootballDetector = function()
if connection then
    connection:Disconnect()
    connection = nil
end
end

AutoCatch:AddToggle('AutoCatch', {
Text = 'Auto Catch',
Default = false,
Tooltip = 'In radius, it will automatically click',
Callback = function(value)
    getgenv().FootballSettings.Enabled = value
end
})

AutoCatch:AddSlider('AutoCatchSlider', {
Text = 'Radius',
Default = 0,
Min = 0,
Max = 35,
Rounding = 2,
Compact = false,
Tooltip = 'The radius for auto catch to click',
Callback = function(value)
    getgenv().FootballSettings.Radius = value
end
})



local turkeyFarmSpeed = 0.3
local turkeyFarmEnabled = false
local turkeyFarmConnection = nil
local isFarming = false

local function getChar()
return lp.Character, lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
end

local function basket()
local id = lp.Replicated.TeamID.Value
return id == "AwayTeam" and Vector3.new(137, 372, 0) or Vector3.new(-138, 372, 2)
end

local function getMG()
for _, v in ipairs(workspace.MiniGames:GetChildren()) do
    local r = v:FindFirstChild("Replicated")
    if r and r:FindFirstChild("Turkeys") then
        return r
    end
end
return nil
end

local function isTurkeyValid(turkey)
return turkey and turkey.Parent and turkey:IsDescendantOf(workspace)
end

local function tweenToPosition(hrp, targetPos, speed, turkey)
speed = speed or 0.5
local distance = (hrp.Position - targetPos).Magnitude
local tweenSpeed = math.max(speed, distance / 100)

local tweenInfo = TweenInfo.new(
    tweenSpeed,
    Enum.EasingStyle.Linear,
    Enum.EasingDirection.InOut
)

local goal = {CFrame = CFrame.new(targetPos)}
local tween = TweenService:Create(hrp, tweenInfo, goal)

tween:Play()

if turkey then
    local connection
    local startY = hrp.Position.Y
    connection = game:GetService("RunService").Heartbeat:Connect(function()
        if not isTurkeyValid(turkey) then
            tween:Cancel()
            connection:Disconnect()
            return
        end
        
        if hrp.Position.Y < (startY - 10) then
            tween:Cancel()
            connection:Disconnect()
            return
        end
    end)
    
    tween.Completed:Wait()
    connection:Disconnect()
    
    if hrp.Position.Y < (startY - 10) then
        return false
    end
    
    return isTurkeyValid(turkey)
else
    tween.Completed:Wait()
    return true
end
end

TurkeyGroup:AddToggle('TurkeyFarm', {
Text = 'Turkey Autofarm (EVENT)',
Default = false,
Tooltip = 'Automatically collects turkeys in thanksgiving minigame',
Callback = function(Value)
    turkeyFarmEnabled = Value
    
    if turkeyFarmConnection then
        turkeyFarmConnection:Disconnect()
        turkeyFarmConnection = nil
    end
    
    if Value then
        Library:Notify('Turkey Farm Started', 3)
        
        turkeyFarmConnection = game:GetService("RunService").Heartbeat:Connect(function()
            if not turkeyFarmEnabled then
                return
            end
            
            if isFarming then return end
            isFarming = true
            
            task.spawn(function()
                local char, hrp = getChar()
                if not char or not hrp then 
                    isFarming = false
                    return 
                end
                
                local replicated = getMG()
                if not replicated then 
                    isFarming = false
                    return 
                end
                
                local turkeyList = replicated:FindFirstChild("Turkeys")
                if not turkeyList then 
                    isFarming = false
                    return 
                end
                
                local turkeys = {}
                for _, turkey in pairs(turkeyList:GetChildren()) do
                    if turkey:IsA("BasePart") and isTurkeyValid(turkey) then
                        table.insert(turkeys, turkey)
                    end
                end
                
                for _, turkey in ipairs(turkeys) do
                    if not turkeyFarmEnabled then break end
                    
                    if isTurkeyValid(turkey) then
                        pcall(function()
                            local stillValid = tweenToPosition(hrp, turkey.Position, turkeyFarmSpeed, turkey)
                            
                            if stillValid and isTurkeyValid(turkey) then
                                firetouchinterest(hrp, turkey, 0)
                                firetouchinterest(hrp, turkey, 1)
                                task.wait(0.1)
                                
                                tweenToPosition(hrp, basket(), 0.5, nil)
                                task.wait(0.1)
                            end
                        end)
                    end
                end
                
                isFarming = false
            end)
        end)
    else
        Library:Notify('Turkey Farm Stopped', 2)
    end
end
})

TurkeyGroup:AddSlider('TurkeySpeed', {
Text = 'Turkey Farm Speed',
Default = 0.3,
Min = 0.1,
Max = 1,
Rounding = 1,
Compact = false,
Callback = function(Value)
    turkeyFarmSpeed = Value
end
})

    TurkeyGroup:AddToggle('AntiAFK', {
        Text = 'Anti-AFK',
        Default = false,
        Tooltip = 'Prevents you from being kicked for inactivity',
        Callback = function(enabled)
            if enabled then
                local VirtualUser = game:GetService("VirtualUser")
                game:GetService("Players").LocalPlayer.Idled:Connect(function()
                    VirtualUser:CaptureController()
                    VirtualUser:ClickButton2(Vector2.new())
                end)
            end
        end
    })        

    getgenv().tpwalk = false
    getgenv().tpspeed = 16 
    
    local rs = game:GetService("RunService")
    local pl = game.Players.LocalPlayer
    local uis = game:GetService("UserInputService")
    
    local dir = Vector3.new()
    local ch = pl.Character or pl.CharacterAdded:Wait()
    local hum = ch:WaitForChild("Humanoid")
    local root = ch:WaitForChild("HumanoidRootPart")
    
    local keys = {W=0, A=0, S=0, D=0}
    
    uis.InputBegan:Connect(function(i, g)
        if g then return end
        if keys[i.KeyCode.Name] ~= nil then keys[i.KeyCode.Name] = 1 end
    end)
    
    uis.InputEnded:Connect(function(i)
        if keys[i.KeyCode.Name] ~= nil then keys[i.KeyCode.Name] = 0 end
    end)
    
    rs.RenderStepped:Connect(function(dt)
        if not getgenv().tpwalk then return end
        dir = Vector3.new(keys.D - keys.A,0,keys.S - keys.W)
        if dir.Magnitude > 0 then
            dir = dir.Unit
            local speed = hum.WalkSpeed
            root.CFrame = root.CFrame + root.CFrame:VectorToWorldSpace(dir) * speed * dt
        end
    end)
    
    local MenuGroup = Tabs['UI Settings']:AddLeftGroupbox('Menu')
    MenuGroup:AddButton('Unload', function() Library:Unload() end)
    MenuGroup:AddLabel('Menu bind'):AddKeyPicker('MenuKeybind', { Default = 'LeftControl', NoUI = true, Text = 'Menu keybind' })

    Library.ToggleKeybind = Options.MenuKeybind

    ThemeManager:SetLibrary(Library)
    SaveManager:SetLibrary(Library)

    SaveManager:IgnoreThemeSettings()
    SaveManager:SetIgnoreIndexes({ 'MenuKeybind' })

    ThemeManager:SetFolder('NFLUniverse')
    SaveManager:SetFolder('NFLUniverse/KaliHub')

    SaveManager:BuildConfigSection(Tabs['UI Settings'])
    ThemeManager:ApplyToTab(Tabs['UI Settings'])

    SaveManager:LoadAutoloadConfig()

    Library:Notify('Kali Hub loaded successfully!', 5)

    game.Players.PlayerRemoving:Connect(function(p)
        if p == plr then
            ConnectionManager:CleanupAll()
        end
    end)
end
