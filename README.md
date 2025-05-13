local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local espAtivado = true
local aimbotAtivado = true
local mirando = false

local corDentroVisao = Color3.fromRGB(255, 255, 0) -- Amarelo
local corForaVisao = Color3.fromRGB(255, 0, 0)     -- Vermelho

local caixasPorJogador = {}

-- Criar caixa para um jogador
local function criarCaixa(player)
	if player == LocalPlayer then return end
	if caixasPorJogador[player] then return end

	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Visible = false
	box.Filled = false
	caixasPorJogador[player] = box
end

-- Remover caixa de jogador
local function removerCaixa(player)
	if caixasPorJogador[player] then
		caixasPorJogador[player]:Remove()
		caixasPorJogador[player] = nil
	end
end

-- Aplicar a todos os jogadores existentes
for _, player in ipairs(Players:GetPlayers()) do
	criarCaixa(player)
end

Players.PlayerAdded:Connect(criarCaixa)
Players.PlayerRemoving:Connect(removerCaixa)

-- Encontrar o inimigo mais próximo da mira
local function getAlvoMaisProximo()
	local menorDist = math.huge
	local alvo = nil

	for _, player in ipairs(Players:GetPlayers()) do
		if player == LocalPlayer then continue end
		local char = player.Character
		if char and char:FindFirstChild("Head") then
			local head = char.Head
			local pos, visivel = Camera:WorldToViewportPoint(head.Position)
			if visivel then
				local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)).Magnitude
				if dist < menorDist then
					menorDist = dist
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
		print("ESP:", espAtivado, " | Aimbot:", aimbotAtivado)
	end
end)

UserInputService.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		mirando = false
	end
end)

-- Loop de atualização
RunService.RenderStepped:Connect(function()
	-- Aimbot
	if mirando and aimbotAtivado then
		local alvo = getAlvoMaisProximo()
		if alvo then
			local dir = (alvo.Position - Camera.CFrame.Position).Unit
			Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir)
		end
	end

	-- ESP com caixas
	for player, box in pairs(caixasPorJogador) do
		local char = player.Character
		if not (char and char:FindFirstChild("HumanoidRootPart")) then
			box.Visible = false
			continue
		end

		local hrp = char.HumanoidRootPart
		local min = (hrp.Position - Vector3.new(2, 3, 0))
		local max = (hrp.Position + Vector3.new(2, 3, 0))

		local p1, v1 = Camera:WorldToViewportPoint(Vector3.new(min.X, min.Y, min.Z))
		local p2, v2 = Camera:WorldToViewportPoint(Vector3.new(max.X, max.Y, max.Z))

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
