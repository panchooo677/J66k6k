local _Players = game:GetService('Players')
local _RunService = game:GetService('RunService')
local _UserInputService = game:GetService('UserInputService')
local _LocalPlayer = _Players.LocalPlayer

local u19 = false -- Lock Toggle
local u20 = 0.15  -- Prediction (Lowered slightly for snappier, less "floaty" tracking)
local u33 = nil   -- Current Target
local menuOpen = true 

-- Function to find nearest enemy
function FindNearestEnemy()
    local _huge = math.huge
    local v27 = nil
    local _Camera = workspace.CurrentCamera
    local _ScreenCenter = Vector2.new(_Camera.ViewportSize.X / 2, _Camera.ViewportSize.Y / 2)

    for _, v28 in pairs(_Players:GetPlayers()) do
        if v28 ~= _LocalPlayer then
            local _Character = v28.Character
            if _Character and _Character:FindFirstChild('HumanoidRootPart') and _Character:FindFirstChild('Humanoid') and _Character.Humanoid.Health > 0 then
                local v30, v31 = _Camera:WorldToViewportPoint(_Character.HumanoidRootPart.Position)
                if v31 then
                    local _Magnitude = (Vector2.new(v30.X, v30.Y) - _ScreenCenter).Magnitude
                    if _Magnitude < _huge then
                        v27 = v28
                        _huge = _Magnitude
                    end
                end
            end
        end
    end
    return v27
end

-- Create GUI
local _ScreenGui = Instance.new('ScreenGui')
_ScreenGui.Name = 'CombinedGui'
_ScreenGui.ResetOnSpawn = false
_ScreenGui.Parent = game.CoreGui

-- Main Container Frame
local _MainFrame = Instance.new('Frame')
_MainFrame.Parent = _ScreenGui
_MainFrame.BackgroundTransparency = 1
_MainFrame.Position = UDim2.new(0.8, -100, 0.45, 0)
_MainFrame.Size = UDim2.new(0, 165, 0, 40)

-- 1. THE TOGGLE BUTTON (Open/Close)
local _ToggleBtn = Instance.new('TextButton')
_ToggleBtn.Name = 'ToggleBtn'
_ToggleBtn.Parent = _MainFrame
_ToggleBtn.BackgroundColor3 = Color3.fromRGB(68, 0, 139)
_ToggleBtn.BackgroundTransparency = 0.85
_ToggleBtn.Position = UDim2.new(0, 0, 0, 0)
_ToggleBtn.Size = UDim2.new(0, 45, 1, 0)
_ToggleBtn.Font = Enum.Font.SourceSansBold
_ToggleBtn.Text = 'Close'
_ToggleBtn.TextColor3 = Color3.new(1, 1, 1)
_ToggleBtn.TextScaled = true
_ToggleBtn.Active = true

local _ToggleCorner = Instance.new('UICorner')
_ToggleCorner.CornerRadius = UDim.new(0, 6)
_ToggleCorner.Parent = _ToggleBtn

-- 2. THE LOCK BUTTON (Camlock Target)
local _LockBtn = Instance.new('TextButton')
_LockBtn.Name = 'LockBtn'
_LockBtn.Parent = _MainFrame
_LockBtn.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
_LockBtn.BackgroundTransparency = 0.85
_LockBtn.Position = UDim2.new(0, 50, 0, 0)
_LockBtn.Size = UDim2.new(0, 115, 1, 0)
_LockBtn.Font = Enum.Font.Code
_LockBtn.Text = 'No Target'
_LockBtn.TextColor3 = Color3.new(1, 1, 1)
_LockBtn.TextSize = 14
_LockBtn.TextWrapped = true

local _LockCorner = Instance.new('UICorner')
_LockCorner.CornerRadius = UDim.new(0, 6)
_LockCorner.Parent = _LockBtn

local _LockStroke = Instance.new('UIStroke')
_LockStroke.Parent = _LockBtn
_LockStroke.Color = Color3.fromRGB(68, 0, 139)
_LockStroke.Thickness = 2

-- FASTEST CAMLOCK LOGIC (RenderStep Priority)
-- Priority is Camera + 1 so it overrides default camera movement instantly
_RunService:BindToRenderStep("FastCamLock", Enum.RenderPriority.Camera.Value + 1, function()
    if u19 and u33 then
        local _Character = u33.Character
        local _RootPart = _Character and _Character:FindFirstChild("HumanoidRootPart")
        local _Humanoid = _Character and _Character:FindFirstChild("Humanoid")
        
        if _RootPart and _Humanoid and _Humanoid.Health > 0 then
            local _Camera = workspace.CurrentCamera
            -- Prediction logic: calculates where they are moving for instant snaps
            local _TargetPos = _RootPart.Position + (_RootPart.Velocity * u20)
            
            -- Instant CFrame assignment (Zero smoothing for max speed)
            _Camera.CFrame = CFrame.new(_Camera.CFrame.Position, _TargetPos)
        else
            u19 = false
            u33 = nil
            _LockBtn.Text = "No Target"
        end
    end
end)

-- Button Interaction Logic
_LockBtn.MouseButton1Click:Connect(function()
    u19 = not u19
    if u19 then
        u33 = FindNearestEnemy()
        if u33 then
            _LockBtn.Text = u33.DisplayName
        else
            u19 = false
            _LockBtn.Text = "No Target"
        end
    else
        u33 = nil
        _LockBtn.Text = "No Target"
    end
end)

_ToggleBtn.MouseButton1Click:Connect(function()
    menuOpen = not menuOpen
    if menuOpen then
        _ToggleBtn.Text = "Close"
        _LockBtn.Visible = true
    else
        _ToggleBtn.Text = "Open"
        _LockBtn.Visible = false
    end
end)

-- Dragging logic (Mobile Joystick Fix)
local _dragging = false
local _dragInput = nil
local _dragStartPos = nil
local _frameStartPos = nil

_ToggleBtn.InputBegan:Connect(function(input)
    if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) and not _dragging then
        _dragging = true
        _dragInput = input
        _dragStartPos = input.Position
        _frameStartPos = _MainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                _dragging = false
                _dragInput = nil
            end
        end)
    end
end)

_UserInputService.InputChanged:Connect(function(input)
    if input == _dragInput and _dragging then
        local _delta = input.Position - _dragStartPos
        _MainFrame.Position = UDim2.new(
            _frameStartPos.X.Scale, 
            _frameStartPos.X.Offset + _delta.X, 
            _frameStartPos.Y.Scale, 
            _frameStartPos.Y.Offset + _delta.Y
        )
    end
end)
