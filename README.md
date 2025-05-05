-- LocalScript dentro de LockButton

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer
local LockButton = script.Parent
local LockedTarget = nil
local IsLockedOn = false

-- Criar indicador visual
local lockGui = Instance.new("BillboardGui")
lockGui.Name = "LockIndicator"
lockGui.Size = UDim2.new(0, 50, 0, 50)
lockGui.StudsOffset = Vector3.new(0, 2.5, 0)
lockGui.AlwaysOnTop = true

local indicator = Instance.new("ImageLabel", lockGui)
indicator.Size = UDim2.new(1, 0, 1, 0)
indicator.BackgroundTransparency = 1
indicator.Image = "rbxassetid://6031091002" -- Ícone de alvo
indicator.ImageColor3 = Color3.fromRGB(255, 50, 50)

-- Função para encontrar inimigo mais próximo
local function getClosestTarget()
	local closest = nil
	local minDistance = 100 -- pixels
	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
			local head = player.Character.Head
			local screenPos, onScreen = Camera:WorldToViewportPoint(head.Position)

			if onScreen then
				local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
				local dist = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
				if dist < minDistance then
					minDistance = dist
					closest = head
				end
			end
		end
	end
	return closest
end

-- Clicar no botão ativa/desativa
LockButton.MouseButton1Click:Connect(function()
	if IsLockedOn then
		IsLockedOn = false
		if lockGui.Parent then lockGui.Parent = nil end
		LockedTarget = nil
		LockButton.Text = "🔒"
	else
		local target = getClosestTarget()
		if target then
			LockedTarget = target
			IsLockedOn = true
			lockGui.Parent = target
			LockButton.Text = "🔓"
		end
	end
end)

-- Atualizar câmera travada
RunService.RenderStepped:Connect(function()
	if IsLockedOn and LockedTarget and LockedTarget.Parent then
		local camPos = Camera.CFrame.Position
		local targetPos = LockedTarget.Position
		Camera.CFrame = CFrame.new(camPos, targetPos)
	else
		IsLockedOn = false
		if lockGui.Parent then lockGui.Parent = nil end
		LockButton.Text = "🔒"
	end
end)
