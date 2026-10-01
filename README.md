-- File: FlyGUI.lua
-- Script bay tích hợp GUI Menu nút bấm (Hỗ trợ PC & Mobile)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local FLY_SPEED = 50
local isFlying = false

local bodyVelocity = nil
local bodyGyro = nil

local movementState = {
	Forward = 0,
	Backward = 0,
	Left = 0,
	Right = 0,
	Up = 0,
	Down = 0
}

-- Tạo GUI Menu
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FlyMenuGui"
ScreenGui.ResetOnSpawn = false

-- Đặt GUI vào CoreGui hoặc PlayerGui
if syn and syn.protect_gui then
	syn.protect_gui(ScreenGui)
	ScreenGui.Parent = CoreGui
elseif gethui then
	ScreenGui.Parent = gethui()
else
	ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 160, 0, 80)
MainFrame.Position = UDim2.new(0.05, 0, 0.4, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true -- Tự do kéo thả Menu trên màn hình
MainFrame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainFrame

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, 0, 0, 25)
Title.Position = UDim2.new(0, 0, 0, 5)
Title.BackgroundTransparency = 1
Title.Text = "Fly Menu"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.TextSize = 16
Title.Font = Enum.Font.SourceSansBold
Title.Parent = MainFrame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Name = "ToggleButton"
ToggleButton.Size = UDim2.new(0.8, 0, 0, 35)
ToggleButton.Position = UDim2.new(0.1, 0, 0.45, 0)
ToggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
ToggleButton.Text = "FLY: OFF"
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.TextSize = 14
ToggleButton.Font = Enum.Font.SourceSansBold
ToggleButton.Parent = MainFrame

local ButtonCorner = Instance.new("UICorner")
ButtonCorner.CornerRadius = UDim.new(0, 6)
ButtonCorner.Parent = ToggleButton

-- Hàm bắt đầu bay
local function StartFly()
	local character = LocalPlayer.Character
	if not character or not character:FindFirstChild("HumanoidRootPart") then return end
	
	local rootPart = character.HumanoidRootPart
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	
	if humanoid then
		humanoid:ChangeState(Enum.HumanoidStateType.PlatformStanding)
	end
	
	bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.MaxForce = Vector3.new(1, 1, 1) * 1000000
	bodyVelocity.Velocity = Vector3.zero
	bodyVelocity.Parent = rootPart
	
	bodyGyro = Instance.new("BodyGyro")
	bodyGyro.MaxTorque = Vector3.new(1, 1, 1) * 1000000
	bodyGyro.CFrame = rootPart.CFrame
	bodyGyro.P = 9000
	bodyGyro.Parent = rootPart
	
	isFlying = true
	ToggleButton.Text = "FLY: ON"
	ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
end

-- Hàm dừng bay
local function StopFly()
	isFlying = false
	ToggleButton.Text = "FLY: OFF"
	ToggleButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
	
	if bodyVelocity then bodyVelocity:Destroy() bodyVelocity = nil end
	if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
	
	local character = LocalPlayer.Character
	if character then
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		if humanoid then
			humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
		end
	end
end

-- Hàm Bật/Tắt
local function ToggleFly()
	if isFlying then
		StopFly()
	else
		StartFly()
	end
end

-- Sự kiện bấm nút trên Menu
ToggleButton.MouseButton1Click:Connect(function()
	ToggleFly()
end)

-- Xử lý di chuyển khi bay
RunService.RenderStepped:Connect(function()
	if not isFlying or not bodyVelocity or not bodyGyro then return end
	
	bodyGyro.CFrame = Camera.CFrame
	
	local forwardVector = Camera.CFrame.LookVector
	local rightVector = Camera.CFrame.RightVector
	local upVector = Camera.CFrame.UpVector
	
	local x = movementState.Right - movementState.Left
	local z = movementState.Backward - movementState.Forward
	local y = movementState.Up - movementState.Down
	
	local moveDirection = (forwardVector * -z) + (rightVector * x) + (upVector * y)
	
	if moveDirection.Magnitude > 0 then
		bodyVelocity.Velocity = moveDirection.Unit * FLY_SPEED
	else
		bodyVelocity.Velocity = Vector3.zero
	end
end)

-- Phím tắt phím F (Dành cho PC)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	
	if input.KeyCode == Enum.KeyCode.F then
		ToggleFly()
	end
	
	if input.KeyCode == Enum.KeyCode.W then movementState.Forward = 1 end
	if input.KeyCode == Enum.KeyCode.S then movementState.Backward = 1 end
	if input.KeyCode == Enum.KeyCode.A then movementState.Left = 1 end
	if input.KeyCode == Enum.KeyCode.D then movementState.Right = 1 end
	if input.KeyCode == Enum.KeyCode.Space then movementState.Up = 1 end
	if input.KeyCode == Enum.KeyCode.LeftShift then movementState.Down = 1 end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
	if input.KeyCode == Enum.KeyCode.W then movementState.Forward = 0 end
	if input.KeyCode == Enum.KeyCode.S then movementState.Backward = 0 end
	if input.KeyCode == Enum.KeyCode.A then movementState.Left = 0 end
	if input.KeyCode == Enum.KeyCode.D then movementState.Right = 0 end
	if input.KeyCode == Enum.KeyCode.Space then movementState.Up = 0 end
	if input.KeyCode == Enum.KeyCode.LeftShift then movementState.Down = 0 end
end)
