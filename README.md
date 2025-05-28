loadstring(game:HttpGet("https://raw.githubusercontent.com/Snxdfer/back-ups-for-libs/refs/heads/main/RedzUI.lua"))()

-- Cria a janela principal
MakeWindow({
    Hub = {
        Title = "Dragon Menu - Ilha Bela",
        Animation = "by : Vito0296poq"
    },
    
   Key = {
        KeySystem = false, -- Ativa o sistema de Key
        Title = "Sistema de Chave",
        Description = "Digite a chave correta para continuar.",
        KeyLink = "https://seusite.com/chave", -- Link para obter a chave (opcional)
        Keys = {"1234", "chave-extra"}, -- Chaves válidas
        Notifi = {
            Notifications = true,
            CorrectKey = "Chave correta! Iniciando script...",
            Incorrectkey = "Chave incorreta, tente novamente.",
            CopyKeyLink = "Link copiado!"
        }
    }
})

-- Botão de minimizar
MinimizeButton({
    Image = "rbxassetid://1234567890",
    Size = {40, 40},
    Color = Color3.fromRGB(10, 10, 10),
    Corner = true,
    Stroke = false,
    StrokeColor = Color3.fromRGB(255, 0, 0)
})

-- Criação da aba principal
local Main = MakeTab({Name = "Jogador"})
local Visuais = MakeTab({Name = "Visuais"})
local Player = MakeTab({Name = "Auxílios"})


-- Notificação inicial
-- Removido se não quiser notificação:
-- MakeNotifi({
--     Title = "Dragon Menu",
--     Text = "Carregando com sucesso!",
--     Time = 5
-- })

-- Noclip
local noclipConnection

function toggleNoclip(enable)
    if enable then
        if not noclipConnection then
            noclipConnection = game:GetService("RunService").Stepped:Connect(function()
                for _, part in pairs(game.Players.LocalPlayer.Character:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.CanCollide = false
                    end
                end
            end)
        end
    else
        if noclipConnection then
            noclipConnection:Disconnect()
            noclipConnection = nil
        end
        for _, part in pairs(game.Players.LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end

-- Toggle para ativar/desativar colisão
AddToggle(Main, {
    Name = "Desabilitar Colisões",
    Default = false,
    Callback = function(Value)
        toggleNoclip(Value)
    end
})

-- Infinite Jump
local jumpConnection
local function toggleInfiniteJump(enable)
    if enable then
        if not jumpConnection then
            jumpConnection = game:GetService("UserInputService").JumpRequest:Connect(function()
                local player = game.Players.LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoid = character:FindFirstChildOfClass("Humanoid")
                if humanoid then
                    humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
                end
            end)
        end
    else
        if jumpConnection then
            jumpConnection:Disconnect()
            jumpConnection = nil
        end
    end
end

-- Toggle para ativar/desativar pulo infinito
local Toggle = AddToggle(Main, {
    Name = "Pulos infinito",
    Default = false,
    Callback = function(Value)
        toggleInfiniteJump(Value)
    end
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local velocidadeValor = 25
local velocidadeAtivada = false
local loopVelocidade = nil

-- Spoof de WalkSpeed
local spoofedSpeed = 16

-- Hook metatable
local mt = getrawmetatable(game)
setreadonly(mt, false)

local oldIndex = mt.__index
local oldNewIndex = mt.__newindex

mt.__index = function(t, k)
    if t:IsA("Humanoid") and k == "WalkSpeed" then
        return spoofedSpeed
    end
    return oldIndex(t, k)
end

mt.__newindex = function(t, k, v)
    if t:IsA("Humanoid") and k == "WalkSpeed" then
        -- bloqueia scripts externos de mudar o valor
        if not checkcaller() then
            return
        end
        spoofedSpeed = v
    end
    return oldNewIndex(t, k, v)
end

-- Função para aplicar a velocidade
local function aplicarVelocidade()
    if loopVelocidade then loopVelocidade:Disconnect() end

    loopVelocidade = RunService.Heartbeat:Connect(function()
        if velocidadeAtivada then
            local char = LocalPlayer.Character
            local humanoid = char and char:FindFirstChildWhichIsA("Humanoid")
            if humanoid and humanoid.WalkSpeed ~= velocidadeValor then
                spoofedSpeed = velocidadeValor
                humanoid.WalkSpeed = velocidadeValor
            end
        end
    end)
end

-- Interface
AddSlider(Main, {
    Name = "Velocidade",
    MinValue = 16,
    MaxValue = 180,
    Default = 16,
    Increase = 1,
    Callback = function(Value)
        velocidadeValor = Value
    end
})

AddToggle(Main, {
    Name = "Velocidade (Bypass)",
    Default = false,
    Callback = function(Value)
        velocidadeAtivada = Value

        if Value then
            aplicarVelocidade()
        else
            if loopVelocidade then
                loopVelocidade:Disconnect()
                loopVelocidade = nil
            end
            local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            local humanoid = char:FindFirstChildWhichIsA("Humanoid")
            if humanoid then
                spoofedSpeed = 16
                humanoid.WalkSpeed = 16
            end
        end
    end
})

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local jogadorSelecionado = nil
local observando = false
local observarConnection = nil

-- Função para observar o jogador
local function observarJogador(jogador)
	if observarConnection then
		observarConnection:Disconnect()
	end

	local function setarCamera()
		if jogador.Character and jogador.Character:FindFirstChild("Humanoid") then
			workspace.CurrentCamera.CameraSubject = jogador.Character.Humanoid
			print("Observando: " .. jogador.Name)
		end
	end

	setarCamera()

	observarConnection = jogador.CharacterAdded:Connect(function()
		task.wait(1)
		if observando then
			setarCamera()
		end
	end)
end

-- Função para parar de observar
local function pararObservar()
	if observarConnection then
		observarConnection:Disconnect()
		observarConnection = nil
	end
	observando = false

	if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
		workspace.CurrentCamera.CameraSubject = LocalPlayer.Character.Humanoid
	end

	print("Observação desativada.")
end

-- Caixa de texto para digitar o nome do jogador
AddTextbox(Main, {
	Name = "Digitar ó nome do Jogador",
	Default = "",
	TextDisappear = false,
	Callback = function(nome)
		local player = Players:FindFirstChild(nome)
		if player and player ~= LocalPlayer then
			jogadorSelecionado = player
			print("Jogador selecionado: " .. player.Name)
		else
			warn("Jogador não encontrado ou é você.")
			jogadorSelecionado = nil
		end
	end
})

-- Toggle para ativar/desativar observação
AddToggle(Main, {
	Name = "Observar Jogador",
	Default = false,
	Callback = function(Value)
		if Value then
			if jogadorSelecionado then
				observando = true
				observarJogador(jogadorSelecionado)
			else
				warn("Digite um nome de jogador válido primeiro.")
			end
		else
			pararObservar()
		end
	end
})

-- Parar observação se o jogador sair
Players.PlayerRemoving:Connect(function(player)
	if jogadorSelecionado == player then
		pararObservar()
		jogadorSelecionado = nil
	end
end)

-- Variáveis de controle
local espAtivado = false
local connections = {}

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Função para criar ESP
local function criarESP(player)
    if player == LocalPlayer then return end

    task.spawn(function()
        while espAtivado do
            local char = player.Character
            local head = char and char:FindFirstChild("Head")
            local humanoid = char and char:FindFirstChild("Humanoid")

            if char and head and humanoid and humanoid.Health > 0 then
                local esp = head:FindFirstChild("ESP")
                if not esp then
                    esp = Instance.new("BillboardGui")
                    esp.Name = "ESP"
                    esp.Adornee = head
                    esp.Size = UDim2.new(0, 70, 0, 18) -- menor tamanho
                    esp.StudsOffset = Vector3.new(0, 1.3, 0) -- mais próximo da cabeça
                    esp.AlwaysOnTop = true

                    local text = Instance.new("TextLabel")
                    text.Name = "Texto"
                    text.Size = UDim2.new(1, 0, 1, 0)
                    text.BackgroundTransparency = 1
                    text.TextColor3 = Color3.fromRGB(255, 255, 255)
                    text.TextSize = 10 -- menor texto
                    text.TextScaled = false
                    text.Font = Enum.Font.Gotham
                    text.TextStrokeTransparency = 0.4
                    text.TextStrokeColor3 = Color3.new(0, 0, 0)
                    text.Parent = esp

                    esp.Parent = head

                    humanoid.Died:Connect(function()
                        if esp then esp:Destroy() end
                    end)
                end

                local texto = esp:FindFirstChild("Texto")
                if texto and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("HumanoidRootPart") then
                    local distancia = (LocalPlayer.Character.HumanoidRootPart.Position - char.HumanoidRootPart.Position).Magnitude
                    texto.Text = player.Name .. " - " .. math.floor(distancia) .. "m"
                end
            end
            wait(0.3)
        end
    end)
end

-- Monitorar jogadores
local function monitorarPlayer(player)
    if connections[player] then
        connections[player]:Disconnect()
    end

    connections[player] = player.CharacterAdded:Connect(function()
        wait(1)
        if espAtivado then
            criarESP(player)
        end
    end)

    criarESP(player)
end

-- Toggle para ativar/desativar o ESP
AddToggle(Visuais, {
    Name = "ESP Nome",
    Default = false,
    Callback = function(Value)
        espAtivado = Value

        if espAtivado then
            for _, player in ipairs(Players:GetPlayers()) do
                monitorarPlayer(player)
            end

            connections["PlayerAdded"] = Players.PlayerAdded:Connect(function(player)
                monitorarPlayer(player)
            end)
        else
            for _, player in ipairs(Players:GetPlayers()) do
                local char = player.Character
                if char then
                    local head = char:FindFirstChild("Head")
                    if head then
                        local esp = head:FindFirstChild("ESP")
                        if esp then
                            esp:Destroy()
                        end
                    end
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
    end
})

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local espTags = {}
local espConnections = {}
local espAtivo = false

-- Cria o ESP acima do jogador se for STAFF
local function createESP(player)
	local character = player.Character
	if not character then return end

	local head = character:FindFirstChild("Head")
	if not head then return end

	-- Verifica se já existe ESP
	if espTags[player] then return end

	-- Verifica se existe "staff" em algum TextLabel
	local hasStaffTag = false
	for _, gui in pairs(head:GetChildren()) do
		if gui:IsA("BillboardGui") then
			for _, label in pairs(gui:GetChildren()) do
				if label:IsA("TextLabel") and string.find(label.Text:lower(), "staff") then
					hasStaffTag = true
					break
				end
			end
		end
	end

	if not hasStaffTag then return end

	-- Criar o Billboard ESP
	local billboard = Instance.new("BillboardGui")
	billboard.Name = "ESP_Staff"
	billboard.Adornee = head
	billboard.Size = UDim2.new(0, 100, 0, 20)
	billboard.StudsOffset = Vector3.new(0, 2, 0)
	billboard.AlwaysOnTop = true

	local text = Instance.new("TextLabel")
	text.Size = UDim2.new(1, 0, 1, 0)
	text.BackgroundTransparency = 1
	text.Text = "STAFF"
	text.TextColor3 = Color3.fromRGB(255, 0, 0)
	text.TextStrokeTransparency = 0
	text.Font = Enum.Font.SourceSansBold
	text.TextScaled = true
	text.Parent = billboard

	billboard.Parent = head
	espTags[player] = billboard
end

-- Remove todos os ESPs e desconecta eventos
local function removeAllESP()
	for player, gui in pairs(espTags) do
		if gui and gui.Parent then
			gui:Destroy()
		end
	end
	espTags = {}

	for _, conn in pairs(espConnections) do
		if conn then conn:Disconnect() end
	end
	espConnections = {}
end

-- Ativa ou desativa o ESP
local function toggleEsp(enabled)
	removeAllESP()
	espAtivo = enabled

	if not enabled then return end

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer then
			-- Cria ESP se personagem já existe
			if player.Character then
				createESP(player)
			end

			-- Conecta para novos personagens
			local conn = player.CharacterAdded:Connect(function()
				wait(1)
				if espAtivo then
					createESP(player)
				end
			end)
			table.insert(espConnections, conn)
		end
	end

	-- Detecta novos jogadores
	local conn = Players.PlayerAdded:Connect(function(player)
		local sub = player.CharacterAdded:Connect(function()
			wait(1)
			if espAtivo then
				createESP(player)
			end
		end)
		table.insert(espConnections, sub)
	end)
	table.insert(espConnections, conn)
end

-- Botão na interface
AddToggle(Visuais, {
	Name = "ESP STAFF",
	Default = false,
	Callback = function(Value)
		toggleEsp(Value)
	end
})

AddButton(Player, {
    Name = "Fly GUI Car",
    Callback = function()
        print("Botão foi clicado!")
        pcall(function()
            loadstring(game:HttpGet('https://raw.githubusercontent.com/ScpGuest666/Random-Roblox-script/refs/heads/main/Roblox%20Vehicle%20Fly%20Gui%20script'))()
        end)
    end
})
