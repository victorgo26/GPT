local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local ESPs = {} -- Armazena os Highlights

-- Cores
local COR_VISIVEL = Color3.fromRGB(255, 255, 0) -- Amarelo
local COR_OBSTRUÍDO = Color3.fromRGB(255, 0, 0) -- Vermelho

-- Função para criar Highlight para um personagem
local function criarESP(player)
	if player == LocalPlayer then return end
	if ESPs[player] then return end

	local highlight = Instance.new("Highlight")
	highlight.Adornee = nil
	highlight.FillTransparency = 1 -- Sem preenchimento, só contorno
	highlight.OutlineTransparency = 0
	highlight.OutlineColor = COR_OBSTRUÍDO
	highlight.Parent = game.CoreGui -- ou LocalPlayer.PlayerGui, se preferir

	ESPs[player] = highlight
end

local function removerESP(player)
	if ESPs[player] then
		ESPs[player]:Destroy()
		ESPs[player] = nil
	end
end

-- Inicializar para todos os jogadores
for _, player in pairs(Players:GetPlayers()) do
	criarESP(player)
end

Players.PlayerAdded:Connect(criarESP)
Players.PlayerRemoving:Connect(removerESP)

-- Atualizar ESP constantemente
RunService.RenderStepped:Connect(function()
	for player, highlight in pairs(ESPs) do
		local char = player.Character
		local root = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChildOfClass("Humanoid")

		if char and root and hum and hum.Health > 0 then
			highlight.Adornee = char

			-- Raycast da câmera até o HumanoidRootPart
			local origem = Camera.CFrame.Position
			local direcao = (root.Position - origem).Unit * 500

			local params = RaycastParams.new()
			params.FilterType = Enum.RaycastFilterType.Blacklist
			params.FilterDescendantsInstances = {Camera, LocalPlayer.Character}

			local resultado = Workspace:Raycast(origem, direcao, params)

			if resultado and resultado.Instance:IsDescendantOf(char) then
				-- Visível
				highlight.OutlineColor = COR_VISIVEL
			else
				-- Obstruído
				highlight.OutlineColor = COR_OBSTRUÍDO
			end

			highlight.Enabled = true
		else
			highlight.Enabled = false
		end
	end
end)
