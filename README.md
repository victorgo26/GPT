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

-- Cores
local corLinha = Color3.fromRGB(255, 0, 0)
local corFOV = Color3.fromRGB(0, 255, 255)

-- FOV circle
local fovCircle = Drawing.new("Circle")
fovCircle.Radius = FOV_RADIUS
fovCircle.Thickness = 1
fovCircle.Filled = false
fovCircle.Color = corFOV
fovCircle.Visible = true

-- Status
local statusText = Drawing.new("Text")
statusText.Size = 18
statusText.Color = Color3.fromRGB(255, 255, 255)
statusText.Position = Vector2.new(10, 10)
statusText.Text = "ESP: ON | Aimbot: ON"
statusText.Outline = true
statusText.Visible = true

-- Esqueleto (linhas conectando partes)
local conexoes = {
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
	{"RightLowerLeg", "RightFoot"}
}

local desenhos = {}

-- Checa inimigos
local function ehInimigo(player)
	return player ~= LocalPlayer and (not player.Team or player.Team ~= LocalPlayer.Team)
end

-- Cria linhas de esqueleto
local function criarDesenhos(player)
	if desenhos[player] then return end
	desenhos[player] = {}
	for _, pair in ipairs(conexoes) do
		local linha = Drawing.new("Line")
		linha.Thickness = 2
		linha.Color = corLinha
		linha.Visible = false
		table.insert(desenhos
