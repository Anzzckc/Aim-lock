-- AIM LOCK PRO v1.4 - Cải thiện GUI bo tròn và nút bấm On/Off
-- Mục tiêu: Tạo GUI bo tròn, màu đen, với nút bấm On/Off.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

--------------------------------------------------------------------------------
-- Cấu hình (Có thể điều chỉnh)
--------------------------------------------------------------------------------
local settings = {
    keyToToggle = Enum.KeyCode.Q,       -- Phím tắt để bật/tắt aim lock
    aimlockSpeed = 0.1,                -- Tốc độ quay camera (0.1 là vừa phải)
    searchRadius = 250,                -- Bán kính tìm kiếm mục tiêu (tính bằng pixel trên màn hình)
    espFillColor = Color3.fromRGB(255, 0, 0),
    espOutlineColor = Color3.fromRGB(255, 255, 255),
    guiPosition = UDim2.new(0.45, 0, 0.1, 0) -- Vị trí của nút GUI
}

--------------------------------------------------------------------------------
-- Biến trạng thái
--------------------------------------------------------------------------------
local aimlockEnabled = false
local lockedTarget = nil
local currentESP = nil
local toggleBtn, targetLabel -- Khai báo các biến ở đây

--------------------------------------------------------------------------------
-- Hàm tiện ích
--------------------------------------------------------------------------------

local function createESP(targetChar)
    local esp = Instance.new("Highlight")
    esp.Name = "LockESP"
    esp.FillColor = settings.espFillColor
    esp.OutlineColor = settings.espOutlineColor
    esp.FillTransparency = 0.5
    esp.OutlineTransparency = 0
    esp.Adornee = targetChar
    esp.Parent = CoreGui
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

    for _, other in ipairs(Players:GetPlayers()) do
        if other ~= player and other.Character and other.Character:FindFirstChild("HumanoidRootPart") then
            local part = other.Character.HumanoidRootPart
            local screenPos, onScreen = camera:WorldToViewportPoint(part.Position)
            local screenDist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude

            if onScreen and screenDist < settings.searchRadius and screenDist < closestDist then
                closest = other
                closestDist = screenDist
            end
        end
    end
    return closest
end

-- Cập nhật trạng thái và GUI khi aim lock được bật/tắt
local function toggleAimlock()
    aimlockEnabled = not aimlockEnabled
    
    if aimlockEnabled then
        toggleBtn.Text = "On"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(30, 255, 30) -- Màu xanh lá khi bật
        toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    else
        lockedTarget = nil
        removeESP()
        toggleBtn.Text = "Off"
        toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40) -- Màu đen khi tắt
        toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        targetLabel.Text = ""
    end
end

-- Tạo GUI với nút bấm On/Off
local function createGUI()
    local gui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))
    gui.Name = "AimLockGUI"
    gui.ResetOnSpawn = false

    toggleBtn = Instance.new("TextButton")
    toggleBtn.Size = UDim2.new(0, 100, 0, 50)
    toggleBtn.Position = settings.guiPosition
    toggleBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40) -- Màu đen mặc định
    toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggleBtn.Text = "Off"
    toggleBtn.Parent = gui
    toggleBtn.Draggable = true
    toggleBtn.Active = true
    toggleBtn.Font = Enum.Font.SourceSansBold
    toggleBtn.TextSize = 20

    -- Bo tròn góc cho nút
    local corner = Instance.new("UICorner", toggleBtn)
    corner.CornerRadius = UDim.new(0.25, 0) -- Độ bo tròn vừa phải

    targetLabel = Instance.new("TextLabel", gui)
    targetLabel.Size = UDim2.new(0, 300, 0, 25)
    targetLabel.Position = UDim2.new(0.35, 0, 0.17, 0)
    targetLabel.BackgroundTransparency = 1
    targetLabel.TextColor3 = Color3.new(1, 0.3, 0.3)
    targetLabel.Text = ""
end

-- Marker để đánh dấu mục tiêu
local marker = Instance.new("BillboardGui")
marker.Size = UDim2.new(0, 10, 0, 10)
marker.AlwaysOnTop = true
marker.Enabled = false
local dot = Instance.new("Frame", marker)
dot.Size = UDim2.new(1, 0, 1, 0)
dot.BackgroundColor3 = Color3.new(1, 0, 0)
dot.BorderSizePixel = 0

--------------------------------------------------------------------------------
-- Khởi tạo và Kết nối sự kiện
--------------------------------------------------------------------------------

createGUI()

toggleBtn.MouseButton1Click:Connect(toggleAimlock)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == settings.keyToToggle then
        toggleAimlock()
    end
end)

RunService.Heartbeat:Connect(function()
    if not aimlockEnabled then 
        marker.Enabled = false
        return 
    end

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

        camera.CFrame = camera.CFrame:Lerp(CFrame.new(camPos, targetPos), settings.aimlockSpeed)

        marker.Adornee = part
        if not marker.Parent then
            marker.Parent = CoreGui
        end
        marker.Enabled = true

        targetLabel.Text = "Locking: " .. lockedTarget.Name
    else
        removeESP()
        marker.Enabled = false
        targetLabel.Text = ""
    end
end)
