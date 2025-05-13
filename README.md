local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações
local FOV_RADIUS = 100
local AIMBOT_MAX_DIST = 1000
local espAtivado = true
local aimbotAtivado = true
local mirando = false

local corDentroVisao = Color3.fromRGB(255, 255, 0) -- Amarelo
local corForaVisao = Color3.fromRGB(255, 0, 0)     -- Vermelho
local corFOV = Color3.fromRGB(0, 255, 255)

local caixasPorJogador = {}

-- Círculo de FOV
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = FOV_RADIUS
fovCircle.Thickness = 1
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Color = corFOV
fovCircle.Visible = true

-- GUI simples com Drawing
local statusText = Drawing.new("Text")
statusText.Size = 18
statusText.Color = Color3.fromRGB(255, 255, 255)
statusText.Position = Vector2.new(10, 10)
statusText.Text = "ESP: ON | Aimbot: ON"
statusText.Outline = true
statusText.Visible = true

-- Criar caixa
local function criarCaixa(player)
	if player == LocalPlayer then return end
	if caixasPorJogador[player] then return end

	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Filled = false
	box.Visible = false
	caixasPorJogador[player] = box
end

-- Remover caixa
local function removerCaixa(player)
	if caixasPorJogador[player] then
		caixasPorJogador[player]:Remove()
		caixasPorJogador[player] = nil
	end
end

-- Jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
	criarCaixa(player)
end

Players.PlayerAdded:Connect(criarCaixa)
Players.PlayerRemoving:Connect(removerCaixa)

-- Inimigo? (verifica Team)
local function ehInimigo(player)
	if not player.Team or not LocalPlayer.Team then return true end
	return player.Team ~= LocalPlayer.Team
end

-- Alvo mais próximo dentro do FOV
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
				local dist2D = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
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

-- Input do usuário
UserInputService.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = true
	elseif input.KeyCode == Enum.KeyCode.F4 then
		espAtivado = not espAtivado
		aimbotAtivado = not aimbotAtivado
		statusText.Text = "ESP: " .. (espAtivado and "ON" or "OFF") .. " | Aimbot: " .. (aimbotAtivado and "ON" or "OFF")
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = false
	end
end)

-- Loop principal
RunService.RenderStepped:Connect(function()
	local centro = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
	fovCircle.Position = centro

	-- Aimbot
	if mirando and aimbotAtivado then
		local alvo = getAlvoMaisProximo()
		if alvo then
			local dir = (alvo.Position - Camera.CFrame.Position).Unit
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir)
		end
	end

	-- ESP
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

		if (v1 and v2) and espAtivado then
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
