local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local espAtivado = true
local aimbotAtivado = true

local corDentroVisao = Color3.fromRGB(255, 255, 0) -- Amarelo
local corForaVisao = Color3.fromRGB(255, 0, 0)     -- Vermelho

local ligacoes = {
	{"Head", "UpperTorso"},
	{"UpperTorso", "LowerTorso"},
	{"UpperTorso", "LeftUpperArm"},
	{"LeftUpperArm", "LeftLowerArm"},
	{"LeftLowerArm", "LeftHand"},
	{"UpperTorso", "RightUpperArm"},
	{"RightUpperArm", "RightLowerArm"},
	{"RightLowerArm", "RightHand"},
	{"LowerTorso", "LeftUpperLeg"},
	{"LeftUpperLeg", "LeftLowerLeg"},
	{"LeftLowerLeg", "LeftFoot"},
	{"LowerTorso", "RightUpperLeg"},
	{"RightUpperLeg", "RightLowerLeg"},
	{"RightLowerLeg", "RightFoot"},
}

local desenhosPorJogador = {}

-- Criar linhas do esqueleto
local function criarDesenhos(player)
	if player == LocalPlayer then return end
	if desenhosPorJogador[player] then return end

	local linhas = {}
	for _, par in ipairs(ligacoes) do
		local linha = Drawing.new("Line")
		linha.Thickness = 2
		linha.Visible = false
		table.insert(linhas, {De = par[1], Para = par[2], Linha = linha})
	end
	desenhosPorJogador[player] = linhas
end

-- Remover linhas
local function removerDesenhos(player)
	if desenhosPorJogador[player] then
		for _, info in ipairs(desenhosPorJogador[player]) do
			info.Linha:Remove()
		end
		desenhosPorJogador[player] = nil
	end
end

-- Inicial
for _, player in ipairs(Players:GetPlayers()) do
	criarDesenhos(player)
end

Players.PlayerAdded:Connect(criarDesenhos)
Players.PlayerRemoving:Connect(removerDesenhos)

-- Função para encontrar o inimigo mais próximo
local function getAlvoMaisProximo()
	local menorDist = math.huge
	local alvo = nil

	for _, player in ipairs(Players:GetPlayers()) do
		if player == LocalPlayer then continue end
		local char = player.Character
		if char and char:FindFirstChild("Head") then
			local head = char:FindFirstChild("Head")
			local screenPos, visivel = Camera:WorldToViewportPoint(head.Position)
			if visivel then
				local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)).Magnitude
				if dist < menorDist then
					menorDist = dist
					alvo = head
				end
			end
		end
	end

	return alvo
end

-- Variável para saber se o botão do mouse está pressionado
local mirando = false

-- Detecção de pressionamento de tecla
UserInputService.InputBegan:Connect(function(input, gp)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = true
	elseif input.KeyCode == Enum.KeyCode.F4 then
		-- Alternar ativação/desativação do ESP e Aimbot
		espAtivado = not espAtivado
		aimbotAtivado = not aimbotAtivado
		print("ESP Ativado:", espAtivado, " | Aimbot Ativado:", aimbotAtivado)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = false
	end
end)

-- Atualização a cada frame
RunService.RenderStepped:Connect(function()
	-- Aimbot: Ajusta a câmera para mirar no inimigo mais próximo
	if mirando and aimbotAtivado then
		local alvo = getAlvoMaisProximo()
		if alvo then
			local dir = (alvo.Position - Camera.CFrame.Position).Unit
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir)
		end
	end

	-- ESP: Exibir linhas de esqueleto
	for player, linhas in pairs(desenhosPorJogador) do
		local char = player.Character
		if not (char and char:FindFirstChild("HumanoidRootPart")) then
			for _, info in ipairs(linhas) do
				info.Linha.Visible = false
			end
			continue
		end

		local hrp = char:FindFirstChild("HumanoidRootPart")
		local direcao = (hrp.Position - Camera.CFrame.Position).Unit
		local alinhamento = direcao:Dot(Camera.CFrame.LookVector)
		local estaNoCampo = alinhamento > 0.5
		local corAtual = estaNoCampo and corDentroVisao or corForaVisao

		for _, info in ipairs(linhas) do
			local parte1 = char:FindFirstChild(info.De)
			local parte2 = char:FindFirstChild(info.Para)
			local linha = info.Linha

			if parte1 and parte2 and espAtivado then
				local pos1, vis1 = Camera:WorldToViewportPoint(parte1.Position)
				local pos2, vis2 = Camera:WorldToViewportPoint(parte2.Position)

				if vis1 and vis2 then
					linha.From = Vector2.new(pos1.X, pos1.Y)
					linha.To = Vector2.new(pos2.X, pos2.Y)
					linha.Color = corAtual
					linha.Visible = true
				else
					linha.Visible = false
				end
			else
				linha.Visible = false
			end
		end
	end
end)
