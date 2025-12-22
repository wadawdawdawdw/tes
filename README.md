--// STUART HUB - Aimbot + FOV + ESP Tronco
--// LocalScript | StarterPlayer > StarterPlayerScripts

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
local bypass = false
local lockedTarget = nil

-- FORÇA DO AIMBOT
local aimStrength = 0.15 -- quanto maior, mais forte (0.05 a 1)

-- ESP
local espEnabled = false
local espObjects = {}

-- =========================
-- UI
-- =========================
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.Name = "StuartHUB"

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 320, 0, 470)
MainFrame.Position = UDim2.new(0.35, 0, 0.25, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0,12)

-- Título
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
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 14
	btn.TextColor3 = Color3.new(1,1,1)
	btn.BackgroundColor3 = Color3.fromRGB(40,40,40)
	btn.BorderSizePixel = 0
	Instance.new("UICorner", btn)
	return btn
end

-- =========================
-- BOTÕES
-- =========================
local AimBtn      = createButton("AIMBOT [OFF]", 60)
local FovBtn      = createButton("FOV CHECK [OFF]", 105)
local PlusBtn     = createButton("AUMENTAR CÍRCULO", 150)
local MinusBtn    = createButton("DIMINUIR CÍRCULO", 195)
local DeadBtn     = createButton("IGNORAR MORTOS [OFF]", 240)
local EspBtn      = createButton("ESP TRONCO [OFF]", 285)
local WeakAimBtn  = createButton("AIMBOT FRACO", 330)
local StrongAimBtn= createButton("AIMBOT FORTE", 375)
local BypassBtn   = createButton("BYPASS", 420)
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
	if not player.Character then return false end
	local hum = player.Character:FindFirstChild("Humanoid")
	return hum and hum.Health > 0
end

local function getClosestPlayer()
	local closest, shortest = nil, math.huge

	for _,plr in pairs(Players:GetPlayers()) do
		if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
			if ignoreDead and not isAlive(plr) then continue end

			local head = plr.Character.Head
			local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X,pos.Y) -
					Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)).Magnitude

				if fovEnabled and dist > fovRadius then continue end

				if dist < shortest then
					shortest = dist
					closest = head
				end
			end
		end
	end
	return closest
end

-- =========================
-- ESP
-- =========================
local function createESP(player)
	if player == LocalPlayer then return end
	if not player.Character then return end

	local hrp = player.Character:FindFirstChild("HumanoidRootPart")
	if not hrp or espObjects[player] then return end

	local box = Instance.new("BoxHandleAdornment")
	box.Adornee = hrp
	box.Size = Vector3.new(2,3,1)
	box.Color3 = Color3.fromRGB(255,0,0)
	box.Transparency = 0.3
	box.AlwaysOnTop = true
	box.ZIndex = 10
	box.Parent = hrp

	espObjects[player] = box
end

local function removeESP(player)
	if espObjects[player] then
		espObjects[player]:Destroy()
		espObjects[player] = nil
	end
end

local function updateESP()
	for _,plr in pairs(Players:GetPlayers()) do
		if espEnabled then
			createESP(plr)
		else
			removeESP(plr)
		end
	end
end

Players.PlayerRemoving:Connect(removeESP)

-- =========================
-- INPUT
-- =========================
UserInputService.InputBegan:Connect(function(input)
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
-- BOTÕES LÓGICA
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

EspBtn.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	EspBtn.Text = "ESP TRONCO [" .. (espEnabled and "ON" or "OFF") .. "]"
	updateESP()
end)

-- Aimbot fraco
WeakAimBtn.MouseButton1Click:Connect(function()
	aimStrength = math.clamp(aimStrength - 0.05, 0.05, 1)
end)

-- Aimbot forte
StrongAimBtn.MouseButton1Click:Connect(function()
	aimStrength = math.clamp(aimStrength + 0.05, 0.05, 1)
end)

BypassBtn.MouseButton1Click:Connect(function()
	bypass = true
	ScreenGui.Enabled = false
	FovCircle.Visible = false
	aimbotEnabled = false
	fovEnabled = false
	espEnabled = false
	lockedTarget = nil
	updateESP()
end)

-- =========================
-- LOOP
-- =========================
RunService.RenderStepped:Connect(function()
	if bypass then return end

	FovCircle.Position = Vector2.new(
		Camera.ViewportSize.X / 2,
		Camera.ViewportSize.Y / 2
	)
	FovCircle.Radius = fovRadius

	if aimbotEnabled and holdingRightClick and lockedTarget then
		local currentCF = Camera.CFrame
		local targetCF = CFrame.new(currentCF.Position, lockedTarget.Position)
		Camera.CFrame = currentCF:Lerp(targetCF, aimStrength)
	end
end)
