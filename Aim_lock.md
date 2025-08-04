-- AIM LOCK PRO v2.0 - by An (Original GUI with iPhone-like Rounded Corners, Instant Aim Lock, HP Display)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera
local CoreGui = game:GetService("CoreGui")

-- Cài đặt có thể tùy chỉnh
local Settings = {
	AimlockEnabled = false,
	ESPEnabled = true,
	MaxDistance = 500, -- Khoảng cách tối đa để khóa mục tiêu
	LockPart = "Head", -- Bộ phận khóa: "Head" hoặc "HumanoidRootPart"
	ToggleKey = Enum.KeyCode.Q, -- Phím bật/tắt aim lock
	ESPColor = Color3.fromRGB(255, 0, 0), -- Màu ESP
}

local lockedTarget = nil
local currentESP = nil

-- GUI setup (giữ giao diện gốc)
local gui = Instance.new("ScreenGui")
gui.Name = "AimLockGUI"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Nút bật/tắt aim lock
local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 100, 0, 50)
toggleBtn.Position = UDim2.new(0.45, 0, 0.1, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Text = "off"
toggleBtn.Font = Enum.Font.SourceSansSemibold
toggleBtn.TextSize = 18
toggleBtn.Active = true
toggleBtn.Draggable = true
toggleBtn.Parent = gui

-- Thêm UICorner cho toggleBtn
local toggleBtnCorner = Instance.new("UICorner")
toggleBtnCorner.CornerRadius = UDim.new(0, 10) -- Bo tròn giống iPhone
toggleBtnCorner.Parent = toggleBtn

-- Nhãn hiển thị mục tiêu và HP
local targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(0, 300, 0, 25)
targetLabel.Position = UDim2.new(0.35, 0, 0.17, 0)
targetLabel.BackgroundTransparency = 1
targetLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
targetLabel.Text = ""
targetLabel.Font = Enum.Font.SourceSansSemibold
targetLabel.TextSize = 16
targetLabel.Parent = gui

-- Marker cho mục tiêu
local marker = Instance.new("BillboardGui")
marker.Size = UDim2.new(0, 10, 0, 10)
marker.AlwaysOnTop = true
marker.Enabled = false
local dot = Instance.new("Frame", marker)
dot.Size = UDim2.new(1, 0, 1, 0)
dot.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
dot.BorderSizePixel = 0
-- Thêm UICorner cho dot
local dotCorner = Instance.new("UICorner")
dotCorner.CornerRadius = UDim.new(0.5, 0) -- Hình tròn hoàn toàn
dotCorner.Parent = dot
marker.Parent = CoreGui

-- Tạo ESP highlight
local function createESP(targetChar)
	if not Settings.ESPEnabled then return nil end
	local esp = Instance.new("Highlight")
	esp.Name = "LockESP"
	esp.FillColor = Settings.ESPColor
	esp.OutlineColor = Color3.fromRGB(255, 255, 255)
	esp.FillTransparency = 0.5
	esp.OutlineTransparency = 0
	esp.Adornee = targetChar
	esp.Parent = CoreGui
	return esp
end

-- Xóa ESP
local function removeESP()
	if currentESP then
		currentESP:Destroy()
		currentESP = nil
	end
end

-- Tìm mục tiêu gần nhất
local function getTarget()
	local closest, closestDist = nil, math.huge
	local myChar = player.Character
	if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end
	local myPos = myChar.HumanoidRootPart.Position
	local screenCenter = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)

	for _, other in pairs(Players:GetPlayers()) do
		if other ~= player and other.Character and other.Character:FindFirstChild("HumanoidRootPart") then
			local part = other.Character:FindFirstChild(Settings.LockPart) or other.Character.HumanoidRootPart
			local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
			local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
			local worldDist = (myPos - part.Position).Magnitude

			if onScreen and screenDist < closestDist and worldDist <= Settings.MaxDistance then
				closest = other
				closestDist = screenDist
			end
		end
	end

	return closest
end

-- Xử lý nút bật/tắt aim lock
toggleBtn.MouseButton1Click:Connect(function()
	Settings.AimlockEnabled = not Settings.AimlockEnabled
	toggleBtn.Text = Settings.AimlockEnabled and "on" or "off"
	if not Settings.AimlockEnabled then
		lockedTarget = nil
		removeESP()
		marker.Enabled = false
		targetLabel.Text = ""
	end
end)

-- Xử lý phím tắt
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.KeyCode == Settings.ToggleKey then
		Settings.AimlockEnabled = not Settings.AimlockEnabled
		toggleBtn.Text = Settings.AimlockEnabled and "on" or "off"
		if not Settings.AimlockEnabled then
			lockedTarget = nil
			removeESP()
			marker.Enabled = false
			targetLabel.Text = ""
		end
	end
end)

-- Cập nhật aim lock
RunService.RenderStepped:Connect(function()
	if not Settings.AimlockEnabled then return end

	-- Kiểm tra và cập nhật mục tiêu
	if not lockedTarget or not lockedTarget.Character or not lockedTarget.Character:FindFirstChild(Settings.LockPart) then
		lockedTarget = getTarget()
		removeESP()
		if lockedTarget and lockedTarget.Character then
			currentESP = createESP(lockedTarget.Character)
		end
	end

	-- Xử lý khóa mục tiêu
	if lockedTarget and lockedTarget.Character and lockedTarget.Character:FindFirstChild(Settings.LockPart) then
		local part = lockedTarget.Character:FindFirstChild(Settings.LockPart) or lockedTarget.Character.HumanoidRootPart
		local camPos = camera.CFrame.Position
		local targetPos = part.Position

		-- Aim lock tức thì (khóa cứng, không mượt)
		camera.CFrame = CFrame.new(camPos, targetPos)

		-- Cập nhật marker
		marker.Adornee = part
		marker.Enabled = true

		-- Hiển thị tên và HP
		local humanoid = lockedTarget.Character:FindFirstChild("Humanoid")
		if humanoid then
			local health = math.floor(humanoid.Health + 0.5) -- Làm tròn số máu
			local maxHealth = math.floor(humanoid.MaxHealth + 0.5)
			targetLabel.Text = "Locking: " .. lockedTarget.Name .. " (HP: " .. health .. "/" .. maxHealth .. ")"
		else
			targetLabel.Text = "Locking: " .. lockedTarget.Name .. " (HP: N/A)"
		end
	else
		removeESP()
		marker.Enabled = false
		targetLabel.Text = ""
	end
end)
