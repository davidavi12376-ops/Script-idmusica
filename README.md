-- Script de Playlist Local no Carro - Car Dealership Tycoon
-- Só você escuta | Delta / Xeno

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local SoundService = game:GetService("SoundService")

-- ==================== LISTA DE MÚSICAS ====================
-- Coloque aqui os IDs que você quer na playlist
local MusicList = {
    1845577532,  -- Exemplo 1
    1837879082,  -- Exemplo 2
    142376088,   -- Exemplo 3
    9043876995,  -- Exemplo 4
    -- Adicione mais IDs abaixo (só o número)
}

-- ==========================================================

local currentSound = nil
local playlistRunning = false
local currentIndex = 1

-- Função pra achar o carro do jogador
local function getMyCar()
    for _, car in pairs(workspace:FindFirstChild("Cars") and workspace.Cars:GetChildren() or {}) do
        local stats = car:FindFirstChild("Stats")
        if stats and stats:FindFirstChild("Owner") and stats.Owner.Value == LocalPlayer.Name then
            return car
        end
    end
    return nil
end

-- Função pra tocar uma música específica
local function playMusic(id)
    if currentSound then
        currentSound:Stop()
        currentSound:Destroy()
        currentSound = nil
    end

    local sound = Instance.new("Sound")
    sound.Name = "LocalCarMusic"
    sound.SoundId = "rbxassetid://" .. tostring(id)
    sound.Volume = 1.5
    sound.Looped = false -- importante: false pra playlist avançar
    sound.RollOffMaxDistance = 50
    sound.RollOffMinDistance = 10

    local car = getMyCar()
    if car then
        local root = car.PrimaryPart or car:FindFirstChild("Chassis") or car:FindFirstChildWhichIsA("BasePart")
        if root then
            sound.Parent = root
        else
            sound.Parent = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or SoundService
        end
    else
        sound.Parent = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or SoundService
    end

    sound:Play()
    currentSound = sound
    print("[Playlist] Tocando ID:", id)
    return sound
end

local function stopMusic()
    playlistRunning = false
    if currentSound then
        currentSound:Stop()
        currentSound:Destroy()
        currentSound = nil
    end
    print("[Playlist] Parada")
end

-- Função da Playlist
local function startPlaylist()
    if #MusicList == 0 then
        warn("Lista de músicas está vazia!")
        return
    end

    if playlistRunning then
        return -- já está tocando
    end

    playlistRunning = true
    currentIndex = 1

    local function playNext()
        if not playlistRunning then return end

        if currentIndex > #MusicList then
            currentIndex = 1 -- volta pro início (loop da playlist)
        end

        local id = MusicList[currentIndex]
        local sound = playMusic(id)

        -- Quando a música acabar, toca a próxima
        sound.Ended:Connect(function()
            if playlistRunning then
                currentIndex = currentIndex + 1
                playNext()
            end
        end)
    end

    playNext()
end

-- ========== GUI ==========
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "LocalMusicGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = game:GetService("CoreGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 280, 0, 160)
Frame.Position = UDim2.new(0, 20, 0.5, -80)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
Frame.BorderSizePixel = 0
Frame.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 10)
UICorner.Parent = Frame

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.Text = "🎵 Playlist Local (só você)"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.Parent = Frame

local Info = Instance.new("TextLabel")
Info.Size = UDim2.new(1, -20, 0, 25)
Info.Position = UDim2.new(0, 10, 0, 32)
Info.BackgroundTransparency = 1
Info.Text = "Músicas na lista: " .. #MusicList
Info.TextColor3 = Color3.fromRGB(180, 180, 180)
Info.Font = Enum.Font.Gotham
Info.TextSize = 13
Info.TextXAlignment = Enum.TextXAlignment.Left
Info.Parent = Frame

local PlayButton = Instance.new("TextButton")
PlayButton.Size = UDim2.new(0.9, 0, 0, 38)
PlayButton.Position = UDim2.new(0.05, 0, 0, 65)
PlayButton.BackgroundColor3 = Color3.fromRGB(0, 170, 80)
PlayButton.Text = "▶ Iniciar Playlist"
PlayButton.TextColor3 = Color3.fromRGB(255, 255, 255)
PlayButton.Font = Enum.Font.GothamBold
PlayButton.TextSize = 15
PlayButton.Parent = Frame

local PlayCorner = Instance.new("UICorner")
PlayCorner.CornerRadius = UDim.new(0, 6)
PlayCorner.Parent = PlayButton

local StopButton = Instance.new("TextButton")
StopButton.Size = UDim2.new(0.9, 0, 0, 38)
StopButton.Position = UDim2.new(0.05, 0, 0, 110)
StopButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
StopButton.Text = "⏹ Parar Playlist"
StopButton.TextColor3 = Color3.fromRGB(255, 255, 255)
StopButton.Font = Enum.Font.GothamBold
StopButton.TextSize = 15
StopButton.Parent = Frame

local StopCorner = Instance.new("UICorner")
StopCorner.CornerRadius = UDim.new(0, 6)
StopCorner.Parent = StopButton

-- Botões
PlayButton.MouseButton1Click:Connect(function()
    startPlaylist()
end)

StopButton.MouseButton1Click:Connect(function()
    stopMusic()
end)

print("✅ Playlist Local carregada! Total de músicas:", #MusicList)
