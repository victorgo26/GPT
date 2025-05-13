-- Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações padrões
local FOV_RADIUS = 100
local AIMBOT_MAX_DIST = 1000
local ESP_ATIVADO = true
local AIMBOT_ATIVADO = true
local MOUSE_AIMBOT_KEY = Enum.UserInputType.MouseButton2
local corDentroVisao = Color3.fromRGB(255, 255, 0)
local corForaVisao = Color3.fromRGB(255, 0, 0)
local corFOV = Color3.fromRGB(0, 255, 255)

-- Drawing
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Visible = true
fovCircle.Color = corFOV

local caixasPorJogador = {}

-- GUI
local statusText = Drawing.new("Text")
statusText.Size = 18
statusText.Color = Color3.fromRGB(255, 255, 255)
statusText.Position = Vector2.new(10, 10)
statusText.Outline = true
statusText.Visible = true

-- Criar Caixa
local function criarCaixa(player)
    if player == LocalPlayer then return end
    if caixasPorJogador[player] then return end

    local box = Drawing.new("Square")
    box.Thickness = 2
    box.Filled = false
    box.Visible = false
    caixasPorJogador[player] = box
end

-- Remover Caixa
local function removerCaixa(player)
    if caixasPorJogador[player] then
        caixasPorJogador[player]:Remove()
        caixasPorJogador[player] = nil
    end
end

for _, player in ipairs(Players:GetPlayers()) do criarCaixa(player) end
Players.PlayerAdded:Connect(criarCaixa)
Players.PlayerRemoving:Connect(removerCaixa)

-- Checar inimigo
local function ehInimigo(player)
    if not player.Team or not LocalPlayer.Team then return true end
    return player.Team ~= LocalPlayer.Team
end

-- Aimbot
local mirando = false
UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == MOUSE_AIMBOT_KEY then mirando = true end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == MOUSE_AIMBOT_KEY then mirando = false end
end)

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

-- Rayfield UI
loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Rayfield/main/source"))()

local Window = Rayfield:CreateWindow({Name = "Aimbot & ESP", ConfigurationSaving = {Enabled = false}})

local MainTab = Window:CreateTab("Config", 4483362458)

MainTab:CreateToggle({
    Name = "ESP",
    CurrentValue = ESP_ATIVADO,
    Callback = function(v) ESP_ATIVADO = v end
})

MainTab:CreateToggle({
    Name = "Aimbot",
    CurrentValue = AIMBOT_ATIVADO,
    Callback = function(v) AIMBOT_ATIVADO = v end
})

MainTab:CreateSlider({
    Name = "FOV Radius",
    Range = {50, 300},
    Increment = 1,
    CurrentValue = FOV_RADIUS,
    Callback = function(v) FOV_RADIUS = v fovCircle.Radius = v end
})

MainTab:CreateDropdown({
    Name = "Tecla do Aimbot",
    Options = {"MouseButton2", "MouseButton1", "E", "Q"},
    CurrentOption = "MouseButton2",
    Callback = function(v)
        local inputMap = {
            ["MouseButton1"] = Enum.UserInputType.MouseButton1,
            ["MouseButton2"] = Enum.UserInputType.MouseButton2,
            ["E"] = Enum.KeyCode.E,
            ["Q"] = Enum.KeyCode.Q
        }
        MOUSE_AIMBOT_KEY = inputMap[v]
    end
})

MainTab:CreateColorPicker({
    Name = "Cor Visível (ESP)",
    Color = corDentroVisao,
    Callback = function(c) corDentroVisao = c end
})

MainTab:CreateColorPicker({
    Name = "Cor Oculto (ESP)",
    Color = corForaVisao,
    Callback = function(c) corForaVisao = c end
})

MainTab:CreateColorPicker({
    Name = "Cor do FOV",
    Color = corFOV,
    Callback = function(c) corFOV = c fovCircle.Color = c end
})

-- Loop
RunService.RenderStepped:Connect(function()
    fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    statusText.Text = "ESP: " .. (ESP_ATIVADO and "ON" or "OFF") .. " | Aimbot: " .. (AIMBOT_ATIVADO and "ON" or "OFF")

    -- Aimbot
    if mirando and AIMBOT_ATIVADO then
        local alvo = getAlvoMaisProximo()
        if alvo then
            local dir = (alvo.Position - Camera.CFrame.Position).Unit
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + dir)
        end
    end

    -- ESP
    for player, box in pairs(caixasPorJogador) do
        local char = player.Character
        if not char or not char:FindFirstChild("HumanoidRootPart") or not ehInimigo(player) then
            box.Visible = false
            continue
        end

        local hrp = char.HumanoidRootPart
        local min = hrp.Position - Vector3.new(2, 3, 0)
        local max = hrp.Position + Vector3.new(2, 3, 0)

        local p1, v1 = Camera:WorldToViewportPoint(min)
        local p2, v2 = Camera:WorldToViewportPoint(max)

        if (v1 and v2) and ESP_ATIVADO then
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
