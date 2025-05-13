local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local FOV_RADIUS = 100
local AIMBOT_MAX_DIST = 1000
local espAtivado = true
local aimbotAtivado = true
local mirando = false

local corDentroVisao = Color3.fromRGB(255, 255, 0)
local corForaVisao = Color3.fromRGB(255, 0, 0)
local corFOV = Color3.fromRGB(0, 255, 255)

local partesParaMostrar = {
	"Head",
	"Torso", "UpperTorso", "LowerTorso",
	"LeftArm", "LeftUpperArm", "LeftLowerArm",
	"RightArm", "RightUpperArm", "RightLowerArm",
	"LeftLeg", "LeftUpperLeg", "LeftLowerLeg",
	"RightLeg", "RightUpperLeg", "RightLowerLeg"
}

local desenhosPorJogador = {}

-- FOV circle
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = FOV_RADIUS
fovCircle.Thickness = 1
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Color = corFOV
fovCircle.Visible = true

-- GUI texto
local statusText = Drawing.new("Text")
statusText.Size = 18
statusText.Color = Color3.fromRGB(255, 255, 255)
statusText.Position = Vector2.new(10, 10)
statusText.Text = "ESP: ON | Aimbot: ON"
statusText.Outline = true
statusText.Visible = true

-- Verifica se é inimigo
local function ehInimigo(player)
	if not player.Team or not LocalPlayer.Team then return true end
	return player.Team ~= LocalPlayer.Team
end

-- Cria desenhos por parte
local function criarDesenhos(player)
	if player == LocalPlayer then return end
	if desenhosPorJogador[player] then return end

	local lista = {}
	for _, nomeParte in ipairs(partesParaMostrar) do
		local box = Drawing.new("Square")
		box.Thickness = 1.5
		box.Filled = false
		box.Visible = false
		table.insert(lista, {nome = nomeParte, desenho = box})
	end
	desenhosPorJogador[player] = lista
end

-- Remove desenhos
local function removerDesenhos(player)
	if desenhosPorJogador[player] then
		for _, parteInfo in ipairs(desenhosPorJogador[player]) do
			parteInfo.desenho:Remove()
		end
		desenhosPorJogador[player] = nil
	end
end

-- Jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
	criarDesenhos(player)
end

Players.PlayerAdded:Connect(criarDesenhos)
Players.PlayerRemoving:Connect(removerDesenhos)

-- Alvo mais próximo
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

-- Input
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

-- Loop
RunService.RenderStepped:Connect(function()
	fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)

	-- Aimbot
	if mirando and aimbotAtivado then
		local alvo = getAlvoMaisProximo()
		if alvo then
			local dir = (alvo.Position - Camera.CFrame.Position).Unit
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir)
		end
	end

	-- ESP por partes
	for player, partes in pairs(desenhosPorJogador) do
		local char = player.Character
		if not (char and char:FindFirstChild("HumanoidRootPart") and ehInimigo(player)) then
			for _, p in ipairs(partes) do p.desenho.Visible = false end
			continue
		end

		local hrp = char.HumanoidRootPart
		local direcao = (hrp.Position - Camera.CFrame.Position).Unit
		local alinhado = direcao:Dot(Camera.CFrame.LookVector)
		local corAtual = alinhado > 0.5 and corDentroVisao or corForaVisao

		for _, parteInfo in ipairs(partes) do
			local parte = char:FindFirstChild(parteInfo.nome)
			local desenho = parteInfo.desenho

			if parte and espAtivado then
				local tamanho = parte.Size or Vector3.new(1,1,1)
				local corner1 = parte.Position - (tamanho / 2)
				local corner2 = parte.Position + (tamanho / 2)

				local p1, v1 = Camera:WorldToViewportPoint(corner1)
				local p2, v2 = Camera:WorldToViewportPoint(corner2)

				if v1 and v2 then
					local topLeft = Vector2.new(math.min(p1.X, p2.X), math.min(p1.Y, p2.Y))
					local width = math.abs(p2.X - p1.X)
					local height = math.abs(p2.Y - p1.Y)

					desenho.Position = topLeft
					desenho.Size = Vector2.new(width, height)
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
