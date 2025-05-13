local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configs iniciais
local FOV_RADIUS = 100
local AIMBOT_MAX_DIST = 1000
local ESP_ENABLED = true
local AIMBOT_ENABLED = true
local MIRANDO = false
local AIMBOT_KEY = Enum.UserInputType.MouseButton2

local corDentroVisao = Color3.fromRGB(255, 255, 0)
local corForaVisao = Color3.fromRGB(255, 0, 0)
local corFOV = Color3.fromRGB(0, 255, 255)

-- Drawing FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Visible = true

-- Status Text
local statusText = Drawing.new("Text")
statusText.Size = 18
statusText.Position = Vector2.new(10, 10)
statusText.Color = Color3.fromRGB(255, 255, 255)
statusText.Outline = true
statusText.Visible = true

-- Caixa ESP
local caixasPorJogador = {}

local function criarCaixa(player)
	if player == LocalPlayer then return end
	if caixasPorJogador[player] then return end

	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Filled = false
	box.Visible = false
	caixasPorJogador[player] = box
end

local function removerCaixa(player)
	if caixasPorJogador[player] then
		caixasPorJogador[player]:Remove()
		caixasPorJogador[player] = nil
	end
end

for _, p in ipairs(Players:GetPlayers()) do criarCaixa(p) end
Players.PlayerAdded:Connect(criarCaixa)
Players.PlayerRemoving:Connect(removerCaixa)

local function ehInimigo(player)
	if not player.Team or not LocalPlayer.Team then return true end
	return player.Team ~= LocalPlayer.Team
end

local function getAlvoMaisProximo()
	local menorDist = math.huge
	local alvo = nil

	for _, player in ipairs(Players:GetPlayers()) do
		if player == LocalPlayer or not ehInimigo(player) then continue end
		local char = player.Character
		if char and char:FindFirstChild("Head") then
			local head = char.Head
			local pos, visivel = Camera:WorldToViewportPoint(head.Position)
			if visivel then
				local dist2D = (Vector2.new(pos.X, pos.Y) - Camera.ViewportSize / 2).Magnitude
				local dist3D = (Camera.CFrame.Position - head.Position).Magnitude
				if dist2D < FOV_RADIUS and dist3D < AIMBOT_MAX_DIST and dist2D < menorDist then
					menorDist = dist2D
					alvo = head
				end
			end
		end
	end

	return alvo
end

UserInputService.InputBegan:Connect(function(input)
	if input.UserInputType == AIMBOT_KEY then
		MIRANDO = true
	elseif input.KeyCode == Enum.KeyCode.F4 then
		ESP_ENABLED = not ESP_ENABLED
		AIMBOT_ENABLED = not AIMBOT_ENABLED
		statusText.Text = "ESP: " .. (ESP_ENABLED and "ON" or "OFF") .. " | Aimbot: " .. (AIMBOT_ENABLED and "ON" or "OFF")
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == AIMBOT_KEY then
		MIRANDO = false
	end
end)

-- PAINEL GUI
local gui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
gui.Name = "PainelAimbot"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 200, 0, 240)
frame.Position = UDim2.new(0, 10, 0, 100)
frame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
frame.BorderSizePixel = 0

-- Slider de FOV
local fovLabel = Instance.new("TextLabel", frame)
fovLabel.Text = "FOV Radius"
fovLabel.Size = UDim2.new(1, 0, 0, 20)
fovLabel.TextColor3 = Color3.new(1, 1, 1)
fovLabel.BackgroundTransparency = 1

local fovSlider = Instance.new("TextBox", frame)
fovSlider.Position = UDim2.new(0, 0, 0, 25)
fovSlider.Size = UDim2.new(1, 0, 0, 25)
fovSlider.Text = tostring(FOV_RADIUS)

fovSlider.FocusLost:Connect(function()
	local val = tonumber(fovSlider.Text)
	if val then FOV_RADIUS = val end
end)

-- Mudança de tecla
local keyLabel = Instance.new("TextLabel", frame)
keyLabel.Text = "Tecla do Aimbot"
keyLabel.Size = UDim2.new(1, 0, 0, 20)
keyLabel.Position = UDim2.new(0, 0, 0, 60)
keyLabel.TextColor3 = Color3.new(1, 1, 1)
keyLabel.BackgroundTransparency = 1

local keyBox = Instance.new("TextBox", frame)
keyBox.Position = UDim2.new(0, 0, 0, 85)
keyBox.Size = UDim2.new(1, 0, 0, 25)
keyBox.Text = "MouseButton2"

keyBox.FocusLost:Connect(function()
	local val = keyBox.Text
	if val == "MouseButton2" then
		AIMBOT_KEY = Enum.UserInputType.MouseButton2
	elseif val == "MouseButton1" then
		AIMBOT_KEY = Enum.UserInputType.MouseButton1
	end
end)

-- Mudança de cor FOV
local colorFovBox = Instance.new("TextBox", frame)
colorFovBox.Position = UDim2.new(0, 0, 0, 125)
colorFovBox.Size = UDim2.new(1, 0, 0, 25)
colorFovBox.Text = "0,255,255"

local function strToColor3(str)
	local r, g, b = str:match("(%d+),(%d+),(%d+)")
	return Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b))
end

colorFovBox.FocusLost:Connect(function()
	local color = strToColor3(colorFovBox.Text)
	if color then corFOV = color fovCircle.Color = color end
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
	local centro = Camera.ViewportSize / 2
	fovCircle.Position = centro
	fovCircle.Radius = FOV_RADIUS

	if MIRANDO and AIMBOT_ENABLED then
		local alvo = getAlvoMaisProximo()
		if alvo then
			local dir = (alvo.Position - Camera.CFrame.Position).Unit
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir)
		end
	end

	for player, box in pairs(caixasPorJogador) do
		local char = player.Character
		if not char or not char:FindFirstChild("HumanoidRootPart") or not ehInimigo(player) then
			box.Visible = false
			continue
		end

		local hrp = char.HumanoidRootPart
		local min = hrp.Position - Vector3.new(2, 3, 0)
		local max = hrp.Position + Vector3.new(2, 3, 0)

		local p1, v1 = Camera:WorldToViewportPoint(min)
		local p2, v2 = Camera:WorldToViewportPoint(max)

		if (v1 and v2) and ESP_ENABLED then
			local topLeft = Vector2.new(math.min(p1.X, p2.X), math.min(p1.Y, p2.Y))
			local width = math.abs(p2.X - p1.X)
			local height = math.abs(p2.Y - p1.Y)

			box.Position = topLeft
			box.Size = Vector2.new(width, height)

			local direcao = (hrp.Position - Camera.CFrame.Position).Unit
			local alinhamento = direcao:Dot(Camera.CFrame.LookVector)
			box.Color = alinhamento > 0.5 and corDentroVisao or corForaVisao

			box.Visible = true
		else
			box.Visible = false
		end
	end
end)
