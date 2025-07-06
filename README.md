**-- Skeleton ESP for all players except yourself (R6 and R15)
-- Requires executor with Drawing API (e.g., Synapse X, KRNL, Script-Ware)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Skeleton connections
local skeletonR15 = {
    {"Head", "UpperTorso"},
    {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"},
    {"UpperTorso", "RightUpperArm"},
    {"LeftUpperArm", "LeftLowerArm"},
    {"LeftLowerArm", "LeftHand"},
    {"RightUpperArm", "RightLowerArm"},
    {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"},
    {"LowerTorso", "RightUpperLeg"},
    {"LeftUpperLeg", "LeftLowerLeg"},
    {"LeftLowerLeg", "LeftFoot"},
    {"RightUpperLeg", "RightLowerLeg"},
    {"RightLowerLeg", "RightFoot"},
}

local skeletonR6 = {
    {"Head", "Torso"},
    {"Torso", "Left Arm"},
    {"Torso", "Right Arm"},
    {"Torso", "Left Leg"},
    {"Torso", "Right Leg"},
}

local function createLine()
    local line = Drawing.new("Line")
    line.Color = Color3.fromRGB(0,255,0)
    line.Thickness = 2
    line.Transparency = 1
    line.Visible = false
    return line
end

-- Store lines for each player
local playerLines = {}

local function getSkeletonType(character)
    if character:FindFirstChild("UpperTorso") then
        return skeletonR15
    else
        return skeletonR6
    end
end

local function setupPlayerESP(player)
    if player == LocalPlayer then return end
    local function onCharacterAdded(character)
        -- Clean up old lines
        if playerLines[player] then
            for _, line in ipairs(playerLines[player]) do
                line:Remove()
            end
        end
        local skeleton = getSkeletonType(character)
        playerLines[player] = {}
        for i = 1, #skeleton do
            playerLines[player][i] = createLine()
        end
    end
    if player.Character then
        onCharacterAdded(player.Character)
    end
    player.CharacterAdded:Connect(onCharacterAdded)
end

-- Setup ESP for all current players
for _, player in ipairs(Players:GetPlayers()) do
    setupPlayerESP(player)
end

-- Setup ESP for players who join later
Players.PlayerAdded:Connect(setupPlayerESP)

-- Remove lines when player leaves
Players.PlayerRemoving:Connect(function(player)
    if playerLines[player] then
        for _, line in ipairs(playerLines[player]) do
            line:Remove()
        end
        playerLines[player] = nil
    end
end)

-- Update all lines every frame
RunService.RenderStepped:Connect(function()
    for player, lines in pairs(playerLines) do
        local character = player.Character
        if character and character.Parent then
            local skeleton = getSkeletonType(character)
            for i, pair in ipairs(skeleton) do
                local part0 = character:FindFirstChild(pair[1])
                local part1 = character:FindFirstChild(pair[2])
                if part0 and part1 then
                    local pos0, onscreen0 = camera:WorldToViewportPoint(part0.Position)
                    local pos1, onscreen1 = camera:WorldToViewportPoint(part1.Position)
                    if onscreen0 and onscreen1 then
                        lines[i].From = Vector2.new(pos0.X, pos0.Y)
                        lines[i].To = Vector2.new(pos1.X, pos1.Y)
                        lines[i].Visible = true
                    else
                        lines[i].Visible = false
                    end
                else
                    lines[i].Visible = false
                end
            end
        else
            for _, line in ipairs(lines) do
                line.Visible = false
            end
        end
    end
end)**
