local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local corDentroVisao = Color3.fromRGB(255, 255, 0) -- Amarelo
local corForaVisao = Color3.fromRGB(255, 0, 0)     -- Vermelho
local espAtivado = true

-- Ligações entre partes do corpo (para formar o esqueleto)
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

-- Guardar desenhos por jogador
local desenhosPorJogador = {}

-- Criar linhas para o jogador
local function criarDesenhos(player)
	if player == LocalPlayer then return end
	if desenhosPorJogador[player] then return end

	local linhas = {}
	for _, par in ipairs(ligacoes) do
		local linha = Drawing.new("Line")
		linha.Thickness = 2
		linha.Visible = false
		table.insert(linhas, {de = par[1], para = par[2], desenho = linha})
	end
	desenhosPorJogador[player] = linhas
end

-- Remover os desenhos
local function removerDesenhos(player)
	if desenhosPorJogador[player] then
		for _, info in ipairs(desenhosPorJogador[player]) do
			info.desenho:Remove()
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

-- Atualização a cada frame
RunService.RenderStepped:Connect(function()
	for player, linhas in pairs(desenhosPorJogador) do
		local char = player.Character
		if not (char and char:FindFirstChild("HumanoidRootPart")) then
			for _, info in ipairs(linhas) do
				info.desenho.Visible = false
			end
			continue
		end

		local hrp = char.HumanoidRootPart
		local direcao = (hrp.Position - Camera.CFrame.Position).Unit
		local alinhamento = direcao:Dot(Camera.CFrame.LookVector)
		local estaNoCampo = alinhamento > 0.5
		local corAtual = estaNoCampo and corDentroVisao or corForaVisao

		for _, info in ipairs(linhas) do
			local parte1 = char:FindFirstChild(info.de)
			local parte2 = char:FindFirstChild(info.para)
			local desenho = info.desenho

			if parte1 and parte2 then
				local p1, v1 = Camera:WorldToViewportPoint(parte1.Position)
				local p2, v2 = Camera:WorldToViewportPoint(parte2.Position)
				if v1 and v2 and espAtivado then
					desenho.From = Vector2.new(p1.X, p1.Y)
					desenho.To = Vector2.new(p2.X, p2.Y)
					desenho.Color = corAtual
					desenho.Visible = true
				else
					desenho.Visible = false
				end
			else
				desenho.Visible = false
			end
		end
	end
end)
