--// STUART HUB - Aimbot + FOV + HP ESP
--// LocalScript | StarterPlayer > StarterPlayerScripts

-- SERVICES
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- =========================
-- CONFIG
-- =========================
local aimbotEnabled = false
local fovEnabled = false
local ignoreDead = false
local ignoreTeam = false -- 🔹 NOVO
local holdingRightClick = false
local fovRadius = 120
local lockedTarget = nil
local aimStrength = 0.15
local scriptDisabled = false

-- HP ESP
local hpEspEnabled = false
local healthBars = {}

-- =========================
-- UI
-- =========================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "StuartHUB"
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 320, 0, 510)
MainFrame.Position = UDim2.new(0.35, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0,12)

local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(0,255,255)

local Logo = Instance.new("ImageLabel", MainFrame)
Logo.Size = UDim2.new(0,40,0,40)
Logo.Position = UDim2.new(0,10,0,5)
Logo.BackgroundTransparency = 1
Logo.Image = "rbxassetid://13799217063"

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1,0,0,50)
Title.Text = "Stuart HUB"
Title.TextColor3 = Color3.fromRGB(0,255,255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 26
Title.BackgroundTransparency = 1

-- =========================
-- FUNÇÃO BOTÃO
-- =========================
local function createButton(text, posY)
	local btn = Instance.new("TextButton", MainFrame)
	btn.Size = UDim2.new(0.9,0,0,35)
	btn.Position = UDim2.new(0.05,0,0,posY)
	btn.Text = text
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 14
	btn.TextColor3 = Color3.new(1,1,1)
	btn.BackgroundColor3 = Color3.fromRGB(35,35,35)
	btn.BorderSizePixel = 0
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0,8)
	local s = Instance.new("UIStroke", btn)
	s.Thickness = 1
	s.Color = Color3.fromRGB(80,80,80)
	return btn
end

-- =========================
-- BOTÕES
-- =========================
local AimBtn       = createButton("AIMBOT [OFF]", 60)
local FovBtn       = createButton("FOV CHECK [OFF]", 105)
local PlusBtn      = createButton("AUMENTAR CÍRCULO", 150)
local MinusBtn     = createButton("DIMINUIR CÍRCULO", 195)
local DeadBtn      = createButton("IGNORAR MORTOS [OFF]", 240)
local TeamBtn      = createButton("IGNORAR TIME [OFF]", 285) -- 🔹 NOVO
local WeakAimBtn   = createButton("AIMBOT FRACO", 330)
local StrongAimBtn = createButton("AIMBOT FORTE", 375)
local HpEspBtn     = createButton("HP ESP [OFF]", 420)
local BypassBtn    = createButton("BYPASS", 465)
BypassBtn.BackgroundColor3 = Color3.fromRGB(120,0,0)

-- =========================
-- FOV CIRCLE
-- =========================
local FovCircle = Drawing.new("Circle")
FovCircle.Color = Color3.fromRGB(0,255,255)
FovCircle.Thickness = 1
FovCircle.NumSides = 100
FovCircle.Filled = false
FovCircle.Visible = false

-- =========================
-- FUNÇÕES AIMBOT
-- =========================
local function isAlive(player)
	local hum = player.Character and player.Character:FindFirstChild("Humanoid")
	return hum and hum.Health > 0
end

local function getClosestPlayer()
	if scriptDisabled then return nil end
	local closest, shortest = nil, math.huge

	for _,plr in pairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
			
			-- 🔹 IGNORA PLAYER DO MESMO TIME
			if ignoreTeam and plr.Team == LocalPlayer.Team then
				continue
			end

			if ignoreDead and not isAlive(plr) then
				continue
			end

			local pos, onScreen = Camera:WorldToViewportPoint(plr.Character.Head.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X,pos.Y) -
					Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude

				if fovEnabled and dist > fovRadius then
					continue
				end

				if dist < shortest then
					shortest = dist
					closest = plr.Character.Head
				end
			end
		end
	end
	return closest
end

-- =========================
-- HP ESP
-- =========================
local function createHealthBar(player)
	if player == LocalPlayer or not player.Character then return end

	local char = player.Character
	local hum = char:FindFirstChildOfClass("Humanoid")
	local root = char:FindFirstChild("HumanoidRootPart")
	if not hum or not root then return end

	local bb = Instance.new("BillboardGui")
	bb.Size = UDim2.new(0,6,0,50)
	bb.StudsOffset = Vector3.new(2.5,0,0)
	bb.AlwaysOnTop = true
	bb.Adornee = root
	bb.Parent = root

	local bg = Instance.new("Frame", bb)
	bg.Size = UDim2.new(1,0,1,0)
	bg.BackgroundColor3 = Color3.fromRGB(25,25,25)
	bg.BorderSizePixel = 0

	local bar = Instance.new("Frame", bg)
	bar.AnchorPoint = Vector2.new(0,1)
	bar.Position = UDim2.new(0,0,1,0)
	bar.Size = UDim2.new(1,0,1,0)
	bar.BorderSizePixel = 0

	local function update()
		local hp = hum.Health / hum.MaxHealth
		bar.Size = UDim2.new(1,0,hp,0)

		if hp > 0.6 then
			bar.BackgroundColor3 = Color3.fromRGB(0,255,0)
		elseif hp > 0.3 then
			bar.BackgroundColor3 = Color3.fromRGB(255,170,0)
		else
			bar.BackgroundColor3 = Color3.fromRGB(255,0,0)
		end
	end

	hum.HealthChanged:Connect(update)
	update()
	healthBars[player] = bb
end

local function removeHealthBars()
	for _,v in pairs(healthBars) do
		if v then v:Destroy() end
	end
	healthBars = {}
end

-- =========================
-- INPUT
-- =========================
UserInputService.InputBegan:Connect(function(input)
	if scriptDisabled then return end
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		holdingRightClick = true
		lockedTarget = getClosestPlayer()
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		holdingRightClick = false
		lockedTarget = nil
	end
end)

-- =========================
-- BOTÕES (FUNÇÕES)
-- =========================
AimBtn.MouseButton1Click:Connect(function()
	aimbotEnabled = not aimbotEnabled
	AimBtn.Text = "AIMBOT ["..(aimbotEnabled and "ON" or "OFF").."]"
end)

FovBtn.MouseButton1Click:Connect(function()
	fovEnabled = not fovEnabled
	FovBtn.Text = "FOV CHECK ["..(fovEnabled and "ON" or "OFF").."]"
	FovCircle.Visible = fovEnabled
end)

PlusBtn.MouseButton1Click:Connect(function()
	fovRadius += 10
end)

MinusBtn.MouseButton1Click:Connect(function()
	fovRadius = math.max(30, fovRadius - 10)
end)

DeadBtn.MouseButton1Click:Connect(function()
	ignoreDead = not ignoreDead
	DeadBtn.Text = "IGNORAR MORTOS ["..(ignoreDead and "ON" or "OFF").."]"
end)

TeamBtn.MouseButton1Click:Connect(function()
	ignoreTeam = not ignoreTeam
	TeamBtn.Text = "IGNORAR TIME ["..(ignoreTeam and "ON" or "OFF").."]"
end)

WeakAimBtn.MouseButton1Click:Connect(function()
	aimStrength = math.clamp(aimStrength - 0.05, 0.05, 1)
end)

StrongAimBtn.MouseButton1Click:Connect(function()
	aimStrength = math.clamp(aimStrength + 0.05, 0.05, 1)
end)

HpEspBtn.MouseButton1Click:Connect(function()
	hpEspEnabled = not hpEspEnabled
	HpEspBtn.Text = "HP ESP ["..(hpEspEnabled and "ON" or "OFF").."]"

	if hpEspEnabled then
		for _,p in pairs(Players:GetPlayers()) do
			createHealthBar(p)
		end
	else
		removeHealthBars()
	end
end)

BypassBtn.MouseButton1Click:Connect(function()
	scriptDisabled = true
	removeHealthBars()
	FovCircle.Visible = false
	ScreenGui.Enabled = false
end)

-- =========================
-- LOOP
-- =========================
RunService.RenderStepped:Connect(function()
	if scriptDisabled then return end

	FovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
	FovCircle.Radius = fovRadius

	if aimbotEnabled and holdingRightClick and lockedTarget then
		local cf = Camera.CFrame
		Camera.CFrame = cf:Lerp(
			CFrame.new(cf.Position, lockedTarget.Position),
			aimStrength
		)
	end
end)

-- =========================
-- RESPAWN
-- =========================
Players.PlayerAdded:Connect(function(p)
	p.CharacterAdded:Connect(function()
		task.wait(1)
		if hpEspEnabled then
			createHealthBar(p)
		end
	end)
end)
