local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Tapiiox hub",
    SubTitle = "by mr, tapiiox",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = false, -- Desativa o efeito blur do Windows 11
    Theme = "Darker", -- CORRIGIDO: "Sakura" não existe na biblioteca original. Use "Darker", "Dark", "Aqua" ou "Light"
    MinimizeKey = Enum.KeyCode.RightControl
})

-- Força o Fluent original a tirar a transparência do fundo
Fluent:ToggleTransparency(false)

-- Notificação apenas para testar se carregou
Fluent:Notify({
    Title = "Tapiiox hub",
    Content = "welcome tapiiox hub!",
    Duration = 3
})
local ScreenGui = Instance.new("ScreenGui")
local ToggleBtn = Instance.new("ImageButton")
local UICorner = Instance.new("UICorner")

ScreenGui.Parent = game.CoreGui
ScreenGui.ResetOnSpawn = false
ScreenGui.ResetOnSpawn = true

ToggleBtn.Parent = ScreenGui
ToggleBtn.Size = UDim2.new(0, 60, 0, 60)
ToggleBtn.Position = UDim2.new(0, 20, 0.5, 0)
ToggleBtn.BackgroundTransparency = 1
ToggleBtn.Image = "rbxassetid://132313789987260"

UICorner.CornerRadius = UDim.new(1, 0)
UICorner.Parent = ToggleBtn

-- Abre/fecha igual RightControl
ToggleBtn.MouseButton1Click:Connect(function()
    game:GetService("VirtualInputManager"):SendKeyEvent(true, Enum.KeyCode.RightControl, false, game)
    task.wait(0.1)
    game:GetService("VirtualInputManager"):SendKeyEvent(false, Enum.KeyCode.RightControl, false, game)
end)

-- Arrastar a bolinha
local dragging, dragStart, startPos

ToggleBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = ToggleBtn.Position
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        ToggleBtn.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

game:GetService("UserInputService").InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
        local delta = input.Position - dragStart
        ToggleBtn.Position = UDim2.new(
            startPos.X.Scale, 
            startPos.X.Offset + delta.X, 
            startPos.Y.Scale, 
            startPos.Y.Offset + delta.Y
        )
    end
end)

game:GetService("UserInputService").InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)
-- ========================================================
-- 1. PRIMEIRO: CRIA TODAS AS ABAS DO MENU
-- ========================================================
local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "" })
}

-- [[ GERENCIADORES INTERNOS ]]
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:SetIgnoreIndexes({})


local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")

_G.PuxarBolaActive = false

-- Removemos a variável fixa lá de cima e criamos uma função para achar a bola atualizada
local function getSoccerBall()
    local footballField = Workspace:FindFirstChild("FootballField")
    return footballField and footballField:FindFirstChild("SoccerBall")
end

-- TOGGLE: Puxa e mantém a bola no jogador
Tabs.Main:AddToggle("PuxarBolaToggle", {
    Title = "Bring ball",
    Default = false,
    Callback = function(Value)
        _G.PuxarBolaActive = Value
        
        if Value then
            task.spawn(function()
                while _G.PuxarBolaActive do
                    local character = Players.LocalPlayer.Character
                    local myHrp = character and character:FindFirstChild("HumanoidRootPart")
                    
                    -- Procuramos a bola ATUAL no frame atual
                    local currentBall = getSoccerBall()
                    
                    if myHrp and currentBall and currentBall:IsA("BasePart") then
                        -- Zera as velocidades físicas para ela não sair voando com o delay do teletransporte
                        currentBall.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                        currentBall.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                        
                        -- Teletransporta a bola para ficar exatamente 3 studs à sua frente
                        currentBall.CFrame = myHrp.CFrame * CFrame.new(0, -1, -3)
                    end
                    task.wait() -- Roda em cada frame (Heartbeat aproximado) para prender bem a bola
                end
            end)
        end
    end
})

Tabs.Main:AddButton({
    Title = "Trocar de Servidor",
    Description = "Entra em outro servidor público com vagas livres",
    Callback = function()
        local TeleportService = game:GetService("TeleportService")
        local HttpService = game:GetService("HttpService")
        local url = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        
        local success, result = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
        end)
        
        if success and result and result.data then
            for _, server in ipairs(result.data) do
                if server.playing < server.maxPlayers and server.id ~= game.JobId then
                    pcall(function()
                        TeleportService:TeleportToPlaceInstance(game.PlaceId, server.id, Players.LocalPlayer)
                    end)
                    return
                end
            end
        end
    end
})

local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")

-- Variáveis de controle
local quantidadeBrings = 1

-- Referência da bola
local footballField = Workspace:FindFirstChild("FootballField")
local soccerBall = footballField and footballField:FindFirstChild("SoccerBall")

-- 1. SLIDER: Define a quantidade de vezes que a bola será puxada
Tabs.Main:AddSlider("BringSlider", {
    Title = "Quantidade de Brings",
    Description = "Quantas vezes a bola será teleportada em sequência ao clicar",
    Min = 1,
    Max = 10,
    Default = 1,
    Rounding = 0,
    Callback = function(Value)
        quantidadeBrings = Value
    end
})

-- 2. BUTTON: Executa a quantidade exata de brings jogando a bola na sua frente
Tabs.Main:AddButton({
    Title = "Dar Bring na Bola",
    Description = "Puxa a bola para a sua frente com base no slider",
    Callback = function()
        local character = Players.LocalPlayer.Character
        local myHrp = character and character:FindFirstChild("HumanoidRootPart")
        
        -- Função interna para achar a bola atualizada no Workspace
        local function getSoccerBall()
            local footballField = Workspace:FindFirstChild("FootballField")
            return footballField and footballField:FindFirstChild("SoccerBall")
        end
        
        -- Pega a bola atual antes de iniciar o processo
        local currentBall = getSoccerBall()
        
        if myHrp and currentBall and currentBall:IsA("BasePart") then
            task.spawn(function()
                -- Loop que roda o número exato de vezes do Slider
                for i = 1, quantidadeBrings do
                    -- Atualiza a referência da bola DENTRO do loop, caso ela respawne enquanto o loop roda
                    currentBall = getSoccerBall()
                    
                    if currentBall and currentBall:IsA("BasePart") then
                        -- Zera a física para ela não bugar ou sair voando de perto
                        currentBall.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                        currentBall.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
                        
                        -- CFrame.new(0, 0, -3) joga a bola 3 studs à frente da direção que seu personagem está olhando
                        currentBall.CFrame = myHrp.CFrame * CFrame.new(0, 0, -3)
                    end
                    
                    -- Micro intervalo para o servidor computar cada posição
                    task.wait(0.05)
                end
            end)
        end
    end
})

-- Função interna para achar a bola atualizada no Workspace
local function getSoccerBall()
    local footballField = Workspace:FindFirstChild("FootballField")
    return footballField and footballField:FindFirstChild("SoccerBall")
end

-- Define a tecla padrão do atalho alterada para Q
local AtivarBringKey = Enum.KeyCode.Q

-- 1. Cria um botão na interface apenas para o usuário saber qual é a tecla (Atualizado para [ Q ])
Tabs.Main:AddButton({
    Title = "Atalho do Bring: [ Q ]",
    Description = "Aperte Q no teclado para puxar a bola instantaneamente",
    Callback = function()
        -- Botão visual, não precisa fazer nada ao clicar nele
    end
})

-- 2. Código "Raiz" usando a API nativa do Roblox para escutar o teclado
local UserInputService = game:GetService("UserInputService")

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- gameProcessed impede que a bola seja puxada se o jogador estiver digitando no chat do jogo
    if gameProcessed then return end
    
    -- Verifica se o jogador apertou a tecla certa (Q)
    if input.KeyCode == AtivarBringKey then
        local character = Players.LocalPlayer.Character
        local myHrp = character and character:FindFirstChild("HumanoidRootPart")
        local currentBall = getSoccerBall()
        
        if myHrp and currentBall and currentBall:IsA("BasePart") then
            -- Zera a física e as propriedades antigas para forçar a parada da bola no lag de rede
            currentBall.Velocity = Vector3.new(0, 0, 0)
            currentBall.RotVelocity = Vector3.new(0, 0, 0)
            currentBall.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
            currentBall.AssemblyAngularVelocity = Vector3.new(0, 0, 0)
            
            -- Teletransporta a bola 3 studs à sua frente
            currentBall.CFrame = myHrp.CFrame * CFrame.new(0, 0, -3)
        end
    end
end)
