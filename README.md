-- AIM LOCK + FAVORITE SYSTEM + AUTO SCROLL + ESP TARGET + FPS BOOSTER BY ARTEAM
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

---------------------------------------------------------
-- CONFIGURATION
---------------------------------------------------------
local AIM_PART = "Head"          -- Target tembakan: "Head" atau "HumanoidRootPart"
local FOV_RADIUS = 250           -- Radius pencarian target awal (Auto Mode)
local TOGGLE_KEY = Enum.KeyCode.E-- Tombol Keyboard Toggle Lock

local isStickyActive = false
local isAutoScrollOn = false     -- Toggle Auto Scroll Target Favorit
local isFpsBoosterOn = false
local lockedTargetPlayer = nil
local selectedTargetName = "AUTO (TERDEKAT)"
local favoritePlayers = {}       -- List Nama Favorit
local targetHighlight = nil      -- Instance Highlight ESP

---------------------------------------------------------
-- 1. UTILITY & ESP FUNCTIONS
---------------------------------------------------------
local function getClosestEnemy()
	local closestPlayer = nil
	local shortestDistance = FOV_RADIUS
	local viewportCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

	for _, player in ipairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character then
			local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
			local targetPart = player.Character:FindFirstChild(AIM_PART)

			if humanoid and humanoid.Health > 0 and targetPart then
				local screenPos, isOnScreen = Camera:WorldToViewportPoint(targetPart.Position)

				if isOnScreen then
					local distance = (Vector2.new(screenPos.X, screenPos.Y) - viewportCenter).Magnitude
					if distance < shortestDistance then
						shortestDistance = distance
						closestPlayer = player
					end
				end
			end
		end
	end

	return closestPlayer
end

-- ESP Target Highlight Manager
local function updateTargetESP(targetPlayer)
	if targetHighlight then
		targetHighlight:Destroy()
		targetHighlight = nil
	end

	if targetPlayer and targetPlayer.Character then
		targetHighlight = Instance.new("Highlight")
		targetHighlight.Name = "TargetLockESP"
		targetHighlight.FillColor = Color3.fromRGB(255, 30, 30)
		targetHighlight.FillTransparency = 0.5
		targetHighlight.OutlineColor = Color3.fromRGB(255, 255, 255)
		targetHighlight.OutlineTransparency = 0
		targetHighlight.Adornee = targetPlayer.Character
		targetHighlight.Parent = targetPlayer.Character
	end
end

-- Fungsi mencari pemain favorit berikutnya yang masih hidup
local function getNextAliveFavoritePlayer()
	for playerName, _ in pairs(favoritePlayers) do
		local plr = Players:FindFirstChild(playerName)
		if plr and plr ~= LocalPlayer and plr.Character then
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health > 0 then
				return plr
			end
		end
	end
	return nil
end

---------------------------------------------------------
-- 2. GUI CREATION (MODERN UI)
---------------------------------------------------------
if PlayerGui:FindFirstChild("StickyAimGui") then
	PlayerGui.StickyAimGui:Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "StickyAimGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = PlayerGui

-- Main Container Frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 270, 0, 370)
mainFrame.Position = UDim2.new(0.05, 0, 0.2, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 17, 23)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local frameCorner = Instance.new("UICorner")
frameCorner.CornerRadius = UDim.new(0, 10)
frameCorner.Parent = mainFrame

local frameStroke = Instance.new("UIStroke")
frameStroke.Color = Color3.fromRGB(0, 170, 255)
frameStroke.Thickness = 1.5
frameStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
frameStroke.Parent = mainFrame

-- Header
local headerFrame = Instance.new("Frame")
headerFrame.Size = UDim2.new(1, 0, 0, 32)
headerFrame.BackgroundColor3 = Color3.fromRGB(22, 25, 35)
headerFrame.Parent = mainFrame

local headerCorner = Instance.new("UICorner")
headerCorner.CornerRadius = UDim.new(0, 10)
headerCorner.Parent = headerFrame

local headerFix = Instance.new("Frame")
headerFix.Size = UDim2.new(1, 0, 0, 10)
headerFix.Position = UDim2.new(0, 0, 1, -10)
headerFix.BackgroundColor3 = Color3.fromRGB(22, 25, 35)
headerFix.BorderSizePixel = 0
headerFix.Parent = headerFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, -60, 1, 0)
titleLabel.Position = UDim2.new(0, 12, 0, 0)
titleLabel.Text = "ARTEAM AIM & FAVORITES"
titleLabel.TextColor3 = Color3.fromRGB(0, 210, 255)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 11
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.BackgroundTransparency = 1
titleLabel.Parent = headerFrame

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 22, 0, 22)
closeBtn.Position = UDim2.new(1, -27, 0, 5)
closeBtn.Text = "âœ•"
closeBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
closeBtn.BackgroundColor3 = Color3.fromRGB(35, 25, 30)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 11
closeBtn.Parent = headerFrame

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeBtn

local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 22, 0, 22)
minBtn.Position = UDim2.new(1, -54, 0, 5)
minBtn.Text = "â”€"
minBtn.TextColor3 = Color3.fromRGB(200, 200, 200)
minBtn.BackgroundColor3 = Color3.fromRGB(30, 35, 45)
minBtn.Font = Enum.Font.GothamBold
minBtn.TextSize = 10
minBtn.Parent = headerFrame

local minCorner = Instance.new("UICorner")
minCorner.CornerRadius = UDim.new(0, 6)
minCorner.Parent = minBtn

local openButton = Instance.new("TextButton")
openButton.Name = "OpenButton"
openButton.Size = UDim2.new(0, 110, 0, 32)
openButton.Position = UDim2.new(0.05, 0, 0.2, 0)
openButton.Text = "âš¡ OPEN MENU"
openButton.TextColor3 = Color3.fromRGB(255, 255, 255)
openButton.BackgroundColor3 = Color3.fromRGB(0, 150, 230)
openButton.Font = Enum.Font.GothamBold
openButton.TextSize = 11
openButton.Visible = false
openButton.Active = true
openButton.Draggable = true
openButton.Parent = screenGui

local openCorner = Instance.new("UICorner")
openCorner.CornerRadius = UDim.new(0, 8)
openCorner.Parent = openButton

---------------------------------------------------------
-- UI CONTENT ELEMENTS
---------------------------------------------------------
local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, -20, 0, 20)
statusLabel.Position = UDim2.new(0, 10, 0, 36)
statusLabel.Text = "Status Target: Standby"
statusLabel.TextColor3 = Color3.fromRGB(150, 160, 180)
statusLabel.Font = Enum.Font.GothamMedium
statusLabel.TextSize = 10
statusLabel.BackgroundTransparency = 1
statusLabel.Parent = mainFrame

-- Target Dropdown Button
local targetDropBtn = Instance.new("TextButton")
targetDropBtn.Name = "TargetDropBtn"
targetDropBtn.Size = UDim2.new(1, -20, 0, 28)
targetDropBtn.Position = UDim2.new(0, 10, 0, 58)
targetDropBtn.Text = "Pilih Target: AUTO (TERDEKAT) â–¾"
targetDropBtn.TextColor3 = Color3.fromRGB(240, 240, 240)
targetDropBtn.BackgroundColor3 = Color3.fromRGB(25, 30, 42)
targetDropBtn.Font = Enum.Font.GothamBold
targetDropBtn.TextSize = 10
targetDropBtn.Parent = mainFrame

local dropCorner = Instance.new("UICorner")
dropCorner.CornerRadius = UDim.new(0, 6)
dropCorner.Parent = targetDropBtn

-- Dropdown Frame
local targetListFrame = Instance.new("ScrollingFrame")
targetListFrame.Name = "TargetListFrame"
targetListFrame.Size = UDim2.new(1, -20, 0, 110)
targetListFrame.Position = UDim2.new(0, 10, 0, 88)
targetListFrame.BackgroundColor3 = Color3.fromRGB(20, 24, 34)
targetListFrame.BorderSizePixel = 0
targetListFrame.ScrollBarThickness = 3
targetListFrame.ScrollBarImageColor3 = Color3.fromRGB(0, 170, 255)
targetListFrame.Visible = false
targetListFrame.ZIndex = 20
targetListFrame.Parent = mainFrame

local listCorner = Instance.new("UICorner")
listCorner.CornerRadius = UDim.new(0, 6)
listCorner.Parent = targetListFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Parent = targetListFrame
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Padding = UDim.new(0, 2)

-- Section Title Favorit
local favTitleLabel = Instance.new("TextLabel")
favTitleLabel.Size = UDim2.new(1, -20, 0, 16)
favTitleLabel.Position = UDim2.new(0, 10, 0, 90)
favTitleLabel.Text = "â˜… DAFTAR TARGET FAVORIT"
favTitleLabel.TextColor3 = Color3.fromRGB(255, 200, 50)
favTitleLabel.Font = Enum.Font.GothamBold
favTitleLabel.TextSize = 10
favTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
favTitleLabel.BackgroundTransparency = 1
favTitleLabel.Parent = mainFrame

-- Favorite Frame
local favListFrame = Instance.new("ScrollingFrame")
favListFrame.Name = "FavListFrame"
favListFrame.Size = UDim2.new(1, -20, 0, 70)
favListFrame.Position = UDim2.new(0, 10, 0, 108)
favListFrame.BackgroundColor3 = Color3.fromRGB(22, 26, 36)
favListFrame.BorderSizePixel = 0
favListFrame.ScrollBarThickness = 3
favListFrame.ScrollBarImageColor3 = Color3.fromRGB(255, 200, 50)
favListFrame.Parent = mainFrame

local favListCorner = Instance.new("UICorner")
favListCorner.CornerRadius = UDim.new(0, 6)
favListCorner.Parent = favListFrame

local favListLayout = Instance.new("UIListLayout")
favListLayout.Parent = favListFrame
favListLayout.SortOrder = Enum.SortOrder.LayoutOrder
favListLayout.Padding = UDim.new(0, 2)

-- Auto Scroll Button (Baru)
local autoScrollBtn = Instance.new("TextButton")
autoScrollBtn.Name = "AutoScrollBtn"
autoScrollBtn.Size = UDim2.new(1, -20, 0, 30)
autoScrollBtn.Position = UDim2.new(0, 10, 0, 185)
autoScrollBtn.Text = "AUTO SCROLL TARGET: OFF"
autoScrollBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
autoScrollBtn.BackgroundColor3 = Color3.fromRGB(40, 48, 65)
autoScrollBtn.Font = Enum.Font.GothamBold
autoScrollBtn.TextSize = 10
autoScrollBtn.Parent = mainFrame

local autoScrollCorner = Instance.new("UICorner")
autoScrollCorner.CornerRadius = UDim.new(0, 6)
autoScrollCorner.Parent = autoScrollBtn

-- Toggle Lock Button
local stickyBtn = Instance.new("TextButton")
stickyBtn.Name = "StickyBtn"
stickyBtn.Size = UDim2.new(1, -20, 0, 30)
stickyBtn.Position = UDim2.new(0, 10, 0, 222)
stickyBtn.Text = "LOCK AIM: OFF [E]"
stickyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stickyBtn.BackgroundColor3 = Color3.fromRGB(180, 45, 55)
stickyBtn.Font = Enum.Font.GothamBold
stickyBtn.TextSize = 10
stickyBtn.Parent = mainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 6)
btnCorner.Parent = stickyBtn

-- FPS Booster Button
local fpsBtn = Instance.new("TextButton")
fpsBtn.Name = "FpsBtn"
fpsBtn.Size = UDim2.new(1, -20, 0, 30)
fpsBtn.Position = UDim2.new(0, 10, 0, 259)
fpsBtn.Text = "FPS BOOSTER: OFF"
fpsBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
fpsBtn.BackgroundColor3 = Color3.fromRGB(180, 45, 55)
fpsBtn.Font = Enum.Font.GothamBold
fpsBtn.TextSize = 10
fpsBtn.Parent = mainFrame

local fpsCorner = Instance.new("UICorner")
fpsCorner.CornerRadius = UDim.new(0, 6)
fpsCorner.Parent = fpsBtn

-- Watermark
local watermark = Instance.new("TextLabel")
watermark.Size = UDim2.new(1, 0, 0, 16)
watermark.Position = UDim2.new(0, 0, 0, 348)
watermark.Text = "ARTEAM | TikTok: ihateyouforeverandeverrr"
watermark.TextColor3 = Color3.fromRGB(90, 100, 120)
watermark.Font = Enum.Font.GothamMedium
watermark.TextSize = 8
watermark.BackgroundTransparency = 1
watermark.Parent = mainFrame

---------------------------------------------------------
-- 3. FAVORITE SYSTEM LOGIC
---------------------------------------------------------
local function updateFavoriteListUI()
	for _, child in ipairs(favListFrame:GetChildren()) do
		if child:IsA("Frame") then
			child:Destroy()
		end
	end

	local count = 0
	for playerName, displayName in pairs(favoritePlayers) do
		count = count + 1

		local itemFrame = Instance.new("Frame")
		itemFrame.Size = UDim2.new(1, 0, 0, 22)
		itemFrame.BackgroundTransparency = 1
		itemFrame.Parent = favListFrame

		local selectBtn = Instance.new("TextButton")
		selectBtn.Size = UDim2.new(1, -26, 1, 0)
		selectBtn.Position = UDim2.new(0, 0, 0, 0)
		selectBtn.Text = "  â˜… " .. displayName
		selectBtn.TextColor3 = Color3.fromRGB(255, 215, 80)
		selectBtn.BackgroundColor3 = Color3.fromRGB(30, 36, 50)
		selectBtn.Font = Enum.Font.GothamBold
		selectBtn.TextSize = 9
		selectBtn.TextXAlignment = Enum.TextXAlignment.Left
		selectBtn.Parent = itemFrame

		local sCorner = Instance.new("UICorner")
		sCorner.CornerRadius = UDim.new(0, 4)
		sCorner.Parent = selectBtn

		selectBtn.MouseButton1Click:Connect(function()
			selectedTargetName = playerName
			targetDropBtn.Text = "Target: " .. displayName .. " â–¾"
			
			if isStickyActive then
				lockedTargetPlayer = Players:FindFirstChild(playerName)
				updateTargetESP(lockedTargetPlayer)
				if lockedTargetPlayer and lockedTargetPlayer.Character then
					statusLabel.Text = "Melekat ke: " .. lockedTargetPlayer.DisplayName
					statusLabel.TextColor3 = Color3.fromRGB(0, 230, 120)
				end
			end
		end)

		-- Tombol Hapus Khusus Panel Favorit
		local removeBtn = Instance.new("TextButton")
		removeBtn.Size = UDim2.new(0, 22, 1, 0)
		removeBtn.Position = UDim2.new(1, -22, 0, 0)
		removeBtn.Text = "âœ•"
		removeBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
		removeBtn.BackgroundColor3 = Color3.fromRGB(45, 25, 30)
		removeBtn.Font = Enum.Font.GothamBold
		removeBtn.TextSize = 9
		removeBtn.Parent = itemFrame

		local rCorner = Instance.new("UICorner")
		rCorner.CornerRadius = UDim.new(0, 4)
		rCorner.Parent = removeBtn

		removeBtn.MouseButton1Click:Connect(function()
			favoritePlayers[playerName] = nil
			updateFavoriteListUI()
		end)
	end

	favListFrame.CanvasSize = UDim2.new(0, 0, 0, count * 24)
end

-- Hapus Favorit jika Leave
Players.PlayerRemoving:Connect(function(playerLeft)
	if favoritePlayers[playerLeft.Name] then
		favoritePlayers[playerLeft.Name] = nil
		updateFavoriteListUI()
	end
	
	if lockedTargetPlayer == playerLeft then
		lockedTargetPlayer = nil
		updateTargetESP(nil)
		statusLabel.Text = "Target keluar dari game!"
		statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
	end
end)

---------------------------------------------------------
-- 4. DROPDOWN LIST LOGIC (â­ ONLY)
---------------------------------------------------------
local function updateTargetList()
	for _, child in ipairs(targetListFrame:GetChildren()) do
		if child:IsA("Frame") or child:IsA("TextButton") then
			child:Destroy()
		end
	end

	local players = Players:GetPlayers()
	targetListFrame.CanvasSize = UDim2.new(0, 0, 0, (#players + 1) * 24)

	local autoBtn = Instance.new("TextButton")
	autoBtn.Size = UDim2.new(1, 0, 0, 22)
	autoBtn.Text = " AUTO (TERDEKAT)"
	autoBtn.TextColor3 = Color3.fromRGB(0, 230, 120)
	autoBtn.BackgroundColor3 = Color3.fromRGB(28, 34, 48)
	autoBtn.Font = Enum.Font.GothamBold
	autoBtn.TextSize = 9
	autoBtn.TextXAlignment = Enum.TextXAlignment.Left
	autoBtn.ZIndex = 21
	autoBtn.Parent = targetListFrame

	autoBtn.MouseButton1Click:Connect(function()
		selectedTargetName = "AUTO (TERDEKAT)"
		targetDropBtn.Text = "Pilih Target: AUTO â–¾"
		targetListFrame.Visible = false
	end)

	for _, p in ipairs(players) do
		if p ~= LocalPlayer then
			local itemFrame = Instance.new("Frame")
			itemFrame.Size = UDim2.new(1, 0, 0, 22)
			itemFrame.BackgroundTransparency = 1
			itemFrame.ZIndex = 21
			itemFrame.Parent = targetListFrame

			local pBtn = Instance.new("TextButton")
			pBtn.Size = UDim2.new(1, -26, 1, 0)
			pBtn.Text = " " .. p.DisplayName .. " (@" .. p.Name .. ")"
			pBtn.TextColor3 = Color3.fromRGB(210, 215, 225)
			pBtn.BackgroundColor3 = Color3.fromRGB(28, 34, 48)
			pBtn.Font = Enum.Font.GothamMedium
			pBtn.TextSize = 9
			pBtn.TextXAlignment = Enum.TextXAlignment.Left
			pBtn.ZIndex = 21
			pBtn.Parent = itemFrame

			pBtn.MouseButton1Click:Connect(function()
				selectedTargetName = p.Name
				targetDropBtn.Text = "Target: " .. p.DisplayName .. " â–¾"
				targetListFrame.Visible = false
			end)

			local starBtn = Instance.new("TextButton")
			starBtn.Size = UDim2.new(0, 22, 1, 0)
			starBtn.Position = UDim2.new(1, -22, 0, 0)
			starBtn.Text = "â­"
			starBtn.BackgroundColor3 = Color3.fromRGB(40, 48, 65)
			starBtn.Font = Enum.Font.GothamBold
			starBtn.TextSize = 9
			starBtn.ZIndex = 22
			starBtn.Parent = itemFrame

			local sCorner = Instance.new("UICorner")
			sCorner.CornerRadius = UDim.new(0, 4)
			sCorner.Parent = starBtn

			starBtn.MouseButton1Click:Connect(function()
				favoritePlayers[p.Name] = p.DisplayName
				updateFavoriteListUI()
			end)
		end
	end
end

targetDropBtn.MouseButton1Click:Connect(function()
	updateTargetList()
	targetListFrame.Visible = not targetListFrame.Visible
end)

---------------------------------------------------------
-- 5. AUTO SCROLL & FPS BOOSTER LOGIC
---------------------------------------------------------
autoScrollBtn.MouseButton1Click:Connect(function()
	isAutoScrollOn = not isAutoScrollOn
	if isAutoScrollOn then
		autoScrollBtn.Text = "AUTO SCROLL TARGET: ON"
		autoScrollBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 90)
	else
		autoScrollBtn.Text = "AUTO SCROLL TARGET: OFF"
		autoScrollBtn.BackgroundColor3 = Color3.fromRGB(40, 48, 65)
	end
end)

local function toggleFpsBooster()
	isFpsBoosterOn = not isFpsBoosterOn

	if isFpsBoosterOn then
		fpsBtn.Text = "FPS BOOSTER: ON"
		fpsBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 90)

		pcall(function()
			Lighting.GlobalShadows = false
			Lighting.FogEnd = 9e9
			Lighting.ShadowSoftness = 0
			
			for _, v in ipairs(workspace:GetDescendants()) do
				if v:IsA("BasePart") then
					v.Material = Enum.Material.SmoothPlastic
					v.CastShadow = false
				elseif v:IsA("Decal") or v:IsA("Texture") then
					v.Transparency = 1
				elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
					v.Enabled = false
				elseif v:IsA("PostEffect") then
					v.Enabled = false
				end
			end
		end)
	else
		fpsBtn.Text = "FPS BOOSTER: OFF"
		fpsBtn.BackgroundColor3 = Color3.fromRGB(180, 45, 55)

		pcall(function()
			Lighting.GlobalShadows = true
		end)
	end
end

fpsBtn.MouseButton1Click:Connect(toggleFpsBooster)

---------------------------------------------------------
-- 6. AIM LOCK & TOGGLE LOGIC
---------------------------------------------------------
local function toggleStickyMode()
	isStickyActive = not isStickyActive

	if isStickyActive then
		if selectedTargetName == "AUTO (TERDEKAT)" then
			lockedTargetPlayer = getClosestEnemy()
		else
			lockedTargetPlayer = Players:FindFirstChild(selectedTargetName)
		end

		if lockedTargetPlayer and lockedTargetPlayer.Character then
			stickyBtn.Text = "LOCK AIM: ON (LOCKED)"
			stickyBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 90)
			frameStroke.Color = Color3.fromRGB(0, 230, 120)
			statusLabel.Text = "Melekat ke: " .. lockedTargetPlayer.DisplayName
			statusLabel.TextColor3 = Color3.fromRGB(0, 230, 120)
			updateTargetESP(lockedTargetPlayer)
		else
			isStickyActive = false
			lockedTargetPlayer = nil
			updateTargetESP(nil)
			stickyBtn.Text = "LOCK AIM: OFF [E]"
			stickyBtn.BackgroundColor3 = Color3.fromRGB(180, 45, 55)
			frameStroke.Color = Color3.fromRGB(0, 170, 255)
			statusLabel.Text = "Target tidak ditemukan!"
			statusLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
		end
	else
		lockedTargetPlayer = nil
		updateTargetESP(nil)
		stickyBtn.Text = "LOCK AIM: OFF [E]"
		stickyBtn.BackgroundColor3 = Color3.fromRGB(180, 45, 55)
		frameStroke.Color = Color3.fromRGB(0, 170, 255)
		statusLabel.Text = "Status Target: Standby"
		statusLabel.TextColor3 = Color3.fromRGB(150, 160, 180)
	end
end

stickyBtn.MouseButton1Click:Connect(toggleStickyMode)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == TOGGLE_KEY then
		toggleStickyMode()
	end
end)

minBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = false
	openButton.Visible = true
	targetListFrame.Visible = false
end)

openButton.MouseButton1Click:Connect(function()
	mainFrame.Visible = true
	openButton.Visible = false
end)

closeBtn.MouseButton1Click:Connect(function()
	updateTargetESP(nil)
	screenGui:Destroy()
end)

---------------------------------------------------------
-- 7. RENDER STEPPED (AIM LOCK + AUTO SCROLL SWITCHING)
---------------------------------------------------------
RunService.RenderStepped:Connect(function()
	if isStickyActive then
		local isValidTarget = false

		if lockedTargetPlayer and lockedTargetPlayer.Character then
			local char = lockedTargetPlayer.Character
			local targetPart = char:FindFirstChild(AIM_PART)
			local humanoid = char:FindFirstChildOfClass("Humanoid")

			if humanoid and humanoid.Health > 0 and targetPart then
				isValidTarget = true
				Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPart.Position)
			end
		end

		-- Jika Target Mati / Tidak Valid:
		if not isValidTarget then
			if isAutoScrollOn then
				-- Cari pemain favorit lain yang masih hidup
				local nextTarget = getNextAliveFavoritePlayer()
				if nextTarget then
					lockedTargetPlayer = nextTarget
					selectedTargetName = nextTarget.Name
					targetDropBtn.Text = "Target: " .. nextTarget.DisplayName .. " â–¾"
					statusLabel.Text = "Auto Switch: " .. nextTarget.DisplayName
					statusLabel.TextColor3 = Color3.fromRGB(0, 230, 120)
					updateTargetESP(nextTarget)
				else
					toggleStickyMode() -- Matikan lock jika tidak ada favorit lain yang hidup
				end
			else
				toggleStickyMode() -- Matikan lock biasa jika Auto Scroll OFF
			end
		end
	end
end)
