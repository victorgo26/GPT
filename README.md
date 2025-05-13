local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local corDentroVisao = Color3.fromRGB(255, 255, 0) -- Amarelo
local corForaVisao = Color3.fromRGB(255, 0, 0)     -- Vermelho
local espAtivado = true

-- Ligações entre partes do corpo para formar o esqueleto
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

-- Tabela para armazenar os desenhos por jogador
local desenhosPorJogador = {}

-- Cria as linhas para o jogador
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

-- Remove desenhos do jogador
local function removerDesenhos(player)
	if desenhosPorJogador[player] then
		for _, info in ipairs(desenhosPorJogador[player]) do
			info.Linha:Remove()
		end
		desenhosPorJogador[player] = nil
	end
end

-- Inicializar para jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
	criarDesenhos(player)
end

-- Conectar eventos
Players.PlayerAdded:Connect(criarDesenhos)
Players.PlayerRemoving:Connect(removerDesenhos)

-- Atualização por frame
RunService.RenderStepped:Connect(function()
	for player, linhas in pairs(desenhosPorJogador) do
		local char = player.Character
		if not (char and char:FindFirstChild("HumanoidRootPart")) then
			for _, info in ipairs(linhas) do
				info.Linha.Visible = false
			end
			continue
		end

		-- Verificar se o personagem está no campo de visão
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
