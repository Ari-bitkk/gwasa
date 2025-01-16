-- Configuração inicial
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local espEnabled = false -- Controla se o ESP está ativo
local espConnections = {} -- Armazena as conexões para desativar o ESP

-- Função para criar ESP para cada jogador
local function createESP(player)
    if player == LocalPlayer then return end

    local function setupCharacter(character)
        if not espEnabled then return end

        local billboard = Instance.new("BillboardGui")
        billboard.Name = "ESP"
        billboard.Adornee = character:WaitForChild("HumanoidRootPart")
        billboard.Size = UDim2.new(5, 0, 5, 0)
        billboard.StudsOffset = Vector3.new(0, 3, 0)
        billboard.AlwaysOnTop = true

        -- Cria a imagem
        local imageLabel = Instance.new("ImageLabel", billboard)
        imageLabel.Size = UDim2.new(1, 0, 1, 0)
        imageLabel.Image = "rbxassetid://ID_DA_IMAGEM" -- Substitua pelo ID da imagem desejada
        imageLabel.BackgroundTransparency = 1

        -- Ajusta tamanho conforme distância
        local connection = RunService.RenderStepped:Connect(function()
            if character and character:FindFirstChild("HumanoidRootPart") then
                local distance = (LocalPlayer.Character.HumanoidRootPart.Position - character.HumanoidRootPart.Position).Magnitude
                billboard.Size = UDim2.new(math.clamp(distance / 10, 5, 20), 0, math.clamp(distance / 10, 5, 20), 0)
            end
        end)

        -- Salva a conexão para desativar depois
        table.insert(espConnections, connection)
        billboard.Parent = character:WaitForChild("HumanoidRootPart")
    end

    -- Conecta ao personagem existente e aos futuros personagens
    if player.Character then
        setupCharacter(player.Character)
    end
    player.CharacterAdded:Connect(setupCharacter)
end

-- Habilita o ESP
local function enableESP()
    espEnabled = true
    for _, player in pairs(Players:GetPlayers()) do
        createESP(player)
    end
    Players.PlayerAdded:Connect(createESP)
end

-- Desabilita o ESP
local function disableESP()
    espEnabled = false
    for _, connection in ipairs(espConnections) do
        connection:Disconnect()
    end
    espConnections = {}

    -- Remove as BillboardGuis criadas
    for _, player in pairs(Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local billboard = player.Character.HumanoidRootPart:FindFirstChild("ESP")
            if billboard then
                billboard:Destroy()
            end
        end
    end
end

-- Ativa/desativa o ESP com as teclas Insert e Delete
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end

    if input.KeyCode == Enum.KeyCode.Insert then
        enableESP()
    elseif input.KeyCode == Enum.KeyCode.Delete then
        disableESP()
    end
end)
