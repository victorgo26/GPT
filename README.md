local function criarDesenhos(player)
	if player == LocalPlayer then return end
	if desenhosPorJogador[player] then return end

	local box = Drawing.new("Square") -- Caixa principal
	box.Thickness = 1.5
	box.Filled = false
	box.Visible = false
	box.ZIndex = 2

	local outline = Drawing.new("Square") -- Contorno
	outline.Thickness = 4
	outline.Filled = false
	outline.Color = Color3.fromRGB(255, 0, 0)
	outline.Visible = false
	outline.ZIndex = 1

	desenhosPorJogador[player] = {box = box, outline = outline}
end

RunService.RenderStepped:Connect(function()
	for player, caixas in pairs(desenhosPorJogador) do
		local char = player.Character
		if not (char and char:FindFirstChild("HumanoidRootPart") and ehInimigo(player)) then
			caixas.box.Visible = false
			caixas.outline.Visible = false
			continue
		end

		local partesVisiveis = {}
		for _, parte in ipairs(char:GetChildren()) do
			if parte:IsA("BasePart") then
				local pos, visivel = Camera:WorldToViewportPoint(parte.Position)
				if visivel then
					table.insert(partesVisiveis, pos)
				end
			end
		end

		if #partesVisiveis == 0 or not espAtivado then
			caixas.box.Visible = false
			caixas.outline.Visible = false
			continue
		end

		local minX, minY = math.huge, math.huge
		local maxX, maxY = -math.huge, -math.huge

		for _, pos in ipairs(partesVisiveis) do
			minX = math.min(minX, pos.X)
			minY = math.min(minY, pos.Y)
			maxX = math.max(maxX, pos.X)
			maxY = math.max(maxY, pos.Y)
		end

		local width = maxX - minX
		local height = maxY - minY
		local pos = Vector2.new(minX, minY)

		caixas.outline.Position = pos
		caixas.outline.Size = Vector2.new(width, height)
		caixas.outline.Visible = true

		caixas.box.Position = pos
		caixas.box.Size = Vector2.new(width, height)
		caixas.box.Color = Color3.fromRGB(255, 255, 255) -- Branco por cima
		caixas.box.Visible = true
	end
end)
