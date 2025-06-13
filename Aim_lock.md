-- AIM LOCK PRO v1.4 - by An (No distance + ESP người bị lock)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local aimlockEnabled = false
local lockedTarget = nil
local currentESP = nil
local keyToToggle = Enum.KeyCode.Q

-- GUI setup
local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
gui.Name = "AimLockGUI"
gui.ResetOnSpawn = false

local toggleBtn = Instance.new("TextButton")
toggleBtn.Size = UDim2.new(0, 100, 0, 50)
toggleBtn.Position = UDim2.new(0.45, 0, 0.1, 0)
toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Text = "off"
toggleBtn.Parent = gui
toggleBtn.Draggable = true
toggleBtn.Active = true

local targetLabel = Instance.new("TextLabel", gui)
targetLabel.Size = UDim2.new(0, 300, 0, 25)
targetLabel.Position = UDim2.new(0.35, 0, 0.17, 0)
targetLabel.BackgroundTransparency = 1
targetLabel.TextColor3 = Color3.new(1, 0.3, 0.3)
targetLabel.Text = ""

local marker = Instance.new("BillboardGui")
marker.Size = UDim2.new(0, 10, 0, 10)
marker.AlwaysOnTop = true
marker.Enabled = false
local dot = Instance.new("Frame", marker)
dot.Size = UDim2.new(1, 0, 1, 0)
dot.BackgroundColor3 = Color3.new(1, 0, 0)
dot.BorderSizePixel = 0

-- Tạo ESP highlight
local function createESP(targetChar)
	local esp = Instance.new("Highlight")
	esp.Name = "LockESP"
	esp.FillColor = Color3.fromRGB(255, 0, 0)
	esp.OutlineColor = Color3.fromRGB(255, 255, 255)
	esp.FillTransparency = 0.5
	esp.OutlineTransparency = 0
	esp.Adornee = targetChar
	esp.Parent = game:GetService("CoreGui")
	return esp
end

local function removeESP()
	if currentESP then
		currentESP:Destroy()
		currentESP = nil
	end
end

local function getTarget()
	local closest, closestDist = nil, math.huge
	local myChar = player.Character
	if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end
	local screenCenter = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)

	for _, other in pairs(Players:GetPlayers()) do
		if other ~= player and other.Character and other.Character:FindFirstChild("HumanoidRootPart") then
			local part = other.Character.HumanoidRootPart
			local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
			local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude

			if onScreen and screenDist < closestDist then
				closest = other
				closestDist = screenDist
			end
		end
	end

	return closest
end

toggleBtn.MouseButton1Click:Connect(function()
	aimlockEnabled = not aimlockEnabled
	toggleBtn.Text = aimlockEnabled and "on" or "off"
	if not aimlockEnabled then
		lockedTarget = nil
		removeESP()
		marker.Enabled = false
		targetLabel.Text = ""
	end
end)

UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.KeyCode == keyToToggle then
		aimlockEnabled = not aimlockEnabled
		toggleBtn.Text = aimlockEnabled and "on" or "off"
		if not aimlockEnabled then
			lockedTarget = nil
			removeESP()
			marker.Enabled = false
			targetLabel.Text = ""
		end
	end
end)

RunService.Heartbeat:Connect(function()
	if not aimlockEnabled then return end

	if not lockedTarget or not lockedTarget.Character or not lockedTarget.Character:FindFirstChild("HumanoidRootPart") then
		removeESP()
		lockedTarget = getTarget()

		if lockedTarget and lockedTarget.Character then
			removeESP()
			currentESP = createESP(lockedTarget.Character)
		end
	end

	if lockedTarget and lockedTarget.Character and lockedTarget.Character:FindFirstChild("HumanoidRootPart") then
		local part = lockedTarget.Character.HumanoidRootPart
		local camPos = camera.CFrame.Position
		local targetPos = part.Position

		camera.CFrame = CFrame.new(camPos, camPos:Lerp(targetPos, 0.1))

		marker.Adornee = part
		if not marker.Parent then
			marker.Parent = game:GetService("CoreGui")
		end
		marker.Enabled = true

		targetLabel.Text = "Locking: " .. lockedTarget.Name
	else
		removeESP()
		marker.Enabled = false
		targetLabel.Text = ""
	end
end)
