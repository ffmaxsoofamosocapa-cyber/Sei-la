local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local fadeTime, tweenTime = 0.5, 0.5
local imageId = "rbxassetid://133329932870887"

local function createBillboard(offset, text)
	local character = player.Character or player.CharacterAdded:Wait()
	local hrp = character:WaitForChild("HumanoidRootPart")

	local bb = Instance.new("BillboardGui")
	bb.Adornee = hrp
	bb.Size = UDim2.fromScale(5, 5)
	bb.StudsOffset = offset
	bb.AlwaysOnTop = true
	bb.Parent = character

	local img = Instance.new("ImageLabel")
	img.Size = UDim2.fromScale(0.77, 0.77)
	img.Position = UDim2.fromScale(0.115, 0.115)
	img.Image = imageId
	img.BackgroundTransparency = 1
	img.Parent = bb

	local txt = Instance.new("TextLabel")
	txt.Size = UDim2.fromScale(0.59, 0.59)
	txt.Position = UDim2.fromScale(0.205, 0.205)
	txt.BackgroundTransparency = 1
	txt.TextScaled = true
	txt.Font = Enum.Font.SpecialElite -- MANGA-LIKE FONT
	txt.TextColor3 = Color3.new()
	txt.TextStrokeTransparency = 1
	txt.Text = text
	txt.Parent = img

	task.spawn(function()
		task.wait(3)
		TweenService:Create(bb, TweenInfo.new(tweenTime, Enum.EasingStyle.Quad), {
			StudsOffset = bb.StudsOffset + Vector3.new((offset.X > 0 and 1 or -1) * 16, 0, 0)
		}):Play()

		for i = 0, 1, 1 / (fadeTime * 60) do
			img.ImageTransparency = i
			txt.TextTransparency = i
			task.wait(1 / 60)
		end

		bb:Destroy()
	end)
end

local function triggerBillboards()
	createBillboard(Vector3.new(-5, 0, 0), "A Criação...")
	createBillboard(Vector3.new(5, 0, 0), "Perfeita.")
end

if player.Character then
	triggerBillboards()
else
	player.CharacterAdded:Once(triggerBillboards)
end
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local root = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

-- Play blue/red animation
local function playBlueRedAnimation()
	local anim = Instance.new("Animation")
	anim.AnimationId = "rbxassetid://48138189"
	local track = humanoid:LoadAnimation(anim)
	track:Play(0.5)
	track.TimePosition = 1
	track:AdjustSpeed(0)
	task.delay(4.3, function()
		track:Stop(0.5)
	end)
end

-- Play purple spawn animation
local function playPurpleAnimation()
	local anim = Instance.new("Animation")
	anim.AnimationId = "rbxassetid://193308129"
	local track = humanoid:LoadAnimation(anim)
	track:Play(0.5)
	track.TimePosition = 0.5
	track:AdjustSpeed(0)
	task.delay(3, function()
		track:Stop(0.5)
	end)
end

-- Fire effect
local function addFireEffect(part, color)
	local fire = Instance.new("Fire")
	fire.Color = color
	fire.SecondaryColor = color
	fire.Heat = 10
	fire.Size = part.Size.X * 2
	fire.Parent = part
end

-- Create glowing ball with fire
local function createBall(color, size, canCollide)
	local ball = Instance.new("Part")
	ball.Shape = Enum.PartType.Ball
	ball.Size = size
	ball.Anchored = false
	ball.CanCollide = canCollide
	ball.Material = Enum.Material.Neon
	ball.Color = color
	ball.Name = "PurpleBall"
	ball.Parent = workspace
	addFireEffect(ball, color)

	-- Remove gravity
	local antiGravity = Instance.new("BodyForce")
	antiGravity.Force = Vector3.new(0, ball:GetMass() * workspace.Gravity, 0)
	antiGravity.Parent = ball

	return ball
end

-- Create non-glowing block that tweens downward and disappears
local function createSideBlock(position)
	local block = Instance.new("Part")
	block.Size = Vector3.new(12, 12, 12)
	block.Anchored = true
	block.CanCollide = false
	block.Material = Enum.Material.SmoothPlastic

	local rayOrigin = position
	local rayDirection = Vector3.new(0, -100, 0)
	local raycastParams = RaycastParams.new()
	raycastParams.FilterDescendantsInstances = {}
	raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

	local result = workspace:Raycast(rayOrigin, rayDirection, raycastParams)
	if result and result.Instance and result.Instance:IsA("BasePart") then
		block.Color = result.Instance.Color
	else
		block.Color = Color3.fromRGB(200, 200, 200)
	end

	block.CFrame = CFrame.new(position) * CFrame.Angles(
		math.rad(math.random(0, 360)),
		math.rad(math.random(0, 360)),
		math.rad(math.random(0, 360))
	)

	block.Parent = workspace

	local goal = { Position = block.Position - Vector3.new(0, 7, 0) }
	local tween = TweenService:Create(block, TweenInfo.new(0.8, Enum.EasingStyle.Sine), goal)
	tween:Play()
	tween.Completed:Connect(function()
		block:Destroy()
	end)
end

-- Start red/blue animation
playBlueRedAnimation()

-- Red and blue balls
local redBall = createBall(Color3.fromRGB(255, 0, 0), Vector3.new(6, 6, 6), true)
local blueBall = createBall(Color3.fromRGB(255, 255, 255), Vector3.new(6, 6, 6), true)

-- Follow player
local followConnection = RunService.RenderStepped:Connect(function()
	if root and root.Parent then
		local back = -root.CFrame.LookVector * 5
		local left = -root.CFrame.RightVector * 10
		local right = root.CFrame.RightVector * 10
		redBall.Position = root.Position + back + left
		blueBall.Position = root.Position + back + right
	end
end)

-- Wait then merge
task.wait(3)
followConnection:Disconnect()

local mergePos = root.Position + root.CFrame.LookVector * 25
local tweenInfo = TweenInfo.new(1, Enum.EasingStyle.Sine)
TweenService:Create(redBall, tweenInfo, {Position = mergePos}):Play()
TweenService:Create(blueBall, tweenInfo, {Position = mergePos}):Play()

task.wait(1)
redBall:Destroy()
blueBall:Destroy()

-- Play purple animation
playPurpleAnimation()

-- Create purple ball
local purpleBall = createBall(Color3.fromRGB(0, 0, 0), Vector3.new(48, 48, 48), false)
purpleBall.Position = mergePos

-- Blackhole attraction (fixed with distance check)
local blockSpawnerConnection
local pullConnection

local function attractNearbyParts()
	if not purpleBall or not purpleBall.Parent then
		if pullConnection then
			pullConnection:Disconnect()
			pullConnection = nil
		end
		return
	end

	local maxDistance = 150
	local pullRadius = 100
	local pullForce = 400000

	local region = Region3.new(
		purpleBall.Position - Vector3.new(maxDistance, maxDistance, maxDistance),
		purpleBall.Position + Vector3.new(maxDistance, maxDistance, maxDistance)
	)

	local ignoreList = {character, purpleBall}
	for _, plr in pairs(Players:GetPlayers()) do
		if plr.Character then
			table.insert(ignoreList, plr.Character)
		end
	end

	local parts = workspace:FindPartsInRegion3WithIgnoreList(region, ignoreList, math.huge)

	for _, part in ipairs(parts) do
		if part:IsA("BasePart") and not part.Anchored then
			local distance = (part.Position - purpleBall.Position).Magnitude
			local velocityControl = part:FindFirstChild("BlackholePull")

			if distance <= pullRadius then
				if not velocityControl then
					velocityControl = Instance.new("BodyVelocity")
					velocityControl.Name = "BlackholePull"
					velocityControl.MaxForce = Vector3.new(pullForce, pullForce, pullForce)
					velocityControl.P = 4000000
					velocityControl.Parent = part
				end
				local direction = (purpleBall.Position - part.Position).Unit
				velocityControl.Velocity = direction * 300
			else
				if velocityControl then
					velocityControl:Destroy()
				end
			end
		end
	end
end

-- Start purple movement + effects
task.delay(1, function()
	local destination = purpleBall.Position + root.CFrame.LookVector * 500
	local moveTween = TweenService:Create(purpleBall, TweenInfo.new(8, Enum.EasingStyle.Linear), {Position = destination})
	moveTween:Play()

	blockSpawnerConnection = RunService.RenderStepped:Connect(function()
		if purpleBall and purpleBall.Parent then
			local pos = purpleBall.Position
			local rightVec = purpleBall.CFrame.RightVector
			createSideBlock(pos - rightVec * 8)
			createSideBlock(pos + rightVec * 8)
		end
	end)

	pullConnection = RunService.Heartbeat:Connect(attractNearbyParts)
end)

-- Cleanup
task.delay(7, function()
	if purpleBall and purpleBall.Parent then
		purpleBall:Destroy()
	end
	if blockSpawnerConnection then
		blockSpawnerConnection:Disconnect()
	end
	if pullConnection then
		pullConnection:Disconnect()
	end
end)

-- camera shake
local Intensity = 0.8
local ShakeLength = 3
local FadeLength = 1

-- don't change anything here unless you know
local rn = Random.new()
local int = Instance.new("NumberValue")
int.Value = Intensity
local b

b = game:GetService("RunService").Heartbeat:Connect(function()
	local char = game.Players.LocalPlayer.Character
	if char and char:FindFirstChild("Humanoid") then
		char.Humanoid.CameraOffset = Vector3.new(
			rn:NextNumber(-int.Value, int.Value),
			rn:NextNumber(-int.Value, int.Value),
			rn:NextNumber(-int.Value, int.Value)
		)
	end
end)

-- Run the shake and fade asynchronously
task.spawn(function()
	task.wait(ShakeLength)
	local p = game:GetService("TweenService"):Create(int, TweenInfo.new(FadeLength, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {Value = 0})
	p:Play()
	p.Completed:Wait()
	b:Disconnect()
end)
local Text = " Buraco negro."

-- don't change anything here unless you know
game:GetService("Chat"):Chat(game.Players.LocalPlayer.Character, Text)