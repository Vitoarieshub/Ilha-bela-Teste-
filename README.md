-- Carregar Fluent
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

-- Função para enviar notificações
local function notify(title, content)
    Fluent:Notify({
        Title = title,
        Content = content,
        Duration = 3 -- Define a duração da notificação para 3 segundos
    })
end

-- Aviso ao executar
notify("Executado com Sucesso!", "Seja bem vindo.")

-- Criar a janela principal
local Window = Fluent:CreateWindow({
    Title = "Dragon Menu | Universal legít  " .. Fluent.Version,
    TabWidth = 90,
    Size = UDim2.fromOffset(370, 300),
    Theme = "Dark"
})

-- Tabela de abas
local Tabs = {
    Main = Window:AddTab({ Title = "Ínício" }),
    Visual = Window:AddTab({ Title = "Visual" }),
    Players = Window:AddTab({ Title = "Jogadores" }),
}

-- Funções utilitárias
local function setHumanoidProperty(property, value)
    local player = game.Players.LocalPlayer
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")
    humanoid[property] = value
    print(property .. " ajustado para:", value)
end

-- Infinite Jump
local jumpConnection
local function toggleInfiniteJump(enable)
    if enable then
        if not jumpConnection then
            jumpConnection = game:GetService("UserInputService").JumpRequest:Connect(function()
                local player = game.Players.LocalPlayer
                local character = player.Character or player.CharacterAdded:Wait()
                local humanoid = character:WaitForChild("Humanoid")
                humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end)
        end
    elseif jumpConnection then
        jumpConnection:Disconnect()
        jumpConnection = nil
    end
end

-- Noclip
local noclipConnection
local function toggleNoclip(enable)
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
        notify("Travessa Paredes", "Você pode travessar paredes agora!")
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
        notify("Travessa Paredes", "Foi Desativado.")
    end
end

-- Aba: Início
Tabs.Main:AddParagraph({ Title = "Programador Victor", Content = "Scripts personalizados" })

Tabs.Main:AddButton({
    Title = "Fly",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/Vitoarieshub/Fly-universal-/refs/heads/main/README.md"))()
    end
})

Tabs.Main:AddToggle("noclip", {
    Title = "Travessa Paredes",
    Description = "Ativa/desativa a travessia de paredes",
    Default = false,
    Callback = toggleNoclip
})

Tabs.Main:AddToggle("infjump", {
    Title = "Pulo infinito",
    Description = "Ativa/desativa o pulo infinito",
    Default = false,
    Callback = function(state)
        notify(
            state and "Pulo infinito" or "Pulo infinito", 
            state and "Pulo infinito ativado com sucesso!" or "Pulo infinito desativado."
        )
        toggleInfiniteJump(state)
    end
})

local velocidadeAtiva = false
local velocidadeValor = 20 -- valor padrão

Tabs.Main:AddInput("WalkSpeedManual", {
    Title = "Velocidade",
    Placeholder = "Digite a velocidade...",
    Numeric = true,
    Finished = true, -- aplica ao sair da caixa
    Callback = function(inputValue)
        local value = tonumber(inputValue)
        if value and value >= 0 and value <= 200 then
            velocidadeValor = value
            if velocidadeAtiva then
                setHumanoidProperty("WalkSpeed", velocidadeValor)
            end
        end
    end
})

-- Toggle para ativar/desativar a velocidade
Tabs.Main:AddToggle("velocidade", {
    Title = "Velocidade",
    Description = "Ativa/desativa a velocidade",
    Default = false,
    Callback = function(state)
        velocidadeAtiva = state
        if state then
            setHumanoidProperty("WalkSpeed", velocidadeValor)
        else
            setHumanoidProperty("WalkSpeed", 20) -- valor padrão do Roblox
        end
    end
})

Tabs.Main:AddParagraph({ Title = "Segue nas redes sociais", Content = "Kwai:Vitoroficial Insta:vitoroemanuel"})

-- Variável para armazenar o estado do ESP
local espAtivado = false
local connections = {}

Tabs.Visual:AddToggle("esp_nome_distancia", {
    Title = "ESP Nome",
    Description = "Ativa/Desativa ESP Nome e Distância",
    Default = false,
    Callback = function(state)
        espAtivado = state
        local Players = game:GetService("Players")
        local LocalPlayer = Players.LocalPlayer

        -- Função para criar ESP
        local function criarESP(player)
            if player == LocalPlayer or not espAtivado then return end

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
                            esp.Size = UDim2.new(0, 150, 0, 60)
                            esp.StudsOffset = Vector3.new(0, 2.5, 0)
                            esp.AlwaysOnTop = true

                            -- Nome do jogador
                            local nome = Instance.new("TextLabel")
                            nome.Name = "Nome"
                            nome.Size = UDim2.new(1, 0, 0.5, 0)
                            nome.Position = UDim2.new(0, 0, 0, 0)
                            nome.BackgroundTransparency = 1
                            nome.TextColor3 = Color3.fromRGB(255, 255, 255)
                            nome.TextSize = 13
                            nome.TextScaled = false
                            nome.Font = Enum.Font.GothamSemibold
                            nome.TextStrokeTransparency = 0.4
                            nome.TextStrokeColor3 = Color3.new(0, 0, 0)
                            nome.Parent = esp

                            -- Distância
                            local distanciaTxt = Instance.new("TextLabel")
                            distanciaTxt.Name = "Distancia"
                            distanciaTxt.Size = UDim2.new(1, 0, 0.5, 0)
                            distanciaTxt.Position = UDim2.new(0, 0, 0.5, 0)
                            distanciaTxt.BackgroundTransparency = 1
                            distanciaTxt.TextColor3 = Color3.fromRGB(255, 255, 255)
                            distanciaTxt.TextTransparency = 0.3
                            distanciaTxt.TextSize = 12
                            distanciaTxt.TextScaled = false
                            distanciaTxt.Font = Enum.Font.Gotham
                            distanciaTxt.TextStrokeTransparency = 1
                            distanciaTxt.Parent = esp

                            esp.Parent = head

                            humanoid.Died:Connect(function()
                                if esp then esp:Destroy() end
                            end)
                        end

                        local nome = esp:FindFirstChild("Nome")
                        local distanciaTxt = esp:FindFirstChild("Distancia")
                        local root1 = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                        local root2 = char:FindFirstChild("HumanoidRootPart")

                        if nome and distanciaTxt and root1 and root2 then
                            local distancia = (root1.Position - root2.Position).Magnitude
                            nome.Text = player.Name
                            distanciaTxt.Text = "[" .. math.floor(distancia) .. "m]"
                        end
                    end
                    wait(0.3)
                end
            end)
        end

        -- Monitorar jogador
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

        if espAtivado then
            for _, player in ipairs(Players:GetPlayers()) do
                monitorarPlayer(player)
            end

            connections["PlayerAdded"] = Players.PlayerAdded:Connect(function(player)
                monitorarPlayer(player)
            end)
        else
            -- Desativar ESP
            for _, player in ipairs(Players:GetPlayers()) do
                local head = player.Character and player.Character:FindFirstChild("Head")
                if head then
                    local esp = head:FindFirstChild("ESP")
                    if esp then
                        esp:Destroy()
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

local espConnections = {}
local drawingObjects = {}

local function createESP(player)
    local head = player.Character:FindFirstChild("Head")
    if not head then return end

    -- Verifica se o player só tem a cabeça
    if #player.Character:GetChildren() > 3 then return end -- (Head, Humanoid, HumanoidRootPart = 3 padrão)

    -- Verifica se tem "Staff" escrito
    local hasStaffTag = false
    for _, gui in pairs(head:GetChildren()) do
        if gui:IsA("BillboardGui") then
            for _, child in pairs(gui:GetChildren()) do
                if child:IsA("TextLabel") and string.find(string.lower(child.Text), "staff") then
                    hasStaffTag = true
                    break
                end
            end
        end
    end

    if not hasStaffTag then return end

    -- Cria ESP (linha)
    local line = Drawing.new("Line")
    line.Thickness = 2
    line.Transparency = 1
    line.Visible = true

    drawingObjects[player] = line

    espConnections[player] = game:GetService("RunService").RenderStepped:Connect(function()
        if player.Character and player.Character:FindFirstChild("Head") then
            local pos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(player.Character.Head.Position)
            if onScreen then
                line.Visible = true
                line.From = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y)
                line.To = Vector2.new(pos.X, pos.Y)
                local t = tick()
                line.Color = Color3.fromRGB(
                    math.sin(t) * 127 + 128,
                    math.sin(t + 2) * 127 + 128,
                    math.sin(t + 4) * 127 + 128
                )
            else
                line.Visible = false
            end
        else
            line.Visible = false
        end
    end)
end

local function removeESP()
    for player, conn in pairs(espConnections) do
        conn:Disconnect()
    end
    for player, obj in pairs(drawingObjects) do
        obj:Remove()
    end
    espConnections = {}
    drawingObjects = {}
end

-- Toggle callback
local function toggleEsp(enabled)
    removeESP()
    if enabled then
        for _, player in pairs(game.Players:GetPlayers()) do
            if player ~= game.Players.LocalPlayer then
                createESP(player)
            end
        end
        game.Players.PlayerAdded:Connect(function(player)
            player.CharacterAdded:Connect(function()
                wait(1)
                createESP(player)
            end)
        end)
    end
end

Tabs.Visual:AddToggle("Esp", {
    Title = "Esp Staff",
    Description = "Ativa/desativa Esp Staff",
    Default = false,
    Callback = toggleEsp
})

local player = game.Players.LocalPlayer
local hrp = player.Character:WaitForChild("HumanoidRootPart")
local TweenService = game:GetService("TweenService")

local autoFarmAtivo = false

-- Caminhar até uma posição
local function moverPara(pos, tempo)
    local tweenInfo = TweenInfo.new(tempo or 2, Enum.EasingStyle.Linear)
    local goal = {CFrame = CFrame.new(pos)}
    local tween = TweenService:Create(hrp, tweenInfo, goal)
    tween:Play()
    tween.Completed:Wait()
end

-- Detecta prompt na bancada
local function encontrarPromptBalcao()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ProximityPrompt") and obj.Parent and obj.Parent.Name:lower():find("lanche") then
            return obj
        end
    end
end

-- Detecta marcador de entrega
local function encontrarMarcadorEntrega()
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("BillboardGui") and v.Name:lower():find("entrega") then
            local adornee = v.Adornee
            if adornee then
                return adornee.Position
            end
        end
    end
end

-- Loop de Auto Farm
task.spawn(function()
    while true do
        if autoFarmAtivo then
            local prompt = encontrarPromptBalcao()
            if prompt then
                local pos = prompt.Parent.Position + Vector3.new(0, 2, 0)
                moverPara(pos, 2)
                fireproximityprompt(prompt)
            end
            wait(1)

            local entregaPos = encontrarMarcadorEntrega()
            if entregaPos then
                moverPara(entregaPos + Vector3.new(0, 2, 0), 3)
            end
            wait(1.5)
        end
        task.wait(0.5)
    end
end)

-- Toggle para ativar/desativar o Auto Farm
Tabs.Players:AddToggle("ToggleAutoFarmLanche", {
    Title = "Auto Farm Lanches",
    Description = "Liga ou desliga o farm automático de entregas",
    Default = false,
    Callback = function(state)
        autoFarmAtivo = state
    end
})
