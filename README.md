--// STUART HUB - MODIFICADO (LAYOUT HORIZONTAL + TECLA K)
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
local ignoreTeam = false 
local holdingRightClick = false
local fovRadius = 120
local lockedTarget = nil
local aimStrength = 0.15
local scriptDisabled = false
local uiVisible = true -- Controle de visibilidade

-- HP ESP
local hpEspEnabled = false
local healthBars = {}

-- =========================
-- UI PRINCIPAL (HORIZONTAL)
-- =========================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "StuartHUB_Mod"
ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 600, 0, 250) -- Retângulo Horizontal
MainFrame.Position = UDim2.new(0.5, -300, 0.4, -125)
MainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0,12)

local Stroke = Instance.new("UIStroke", MainFrame)
Stroke.Thickness = 2
Stroke.Color = Color3.fromRGB(0,255,255)

-- LOGO NO CANTO SUPERIOR DIREITO
local Logo = Instance.new("ImageLabel", MainFrame)
Logo.Size = UDim2.new(0,50,0,50)
Logo.Position = UDim2.new(1, -60, 0, 10) -- Canto superior direito
Logo.BackgroundTransparency = 1
Logo.Image = "rbxassetid://13799217063"

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(0, 200, 0, 40)
Title.Position = UDim2.new(0, 15, 0, 5)
Title.Text = "Stuart HUB"
Title.TextColor3 = Color3.fromRGB(0,255,255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 22
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1

-- CONTAINER PARA BOTÕES (LADO ESQUERDO)
local ButtonContainer = Instance.new("Frame", MainFrame)
ButtonContainer.Size = UDim2.new(0, 450, 0, 180)
ButtonContainer.Position = UDim2.new(0, 15, 0, 50)
ButtonContainer.BackgroundTransparency = 1

local GridLayout = Instance.new("UIGridLayout", ButtonContainer)
GridLayout.CellSize = UDim2.new(0, 215, 0, 32)
GridLayout.CellPadding = UDim2.new(0, 10, 0, 8)
GridLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- =========================
-- FUNÇÃO BOTÃO
-- =========================
local function createButton(text)
	local btn = Instance.new("TextButton", ButtonContainer)
	btn.Size = UDim2.new(0,0,0,0) -- Controlado pelo Grid
	btn.Text = text
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 13
	btn.TextColor3 = Color3.new(1,1,1)
	btn.BackgroundColor3 = Color3.fromRGB(35,35,35)
	btn.BorderSizePixel = 0
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0,6)
	local s = Instance.new("UIStroke", btn)
	s.Thickness = 1
	s.Color = Color3.fromRGB(80,80,80)
	return btn
end

-- =========================
-- BOTÕES
-- =========================
local AimBtn       = createButton("AIMBOT [OFF]")
local FovBtn       = createButton("FOV CHECK [OFF]")
local PlusBtn      = createButton("AUMENTAR CÍRCULO")
local MinusBtn     = createButton("DIMINUIR CÍRCULO")
local DeadBtn      = createButton("IGNORAR MORTOS [OFF]")
local TeamBtn      = createButton("IGNORAR TIME [OFF]")
local WeakAimBtn   = createButton("AIMBOT FRACO")
local StrongAimBtn = createButton("AIMBOT FORTE")
local HpEspBtn     = createButton("HP ESP [OFF]")
local BypassBtn    = createButton("FECHAR SCRIPT")
BypassBtn.BackgroundColor3 = Color3.fromRGB(120,0,0)

-- =========================
-- FOV CIRCLE
-- =========================
local FovCircle = Drawing.new("Circle")
FovCircle.Color = Color3.fromRGB(0,255,255)
FovCircle.Thickness = 1
FovCircle.NumSides = 100
FovCircle.Visible = false

-- =========================
-- LÓGICA DE VISIBILIDADE (TECLA K)
-- =========================
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.KeyCode == Enum.KeyCode.K then
		uiVisible = not uiVisible
		MainFrame.Visible = uiVisible
	end
end)

-- =========================
-- FUNÇÕES AIMBOT & ESP (MANTIDAS)
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
			if ignoreTeam and plr.Team == LocalPlayer.Team then continue end
			if ignoreDead and not isAlive(plr) then continue end
			local pos, onScreen = Camera:WorldToViewportPoint(plr.Character.Head.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X,pos.Y) - Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude
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

local function createHealthBar(player)
	if player == LocalPlayer or not player.Character then return end
	local char = player.Character
	local hum = char:FindFirstChildOfClass("Humanoid")
	local root = char:FindFirstChild("HumanoidRootPart")
	if not hum or not root then return end
	local bb = Instance.new("BillboardGui", root)
	bb.Size = UDim2.new(0,6,0,50)
	bb.StudsOffset = Vector3.new(2.5,0,0)
	bb.AlwaysOnTop = true
	bb.Adornee = root
	local bg = Instance.new("Frame", bb)
	bg.Size = UDim2.new(1,0,1,0)
	bg.BackgroundColor3 = Color3.fromRGB(25,25,25)
	local bar = Instance.new("Frame", bg)
	bar.AnchorPoint = Vector2.new(0,1)
	bar.Position = UDim2.new(0,0,1,0)
	bar.Size = UDim2.new(1,0,1,0)
	local function update()
		local hp = hum.Health / hum.MaxHealth
		bar.Size = UDim2.new(1,0,hp,0)
		bar.BackgroundColor3 = hp > 0.6 and Color3.fromRGB(0,255,0) or (hp > 0.3 and Color3.fromRGB(255,170,0) or Color3.fromRGB(255,0,0))
	end
	hum.HealthChanged:Connect(update)
	update()
	healthBars[player] = bb
end

local function removeHealthBars()
	for _,v in pairs(healthBars) do if v then v:Destroy() end end
	healthBars = {}
end

-- =========================
-- EVENTOS DE INPUT E BOTÕES
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

AimBtn.MouseButton1Click:Connect(function()
	aimbotEnabled = not aimbotEnabled
	AimBtn.Text = "AIMBOT ["..(aimbotEnabled and "ON" or "OFF").."]"
end)

FovBtn.MouseButton1Click:Connect(function()
	fovEnabled = not fovEnabled
	FovBtn.Text = "FOV CHECK ["..(fovEnabled and "ON" or "OFF").."]"
	FovCircle.Visible = fovEnabled
end)

PlusBtn.MouseButton1Click:Connect(function() fovRadius += 15 end)
MinusBtn.MouseButton1Click:Connect(function() fovRadius = math.max(30, fovRadius - 15) end)

DeadBtn.MouseButton1Click:Connect(function()
	ignoreDead = not ignoreDead
	DeadBtn.Text = "IGNORAR MORTOS ["..(ignoreDead and "ON" or "OFF").."]"
end)

TeamBtn.MouseButton1Click:Connect(function()
	ignoreTeam = not ignoreTeam
	TeamBtn.Text = "IGNORAR TIME ["..(ignoreTeam and "ON" or "OFF").."]"
end)

WeakAimBtn.MouseButton1Click:Connect(function() aimStrength = 0.08 end)
StrongAimBtn.MouseButton1Click:Connect(function() aimStrength = 0.25 end)

HpEspBtn.MouseButton1Click:Connect(function()
	hpEspEnabled = not hpEspEnabled
	HpEspBtn.Text = "HP ESP ["..(hpEspEnabled and "ON" or "OFF").."]"
	if hpEspEnabled then
		for _,p in pairs(Players:GetPlayers()) do createHealthBar(p) end
	else
		removeHealthBars()
	end
end)

BypassBtn.MouseButton1Click:Connect(function()
	scriptDisabled = true
	removeHealthBars()
	FovCircle.Visible = false
	ScreenGui:Destroy()
end)

-- =========================
-- LOOP PRINCIPAL
-- =========================
RunService.RenderStepped:Connect(function()
	if scriptDisabled then return end
	FovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
	FovCircle.Radius = fovRadius

	if aimbotEnabled and holdingRightClick and lockedTarget then
		local cf = Camera.CFrame
		Camera.CFrame = cf:Lerp(CFrame.new(cf.Position, lockedTarget.Position), aimStrength)
	end
end)

Players.PlayerAdded:Connect(function(p)
	p.CharacterAdded:Connect(function()
		task.wait(1)
		if hpEspEnabled then createHealthBar(p) end
	end)
end)
