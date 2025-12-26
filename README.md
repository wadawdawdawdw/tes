--// STUART HUB - Aimbot + FOV
--// UI MELHORADA + IMAGEM NO CANTO + BYPASS
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
local holdingRightClick = false
local fovRadius = 120
local lockedTarget = nil
local aimStrength = 0.15
local scriptDisabled = false -- para o BYPASS

-- =========================
-- UI
-- =========================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "StuartHUB"
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 320, 0, 430)
MainFrame.Position = UDim2.new(0.35, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0,12)

-- BORDA NEON
local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(0,255,255)

-- LOGO
local Logo = Instance.new("ImageLabel", MainFrame)
Logo.Size = UDim2.new(0,40,0,40)
Logo.Position = UDim2.new(0,10,0,5)
Logo.BackgroundTransparency = 1
Logo.Image = "rbxassetid://13799217063"

-- TÍTULO
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

-- BOTÕES
local AimBtn       = createButton("AIMBOT [OFF]", 60)
local FovBtn       = createButton("FOV CHECK [OFF]", 105)
local PlusBtn      = createButton("AUMENTAR CÍRCULO", 150)
local MinusBtn     = createButton("DIMINUIR CÍRCULO", 195)
local DeadBtn      = createButton("IGNORAR MORTOS [OFF]", 240)
local WeakAimBtn   = createButton("AIMBOT FRACO", 285)
local StrongAimBtn = createButton("AIMBOT FORTE", 330)
local BypassBtn    = createButton("BYPASS", 375)
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
			if ignoreDead and not isAlive(plr) then continue end
			local pos, onScreen = Camera:WorldToViewportPoint(plr.Character.Head.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X,pos.Y) -
					Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude
				if fovEnabled and dist > fovRadius then continue end
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
	if scriptDisabled then return end
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		holdingRightClick = false
		lockedTarget = nil
	end
end)

-- =========================
-- BOTÕES
-- =========================
AimBtn.MouseButton1Click:Connect(function()
	if scriptDisabled then return end
	aimbotEnabled = not aimbotEnabled
	AimBtn.Text = "AIMBOT ["..(aimbotEnabled and "ON" or "OFF").."]"
end)

FovBtn.MouseButton1Click:Connect(function()
	if scriptDisabled then return end
	fovEnabled = not fovEnabled
	FovBtn.Text = "FOV CHECK ["..(fovEnabled and "ON" or "OFF").."]"
	FovCircle.Visible = fovEnabled
end)

PlusBtn.MouseButton1Click:Connect(function()
	if scriptDisabled then return end
	fovRadius += 10
end)

MinusBtn.MouseButton1Click:Connect(function()
	if scriptDisabled then return end
	fovRadius = math.max(30, fovRadius - 10)
end)

DeadBtn.MouseButton1Click:Connect(function()
	if scriptDisabled then return end
	ignoreDead = not ignoreDead
	DeadBtn.Text = "IGNORAR MORTOS ["..(ignoreDead and "ON" or "OFF").."]"
end)

WeakAimBtn.MouseButton1Click:Connect(function()
	if scriptDisabled then return end
	aimStrength = math.clamp(aimStrength - 0.05, 0.05, 1)
end)

StrongAimBtn.MouseButton1Click:Connect(function()
	if scriptDisabled then return end
	aimStrength = math.clamp(aimStrength + 0.05, 0.05, 1)
end)

BypassBtn.MouseButton1Click:Connect(function()
	scriptDisabled = true
	aimbotEnabled = false
	fovEnabled = false
	holdingRightClick = false
	lockedTarget = nil
	FovCircle.Visible = false
	ScreenGui.Enabled = false
end)

-- =========================
-- LOOP
-- =========================
RunService.RenderStepped:Connect(function()
	if scriptDisabled then return end

	FovCircle.Position = Vector2.new(
		Camera.ViewportSize.X / 2,
		Camera.ViewportSize.Y / 2
	)
	FovCircle.Radius = fovRadius

	if aimbotEnabled and holdingRightClick and lockedTarget then
		local cf = Camera.CFrame
		Camera.CFrame = cf:Lerp(
			CFrame.new(cf.Position, lockedTarget.Position),
			aimStrength
		)
	end
end)
