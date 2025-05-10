-- script do sayz
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--
local isAiming = true
local currentTarget = nil

-- 
local highlights = {}

-- 

local function isOnScreen(part)
	local _, onScreen = Camera:WorldToViewportPoint(part.Position)
	return onScreen
end

local function isAlive(character)
	local humanoid = character:FindFirstChildWhichIsA("Humanoid")
	return humanoid and humanoid.Health > 0
end

local function getClosestValidPlayer()
	local myCharacter = LocalPlayer.Character
	if not myCharacter or not myCharacter:FindFirstChild("HumanoidRootPart") then return nil end

	local myPos = myCharacter.HumanoidRootPart.Position
	local closestDist = math.huge
	local closestChar = nil

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local char = player.Character
			local part = char.HumanoidRootPart
			local dist = (myPos - part.Position).Magnitude

			if dist < closestDist and isAlive(char) and isOnScreen(part) then
				closestDist = dist
				closestChar = char
			end
		end
	end

	return closestChar
end


RunService.RenderStepped:Connect(function()
	if not isAiming then return end

	if not currentTarget or not isAlive(currentTarget) or not isOnScreen(currentTarget.HumanoidRootPart) then
		currentTarget = getClosestValidPlayer()
	end

	if currentTarget and currentTarget:FindFirstChild("HumanoidRootPart") then
		local targetPos = currentTarget.HumanoidRootPart.Position
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPos)
	end
end)

-- Alterna a mira com tecla E
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if not gameProcessed and input.KeyCode == Enum.KeyCode.E then
		isAiming = not isAiming
		if not isAiming then
			currentTarget = nil
		end
	end
end)


local function applyHighlightToCharacter(character, player)
	if highlights[player] then
		highlights[player]:Destroy()
		highlights[player] = nil
	end

	local highlight = Instance.new("Highlight")
	highlight.Name = "ESPHighlight"
	highlight.FillColor = Color3.fromRGB(255, 0, 0)
	highlight.OutlineColor = Color3.new(1, 1, 1)
	highlight.FillTransparency = 0.3
	highlight.OutlineTransparency = 0
	highlight.Adornee = character
	highlight.Parent = character

	highlights[player] = highlight
end

local function setupPlayer(player)
	if player == LocalPlayer then return end

	player.CharacterAdded:Connect(function(character)
		local hrp = character:WaitForChild("HumanoidRootPart", 5)
		if hrp then
			applyHighlightToCharacter(character, player)
		end
	end)

	if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
		applyHighlightToCharacter(player.Character, player)
	end
end

-- Aplica a todos os jogadores
for _, player in ipairs(Players:GetPlayers()) do
	setupPlayer(player)
end


Players.PlayerAdded:Connect(setupPlayer)


Players.PlayerRemoving:Connect(function(player)
	if highlights[player] then
		highlights[player]:Destroy()
		highlights[player] = nil
	end
end)

