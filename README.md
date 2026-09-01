local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local isActive = false
local moveSpeed = 1
local sensitivity = 0.3

local cameraCFrame = CFrame.new()
local rotX, rotY = 0, 0

-- ฟังก์ชันล็อก/ปลดล็อกการขยับของตัวละคร
local function setCharacterFrozen(frozen)
	local character = player.Character
	if character then
		local humanoidRootPart = character:FindFirstChild("HumanoidRootPart")
		if humanoidRootPart then
			humanoidRootPart.Anchored = frozen
		end
	end
end

-- ฟังก์ชันจัดการการเปิด/ปิด Freecam
local function toggleFreecam()
	isActive = not isActive
	
	if isActive then
		camera.CameraType = Enum.CameraType.Scriptable
		cameraCFrame = camera.CFrame
		local yaw, pitch, _ = cameraCFrame:ToOrientation()
		rotX, rotY = yaw, pitch
		
		-- ล็อกตัวละครให้อยู่กับที่ และล็อกเมาส์ให้อยู่ตรงกลาง
		setCharacterFrozen(true)
		UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
	else
		camera.CameraType = Enum.CameraType.Custom
		if player.Character and player.Character:FindFirstChild("Humanoid") then
			camera.CameraSubject = player.Character.Humanoid
		end
		
		-- ปลดล็อกตัวละคร และคืนค่าเมาส์ปกติ
		setCharacterFrozen(false)
		UserInputService.MouseBehavior = Enum.MouseBehavior.Default
	end
end

-- ตรวจจับการกดปุ่ม X
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.X then
		toggleFreecam()
	end
end)

-- ตรวจจับการขยับเมาส์
UserInputService.InputChanged:Connect(function(input)
	if isActive and input.UserInputType == Enum.UserInputType.MouseMovement then
		rotX = rotX - math.rad(input.Delta.Y * sensitivity)
		rotY = rotY - math.rad(input.Delta.X * sensitivity)
		rotX = math.clamp(rotX, math.rad(-89), math.rad(89))
	end
end)
-- อัปเดตตำแหน่งกล้องทุกเฟรม
RunService.RenderStepped:Connect(function(deltaTime)
	if not isActive then return end
	
	-- คำนวณการเคลื่อนที่ของกล้อง (WASD / Q / E)
	local forward = (UserInputService:IsKeyDown(Enum.KeyCode.W) and 1 or 0) - (UserInputService:IsKeyDown(Enum.KeyCode.S) and 1 or 0)
	local right = (UserInputService:IsKeyDown(Enum.KeyCode.D) and 1 or 0) - (UserInputService:IsKeyDown(Enum.KeyCode.A) and 1 or 0)
	local up = (UserInputService:IsKeyDown(Enum.KeyCode.E) and 1 or 0) - (UserInputService:IsKeyDown(Enum.KeyCode.Q) and 1 or 0)
	
	local currentSpeed = UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) and (moveSpeed * 2.5) or moveSpeed
	local rotationCFrame = CFrame.Angles(0, rotY, 0) * CFrame.Angles(rotX, 0, 0)
	local direction = (rotationCFrame * Vector3.new(right, up, -forward))
	
	cameraCFrame = cameraCFrame + (direction * currentSpeed)
	camera.CFrame = CFrame.new(cameraCFrame.Position) * rotationCFrame
end)
