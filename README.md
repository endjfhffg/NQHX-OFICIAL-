-- Check if already running
if getgenv().ESP_RUNNING then
    -- If already running, clean up and stop
    getgenv().ESP_RUNNING = false
    if getgenv().ESP_DRAWINGS then
        for _, d in pairs(getgenv().ESP_DRAWINGS) do
            if d.box then d.box:Remove() end
            if d.name then d.name:Remove() end
            if d.dist then d.dist:Remove() end
        end
    end
    if getgenv().ESP_BUTTON then
        getgenv().ESP_BUTTON:Destroy()
    end
    if getgenv().ESP_GUI then
        getgenv().ESP_GUI:Destroy()
    end
    print("[ESP] Desligado")
    return
end

-- Iniciar
getgenv().ESP_RUNNING = true
getgenv().ESP_DRAWINGS = {}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local ESP_ACTIVE = false

-- Criar botão
local button = Instance.new("TextButton")
button.Size = UDim2.new(0, 120, 0, 40)
button.Position = UDim2.new(0.5, -60, 0.9, 0)
button.Text = "Toggle ESP"
button.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
button.TextColor3 = Color3.new(1, 1, 1)
button.BorderSizePixel = 0
button.AutoButtonColor = true

local screenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
screenGui.ResetOnSpawn = false
button.Parent = screenGui

getgenv().ESP_GUI = screenGui
getgenv().ESP_BUTTON = button

-- Criar ESP para cada player
local function createDrawing(player)
	local box = Drawing.new("Square")
	box.Color = Color3.new(1, 0, 0)
	box.Thickness = 2
	box.Filled = false
	box.Visible = false

	local nameTag = Drawing.new("Text")
	nameTag.Size = 14
	nameTag.Color = Color3.new(1, 1, 1)
	nameTag.Outline = true
	nameTag.Center = true
	nameTag.Visible = false

	local distance = Drawing.new("Text")
	distance.Size = 13
	distance.Color = Color3.new(0, 1, 1)
	distance.Outline = true
	distance.Center = true
	distance.Visible = false

	getgenv().ESP_DRAWINGS[player] = {box = box, name = nameTag, dist = distance}
end

-- Atualizar em tempo real
RunService.RenderStepped:Connect(function()
	if not ESP_ACTIVE or not getgenv().ESP_RUNNING then
		for _, d in pairs(getgenv().ESP_DRAWINGS) do
			d.box.Visible = false
			d.name.Visible = false
			d.dist.Visible = false
		end
		return
	end

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
			local hrp = player.Character.HumanoidRootPart
			local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

			local d = getgenv().ESP_DRAWINGS[player]
			if not d then
				createDrawing(player)
				d = getgenv().ESP_DRAWINGS[player]
			end

			if onScreen then
				local scale = math.clamp((Camera.CFrame.Position - hrp.Position).Magnitude / 100, 0.5, 3)
				local size = Vector2.new(50 / scale, 100 / scale)
				local boxPos = Vector2.new(pos.X - size.X / 2, pos.Y - size.Y / 2)

				d.box.Size = size
				d.box.Position = boxPos
				d.box.Visible = true

				d.name.Text = player.Name
				d.name.Position = Vector2.new(pos.X, boxPos.Y - 15)
				d.name.Visible = true

				d.dist.Text = string.format("%.0f studs", (hrp.Position - Camera.CFrame.Position).Magnitude)
				d.dist.Position = Vector2.new(pos.X, boxPos.Y + size.Y + 5)
				d.dist.Visible = true
			else
				d.box.Visible = false
				d.name.Visible = false
				d.dist.Visible = false
			end
		end
	end
end)

-- Botão toggle
button.MouseButton1Click:Connect(function()
	ESP_ACTIVE = not ESP_ACTIVE
	button.Text = ESP_ACTIVE and "ESP ON" or "ESP OFF"
end)

-- Criar ESP para todos
for _, player in pairs(Players:GetPlayers()) do
	if player ~= LocalPlayer then
		createDrawing(player)
	end
end

Players.PlayerAdded:Connect(function(player)
	if player ~= LocalPlayer then
		createDrawing(player)
	end
end)

print("[ESP] Ligado")
