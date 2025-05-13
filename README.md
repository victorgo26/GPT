local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- CONFIGURAÇÕES
local espAtivado = true
local corForaVisao = Color3.fromRGB(255, 255, 255)
local corDentroVisao = Color3.fromRGB(255, 0, 0)

-- GUI PANEL
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "ESPGUI"
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Position = UDim2.new(0.7, 0, 0.3, 0)
Frame.Size = UDim2.new(0, 250, 0, 200)
Frame.Visible = false
Frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)

local ToggleButton = Instance.new("TextButton", Frame)
ToggleButton.Size = UDim2.new(1, -20, 0, 40)
ToggleButton.Position = UDim2.new(0, 10, 0, 10)
ToggleButton.Text = "Ativar/Desativar ESP"
ToggleButton.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

local DentroColor = Instance.new("TextBox", Frame)
DentroColor.PlaceholderText = "Cor visão (RGB: 255,0,0)"
DentroColor.Size = UDim2.new(1, -20, 0, 30)
DentroColor.Position = UDim2.new(0, 10, 0, 60)

local ForaColor = Instance.new("TextBox", Frame)
ForaColor.PlaceholderText = "Cor fora visão (RGB: 255,255,255)"
ForaColor.Size = UDim2.new(1, -20, 0, 30)
ForaColor.Position = UDim2.new(0, 10, 0, 100)

ToggleButton.MouseButton1Click:Connect(function()
	espAtivado = not espAtivado
end)

DentroColor.FocusLost:Connect(function()
	local r,g,b = DentroColor.Text:match("(%d+),%s*(%d+),%s*(%d+)")
	if r and g and b then
		corDentroVisao = Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b))
	end
end)

ForaColor.FocusLost:Connect(function()
	local r,g,b = ForaColor.Text:match("(%d+),%s*(%d+),%s*(%d+)")
	if r and g and b then
		corForaVisao = Color3.fromRGB(tonumber(r), tonumber(g), tonumber(b))
	end
end)

-- Mostrar/ocultar painel com F4
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if input.KeyCode == Enum.KeyCode.F4 and not gameProcessed then
		Frame.Visible = not Frame.Visible
	end
end)

-- ESP Drawing
local espBoxes = {}

local function createESP(player)
	if player == LocalPlayer then return end
	local box = Drawing.new("Square")
	box.Thickness = 2
	box.Transparency = 1
	box.Filled = false
	box.Visible = false
	espBoxes[player] = box
end

local function removeESP(player)
	if espBoxes[player] then
		espBoxes[player]:Remove()
		espBoxes[player] = nil
	end
end

Players.PlayerAdded:Connect(createESP)
Players.PlayerRemoving:Connect(removeESP)

for _, player in pairs(Players:GetPlayers()) do
	createESP(player)
end

RunService.RenderStepped:Connect(function()
	if not espAtivado then
		for _, box in pairs(espBoxes) do
			box.Visible = false
		end
		return
	end

	for player, box in pairs(espBoxes) do
		local char = player.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		local hum = char and char:FindFirstChildOfClass("Humanoid")

		if hrp and hum and hum.Health > 0 then
			local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
			if onScreen then
				local size = Vector2.new(60, 100) -- Tamanho da caixa
				box.Position = Vector2.new(pos.X - size.X/2, pos.Y - size.Y/2)
				box.Size = size
				box.Visible = true

				-- Verifica se está no campo de visão
				local dir = (hrp.Position - Camera.CFrame.Position).Unit
				local dot = dir:Dot(Camera.CFrame.LookVector)
				if dot > 0.5 then
					box.Color = corDentroVisao
				else
					box.Color = corForaVisao
				end
			else
				box.Visible = false
			end
		else
			box.Visible = false
		end
	end
end)
