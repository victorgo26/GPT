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
local corForaVisao = Color3.fromRGB(255, 0, 0) -- Vermelho
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

-- Criar caixa e esqueleto para um jogador
local function criarCaixa(player)
	if player == LocalPlayer then return end
	if caixasPorJogador[player] then return end

	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Filled = false
	box.Visible = false

	local skeleton = {}
	local parts = {
		"Head", "UpperTorso", "LowerTorso",
		"LeftUpperArm", "LeftLowerArm", "LeftHand",
		"RightUpperArm", "RightLowerArm", "RightHand",
		"LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
		"RightUpperLeg", "RightLowerLeg", "RightFoot",
	}

	for _, name in ipairs(parts) do
		skeleton[name] = nil
	end

	skeleton.Lines = {}
	for i = 1, 12 do
		local line = Drawing.new("Line")
		line.Thickness = 1.5
		line.Transparency = 1
		line.Color = Color3.fromRGB(0, 255, 0)
		line.Visible = false
		table.insert(skeleton.Lines, line)
	end

	caixasPorJogador[player] = {box = box, skeleton = skeleton}

	-- Atualizar ao spawnar
	player.CharacterAdded:Connect(function()
		caixasPorJogador[player].box.Visible = false
		for _, line in ipairs(caixasPorJogador[player].skeleton.Lines) do
			line.Visible = false
		end
	end)
end

-- Remover caixa e esqueleto
local function removerCaixa(player)
	if caixasPorJogador[player] then
		caixasPorJogador[player].box:Remove()
		for _, line in ipairs(caixasPorJogador[player].skeleton.Lines) do
			line:Remove()
		end
		caixasPorJogador[player] = nil
	end
end

-- Jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
	criarCaixa(player)
end

Players.PlayerAdded:Connect(criarCaixa)
Players.PlayerRemoving:Connect(removerCaixa)

-- Verifica se é inimigo (checa time)
local function ehInimigo(player)
	if not player.Team or not LocalPlayer.Team then return true end
	return player.Team ~= LocalPlayer.Team
end

-- Retorna o alvo mais próximo dentro do FOV
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
	local centro = Camera.ViewportSize / 2
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
	for player, dados in pairs(caixasPorJogador) do
		local box = dados.box
		local skeleton = dados.skeleton
		local char = player.Character

		if not char or not char:FindFirstChild("HumanoidRootPart") or not ehInimigo(player) then
			box.Visible = false
			for _, line in ipairs(skeleton.Lines) do
				line.Visible = false
			end
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

		-- Skeleton ESP
		local function getPos(partName)
			local part = char:FindFirstChild(partName)
			if not part then return nil end
			local screenPos, visible = Camera:WorldToViewportPoint(part.Position)
			return visible and Vector2.new(screenPos.X, screenPos.Y) or nil
		end

		local points = {
			Head = getPos("Head"),
			UpperTorso = getPos("UpperTorso"),
			LowerTorso = getPos("LowerTorso"),
			LUA = getPos("LeftUpperArm"),
			LLA = getPos("LeftLowerArm"),
			LH = getPos("LeftHand"),
			RUA = getPos("RightUpperArm"),
			RLA = getPos("RightLowerArm"),
			RH = getPos("RightHand"),
			LUL = getPos("LeftUpperLeg"),
			LLL = getPos("LeftLowerLeg"),
			LF = getPos("LeftFoot"),
			RUL = getPos("RightUpperLeg"),
			RLL = getPos("RightLowerLeg"),
			RF = getPos("RightFoot"),
		}

		local function drawLine(i, from, to)
			if from and to then
				skeleton.Lines[i].From = from
				skeleton.Lines[i].To = to
				skeleton.Lines[i].Visible = true
			else
				skeleton.Lines[i].Visible = false
			end
		end

		-- Desenhar esqueleto
		drawLine(1, points.Head, points.UpperTorso)
		drawLine(2, points.UpperTorso, points.LowerTorso)
		drawLine(3, points.UpperTorso, points.LUA)
		drawLine(4, points.LUA, points.LLA)
		drawLine(5, points.LLA, points.LH)
		drawLine(6, points.UpperTorso, points.RUA)
		drawLine(7, points.RUA, points.RLA)
		drawLine(8, points.RLA, points.RH)
		drawLine(9, points.LowerTorso, points.LUL)
		drawLine(10, points.LUL, points.LLL)
		drawLine(11, points.LLL, points.LF)
		drawLine(12, points.LowerTorso, points.RUL)
		drawLine(13, points.RUL, points.RLL)
		drawLine(14, points.RLL, points.RF)
	end
end)
