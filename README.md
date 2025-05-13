local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- CONFIG
local ESP_ENABLED = true
local AIMBOT_ENABLED = true
local TARGET_PART = "Head" -- ou "HumanoidRootPart"

-- CORES
local COR_VISIVEL = Color3.fromRGB(255, 255, 0) -- Amarelo
local COR_OBSTRUÍDO = Color3.fromRGB(255, 0, 0) -- Vermelho

-- GERENCIAMENTO DE HIGHLIGHT
local ESPs = {}

local function criarESP(player)
	if player == LocalPlayer then return end
	if ESPs[player] then return end

	local highlight = Instance.new("Highlight")
	highlight.FillTransparency = 1
	highlight.OutlineTransparency = 0
	highlight.OutlineColor = COR_OBSTRUÍDO
	highlight.Enabled = true
	highlight.Adornee = nil
	highlight.Parent = game.CoreGui

	ESPs[player] = highlight
end

local function removerESP(player)
	if ESPs[player] then
		ESPs[player]:Destroy()
		ESPs[player] = nil
	end
end

for _, p in pairs(Players:GetPlayers()) do
	criarESP(p)
end

Players.PlayerAdded:Connect(criarESP)
Players.PlayerRemoving:Connect(removerESP)

-- Função para verificar se alvo está visível
local function estaVisivel(alvo)
	local origem = Camera.CFrame.Position
	local direcao = (alvo.Position - origem).Unit * 500
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Blacklist
	params.FilterDescendantsInstances = {LocalPlayer.Character, Camera}

	local resultado = Workspace:Raycast(origem, direcao, params)
	return resultado and resultado.Instance:IsDescendantOf(alvo.Parent)
end

-- ESP LOOP
RunService.RenderStepped:Connect(function()
	if not ESP_ENABLED then
		for _, esp in pairs(ESPs) do
			esp.Enabled = false
		end
		return
	end

	for player, esp in pairs(ESPs) do
		local char = player.Character
		local part = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChildOfClass("Humanoid")

		if char and part and hum and hum.Health > 0 then
			esp.Adornee = char
			esp.Enabled = true

			if estaVisivel(part) then
				esp.OutlineColor = COR_VISIVEL
			else
				esp.OutlineColor = COR_OBSTRUÍDO
			end
		else
			esp.Enabled = false
		end
	end
end)

-- AIMBOT
local aiming = false

UserInputService.InputBegan:Connect(function(input, gp)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		aiming = true
	end
end)

UserInputService.InputEnded:Connect(function(input, gp)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		aiming = false
	end
end)

-- Aimbot Loop
RunService.RenderStepped:Connect(function()
	if not AIMBOT_ENABLED or not aiming then return end

	local alvoMaisProximo, menorDist = nil, math.huge

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			local char = player.Character
			local part = char and char:FindFirstChild(TARGET_PART)
			local hum = char and char:FindFirstChildOfClass("Humanoid")

			if part and hum and hum.Health > 0 then
				local pos, visivel = Camera:WorldToViewportPoint(part.Position)
				if visivel and estaVisivel(part) then
					local mousePos = UserInputService:GetMouseLocation()
					local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude

					if dist < menorDist then
						menorDist = dist
						alvoMaisProximo = part
					end
				end
			end
		end
	end

	if alvoMaisProximo then
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, alvoMaisProximo.Position)
	end
end)

-- PAINEL F4
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "PainelAimbotESP"
gui.ResetOnSpawn = false
gui.Enabled = false

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 300, 0, 200)
frame.Position = UDim2.new(0.5, -150, 0.5, -100)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.Visible = true

local btnESP = Instance.new("TextButton", frame)
btnESP.Text = "ESP: ON"
btnESP.Size = UDim2.new(1, -20, 0, 40)
btnESP.Position = UDim2.new(0, 10, 0, 10)
btnESP.MouseButton1Click:Connect(function()
	ESP_ENABLED = not ESP_ENABLED
	btnESP.Text = "ESP: " .. (ESP_ENABLED and "ON" or "OFF")
end)

local btnAimbot = Instance.new("TextButton", frame)
btnAimbot.Text = "AIMBOT: ON"
btnAimbot.Size = UDim2.new(1, -20, 0, 40)
btnAimbot.Position = UDim2.new(0, 10, 0, 60)
btnAimbot.MouseButton1Click:Connect(function()
	AIMBOT_ENABLED = not AIMBOT_ENABLED
	btnAimbot.Text = "AIMBOT: " .. (AIMBOT_ENABLED and "ON" or "OFF")
end)

local btnTrocarParte = Instance.new("TextButton", frame)
btnTrocarParte.Text = "MIRAR: HEAD"
btnTrocarParte.Size = UDim2.new(1, -20, 0, 40)
btnTrocarParte.Position = UDim2.new(0, 10, 0, 110)
btnTrocarParte.MouseButton1Click:Connect(function()
	if TARGET_PART == "Head" then
		TARGET_PART = "HumanoidRootPart"
		btnTrocarParte.Text = "MIRAR: CORPO"
	else
		TARGET_PART = "Head"
		btnTrocarParte.Text = "MIRAR: HEAD"
	end
end)

-- F4 para mostrar/ocultar painel
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if input.KeyCode == Enum.KeyCode.F4 and not gameProcessed then
		gui.Enabled = not gui.Enabled
	end
end)
