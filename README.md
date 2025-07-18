local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local gui = Instance.new("ScreenGui")
gui.Name = "ShiftLockGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local button = Instance.new("ImageButton", gui)
button.Size = UDim2.new(0, 60, 0, 60)
button.Position = UDim2.new(0, 20, 0.8, 0)
button.BackgroundTransparency = 1  -- fully transparent
button.ZIndex = 10

local corner = Instance.new("UICorner", button)
corner.CornerRadius = UDim.new(1, 0)

local OFF_ICON = "rbxassetid://105987953182009" -- white/off icon
local ON_ICON = "rbxassetid://139211296111194"  -- blue/on icon

local shiftLockEnabled = false

local function updateIcon()
    button.Image = shiftLockEnabled and ON_ICON or OFF_ICON
end

button.MouseButton1Click:Connect(function()
    shiftLockEnabled = not shiftLockEnabled
    updateIcon()
end)

updateIcon()

RunService.RenderStepped:Connect(function()
    if shiftLockEnabled and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
        local root = player.Character.HumanoidRootPart
        local lookVector = camera.CFrame.LookVector
        local flatLook = Vector3.new(lookVector.X, 0, lookVector.Z)
        if flatLook.Magnitude > 0 then
            flatLook = flatLook.Unit
            root.CFrame = CFrame.new(root.Position, root.Position + flatLook)
        end
    end
end)
