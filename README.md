-- Open source, pls Credit me :( 
setclipboard("https://youtube.com/@caothieunang?si=8nCRL2eD9Wu0CsiA")

local StarterGui = game:GetService("StarterGui")
StarterGui:SetCore("SendNotification", {
    Title = "Dead Rail | Kill Aura V4",
    Text = "Script made by Cáo Mod, pls Subscribe",
    Duration = 7
})

local plr = game:GetService("Players").LocalPlayer
local rs = game:GetService("ReplicatedStorage")

local sg, fr, title, btn = Instance.new("ScreenGui"), Instance.new("Frame"), Instance.new("TextLabel"), Instance.new("TextButton")

sg.Parent, sg.ResetOnSpawn = plr:FindFirstChild("PlayerGui"), false
fr.Parent, fr.Size, fr.Position = sg, UDim2.new(0, 140, 0, 80), UDim2.new(0.1, 0, 0.05, 0)
fr.BackgroundColor3, fr.Active, fr.Draggable = Color3.fromRGB(50, 50, 50), true, true

title.Parent, title.Size, title.Position = fr, UDim2.new(1, 0, 0.4, 0), UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency, title.TextColor3 = 1, Color3.fromRGB(255, 255, 255)
title.Font, title.TextSize, title.Text = Enum.Font.Cartoon, 16, "Credit: Cáo Mod"

btn.Parent, btn.Size, btn.Position = fr, UDim2.new(1, 0, 0.6, 0), UDim2.new(0, 0, 0.4, 0)
btn.BackgroundColor3, btn.TextColor3 = Color3.fromRGB(80, 80, 80), Color3.fromRGB(255, 255, 255)
btn.Font, btn.TextSize, btn.Text = Enum.Font.Cartoon, 16, "Kill Aura: OFF"

local auraOn, killDist, killCount = false, 25, 0

-- Function to auto-detect the best damage remote
local function findDamageRemote()
    local possibleNames = { "DealDamage", "TakeDamage", "Attack", "DamageNPC", "Hit", "InflictDamage", "ApplyDamage" }
    
    for _, v in pairs(rs:GetChildren()) do
        for _, name in pairs(possibleNames) do
            if v:IsA("RemoteEvent") and v.Name:lower():find(name:lower()) then
                return v
            end
        end
    end
    return nil
end

local damageRemote = findDamageRemote()

if damageRemote then
    StarterGui:SetCore("SendNotification", {
        Title = "Kill Aura",
        Text = "Using: " .. damageRemote.Name .. " for killing!",
        Duration = 5
    })
else
    StarterGui:SetCore("SendNotification", {
        Title = "Warning",
        Text = "No valid damage function found! Using fallback kill.",
        Duration = 5
    })
end

-- Function to get all NPCs within attack range
local function getNearbyNPCs()
    local root = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
    if not root then return {} end

    local nearbyNPCs = {}
    for _, npc in ipairs(workspace:GetDescendants()) do
        if npc:IsA("Model") and npc:FindFirstChild("HumanoidRootPart") and npc:FindFirstChild("Humanoid") and not game.Players:GetPlayerFromCharacter(npc) then
            if npc.Parent ~= workspace.RuntimeItems then
                local hrp, hum = npc.HumanoidRootPart, npc.Humanoid
                local dist = (hrp.Position - root.Position).Magnitude
                if hum.Health > 0 and dist <= killDist then
                    table.insert(nearbyNPCs, npc)
                end
            end
        end
    end
    return nearbyNPCs
end

-- Function to kill NPCs using the correct method
local function killNPC(npc)
    if not npc then return end
    local hum = npc:FindFirstChild("Humanoid")
    if not hum or hum.Health <= 0 then return end

    if damageRemote then
        damageRemote:FireServer(npc, 9999)  -- Massive damage to ensure real kill
    else
        hum.Health = 0  -- Fallback method (may not work properly)
    end

    killCount = killCount + 1
end

-- Kill aura loop to continuously kill NPCs
local function killAuraLoop()
    while auraOn do
        local targets = getNearbyNPCs()
        for _, npc in pairs(targets) do
            killNPC(npc)
        end
        StarterGui:SetCore("SendNotification", { Title = "Kill Aura", Text = "Killed: " .. killCount .. " NPCs", Duration = 2 })
        task.wait(0.1)
    end
end

btn.MouseButton1Click:Connect(function()
    auraOn = not auraOn
    btn.Text = auraOn and "Kill Aura: ON" or "Kill Aura: OFF"
    killCount = 0
    if auraOn then killAuraLoop() end
end)
