-- SERVICES
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- STATES
local highlightEnabled = false
local namesEnabled = false
local tracesEnabled = false
local rainbowMode = false
local flyEnabled = false
local speedEnabled = false
local jumpEnabled = false
local noclipEnabled = false

-- STORAGE
local highlights = {}
local nameTags = {}
local traces = {}

-- COLORS
local presetColors = {
	{Color = Color3.fromRGB(255,0,0), Name = "Red"},
	{Color = Color3.fromRGB(0,170,255), Name = "Blue"},
	{Color = Color3.fromRGB(0,255,100), Name = "Green"}
}
local currentColor = presetColors[1].Color

-- PLAYER CONTROL
local WalkSpeed = 16
local JumpPower = 50
local flySpeed = 50

-- CLEAR VISUALS
local function clearPlayer(player)
	if highlights[player] then highlights[player]:Destroy() highlights[player]=nil end
	if nameTags[player] then nameTags[player]:Destroy() nameTags[player]=nil end
	if traces[player] then
		for _,obj in pairs(traces[player]) do obj:Destroy() end
		traces[player]=nil
	end
end

-- APPLY VISUALS
local function applyVisuals(player)
	if player == LocalPlayer then return end
	if not player.Character then return end
	local char = player.Character

	if highlightEnabled then
		local h = Instance.new("Highlight")
		h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
		h.FillColor = currentColor
		h.OutlineColor = Color3.new(1,1,1)
		h.Parent = char
		highlights[player] = h
	end

	if namesEnabled then
		local head = char:FindFirstChild("Head")
		if head then
			local bill = Instance.new("BillboardGui", head)
			bill.Size = UDim2.new(0,220,0,45)
			bill.StudsOffset = Vector3.new(0,2.5,0)
			bill.AlwaysOnTop = true
			local label = Instance.new("TextLabel", bill)
			label.Size = UDim2.new(1,0,1,0)
			label.BackgroundTransparency = 1
			label.TextScaled = true
			label.TextStrokeTransparency = 0
			label.TextStrokeColor3 = Color3.new(0,0,0)
			label.TextColor3 = currentColor
			label.Text = player.Name
			nameTags[player] = bill
		end
	end

	if tracesEnabled then
		local root = char:FindFirstChild("HumanoidRootPart")
		local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
		if root and myRoot then
			local a0 = Instance.new("Attachment", myRoot)
			local a1 = Instance.new("Attachment", root)
			local beam = Instance.new("Beam", myRoot)
			beam.Attachment0 = a0
			beam.Attachment1 = a1
			beam.Width0 = 0.35
			beam.Width1 = 0.35
			beam.LightEmission = 1
			beam.Brightness = 5
			beam.FaceCamera = true
			beam.Color = ColorSequence.new(currentColor)
			traces[player] = {beam,a0,a1}
		end
	end
end

-- REFRESH ALL
local function refreshAll()
	for _,p in pairs(Players:GetPlayers()) do
		clearPlayer(p)
		applyVisuals(p)
	end
end

-- GUI CREATION
local gui = Instance.new("ScreenGui")
gui.Name = "ZenthAdmin"
gui.ResetOnSpawn = false
gui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0,360,0,500)
local fullSize = UDim2.new(0,360,0,500)
local minimizedSize = UDim2.new(0,360,0,40)
local minimized = false
main.Position = UDim2.new(0.5,-180,1,100) -- start offscreen
main.BackgroundTransparency = 1
main.BackgroundColor3 = Color3.fromRGB(25,25,25)
main.BorderSizePixel = 0
main.Active = true
main.Draggable = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0,12)
-- INTRO ANIMATION
task.wait()

TweenService:Create(
	main,
	TweenInfo.new(0.6, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
	{
		Position = UDim2.new(0.5,-180,0.5,-250),
		BackgroundTransparency = 0
	}
):Play()

-- TITLE
local titleBar = Instance.new("Frame", main)
titleBar.Size = UDim2.new(1,0,0,40)
titleBar.BackgroundColor3 = Color3.fromRGB(35,35,35)
titleBar.BorderSizePixel = 0

local title = Instance.new("TextLabel", titleBar)
title.Size = UDim2.new(1,-120,1,0)
title.Position = UDim2.new(0,10,0,0)
title.BackgroundTransparency = 1
title.Text = "Zenth Admin Panel"
title.TextColor3 = Color3.new(1,1,1)
title.TextScaled = true

local minimizeBtn = Instance.new("TextButton", titleBar)
minimizeBtn.Size = UDim2.new(0,40,0,30)
minimizeBtn.Position = UDim2.new(1,-90,0,5)
minimizeBtn.Text = "-"
minimizeBtn.BackgroundColor3 = Color3.fromRGB(80,80,0)

local destroyBtn = Instance.new("TextButton", titleBar)
destroyBtn.Size = UDim2.new(0,40,0,30)
destroyBtn.Position = UDim2.new(1,-45,0,5)
destroyBtn.Text = "X"
destroyBtn.BackgroundColor3 = Color3.fromRGB(150,0,0)

-- SCROLLABLE CONTENT
local scrollFrame = Instance.new("ScrollingFrame", main)
scrollFrame.Size = UDim2.new(1,0,1,-40)
scrollFrame.Position = UDim2.new(0,0,0,40)
scrollFrame.BackgroundTransparency = 1
scrollFrame.ScrollBarThickness = 8

local UIListLayout = Instance.new("UIListLayout", scrollFrame)
UIListLayout.Padding = UDim.new(0,10)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- BUTTON CREATOR
local function createButton(text)
	local b = Instance.new("TextButton", scrollFrame)
	b.Size = UDim2.new(1,-20,0,38)
	b.BackgroundColor3 = Color3.fromRGB(45,45,45)
	b.TextColor3 = Color3.new(1,1,1)
	b.TextScaled = true
	b.Text = text
	Instance.new("UICorner", b).CornerRadius = UDim.new(0,8)
	return b
end

-- TOGGLES
local highlightBtn = createButton("Highlights [OFF]")
highlightBtn.MouseButton1Click:Connect(function()
	highlightEnabled = not highlightEnabled

	highlightBtn.Text = "Highlights [" .. (highlightEnabled and "ON" or "OFF") .. "]"
	highlightBtn.BackgroundColor3 = highlightEnabled
		and Color3.fromRGB(30,100,30)
		or Color3.fromRGB(45,45,45)

	refreshAll()
end)

local namesBtn = createButton("Names [OFF]")
namesBtn.MouseButton1Click:Connect(function()
	namesEnabled = not namesEnabled

	namesBtn.Text = "Names [" .. (namesEnabled and "ON" or "OFF") .. "]"
	namesBtn.BackgroundColor3 = namesEnabled
		and Color3.fromRGB(30,100,30)
		or Color3.fromRGB(45,45,45)

	refreshAll()
end)

local tracesBtn = createButton("Traces [OFF]")
tracesBtn.MouseButton1Click:Connect(function()
	tracesEnabled = not tracesEnabled

	tracesBtn.Text = "Traces [" .. (tracesEnabled and "ON" or "OFF") .. "]"
	tracesBtn.BackgroundColor3 = tracesEnabled
		and Color3.fromRGB(30,100,30)
		or Color3.fromRGB(45,45,45)

	refreshAll()
end)

-- COLOR SECTION
local colorLabel = Instance.new("TextLabel", scrollFrame)
colorLabel.Size = UDim2.new(1,-20,0,25)
colorLabel.BackgroundTransparency = 1
colorLabel.Text = "Customize Color"
colorLabel.TextColor3 = Color3.new(1,1,1)
colorLabel.TextScaled = true

local colorFrame = Instance.new("Frame", scrollFrame)
colorFrame.Size = UDim2.new(1,0,0,40)
colorFrame.BackgroundTransparency = 1

local UIListLayoutColors = Instance.new("UIListLayout", colorFrame)
UIListLayoutColors.FillDirection = Enum.FillDirection.Horizontal
UIListLayoutColors.Padding = UDim.new(0,10)
UIListLayoutColors.SortOrder = Enum.SortOrder.LayoutOrder

for _,c in ipairs(presetColors) do
	local btn = Instance.new("TextButton", colorFrame)
	btn.Size = UDim2.new(0,60,1,0)
	btn.BackgroundColor3 = c.Color
	btn.Text = c.Name
	btn.TextColor3 = Color3.new(1,1,1)
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)
	btn.MouseButton1Click:Connect(function()
		rainbowMode = false
		currentColor = c.Color
		refreshAll()
	end)
end

local rainbowBtn = Instance.new("TextButton", colorFrame)
rainbowBtn.Size = UDim2.new(0,80,1,0)
rainbowBtn.BackgroundColor3 = Color3.fromRGB(255,255,255)
rainbowBtn.Text = "🌈 Rainbow"
Instance.new("UICorner", rainbowBtn).CornerRadius = UDim.new(0,6)
rainbowBtn.MouseButton1Click:Connect(function()
	rainbowMode = true
	refreshAll()
end)

-- PLAYER CONTROL LABEL
local controlLabel = Instance.new("TextLabel", scrollFrame)
controlLabel.Size = UDim2.new(1,-20,0,25)
controlLabel.BackgroundTransparency = 1
controlLabel.Text = "Player Controls"
controlLabel.TextColor3 = Color3.new(1,1,1)
controlLabel.TextScaled = true

-- WalkSpeed
local walkBox = Instance.new("TextBox", scrollFrame)
walkBox.Size = UDim2.new(1,-20,0,30)
walkBox.PlaceholderText = "Enter WalkSpeed"
walkBox.Text = tostring(WalkSpeed)
walkBox.TextScaled = true
walkBox.ClearTextOnFocus = false

local walkToggle = createButton("Enable WalkSpeed")
walkToggle.MouseButton1Click:Connect(function()
	speedEnabled = not speedEnabled

	walkToggle.Text = speedEnabled and "Disable WalkSpeed" or "Enable WalkSpeed"
	walkToggle.BackgroundColor3 = speedEnabled and Color3.fromRGB(30,100,30) or Color3.fromRGB(45,45,45)

	if not speedEnabled then
		if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
			LocalPlayer.Character.Humanoid.WalkSpeed = 16
		end
	end
end)

-- JumpPower
local jumpBox = Instance.new("TextBox", scrollFrame)
jumpBox.Size = UDim2.new(1,-20,0,30)
jumpBox.PlaceholderText = "Enter JumpPower"
jumpBox.Text = tostring(JumpPower)
jumpBox.TextScaled = true
jumpBox.ClearTextOnFocus = false

local jumpToggle = createButton("Enable JumpPower")
jumpToggle.MouseButton1Click:Connect(function()
	jumpEnabled = not jumpEnabled

	jumpToggle.Text = jumpEnabled and "Disable JumpPower" or "Enable JumpPower"
	jumpToggle.BackgroundColor3 = jumpEnabled and Color3.fromRGB(30,100,30) or Color3.fromRGB(45,45,45)

	if not jumpEnabled then
		if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
			LocalPlayer.Character.Humanoid.JumpPower = 50
		end
	end
end)

-- Fly
local flyBox = Instance.new("TextBox", scrollFrame)
flyBox.Size = UDim2.new(1,-20,0,30)
flyBox.PlaceholderText = "Fly Speed"
flyBox.Text = tostring(flySpeed)
flyBox.TextScaled = true
flyBox.ClearTextOnFocus = false

local flyToggleBtn = createButton("Enable Fly")
flyToggleBtn.MouseButton1Click:Connect(function()
	flyEnabled = not flyEnabled
	flyToggleBtn.Text = flyEnabled and "Disable Fly" or "Enable Fly"
	-- match the same green/normal styles as your other toggles
	flyToggleBtn.BackgroundColor3 = flyEnabled and Color3.fromRGB(30,100,30) or Color3.fromRGB(45,45,45)
end)

-- Noclip
local noclipBtn = createButton("Enable Noclip")
noclipBtn.MouseButton1Click:Connect(function()
	noclipEnabled = not noclipEnabled

	noclipBtn.Text = noclipEnabled and "Disable Noclip" or "Enable Noclip"
	noclipBtn.BackgroundColor3 = noclipEnabled and Color3.fromRGB(30,100,30) or Color3.fromRGB(45,45,45)
end)

-- MINIMIZE
-- MINIMIZE / MAXIMIZE TOGGLE
local TweenService = game:GetService("TweenService")

minimizeBtn.MouseButton1Click:Connect(function()

	minimized = not minimized

	if minimized then
		-- MINIMIZE
		local tweenMain = TweenService:Create(
			main,
			TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
			{Size = minimizedSize}
		)
		tweenMain:Play()

		local tweenScroll = TweenService:Create(
			scrollFrame,
			TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
			{Size = UDim2.new(1,0,0,0)}
		)
		tweenScroll:Play()

	else
		-- MAXIMIZE
		local tweenMain = TweenService:Create(
			main,
			TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
			{Size = fullSize}
		)
		tweenMain:Play()

		local tweenScroll = TweenService:Create(
			scrollFrame,
			TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
			{Size = UDim2.new(1,0,1,-40)}
		)
		tweenScroll:Play()
	end

end)

-- DESTROY
-- DESTROY (with animation)
destroyBtn.MouseButton1Click:Connect(function()

	for _,player in pairs(Players:GetPlayers()) do
		clearPlayer(player)
	end

	speedEnabled = false
	jumpEnabled = false
	flyEnabled = false
	noclipEnabled = false

	local tween = TweenService:Create(
		main,
		TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
		{
			Position = UDim2.new(0.5,-180,1,100),
			BackgroundTransparency = 1
		}
	)

	tween:Play()
	tween.Completed:Wait()

	gui:Destroy()
end)

-- PLAYER ADDED
Players.PlayerAdded:Connect(function(player)
	player.CharacterAdded:Connect(function()
		task.wait(0.3)
		applyVisuals(player)
	end)
end)

-- FLY & NOCLIP FUNCTION
local function setNoclip(state)
	local char = LocalPlayer.Character
	if not char then return end

	for _, part in ipairs(char:GetDescendants()) do
		if part:IsA("BasePart") then
			part.CanCollide = not state
			part.CanTouch = not state
		end
	end
end

-- MAIN LOOP
-- MAIN LOOP (clean)
RunService.RenderStepped:Connect(function()
	-- rainbow color update
	if rainbowMode then
		currentColor = Color3.fromHSV((tick() % 5) / 5, 1, 1)
	end

	-- Update visuals
	for _, h in pairs(highlights) do
		if h and h.Parent then
			h.FillColor = currentColor
		end
	end

	for _, n in pairs(nameTags) do
		local label = n:FindFirstChildOfClass("TextLabel")
		if label then
			label.TextColor3 = currentColor
		end
	end

	for _, tbl in pairs(traces) do
		if tbl and tbl[1] then
			tbl[1].Color = ColorSequence.new(currentColor)
		end
	end

	-- Ensure character exists before doing player controls
	local char = LocalPlayer.Character
	if not char then
		return
	end

	local humanoid = char:FindFirstChild("Humanoid")
	local hrp = char:FindFirstChild("HumanoidRootPart")

	-- WalkSpeed (applied to humanoid if enabled)
	if humanoid and speedEnabled then
		local val = tonumber(walkBox.Text)
		if val then
			humanoid.WalkSpeed = val
		end
	end

	-- JumpPower
	if humanoid and jumpEnabled then
		local val = tonumber(jumpBox.Text)
		if val then
			humanoid.JumpPower = val
		end
	end

	-- FLY
	if flyEnabled and humanoid and hrp then
		-- force physics state and prevent auto-rotation
		humanoid:ChangeState(Enum.HumanoidStateType.Physics)
		humanoid.AutoRotate = false

		-- BodyVelocity
		local bv = hrp:FindFirstChild("FlyVelocity")
		if not bv then
			bv = Instance.new("BodyVelocity")
			bv.Name = "FlyVelocity"
			bv.Parent = hrp
		end
		bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)

		-- BodyGyro
		local bg = hrp:FindFirstChild("FlyGyro")
		if not bg then
			bg = Instance.new("BodyGyro")
			bg.Name = "FlyGyro"
			bg.Parent = hrp
		end
		bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
		bg.P = 100000
		bg.CFrame = Camera.CFrame

		-- Movement direction (camera-relative)
		local direction = Vector3.new(0,0,0)
		if UserInputService:IsKeyDown(Enum.KeyCode.W) then
			direction = direction + Camera.CFrame.LookVector
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.S) then
			direction = direction - Camera.CFrame.LookVector
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.A) then
			direction = direction - Camera.CFrame.RightVector
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.D) then
			direction = direction + Camera.CFrame.RightVector
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
			direction = direction + Vector3.new(0,1,0)
		end
		if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then
			direction = direction - Vector3.new(0,1,0)
		end

		local speed = tonumber(flyBox.Text) or flySpeed
		if direction.Magnitude > 0 then
			bv.Velocity = direction.Unit * speed
		else
			bv.Velocity = Vector3.new(0,0,0)
		end
	else
		-- cleanup fly objects if present
		if hrp then
			local bv = hrp:FindFirstChild("FlyVelocity")
			local bg = hrp:FindFirstChild("FlyGyro")
			if bv then bv:Destroy() end
			if bg then bg:Destroy() end
		end
		if humanoid then
			humanoid.AutoRotate = true
		end
	end

	-- NOCLIP (apply to character if enabled)
	if noclipEnabled and char then
		for _, part in ipairs(char:GetDescendants()) do
			if part:IsA("BasePart") then
				part.CanCollide = false
				part.CanTouch = false
				-- part.CanQuery = false -- mostly unnecessary, but you can uncomment if desired
			end
		end
	else
		-- Optionally restore collisions when noclip disabled
		if char and not noclipEnabled then
			for _, part in ipairs(char:GetDescendants()) do
				if part:IsA("BasePart") then
					part.CanCollide = true
					part.CanTouch = true
				end
			end
		end
	end
end)

-- CLICK TO TELEPORT TOGGLE
local clickTeleportEnabled = false
local teleportDebounce = false

-- Create Button
local clickTeleportBtn = createButton("Click To Teleport: OFF")
clickTeleportBtn.MouseButton1Click:Connect(function()
	clickTeleportEnabled = not clickTeleportEnabled
	clickTeleportBtn.Text = clickTeleportEnabled and "Click To Teleport: ON" or "Click To Teleport: OFF"
	clickTeleportBtn.BackgroundColor3 = clickTeleportEnabled and Color3.fromRGB(0,170,0) or Color3.fromRGB(170,0,0)
end)

-- Function to safely get HumanoidRootPart
local function getRootPart()
	if LocalPlayer.Character then
		return LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
	end
	return nil
end

-- Mouse teleport logic
local mouse = LocalPlayer:GetMouse()
mouse.Button1Down:Connect(function()
	if clickTeleportEnabled and not teleportDebounce then
		teleportDebounce = true
		local target = mouse.Hit
		if target then
			local root = getRootPart()
			if root then
				root.CFrame = CFrame.new(target.Position + Vector3.new(0,3,0)) -- small Y offset
			end
		end
		task.wait(0.2)
		teleportDebounce = false
	end
end)
