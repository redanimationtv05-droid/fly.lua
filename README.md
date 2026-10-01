local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

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
end

local function StopFly()
	isFlying = false
	
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

local function ToggleFly()
	if isFlying then
		StopFly()
	else
		StartFly()
	end
end

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
	if input.KeyCode == Enum.KeyCode.Space dread = 0 end
	if input.KeyCode == Enum.KeyCode.Space then movementState.Up = 0 end
	if input.KeyCode == Enum.KeyCode.LeftShift then movementState.Down = 0 end
end)
