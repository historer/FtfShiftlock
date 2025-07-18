-- Flee the Facility ShiftLock for Delta (Mobile)
loadstring(game:HttpGet("https://raw.githubusercontent.com/Robobo2022/ShiftLockFix/main/MobileShiftLock.lua"))()

-- If the above fails, use this manual version:
local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Player = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Settings
local SHIFT_KEY = Enum.KeyCode.ButtonR1 -- Mobile shoulder button
local TOUCH_TOGGLE = true -- Two-finger tap toggle
local VISUAL_FEEDBACK = true -- Shows indicator on screen

-- ShiftLock state
local shiftLockEnabled = false
local touchCount = 0
local gui = nil

-- Create visual indicator
if VISUAL_FEEDBACK then
    gui = Instance.new("ScreenGui")
    gui.Name = "ShiftLockIndicator"
    gui.Parent = game:GetService("CoreGui")
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 100, 0, 40)
    frame.Position = UDim2.new(0.5, -50, 1, -100)
    frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    frame.BackgroundTransparency = 0.5
    frame.Parent = gui
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.Text = "ShiftLock: OFF"
    label.TextColor3 = Color3.new(1, 1, 1)
    label.Font = Enum.Font.SourceSansBold
    label.Parent = frame
end

-- Update visual indicator
local function updateIndicator()
    if gui and gui:FindFirstChild("Frame") then
        local label = gui.Frame.TextLabel
        if shiftLockEnabled then
            label.Text = "SHIFTLOCK: ON"
            label.TextColor3 = Color3.fromRGB(0, 255, 0)
        else
            label.Text = "SHIFTLOCK: OFF"
            label.TextColor3 = Color3.fromRGB(255, 0, 0)
        end
    end
end

-- Toggle ShiftLock
local function toggleShiftLock()
    shiftLockEnabled = not shiftLockEnabled
    pcall(function() Player.DevEnableMouseLock = shiftLockEnabled end)
    updateIndicator()
    
    -- Force camera lock if game blocks it
    if shiftLockEnabled then
        RunService:BindToRenderStep("ForceShiftLock", Enum.RenderPriority.Camera.Value + 1, function()
            if Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
                local root = Player.Character.HumanoidRootPart
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, root.Position + Vector3.new(0, 1.5, 0))
            end
        end)
    else
        RunService:UnbindFromRenderStep("ForceShiftLock")
    end
end

-- Shoulder button toggle
UIS.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == SHIFT_KEY and not gameProcessed then
        toggleShiftLock()
    end
end)

-- Two-finger tap toggle
if TOUCH_TOGGLE then
    UIS.TouchStarted:Connect(function()
        touchCount += 1
        if touchCount >= 2 then
            toggleShiftLock()
            task.delay(0.5, function() touchCount = 0 end)
        end
    end)
end

-- Reset on character change
Player.CharacterAdded:Connect(function()
    if shiftLockEnabled then
        toggleShiftLock()
        task.wait(0.1)
        toggleShiftLock()
    end
end)

-- Initial update
updateIndicator()
