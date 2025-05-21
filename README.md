-- Dragon Menu | Ilha Bela v1
debugX = true

-- Carrega Rayfield UI
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Janela principal
local Window = Rayfield:CreateWindow({
	Name = "Dragon Menu | Ilha Bela v1",
	Icon = 0,
	LoadingTitle = "Carregando...",
	LoadingSubtitle = "by Vitor",
	Theme = "Default",
	ConfigurationSaving = {
		Enabled = true,
		FileName = "DragonMenu"
	},
	KeySystem = true,
	KeySettings = {
		Title = "Dragon Menu",
		Subtitle = "Key System",
		Note = "Key: #####",
		FileName = "SiriusKey",
		SaveKey = true,
		Key = "Key_ilhabela"
	}
})

-- Gui Tabs
local mainTab = Window:CreateTab("Main", 4483362458)
local playerTab = Window:CreateTab("Player", 6034996698)

--------------------------------
-- FUNCIONALIDADES MAIN TAB  --
--------------------------------

-- Noclip
mainTab:CreateToggle({
	Name = "Noclip",
	CurrentValue = false,
	Flag = "Toggle_Noclip",
	Callback = function(enabled)
		local player = game.Players.LocalPlayer
		local character = player.Character or player.CharacterAdded:Wait()
		if _G.NoclipConnection then _G.NoclipConnection:Disconnect() end
		if enabled then
			_G.NoclipConnection = game:GetService("RunService").Stepped:Connect(function()
				for _, part in ipairs(character:GetDescendants()) do
					if part:IsA("BasePart") then part.CanCollide = false end
				end
			end)
		else
			_G.NoclipConnection = nil
		end
	end,
})

-- Infinite Jump
local infiniteJumpConnection
mainTab:CreateToggle({
	Name = "Infinite Jump",
	CurrentValue = false,
	Flag = "Toggle_InfiniteJump",
	Callback = function(enabled)
		if enabled then
			infiniteJumpConnection = game:GetService("UserInputService").JumpRequest:Connect(function()
				local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
				if humanoid then humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
			end)
		elseif infiniteJumpConnection then
			infiniteJumpConnection:Disconnect()
			infiniteJumpConnection = nil
		end
	end,
})

-- WalkSpeed
local velocidadeAtivada = false
local velocidadeAtual = 16
mainTab:CreateSlider({
	Name = "WalkSpeed",
	Range = {16, 250},
	Increment = 10,
	Suffix = "Velocidade",
	CurrentValue = 16,
	Flag = "Slider_Walkspeed",
	Callback = function(v)
		velocidadeAtual = v
		if velocidadeAtivada then
			local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
			if humanoid then humanoid.WalkSpeed = v end
		end
	end,
})
mainTab:CreateToggle({
	Name = "Ativar WalkSpeed",
	CurrentValue = false,
	Flag = "Toggle_AtivarVelocidade",
	Callback = function(enabled)
		velocidadeAtivada = enabled
		local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
		if humanoid then humanoid.WalkSpeed = enabled and velocidadeAtual or 16 end
	end,
})

-- JumpPower
local puloAtivado = false
local puloAtual = 50
mainTab:CreateSlider({
	Name = "JumpPower",
	Range = {50, 500},
	Increment = 10,
	Suffix = "Pulo",
	CurrentValue = 50,
	Flag = "Slider_JumpPower",
	Callback = function(v)
		puloAtual = v
		if puloAtivado then
			local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
			if humanoid then humanoid.JumpPower = v end
		end
	end,
})
mainTab:CreateToggle({
	Name = "Ativar JumpPower",
	CurrentValue = false,
	Flag = "Toggle_AtivarPulo",
	Callback = function(enabled)
		puloAtivado = enabled
		local humanoid = game.Players.LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
		if humanoid then humanoid.JumpPower = enabled and puloAtual or 50 end
	end,
})

----------------------------
-- ESP (JOGADORES)        --
----------------------------

local espAtivado = false
local connections = {}
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

playerTab:CreateToggle({
	Name = "ESP (Nomes)",
	CurrentValue = false,
	Flag = "Toggle_ESP",
	Callback = function(enabled)
		espAtivado = enabled

		local function criarESP(player)
			if player == LocalPlayer or not espAtivado then return end
			task.spawn(function()
				while espAtivado and player and player.Character do
					local char = player.Character
					local head = char:FindFirstChild("Head")
					local humanoid = char:FindFirstChild("Humanoid")

					if head and humanoid and humanoid.Health > 0 then
						local esp = head:FindFirstChild("ESP")
						if not esp then
							esp = Instance.new("BillboardGui", head)
							esp.Name = "ESP"
							esp.Adornee = head
							esp.Size = UDim2.new(0, 150, 0, 60)
							esp.StudsOffset = Vector3.new(0, 2.5, 0)
							esp.AlwaysOnTop = true

							local nome = Instance.new("TextLabel", esp)
							nome.Name = "Nome"
							nome.Size = UDim2.new(1, 0, 0.5, 0)
							nome.Position = UDim2.new(0, 0, 0, 0)
							nome.BackgroundTransparency = 1
							nome.TextColor3 = Color3.fromRGB(255, 255, 255)
							nome.TextSize = 13
							nome.Font = Enum.Font.GothamSemibold
							nome.TextStrokeTransparency = 0.4
							nome.TextStrokeColor3 = Color3.new(0, 0, 0)

							local distanciaTxt = Instance.new("TextLabel", esp)
							distanciaTxt.Name = "Distancia"
							distanciaTxt.Size = UDim2.new(1, 0, 0.5, 0)
							distanciaTxt.Position = UDim2.new(0, 0, 0.5, 0)
							distanciaTxt.BackgroundTransparency = 1
							distanciaTxt.TextColor3 = Color3.fromRGB(255, 255, 255)
							distanciaTxt.TextSize = 12
							distanciaTxt.Font = Enum.Font.Gotham
							distanciaTxt.TextTransparency = 0.3
							distanciaTxt.TextStrokeTransparency = 1

							humanoid.Died:Connect(function()
								if esp then esp:Destroy() end
							end)
						end

						local nome = head:FindFirstChild("ESP"):FindFirstChild("Nome")
						local distanciaTxt = head:FindFirstChild("ESP"):FindFirstChild("Distancia")
						local root1 = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
						local root2 = char:FindFirstChild("HumanoidRootPart")

						if nome and distanciaTxt and root1 and root2 then
							local distancia = (root1.Position - root2.Position).Magnitude
							nome.Text = player.Name
							distanciaTxt.Text = "[" .. math.floor(distancia) .. "m]"
						end
					end
					task.wait(0.3)
				end
			end)
		end

		local function monitorarPlayer(player)
			if connections[player] then connections[player]:Disconnect() end
			connections[player] = player.CharacterAdded:Connect(function()
				task.wait(1)
				if espAtivado then criarESP(player) end
			end)
			if player.Character then criarESP(player) end
		end

		if espAtivado then
			for _, player in ipairs(Players:GetPlayers()) do
				monitorarPlayer(player)
			end
			connections["PlayerAdded"] = Players.PlayerAdded:Connect(monitorarPlayer)
		else
			for _, player in ipairs(Players:GetPlayers()) do
				local head = player.Character and player.Character:FindFirstChild("Head")
				if head then
					local esp = head:FindFirstChild("ESP")
					if esp then esp:Destroy() end
				end
				if connections[player] then
					connections[player]:Disconnect()
					connections[player] = nil
				end
			end
			if connections["PlayerAdded"] then
				connections["PlayerAdded"]:Disconnect()
				connections["PlayerAdded"] = nil
			end
		end
	end,
})

-- Carregar configurações salvas
Rayfield:LoadConfiguration()
