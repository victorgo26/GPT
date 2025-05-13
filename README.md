local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local corDentroVisao = Color3.fromRGB(255, 255, 0) -- Amarelo
local corForaVisao = Color3.fromRGB(255, 0, 0)     -- Vermelho
local espAtivado = true

-- Lista de partes que vamos mostrar
local partesParaMostrar = {
	"Head",
	"Torso", "UpperTorso", "LowerTorso",
	"LeftArm", "LeftUpperArm", "LeftLowerArm",
	"RightArm", "RightUpperArm", "RightLowerArm",
	"LeftLeg", "LeftUpperLeg", "LeftLowerLeg",
	"RightLeg", "RightUpperLeg", "RightLowerLeg"
}

-- Criar desenhos para cada parte
local desenhosPorJogador = {}

local function criarDesenhos(player)
	if player == LocalPlayer then return end
	if desenhosPorJogador[player] then return end

	local desenhos = {}
	for _, parte in ipairs(partesParaMostrar) do
		local dot = Drawing.new("Circle")
		dot.Radius = 4
		dot.Thickness = 2
		dot.Visible = false
		dot.Filled = true
		table.insert(desenhos, {nome = parte, desenho = dot})
	end
	desenhosPorJogador[player] = desenhos
end

local function removerDesenhos(player)
	if desenhosPorJogador[player] then
		for _, info in ipairs(desenhosPorJogador[player]) do
			info.desenho:Remove()
		end
		desenhosPorJogador[player] = nil
	end
end

-- Iniciar para os jogadores existentes
for _, player in pairs(Players:GetPlayers()) do
	criarDesenhos(player)
end

-- Conectar jogadores entrando e saindo
Players.PlayerAdded:Connect(criarDesenhos)
Players.PlayerRemoving:Connect(removerDesenhos)

-- Atualizar posição a cada frame
RunService.RenderStepped:Connect(function()
	for player, partes in pairs(desenhosPorJogador) do
		local char = player.Character
		if not (char and char:FindFirstChild("Humanoid") and char:FindFirstChild("HumanoidRootPart")) then
			for _, parteInfo in ipairs(partes) do
				parteInfo.desenho.Visible = false
			end
			continue
		end

		local hrp = char:FindFirstChild("HumanoidRootPart")
		local direcao = (hrp.Position - Camera.CFrame.Position).Unit
		local alinhamento = direcao:Dot(Camera.CFrame.LookVector)
		local estaNoCampo = alinhamento > 0.5

		local corAtual = estaNoCampo and corDentroVisao or corForaVisao

		for _, parteInfo in ipairs(partes) do
			local parte = char:FindFirstChild(parteInfo.nome)
			local desenho = parteInfo.desenho

			if parte then
				local screenPos, visivel = Camera:WorldToViewportPoint(parte.Position)
				if visivel and espAtivado then
					desenho.Position = Vector2.new(screenPos.X, screenPos.Y)
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
