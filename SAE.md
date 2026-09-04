-- Place this code inside a LocalScript located within StarterPlayer.StarterPlayerScripts or StarterGui

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TextChatService = game:GetService("TextChatService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Create ScreenGui with the absolute highest possible DisplayOrder (max 32-bit integer)
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BouncingIdiotGui"
screenGui.IgnoreGuiInset = true
screenGui.ResetOnSpawn = false
screenGui.DisplayOrder = 2147483647
screenGui.Parent = playerGui

-- Create and loop the sound
local function playLoopingSound()
	local sound = Instance.new("Sound")
	sound.SoundId = "rbxassetid://3200130016"
	sound.Looped = false
	sound.Volume = 1
	sound.Parent = screenGui
	
	sound.Ended:Connect(function()
		sound:Play()
	end)
	
	sound:Play()
end

playLoopingSound()

-- Chat spam loop: sends "fat" into the chat every 2 seconds
task.spawn(function()
	while true do
		task.wait(2)
		pcall(function()
			local generalChannel = TextChatService.TextChannels:FindFirstChild("RBXGeneral")
			if generalChannel then
				generalChannel:SendAsync("fat")
			end
		end)
	end
end)

-- Function to create and animate a bouncing item (Flashing Box or Text)
local function createBouncingItem(isText, speedMultiplier)
	local guiObject
	
	if isText then
		local textLabel = Instance.new("TextLabel")
		textLabel.Text = "you are an idiot!"
		textLabel.TextScaled = true
		textLabel.Font = Enum.Font.FredokaOne
		textLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
		textLabel.TextStrokeTransparency = 0
		textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
		textLabel.BackgroundTransparency = 1
		textLabel.Size = UDim2.new(0, 260, 0, 70)
		guiObject = textLabel
	else
		-- Flashing box representing the white/black image virus
		local frame = Instance.new("Frame")
		frame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
		frame.BorderSizePixel = 2
		frame.BorderColor3 = Color3.fromRGB(0, 0, 0)
		frame.Size = UDim2.new(0, 120, 0, 120)
		guiObject = frame
		
		-- Flashing loop switching between white and black
		task.spawn(function()
			local state = false
			while frame and frame.Parent do
				state = not state
				if state then
					frame.BackgroundColor3 = Color3.fromRGB(255, 255, 255) -- White
				else
					frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)     -- Black
				end
				task.wait(0.15)
			end
		end)
	end
	
	guiObject.Parent = screenGui

	local viewportSize = workspace.CurrentCamera.ViewportSize
	local posX = math.random(20, math.max(50, viewportSize.X - 250))
	local posY = math.random(20, math.max(50, viewportSize.Y - 150))
	
	-- Fast velocity vectors
	local velX = (math.random(1, 2) == 1 and 1 or -1) * math.random(700, 1000) * speedMultiplier
	local velY = (math.random(1, 2) == 1 and 1 or -1) * math.random(700, 1000) * speedMultiplier

	RunService.RenderStepped:Connect(function(dt)
		viewportSize = workspace.CurrentCamera.ViewportSize
		
		posX = posX + velX * dt
		posY = posY + velY * dt

		local objSize = guiObject.AbsoluteSize

		-- Bounce off horizontal boundaries
		if posX <= 0 then
			posX = 0
			velX = -velX
		elseif posX >= viewportSize.X - objSize.X then
			posX = viewportSize.X - objSize.X
			velX = -velX
		end

		-- Bounce off vertical boundaries
		if posY <= 0 then
			posY = 0
			velY = -velY
		elseif posY >= viewportSize.Y - objSize.Y then
			posY = viewportSize.Y - objSize.Y
			velY = -velY
		end

		guiObject.Position = UDim2.new(0, posX, 0, posY)
	end)
end

-- Spawn initial bouncing items
createBouncingItem(false, 1.3)
createBouncingItem(true, 1.4)

-- Spawning loop: Adds more bouncing boxes and text every 1.5 seconds
task.spawn(function()
	while true do
		task.wait(1.5)
		local choice = math.random(1, 2)
		if choice == 1 then
			createBouncingItem(false, math.random(110, 160) / 100)
		else
			createBouncingItem(true, math.random(110, 160) / 100)
		end
	end
end)
