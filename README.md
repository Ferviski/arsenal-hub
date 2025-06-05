if getgenv().ArsenalHubLoaded then return end
getgenv().ArsenalHubLoaded = true

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local UIS = game:GetService("UserInputService")

-- Estados
local Aimbot = false
local ESP = false
local MouseRightDown = false

-- GUI
local GUI = Instance.new("ScreenGui", game.CoreGui)
GUI.Name = "ArsenalHubGUI"

-- Painel principal
local Frame = Instance.new("Frame", GUI)
Frame.Size = UDim2.new(0, 180, 0, 120)
Frame.Position = UDim2.new(0, 60, 0, 100)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Visible = false
Instance.new("UICorner", Frame)

-- Botões do painel
local function CreateButton(text, yOffset, callback)
	local btn = Instance.new("TextButton", Frame)
	btn.Size = UDim2.new(0.9, 0, 0, 30)
	btn.Position = UDim2.new(0.05, 0, 0, yOffset)
	btn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	btn.TextColor3 = Color3.new(1, 1, 1)
	btn.Text = text
	btn.Font = Enum.Font.Gotham
	btn.TextSize = 14
	btn.MouseButton1Click:Connect(callback)
	return btn
end

CreateButton("Toggle Aimbot", 10, function() Aimbot = not Aimbot end)
CreateButton("Toggle ESP", 50, function() ESP = not ESP end)

-- Bola flutuante
local ToggleBall = Instance.new("TextButton", GUI)
ToggleBall.Size = UDim2.new(0, 40, 0, 40)
ToggleBall.Position = UDim2.new(0, 10, 0, 90)
ToggleBall.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
ToggleBall.Text = "☰"
ToggleBall.TextSize = 22
ToggleBall.Font = Enum.Font.GothamBold
ToggleBall.TextColor3 = Color3.new(1, 1, 1)
ToggleBall.AutoButtonColor = true
Instance.new("UICorner", ToggleBall)

ToggleBall.MouseButton1Click:Connect(function()
	Frame.Visible = not Frame.Visible
end)

-- Mouse botão direito detectado
UIS.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		MouseRightDown = true
	end
end)

UIS.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton2 then
		MouseRightDown = false
	end
end)

-- ESP inimigos
local function addESP(player)
	if player == LocalPlayer then return end

	local box = Drawing.new("Square")
	box.Color = Color3.fromRGB(255, 0, 0)
	box.Thickness = 2
	box.Filled = false
	box.Visible = false

	local name = Drawing.new("Text")
	name.Color = Color3.fromRGB(255, 0, 0)
	name.Size = 14
	name.Outline = true
	name.Center = true
	name.Visible = false

	RunService.RenderStepped:Connect(function()
		local char = player.Character
		if ESP and char and char:FindFirstChild("HumanoidRootPart") and player.Team ~= LocalPlayer.Team then
			local hrp = char.HumanoidRootPart
			local pos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
			if onScreen then
				local scale = (Camera:WorldToViewportPoint(hrp.Position + Vector3.new(0, 3, 0)).Y -
							   Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 2.5, 0)).Y)
				box.Size = Vector2.new(scale * 0.6, scale)
				box.Position = Vector2.new(pos.X - box.Size.X/2, pos.Y - box.Size.Y/2)
				box.Visible = true

				name.Text = player.Name
				name.Position = Vector2.new(pos.X, pos.Y - box.Size.Y/2 - 15)
				name.Visible = true
			else
				box.Visible = false
				name.Visible = false
			end
		else
			box.Visible = false
			name.Visible = false
		end
	end)
end

for _, p in pairs(Players:GetPlayers()) do addESP(p) end
Players.PlayerAdded:Connect(addESP)

-- Aimbot
RunService.RenderStepped:Connect(function()
	if not Aimbot or not MouseRightDown then return end

	local closest = nil
	local shortest = math.huge
	local mousePos = UIS:GetMouseLocation()

	for _, p in pairs(Players:GetPlayers()) do
		if p ~= LocalPlayer and p.Team ~= LocalPlayer.Team and p.Character and p.Character:FindFirstChild("Head") then
			local head = p.Character.Head
			local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
			if onScreen then
				local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
				if dist < shortest then
					shortest = dist
					closest = head
				end
			end
		end
	end

	if closest then
		Camera.CFrame = CFrame.new(Camera.CFrame.Position, closest.Position)
	end
end)
