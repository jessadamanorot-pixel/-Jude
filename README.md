local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer
local playerGui = localPlayer:WaitForChild("PlayerGui")

local isHpTextActive = false
local isUIVisible = true

-- 1. สร้าง ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PlayerHealthTextUI_System"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- 2. สร้าง UI เมนูสีดำ (ลากขยับตำแหน่งได้)
local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 95)
mainFrame.Position = UDim2.new(0.05, 0, 0.4, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 25)
title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
title.Text = "NAME & HP SYSTEM [Press K to Hide]"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 10
title.Font = Enum.Font.SourceSansBold
title.Parent = mainFrame

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0.9, 0, 0, 40)
toggleBtn.Position = UDim2.new(0.05, 0, 0.42, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 40)
toggleBtn.Text = "HP Text & ESP: OFF"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.SourceSansBold
toggleBtn.TextSize = 13
toggleBtn.Parent = mainFrame

-- 3. ฟังก์ชันสร้างข้อความชื่อ HP และสีคุมตัวผู้เล่น (Highlight ESP)
local function createHealthText(character)
	if not character or character == localPlayer.Character then return end

	local head = character:FindFirstChild("Head")
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if not head or not humanoid then return end

	-- ลบของเดิมก่อนหน้าถ้ามีอยู่
	local existingText = head:FindFirstChild("OverheadHPText")
	if existingText then existingText:Destroy() end
	
	local existingHighlight = character:FindFirstChild("PlayerHighlight")
	if existingHighlight then existingHighlight:Destroy() end

	-- === เพิ่มระบบ Highlight คุมตัวผู้เล่น ===
	local highlight = Instance.new("Highlight")
	highlight.Name = "PlayerHighlight"
	highlight.FillColor = Color3.fromRGB(0, 255, 120) -- สีตัวข้างใน (เขียวสด)
	highlight.FillTransparency = 0.5                  -- ความโปร่งแสงตัว
	highlight.OutlineColor = Color3.fromRGB(255, 255, 255) -- สีเส้นขอบ (ขาว)
	highlight.OutlineTransparency = 0                 -- ความเข้มเส้นขอบ
	highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- เห็นทะลุกำแพง
	highlight.Parent = character

	-- === สร้าง BillboardGui บนหัว ===
	local billboard = Instance.new("BillboardGui")
	billboard.Name = "OverheadHPText"
	billboard.Size = UDim2.new(0, 120, 0, 40)
	billboard.StudsOffset = Vector3.new(0, 4.5, 0)
	billboard.AlwaysOnTop = true
	billboard.Parent = head

	-- ข้อความแสดงชื่อผู้เล่น
	local nameLabel = Instance.new("TextLabel")
	nameLabel.Size = UDim2.new(1, 0, 0, 16)
	nameLabel.BackgroundTransparency = 1
	nameLabel.Text = character.Name
	nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	nameLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
	nameLabel.TextStrokeTransparency = 0
	nameLabel.Font = Enum.Font.SourceSansBold
	nameLabel.TextSize = 13
	nameLabel.Parent = billboard

	-- ข้อความแสดงค่า HP
	local hpLabel = Instance.new("TextLabel")
	hpLabel.Size = UDim2.new(1, 0, 0, 16)
	hpLabel.Position = UDim2.new(0, 0, 0, 18)
	hpLabel.BackgroundTransparency = 1
	hpLabel.Text = "HP: " .. math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth)
	hpLabel.TextColor3 = Color3.fromRGB(0, 255, 120)
	hpLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
	hpLabel.TextStrokeTransparency = 0
	hpLabel.Font = Enum.Font.SourceSansBold
	hpLabel.TextSize = 13
	hpLabel.Parent = billboard

	-- ฟังก์ชันสำหรับอัปเดตสีทั้งข้อความและ Highlight ตาม HP
	local function updateHealthEffects(currentHealth)
		local healthPercent = math.clamp(currentHealth / humanoid.MaxHealth, 0, 1)
		local newColor

		if healthPercent > 0.5 then
			newColor = Color3.fromRGB(0, 255, 120) -- เลือดเยอะ (เขียว)
		elseif healthPercent > 0.25 then
			newColor = Color3.fromRGB(255, 200, 0) -- เลือดปานกลาง (เหลือง)
		else
			newColor = Color3.fromRGB(255, 50, 50)  -- เลือดน้อย (แดง)
		end

		if hpLabel and hpLabel.Parent then
			hpLabel.Text = "HP: " .. math.floor(currentHealth) .. "/" .. math.floor(humanoid.MaxHealth)
			hpLabel.TextColor3 = newColor
		end

		if highlight and highlight.Parent then
			highlight.FillColor = newColor
		end
	end

	-- เรียกใช้อัปเดตสีครั้งแรก
	updateHealthEffects(humanoid.Health)

	-- อัปเดตสีข้อความและ Highlight อัตโนมัติเมื่อโดนความเสียหาย
	humanoid.HealthChanged:Connect(updateHealthEffects)
end

-- 4. ระบบกดปุ่มเปิด-ปิดการแสดงผล
local function toggleHpText()
	isHpTextActive = not isHpTextActive

	if isHpTextActive then
		toggleBtn.Text = "HP Text & ESP: ON"
		toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 150, 40)
	else
		toggleBtn.Text = "HP Text & ESP: OFF"
		toggleBtn.BackgroundColor3 = Color3.fromRGB(150, 40, 40)
	end

	for _, p in pairs(Players:GetPlayers()) do
		if p ~= localPlayer and p.Character then
			if isHpTextActive then
				createHealthText(p.Character)
			else
				-- ลบ Overhead Text
				local head = p.Character:FindFirstChild("Head")
				if head and head:FindFirstChild("OverheadHPText") then
					head.OverheadHPText:Destroy()
				end
				-- ลบ Highlight
				local highlight = p.Character:FindFirstChild("PlayerHighlight")
				if highlight then
					highlight:Destroy()
				end
			end
		end
	end
end

toggleBtn.MouseButton1Click:Connect(toggleHpText)

-- 5. กดปุ่ม K เพื่อซ่อน/แสดง เมนู UI หลัก
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.K then
		isUIVisible = not isUIVisible
		mainFrame.Visible = isUIVisible
	end
end)

-- 6. ระบบลากเคลื่อนย้าย UI (Draggable)
local dragging, dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		startPos = mainFrame.Position
	end
end)

mainFrame.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)
