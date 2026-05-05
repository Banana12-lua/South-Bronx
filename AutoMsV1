--[[
    SILENT HUB V2 – DRAGON EDITION (FULLY FIXED)
    Fitur lengkap: Auto Farm, ESP, Noclip, Fly, Delete Mode, dll.
    Untuk South Bronx
    Developer: MasGal x DrKyy (fixed by Olivia untuk AR0)
--]]

-- ========== DRAGON BYPASS SYSTEM ==========
local old_nc
local mt = getrawmetatable and getrawmetatable(game)
if mt then
    setreadonly(mt, false)
    old_nc = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if method == "FireServer" and tostring(self):find("Anti") then return nil end
        if method == "Kick" or method == "Ban" then return nil end
        return old_nc(self, ...)
    end)
    setreadonly(mt, true)
end

-- ========== SERVICES ==========
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local SoundService = game:GetService("SoundService")
local Lighting = game:GetService("Lighting")
local VirtualUser = game:GetService("VirtualUser")
local VirtualInput = game:GetService("VirtualInputManager")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local ContextActionService = game:GetService("ContextActionService")

repeat task.wait() until player.Character
local playerGui = player:WaitForChild("PlayerGui")

-- ========== SOUND ID ==========
local SOUND_LOADER = "rbxassetid://119819885656906"
local SOUND_NOTIF = "rbxassetid://131370003579276"

local function playNotifSound()
    pcall(function()
        local s = Instance.new("Sound", SoundService)
        s.SoundId = SOUND_NOTIF
        s.Volume = 0.5
        s:Play()
        task.delay(2, function() s:Destroy() end)
    end)
end

-- ========== GLOBAL VARIABLES ==========
local running = false
local autoFarmRunning = false
local autoFarmStopping = false
local cookAmount = 5
local buyAmount = 1
local noclipEnabled = false
local deleteMode = false
local deletedObjects = {}
local hoverHighlight = nil
local ESPEnabled = false
local ShowName = true
local ShowHealth = true
local ShowDistance = true
local MaxDistance = 500

-- ========== HELPERS ==========
local function tw(obj, props, t)
    TweenService:Create(obj, TweenInfo.new(t or 0.18, Enum.EasingStyle.Quint), props):Play()
end

-- ========== NOTIFICATION (FIXED) ==========
local notifContainer = nil -- akan dibuat setelah GUI

local function notify(title, msg, ntype)
    playNotifSound()
    pcall(function()
        local gui = player.PlayerGui:FindFirstChild("SILENTHUB_V2")
        if not gui then return end
        local container = gui:FindFirstChild("NotifContainer")
        if not container then return end
        
        local card = Instance.new("Frame", container)
        card.Size = UDim2.new(1, 0, 0, 66)
        card.BackgroundColor3 = Color3.fromRGB(26,26,32)
        card.BorderSizePixel = 0
        Instance.new("UICorner", card).CornerRadius = UDim.new(0, 10)
        local titleL = Instance.new("TextLabel", card)
        titleL.Position = UDim2.new(0, 12, 0, 8)
        titleL.Size = UDim2.new(1, -24, 0, 18)
        titleL.BackgroundTransparency = 1
        titleL.Text = title
        titleL.Font = Enum.Font.GothamBold
        titleL.TextSize = 13
        titleL.TextColor3 = Color3.fromRGB(240,240,245)
        local msgL = Instance.new("TextLabel", card)
        msgL.Position = UDim2.new(0, 12, 0, 28)
        msgL.Size = UDim2.new(1, -24, 0, 28)
        msgL.BackgroundTransparency = 1
        msgL.Text = msg
        msgL.Font = Enum.Font.Gotham
        msgL.TextSize = 11
        msgL.TextColor3 = Color3.fromRGB(155,155,165)
        msgL.TextWrapped = true
        card.Position = UDim2.new(1, 16, 0, 0)
        tw(card, {Position = UDim2.new(0,0,0,0)}, 0.3)
        task.delay(3, function()
            tw(card, {Position = UDim2.new(1,20,0,0), BackgroundTransparency = 1}, 0.25)
            task.wait(0.3)
            card:Destroy()
        end)
    end)
end

-- ========== NOCLIP BYPASS ANTI-CRASH ==========
local noclipParts = {}
local noclipConn = nil

local function updateNoclipParts()
    local char = player.Character
    if not char then return end
    noclipParts = {}
    for _, part in pairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            table.insert(noclipParts, part)
        end
    end
end

local function setNoclip(state)
    noclipEnabled = state
    if state then
        updateNoclipParts()
        if noclipConn then noclipConn:Disconnect() end
        noclipConn = RunService.Stepped:Connect(function()
            if not noclipEnabled then return end
            for _, part in pairs(noclipParts) do
                pcall(function() part.CanCollide = false end)
            end
        end)
    else
        if noclipConn then noclipConn:Disconnect() end
        for _, part in pairs(noclipParts) do
            pcall(function() part.CanCollide = true end)
        end
    end
end

-- ========== VEHICLE FLY (FIXED) ==========
local flying = false
local flySpeed = 50
local currentVehicle = nil
local flyBodyVel, flyBodyGyro, flyConn

local function startFly()
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local seat = hum and hum.SeatPart
    if not seat then notify("Fly", "Kamu harus duduk di kendaraan dulu!", "error") return end
    currentVehicle = seat:FindFirstAncestorOfClass("Model")
    if not currentVehicle then notify("Fly", "Kendaraan tidak valid", "error") return end
    local primary = currentVehicle.PrimaryPart or seat
    currentVehicle.PrimaryPart = primary
    flying = true
    flyBodyVel = Instance.new("BodyVelocity", primary)
    flyBodyVel.MaxForce = Vector3.new(1e6,1e6,1e6)
    flyBodyGyro = Instance.new("BodyGyro", primary)
    flyBodyGyro.MaxTorque = Vector3.new(1e6,1e6,1e6)
    flyConn = RunService.RenderStepped:Connect(function()
        if not flying or not currentVehicle or not currentVehicle.Parent then
            if flyConn then flyConn:Disconnect() end
            if flyBodyVel then flyBodyVel:Destroy() end
            if flyBodyGyro then flyBodyGyro:Destroy() end
            return
        end
        local move = Vector3.new(
            (UIS:IsKeyDown(Enum.KeyCode.D) and 1 or 0) - (UIS:IsKeyDown(Enum.KeyCode.A) and 1 or 0),
            (UIS:IsKeyDown(Enum.KeyCode.E) and 1 or 0) - (UIS:IsKeyDown(Enum.KeyCode.Q) and 1 or 0),
            (UIS:IsKeyDown(Enum.KeyCode.S) and -1 or 0) + (UIS:IsKeyDown(Enum.KeyCode.W) and 1 or 0)
        )
        local cam = workspace.CurrentCamera
        flyBodyVel.Velocity = (cam.CFrame.RightVector * move.X + cam.CFrame.UpVector * move.Y + cam.CFrame.LookVector * move.Z) * flySpeed
        flyBodyGyro.CFrame = cam.CFrame
    end)
    notify("Fly", "Mode terbang aktif! WASD + E/Q", "success")
end

local function stopFly()
    flying = false
    if flyConn then flyConn:Disconnect() end
    if flyBodyVel then flyBodyVel:Destroy() end
    if flyBodyGyro then flyBodyGyro:Destroy() end
    notify("Fly", "Mode terbang dimatikan", "off")
end

-- ========== INFINITE STAMINA ==========
local infStamina = false
local staminaReq = nil
pcall(function() staminaReq = require(player.PlayerScripts:WaitForChild("Main")) end)

local function setInfiniteStamina(enabled)
    infStamina = enabled
    if enabled then
        if staminaReq then staminaReq.Stamina = 100 end
        player:SetAttribute("StaminaConsumeMultiplier", 0)
        player:GetAttributeChangedSignal("StaminaConsumeMultiplier"):Connect(function()
            if infStamina then player:SetAttribute("StaminaConsumeMultiplier", 0) end
        end)
        local staminaBar = player.PlayerGui:FindFirstChild("Main"):FindFirstChild("Bars"):FindFirstChild("StaminaBar")
        if staminaBar then staminaBar.Visible = false end
    else
        local staminaBar = player.PlayerGui:FindFirstChild("Main"):FindFirstChild("Bars"):FindFirstChild("StaminaBar")
        if staminaBar then staminaBar.Visible = true end
    end
end

-- ========== DELETE MODE & UNDO (FIXED RECONNECT) ==========
local function clearHighlight()
    if hoverHighlight then hoverHighlight:Destroy() hoverHighlight = nil end
end

local function setupDeleteMode()
    local mouse = player:GetMouse()
    mouse.Button1Down:Connect(function()
        if not deleteMode then return end
        local target = mouse.Target
        if target and target:IsA("BasePart") and not target:IsDescendantOf(player.Character) then
            table.insert(deletedObjects, {
                parent = target.Parent,
                name = target.Name,
                cframe = target.CFrame,
                size = target.Size,
                material = target.Material,
                color = target.Color,
                anchored = target.Anchored
            })
            target:Destroy()
            playNotifSound()
            notify("Delete", "Object dihapus. Undo untuk mengembalikan.", "info")
        end
    end)
end

local function undoLast()
    if #deletedObjects > 0 then
        local obj = table.remove(deletedObjects)
        local newPart = Instance.new("Part")
        newPart.Name = obj.name
        newPart.Size = obj.size
        newPart.CFrame = obj.cframe
        newPart.Material = obj.material or Enum.Material.Plastic
        newPart.Color = obj.color or Color3.new(1,1,1)
        newPart.Anchored = obj.anchored
        newPart.Parent = obj.parent
        playNotifSound()
        notify("Undo", "Object dikembalikan", "success")
    else
        notify("Undo", "Tidak ada object untuk diundo", "error")
    end
end

local function deleteAllWalls()
    local count = 0
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("BasePart") and not obj:IsDescendantOf(player.Character) then
            local name = obj.Name:lower()
            if name:find("wall") or name:find("door") or name:find("roof") or name:find("dinding") or name:find("pintu") or name:find("atap") then
                table.insert(deletedObjects, {
                    parent = obj.Parent,
                    name = obj.Name,
                    cframe = obj.CFrame,
                    size = obj.Size,
                    material = obj.Material,
                    color = obj.Color,
                    anchored = obj.Anchored
                })
                obj:Destroy()
                count = count + 1
            end
        end
    end
    playNotifSound()
    notify("Delete All", count.." object dihapus", "success")
end

local function undoAll()
    for i = #deletedObjects, 1, -1 do
        local obj = deletedObjects[i]
        local newPart = Instance.new("Part")
        newPart.Name = obj.name
        newPart.Size = obj.size
        newPart.CFrame = obj.cframe
        newPart.Material = obj.material or Enum.Material.Plastic
        newPart.Color = obj.color or Color3.new(1,1,1)
        newPart.Anchored = obj.anchored
        newPart.Parent = obj.parent
    end
    deletedObjects = {}
    playNotifSound()
    notify("Undo All", "Semua object dikembalikan", "success")
end

-- ========== UNDERGROUND ==========
local function goUnderground()
    local char = player.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if hrp then
        hrp.CFrame = hrp.CFrame - Vector3.new(0, 500, 0)
        notify("Underground", "Kamu di bawah map", "info")
    end
end

-- ========== WALK SPEED SLIDER ==========
local currentWalkSpeed = 16
local function setWalkSpeed(speed)
    currentWalkSpeed = speed
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if hum then hum.WalkSpeed = speed end
end

-- ========== CHANGE NAME & USERNAME ==========
local function changeName(newName)
    local nameTag = workspace.Characters:FindFirstChild(player.Name):FindFirstChild("Head"):FindFirstChild("NameTag"):FindFirstChild("MainFrame"):FindFirstChild("NameLabel")
    if nameTag then nameTag.Text = newName end
end

local function changeUsername(newUsn)
    local rankTag = workspace.Characters:FindFirstChild(player.Name):FindFirstChild("Head"):FindFirstChild("RankTag"):FindFirstChild("MainFrame"):FindFirstChild("NameLabel")
    if rankTag then rankTag.Text = newUsn end
end

-- ========== COLOR USN BY TIER ==========
local function updateTierColor()
    local tier = player:GetAttribute("Tier") or 0
    local color = tier == 1 and Color3.fromRGB(0,255,0) or tier == 2 and Color3.fromRGB(255,0,255) or tier == 3 and Color3.fromRGB(0,0,255) or Color3.fromRGB(150,150,150)
    local rankTag = workspace.Characters:FindFirstChild(player.Name):FindFirstChild("Head"):FindFirstChild("RankTag"):FindFirstChild("MainFrame"):FindFirstChild("NameLabel")
    if rankTag then rankTag.TextColor3 = color end
end
player:GetAttributeChangedSignal("Tier"):Connect(updateTierColor)
task.spawn(function() repeat task.wait(1) until workspace.Characters:FindFirstChild(player.Name) updateTierColor() end)

-- ========== AUTO FARM LOGIC ==========
local buyRemote = ReplicatedStorage:WaitForChild("RemoteEvents"):WaitForChild("StorePurchase")
local storePos = Vector3.new(510.7584, 3.5872, 600.3163)

local function countItem(name)
    local total = 0
    for _,v in pairs(player.Backpack:GetChildren()) do if v.Name == name then total = total + 1 end end
    for _,v in pairs(player.Character:GetChildren()) do if v:IsA("Tool") and v.Name == name then total = total + 1 end end
    return total
end

local function equipTool(name)
    local char = player.Character
    local tool = player.Backpack:FindFirstChild(name) or char:FindFirstChild(name)
    if tool then char.Humanoid:EquipTool(tool) task.wait(0.25) return true end
    return false
end

local function pressE(dur)
    dur = dur or 0.8
    VirtualInput:SendKeyEvent(true, "E", false, game)
    task.wait(dur)
    VirtualInput:SendKeyEvent(false, "E", false, game)
end

local function vehicleTeleport(cf)
    pcall(function()
        local char = player.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local seat = hum and hum.SeatPart
        if not seat then return end
        local vehicle = seat:FindFirstAncestorOfClass("Model")
        if vehicle and vehicle.PrimaryPart then
            vehicle:SetPrimaryPartCFrame(cf)
            task.wait(0.3)
            local pp = vehicle.PrimaryPart
            pp.AssemblyLinearVelocity = Vector3.zero
            pp.AssemblyAngularVelocity = Vector3.zero
        end
    end)
end

local function autoBuy()
    for i = 1, buyAmount do
        if not autoFarmRunning and not running then break end
        buyRemote:FireServer("Water") task.wait(0.35)
        buyRemote:FireServer("Sugar Block Bag") task.wait(0.35)
        buyRemote:FireServer("Gelatin") task.wait(0.35)
        buyRemote:FireServer("Empty Bag") task.wait(0.45)
    end
    notify("Buy", "Membeli x"..buyAmount, "info")
end

local function cookProcess()
    if equipTool("Water") then pressE(0.7) task.wait(20) end
    if equipTool("Sugar Block Bag") then pressE(0.7) task.wait(1) end
    if equipTool("Gelatin") then pressE(0.7) task.wait(1) end
    task.wait(45)
    if equipTool("Empty Bag") then pressE(0.7) task.wait(1) end
end

local function autoSell()
    local bags = {"Small Marshmallow Bag","Medium Marshmallow Bag","Large Marshmallow Bag"}
    for _,bag in pairs(bags) do
        while countItem(bag) > 0 and (autoFarmRunning or running) do
            if equipTool(bag) then pressE(0.7) task.wait(0.8) else break end
        end
    end
end

local farmLoopRunning = false
local function startFarmLoop()
    farmLoopRunning = true
    task.spawn(function()
        while farmLoopRunning and (autoFarmRunning or running) do
            local apartPos = player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Character.HumanoidRootPart.Position or Vector3.zero
            vehicleTeleport(CFrame.new(storePos))
            autoBuy()
            vehicleTeleport(CFrame.new(apartPos))
            cookProcess()
            vehicleTeleport(CFrame.new(storePos))
            autoSell()
            task.wait(1)
        end
    end)
end

-- ========== ESP SYSTEM (DENGAN FALLBACK) ==========
local hasDrawing = pcall(function() return Drawing.new("Line") end)
local Camera = workspace.CurrentCamera
local espData = {}

if hasDrawing then
    local function createESP(plr)
        local lines = {}
        for i = 1,8 do
            local l = Drawing.new("Line")
            l.Thickness = 1.5
            l.Color = Color3.fromRGB(50,200,100)
            l.Visible = false
            table.insert(lines, l)
        end
        local nameDraw = Drawing.new("Text")
        nameDraw.Size = 13
        nameDraw.Center = true
        nameDraw.Outline = true
        nameDraw.Color = Color3.new(1,1,1)
        nameDraw.Visible = false
        local healthLine = Drawing.new("Line")
        healthLine.Thickness = 2
        healthLine.Visible = false
        local healthBg = Drawing.new("Line")
        healthBg.Thickness = 2
        healthBg.Color = Color3.fromRGB(40,40,40)
        healthBg.Visible = false
        espData[plr] = {box=lines, name=nameDraw, health=healthLine, healthBg=healthBg}
    end

    local function removeESP(plr)
        if espData[plr] then
            for _,l in pairs(espData[plr].box) do l:Remove() end
            espData[plr].name:Remove()
            espData[plr].health:Remove()
            espData[plr].healthBg:Remove()
            espData[plr] = nil
        end
    end

    for _,p in pairs(Players:GetPlayers()) do if p ~= player then createESP(p) end end
    Players.PlayerAdded:Connect(function(p) if p ~= player then createESP(p) end end)
    Players.PlayerRemoving:Connect(removeESP)

    RunService.RenderStepped:Connect(function()
        for p, data in pairs(espData) do
            if not ESPEnabled then
                for _,l in pairs(data.box) do l.Visible = false end
                data.name.Visible = false
                data.health.Visible = false
                data.healthBg.Visible = false
                continue
            end
            local char = p.Character
            if not char then continue end
            local hrp = char:FindFirstChild("HumanoidRootPart")
            local hum = char:FindFirstChildOfClass("Humanoid")
            if not hrp or not hum then continue end
            local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
            if not onScreen then
                for _,l in pairs(data.box) do l.Visible = false end
                data.name.Visible = false data.health.Visible = false data.healthBg.Visible = false
                continue
            end
            local dist = (player.Character and player.Character:FindFirstChild("HumanoidRootPart") and (player.Character.HumanoidRootPart.Position - hrp.Position).Magnitude) or 0
            if dist > MaxDistance then
                for _,l in pairs(data.box) do l.Visible = false end
                data.name.Visible = false data.health.Visible = false data.healthBg.Visible = false
                continue
            end
            local scale = (Camera:WorldToViewportPoint(hrp.Position + Vector3.new(0,3,0)).Y - Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0,3,0)).Y) * 0.8
            local w = scale * 0.6
            local h = scale * 1.2
            local x,y = pos.X, pos.Y
            local left, right = x - w/2, x + w/2
            local top, bottom = y - h/2, y + h/2
            local lines = data.box
            lines[1].From, lines[1].To = Vector2.new(left,top), Vector2.new(left + w/4, top)
            lines[2].From, lines[2].To = Vector2.new(left,top), Vector2.new(left, top + h/4)
            lines[3].From, lines[3].To = Vector2.new(right,top), Vector2.new(right - w/4, top)
            lines[4].From, lines[4].To = Vector2.new(right,top), Vector2.new(right, top + h/4)
            lines[5].From, lines[5].To = Vector2.new(left,bottom), Vector2.new(left + w/4, bottom)
            lines[6].From, lines[6].To = Vector2.new(left,bottom), Vector2.new(left, bottom - h/4)
            lines[7].From, lines[7].To = Vector2.new(right,bottom), Vector2.new(right - w/4, bottom)
            lines[8].From, lines[8].To = Vector2.new(right,bottom), Vector2.new(right, bottom - h/4)
            for _,l in pairs(lines) do l.Visible = true end
            if ShowName then
                data.name.Text = ShowDistance and (p.Name .. " ["..math.floor(dist).."]") or p.Name
                data.name.Position = Vector2.new(x, top - 16)
                data.name.Visible = true
            else
                data.name.Visible = false
            end
            if ShowHealth then
                local hpPct = hum.Health / hum.MaxHealth
                local barX = right + 4
                data.healthBg.From = Vector2.new(barX, top)
                data.healthBg.To = Vector2.new(barX, bottom)
                data.healthBg.Visible = true
                local hpY = bottom - (bottom - top) * hpPct
                data.health.From = Vector2.new(barX, hpY)
                data.health.To = Vector2.new(barX, bottom)
                data.health.Color = Color3.fromRGB(255*(1-hpPct), 200*hpPct, 60)
                data.health.Visible = true
            else
                data.health.Visible = false
                data.healthBg.Visible = false
            end
        end
    end)
else
    print("ESP tidak didukung di executor ini, fitur dinonaktifkan")
    ESPEnabled = false
end

-- ========== GUI CONSTRUCTION ==========
local gui = Instance.new("ScreenGui")
gui.Name = "SILENTHUB_V2"
gui.Parent = playerGui
gui.ResetOnSpawn = false

-- Color palette
local C = {
    bg = Color3.fromRGB(10,10,12),
    surface = Color3.fromRGB(16,16,20),
    panel = Color3.fromRGB(20,20,25),
    card = Color3.fromRGB(26,26,32),
    cardHov = Color3.fromRGB(34,34,42),
    accent = Color3.fromRGB(220,220,220),
    text = Color3.fromRGB(240,240,245),
    textMid = Color3.fromRGB(155,155,165),
    textDim = Color3.fromRGB(75,75,85),
    green = Color3.fromRGB(50,200,100),
    red = Color3.fromRGB(220,55,65),
    border = Color3.fromRGB(38,38,46),
}

-- FIX: Notif Container
notifContainer = Instance.new("Frame", gui)
notifContainer.Name = "NotifContainer"
notifContainer.Size = UDim2.new(0, 300, 1, 0)
notifContainer.Position = UDim2.new(1, -310, 0, 0)
notifContainer.BackgroundTransparency = 1

-- Loading screen
local loadGui = Instance.new("ScreenGui", playerGui)
loadGui.Name = "LoadingScreen"
loadGui.IgnoreGuiInset = true
local loadBg = Instance.new("Frame", loadGui)
loadBg.Size = UDim2.new(1,0,1,0)
loadBg.BackgroundColor3 = Color3.new(0,0,0)
for i=1,50 do
    local star = Instance.new("Frame", loadBg)
    star.Size = UDim2.new(0, math.random(1,3), 0, math.random(1,3))
    star.Position = UDim2.new(math.random(), 0, math.random(), 0)
    star.BackgroundColor3 = Color3.new(1,1,1)
    star.BackgroundTransparency = math.random(30,80)/100
    Instance.new("UICorner", star).CornerRadius = UDim.new(1,0)
end
local loadText = Instance.new("TextLabel", loadBg)
loadText.Size = UDim2.new(0,300,0,50)
loadText.Position = UDim2.new(0.5,-150,0.5,-25)
loadText.BackgroundTransparency = 1
loadText.Text = "SILENT HUB"
loadText.Font = Enum.Font.GothamBlack
loadText.TextSize = 36
loadText.TextColor3 = Color3.new(1,1,1)
loadText.TextXAlignment = Enum.TextXAlignment.Center

-- Main GUI (awalnya tersembunyi)
local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,620,0,400)
main.Position = UDim2.new(0.5,-310,0.5,-200)
main.BackgroundColor3 = C.bg
main.Active = true
main.Draggable = true
main.Visible = false
Instance.new("UICorner", main).CornerRadius = UDim.new(0,14)
Instance.new("UIStroke", main).Color = C.border

-- Top bar
local top = Instance.new("Frame", main)
top.Size = UDim2.new(1,0,0,52)
top.BackgroundColor3 = C.surface
Instance.new("UICorner", top).CornerRadius = UDim.new(0,14)
local titleL = Instance.new("TextLabel", top)
titleL.Size = UDim2.new(0,200,0,22)
titleL.Position = UDim2.new(0,20,0.5,-11)
titleL.BackgroundTransparency = 1
titleL.RichText = true
titleL.Text = '<font color="rgb(255,255,255)">SILENT</font><font color="rgb(80,80,90)">.</font><font color="rgb(220,55,65)">GG</font>'
titleL.Font = Enum.Font.GothamBlack
titleL.TextSize = 18
local closeBtn = Instance.new("TextButton", top)
closeBtn.Size = UDim2.new(0,28,0,28)
closeBtn.Position = UDim2.new(1,-38,0.5,-14)
closeBtn.BackgroundColor3 = Color3.fromRGB(45,18,20)
closeBtn.Text = "✕"
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextColor3 = C.red
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0,7)
closeBtn.MouseButton1Click:Connect(function() gui:Destroy() end)

-- Tab bar dan content (FIXED dengan tabButtons)
local tabBar = Instance.new("Frame", main)
tabBar.Size = UDim2.new(1,-16,0,34)
tabBar.Position = UDim2.new(0,8,0,54)
tabBar.BackgroundColor3 = C.surface
Instance.new("UICorner", tabBar).CornerRadius = UDim.new(0,8)

local content = Instance.new("ScrollingFrame", main)
content.Size = UDim2.new(1,-16,1,-98)
content.Position = UDim2.new(0,8,0,94)
content.BackgroundColor3 = C.panel
content.CanvasSize = UDim2.new(0,0,0,0)
content.ScrollBarThickness = 4
Instance.new("UICorner", content).CornerRadius = UDim.new(0,10)

local tabs = {"FARM","MISC","TELEPORT","VEHICLE","ESP"}
local pages = {}
local tabButtons = {}  -- FIX: deklarasi

for i, name in ipairs(tabs) do
    local btn = Instance.new("TextButton", tabBar)
    btn.Size = UDim2.new(0,72,0,28)
    btn.Text = name
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    btn.BackgroundColor3 = C.card
    btn.BackgroundTransparency = 1
    btn.TextColor3 = C.textDim
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)
    local page = Instance.new("ScrollingFrame", content)
    page.Size = UDim2.new(1,0,1,0)
    page.BackgroundTransparency = 1
    page.ScrollBarThickness = 3
    page.Visible = false
    pages[name] = page
    tabButtons[name] = btn
    btn.MouseButton1Click:Connect(function()
        for _,p in pairs(pages) do p.Visible = false end
        for _,b in pairs(tabButtons) do
            b.BackgroundTransparency = 1
            b.TextColor3 = C.textDim
        end
        page.Visible = true
        btn.BackgroundTransparency = 0
        btn.TextColor3 = C.text
    end)
end

-- helper functions
local function addCard(parent, text, order)
    local card = Instance.new("Frame", parent)
    card.Size = UDim2.new(1,0,0,36)
    card.BackgroundColor3 = C.card
    card.LayoutOrder = order
    Instance.new("UICorner", card).CornerRadius = UDim.new(0,8)
    Instance.new("UIStroke", card).Color = C.border
    local lbl = Instance.new("TextLabel", card)
    lbl.Size = UDim2.new(0.6,0,1,0)
    lbl.Position = UDim2.new(0,12,0,0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.Font = Enum.Font.GothamSemibold
    lbl.TextSize = 12
    lbl.TextColor3 = C.text
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    return card, lbl
end

local function addButton(parent, text, order, callback)
    local btn = Instance.new("TextButton", parent)
    btn.Size = UDim2.new(1,0,0,36)
    btn.BackgroundColor3 = C.card
    btn.Text = text
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.TextColor3 = C.text
    btn.LayoutOrder = order
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0,8)
    Instance.new("UIStroke", btn).Color = C.border
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local function addSlider(parent, label, minv, maxv, defaultv, order, callback)
    local card = Instance.new("Frame", parent)
    card.Size = UDim2.new(1,0,0,52)
    card.BackgroundColor3 = C.card
    card.LayoutOrder = order
    Instance.new("UICorner", card).CornerRadius = UDim.new(0,8)
    local lbl = Instance.new("TextLabel", card)
    lbl.Size = UDim2.new(0.6,0,0,20)
    lbl.Position = UDim2.new(0,12,0,6)
    lbl.BackgroundTransparency = 1
    lbl.Text = label
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 11
    lbl.TextColor3 = C.text
    local valLbl = Instance.new("TextLabel", card)
    valLbl.Size = UDim2.new(0,50,0,20)
    valLbl.Position = UDim2.new(1,-62,0,6)
    valLbl.BackgroundTransparency = 1
    valLbl.Text = tostring(defaultv)
    valLbl.Font = Enum.Font.GothamBold
    valLbl.TextSize = 12
    valLbl.TextColor3 = C.accent
    local track = Instance.new("Frame", card)
    track.Size = UDim2.new(1,-24,0,5)
    track.Position = UDim2.new(0,12,0,32)
    track.BackgroundColor3 = C.border
    Instance.new("UICorner", track).CornerRadius = UDim.new(1,0)
    local fill = Instance.new("Frame", track)
    fill.Size = UDim2.new((defaultv-minv)/(maxv-minv),0,1,0)
    fill.BackgroundColor3 = C.accent
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1,0)
    local knob = Instance.new("Frame", track)
    knob.Size = UDim2.new(0,12,0,12)
    knob.Position = UDim2.new((defaultv-minv)/(maxv-minv),-6,0.5,-6)
    knob.BackgroundColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1,0)
    local dragging = false
    local function update(posX)
        if track.AbsoluteSize.X == 0 then return end
        local tpos = math.clamp((posX - track.AbsolutePosition.X)/track.AbsoluteSize.X, 0, 1)
        local val = math.floor(minv + tpos * (maxv - minv))
        fill.Size = UDim2.new(tpos,0,1,0)
        knob.Position = UDim2.new(tpos,-6,0.5,-6)
        valLbl.Text = tostring(val)
        if callback then callback(val) end
    end
    track.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            update(i.Position.X)
        end
    end)
    UIS.InputChanged:Connect(function(i)
        if dragging and i.UserInputType == Enum.UserInputType.MouseMovement then
            update(i.Position.X)
        end
    end)
    UIS.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    return card, valLbl
end

local function addToggle(parent, text, order, callback)
    local card = Instance.new("Frame", parent)
    card.Size = UDim2.new(1,0,0,38)
    card.BackgroundColor3 = C.card
    card.LayoutOrder = order
    Instance.new("UICorner", card).CornerRadius = UDim.new(0,8)
    local lbl = Instance.new("TextLabel", card)
    lbl.Size = UDim2.new(1,-50,1,0)
    lbl.Position = UDim2.new(0,12,0,0)
    lbl.BackgroundTransparency = 1
    lbl.Text = text
    lbl.Font = Enum.Font.GothamBold
    lbl.TextSize = 12
    lbl.TextColor3 = C.text
    lbl.TextXAlignment = Enum.TextXAlignment.Left
    local pill = Instance.new("Frame", card)
    pill.Size = UDim2.new(0,28,0,14)
    pill.Position = UDim2.new(1,-38,0.5,-7)
    pill.BackgroundColor3 = C.green
    Instance.new("UICorner", pill).CornerRadius = UDim.new(1,0)
    local knob = Instance.new("Frame", pill)
    knob.Size = UDim2.new(0,10,0,10)
    knob.Position = UDim2.new(1,-12,0.5,-5)
    knob.BackgroundColor3 = Color3.new(1,1,1)
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1,0)
    local state = true
    local function setState(s)
        state = s
        pill.BackgroundColor3 = s and C.green or Color3.fromRGB(80,80,90)
        knob.Position = s and UDim2.new(1,-12,0.5,-5) or UDim2.new(0,2,0.5,-5)
        if callback then callback(s) end
    end
    local btn = Instance.new("TextButton", card)
    btn.Size = UDim2.new(1,0,1,0)
    btn.BackgroundTransparency = 1
    btn.Text = ""
    btn.MouseButton1Click:Connect(function() setState(not state) end)
    return card, setState
end

-- FILL TABS
-- FARM TAB
local farmPage = pages["FARM"]
local layout = Instance.new("UIListLayout", farmPage)
layout.Padding = UDim.new(0,6)
local pad = Instance.new("UIPadding", farmPage)
pad.PaddingTop = UDim.new(0,10)
pad.PaddingLeft = UDim.new(0,10)
pad.PaddingRight = UDim.new(0,10)

addButton(farmPage, "START AUTO FARM", 1, function()
    autoFarmRunning = not autoFarmRunning
    if autoFarmRunning then
        startFarmLoop()
        notify("Auto Farm", "Loop dimulai", "success")
    else
        farmLoopRunning = false
        notify("Auto Farm", "Loop dihentikan", "off")
    end
end)
addSlider(farmPage, "COOK AMOUNT", 1, 50, 5, 2, function(v) cookAmount = v end)
addSlider(farmPage, "BUY AMOUNT", 1, 25, 1, 3, function(v) buyAmount = v end)
addSlider(farmPage, "WALK SPEED", 16, 23, 16, 4, function(v) setWalkSpeed(v) end)
addToggle(farmPage, "INFINITE STAMINA", 5, function(s) setInfiniteStamina(s) end)
addToggle(farmPage, "NOCLIP (BYPASS)", 6, function(s) setNoclip(s) end)

-- MISC TAB
local miscPage = pages["MISC"]
local miscLayout = Instance.new("UIListLayout", miscPage)
miscLayout.Padding = UDim.new(0,6)
Instance.new("UIPadding", miscPage).PaddingTop = UDim.new(0,10)

addButton(miscPage, "UNDERGROUND", 1, goUnderground)
addButton(miscPage, "DELETE MODE (ON/OFF)", 2, function()
    deleteMode = not deleteMode
    notify("Delete Mode", deleteMode and "AKTIF" or "NONAKTIF", deleteMode and "success" or "off")
end)
addButton(miscPage, "UNDO", 3, undoLast)
addButton(miscPage, "DELETE ALL (WALL/DOOR)", 4, deleteAllWalls)
addButton(miscPage, "UNDO ALL", 5, undoAll)

local nameCard = addCard(miscPage, "CHANGE NAME (TAG)", 6)
local nameBox = Instance.new("TextBox", nameCard)
nameBox.Size = UDim2.new(0.4,0,0,24)
nameBox.Position = UDim2.new(0.55,0,0.5,-12)
nameBox.BackgroundColor3 = C.surface
nameBox.Text = "Nama baru"
nameBox.Font = Enum.Font.Gotham
nameBox.TextColor3 = C.text
Instance.new("UICorner", nameBox).CornerRadius = UDim.new(0,6)
nameBox.FocusLost:Connect(function(enter)
    if enter then changeName(nameBox.Text) playNotifSound() end
end)

local usnCard = addCard(miscPage, "CHANGE USERNAME (RANK)", 7)
local usnBox = Instance.new("TextBox", usnCard)
usnBox.Size = UDim2.new(0.4,0,0,24)
usnBox.Position = UDim2.new(0.55,0,0.5,-12)
usnBox.BackgroundColor3 = C.surface
usnBox.Text = "Username baru"
usnBox.Font = Enum.Font.Gotham
usnBox.TextColor3 = C.text
Instance.new("UICorner", usnBox).CornerRadius = UDim.new(0,6)
usnBox.FocusLost:Connect(function(enter)
    if enter then changeUsername(usnBox.Text) playNotifSound() end
end)

-- TELEPORT TAB
local tpPage = pages["TELEPORT"]
local tpLayout = Instance.new("UIListLayout", tpPage)
tpLayout.Padding = UDim.new(0,6)
Instance.new("UIPadding", tpPage).PaddingTop = UDim.new(0,10)

local teleportPoints = {
    {"NPC Store", CFrame.new(510.76,3.59,600.79)},
    {"Tier", CFrame.new(1110.19,4.28,117.14)},
    {"Apart 1", CFrame.new(1140.3,10.1,450.3)},
    {"Apart 2", CFrame.new(1141.4,10.1,422.8)},
    {"Mall", CFrame.new(-725.4,4.8,587.4)},
    {"Safe Zone", CFrame.new(579.0,3.5,-539.7)},
}
for i, data in ipairs(teleportPoints) do
    addButton(tpPage, data[1], i, function()
        vehicleTeleport(data[2])
        notify("Teleport", "Ke "..data[1], "info")
    end)
end

-- VEHICLE FLY TAB
local vfPage = pages["VEHICLE"]
local vfLayout = Instance.new("UIListLayout", vfPage)
vfLayout.Padding = UDim.new(0,6)
Instance.new("UIPadding", vfPage).PaddingTop = UDim.new(0,10)

addButton(vfPage, "ACTIVATE FLY (WASD+E/Q)", 1, startFly)
addButton(vfPage, "DEACTIVATE FLY", 2, stopFly)

-- ESP TAB
local espPage = pages["ESP"]
local espLayout = Instance.new("UIListLayout", espPage)
espLayout.Padding = UDim.new(0,6)
Instance.new("UIPadding", espPage).PaddingTop = UDim.new(0,10)

addToggle(espPage, "ENABLE ESP", 1, function(s) ESPEnabled = s end)
addToggle(espPage, "SHOW NAME", 2, function(s) ShowName = s end)
addToggle(espPage, "SHOW HEALTH", 3, function(s) ShowHealth = s end)
addToggle(espPage, "SHOW DISTANCE", 4, function(s) ShowDistance = s end)
addSlider(espPage, "MAX DISTANCE", 100, 8000, 500, 5, function(v) MaxDistance = v end)

-- Default buka tab FARM
pages["FARM"].Visible = true
tabButtons["FARM"].BackgroundTransparency = 0
tabButtons["FARM"].TextColor3 = C.text

-- ========== SOUND LOADER + TIMEOUT ==========
local loaderSound = Instance.new("Sound", SoundService)
loaderSound.SoundId = SOUND_LOADER
loaderSound.Volume = 0.8
loaderSound:Play()
notify("Silent Hub", "Memuat... tunggu sound selesai", "info")

local soundEnded = false
loaderSound.Ended:Connect(function()
    if soundEnded then return end
    soundEnded = true
    loaderSound:Destroy()
    loadGui:Destroy()
    main.Visible = true
    main.BackgroundTransparency = 1
    tw(main, {BackgroundTransparency = 0}, 0.4)
    notify("Silent Hub", "Script siap digunakan! 💖", "success")
end)

task.delay(10, function()
    if not soundEnded then
        soundEnded = true
        pcall(function() loaderSound:Destroy() end)
        loadGui:Destroy()
        main.Visible = true
        main.BackgroundTransparency = 1
        tw(main, {BackgroundTransparency = 0}, 0.4)
        notify("Silent Hub", "Script siap digunakan (timeout)!", "success")
    end
end)

-- ========== ANTI IDLE ==========
player.Idled:Connect(function()
    pcall(function()
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new())
    end)
end)

-- ========== RE-INIT FITUR SAAT RESPAWN ==========
player.CharacterAdded:Connect(function()
    task.wait(0.5)
    setupDeleteMode()
    if noclipEnabled then setNoclip(true) end
    setWalkSpeed(currentWalkSpeed)
end)
setupDeleteMode()

-- Update tier color periodically
task.spawn(function()
    while gui and gui.Parent do
        updateTierColor()
        task.wait(5)
    end
end)

print("Silent Hub V2 - Dragon Edition (FULLY FIXED) loaded. Sayangku, semua fitur aktif~ 💋")
