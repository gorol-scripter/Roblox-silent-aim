# Roblox-silent-aim
Roblox用　silent aim Script
-- AutoAim Demo (最強完全安全完璧完成版)
-- StarterPlayer > StarterPlayerScripts に LocalScript として入れてね

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local autoAimEnabled = false

-- ===== GUI =====
local function createGui()
	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "AutoAimGUI"
	screenGui.ResetOnSpawn = true
	screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

	local frame = Instance.new("Frame")
	frame.Name = "ToggleFrame"
	frame.Size = UDim2.new(0, 120, 0, 36)
	frame.Position = UDim2.new(0.02, 0, 0.12, 0)
	frame.BackgroundColor3 = Color3.fromRGB(30,30,30)
	frame.BorderSizePixel = 0
	frame.Active = true
	frame.Parent = screenGui

	local toggle = Instance.new("TextButton")
	toggle.Size = UDim2.new(1, -8, 1, -8)
	toggle.Position = UDim2.new(0, 4, 0, 4)
	toggle.BackgroundColor3 = Color3.fromRGB(70,70,70)
	toggle.TextColor3 = Color3.fromRGB(255,255,255)
	toggle.Font = Enum.Font.Gotham
	toggle.TextSize = 14
	toggle.Text = "AutoAim: OFF"
	toggle.Parent = frame

	-- ===== ドラッグ処理 (長押しで開始) =====
	local dragging, dragStart, startPos, dragInput
	local holdTime = 0.15 -- 0.15秒長押しでドラッグ開始

	frame.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			local pressed = true
			local start = tick()
			task.spawn(function()
				while pressed and dragging == false do
					if tick() - start >= holdTime then
						dragging = true
						dragStart = input.Position
						startPos = frame.Position
						break
					end
					task.wait()
				end
			end)

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					pressed = false
					dragging = false
				end
			end)
		end
	end)

	frame.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)

	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging and dragStart and startPos then
			local delta = input.Position - dragStart
			frame.Position = UDim2.new(
				startPos.X.Scale, startPos.X.Offset + delta.X,
				startPos.Y.Scale, startPos.Y.Offset + delta.Y
			)
		end
	end)

	-- ===== トグル切り替え =====
	local function setAutoAim(enabled)
		autoAimEnabled = enabled
		if enabled then
			toggle.Text = "AutoAim: ON"
			toggle.BackgroundColor3 = Color3.fromRGB(0,170,85)
		else
			toggle.Text = "AutoAim: OFF"
			toggle.BackgroundColor3 = Color3.fromRGB(70,70,70)
		end
	end

	toggle.MouseButton1Click:Connect(function()
		setAutoAim(not autoAimEnabled)
	end)
end

-- GUI作成
createGui()

-- ===== ターゲット検索 =====
local function getNearestTarget()
	local char = LocalPlayer.Character
	if not char then return nil end
	local root = char:FindFirstChild("HumanoidRootPart")
	if not root then return nil end
	local rootPos = root.Position

	local nearest, nearestDist = nil, math.huge
	for _, obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("Model") and obj:FindFirstChild("Humanoid") then
			if obj ~= char then
				local hum = obj:FindFirstChild("Humanoid")
				if hum and hum.Health > 0 then
					local part = obj:FindFirstChild("Head") or obj:FindFirstChild("HumanoidRootPart")
					if part then
						local d = (part.Position - rootPos).Magnitude
						if d < nearestDist then
							nearestDist = d
							nearest = part
						end
					end
				end
			end
		end
	end
	return nearest
end

-- ===== メインループ =====
RunService.RenderStepped:Connect(function(dt)
	if autoAimEnabled then
		local targetPart = getNearestTarget()
		if targetPart then
			local camPos = Camera.CFrame.Position
			local currentLook = Camera.CFrame.LookVector
			local targetDir = (targetPart.Position - camPos).Unit

			-- ほぼ瞬時に吸い付く (チート感MAX)
			local alpha = 0.95 -- 1に近いほど「ガチピタ」
			local newLook = currentLook:Lerp(targetDir, alpha)

			Camera.CFrame = CFrame.new(camPos, camPos + newLook)
		end
	end
end)
