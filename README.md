-- Car Dealership Tycoon - Música fica no carro
-- Só você escuta | Delta / Xeno

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local SoundService = game:GetService("SoundService")
local HttpService = game:GetService("HttpService")

local SAVE_FILE = "CDT_MusicList.json"

local MusicList = {}
if isfile and isfile(SAVE_FILE) then
    local success, data = pcall(function()
        return HttpService:JSONDecode(readfile(SAVE_FILE))
    end)
    if success and type(data) == "table" then
        MusicList = data
    end
end

local currentSound = nil
local playlistRunning = false
local currentIndex = 1
local lastVehicle = nil -- guarda o último carro

-- Acha o carro que você está sentado
local function getCurrentVehicle()
    local character = LocalPlayer.Character
    if not character then return nil end

    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid or not humanoid.SeatPart then return nil end

    local seat = humanoid.SeatPart
    local current = seat

    while current and current.Parent do
        if current:IsA("Model") then
            if current:FindFirstChild("Stats") or current:FindFirstChild("Chassis") or current:FindFirstChildWhichIsA("VehicleSeat") then
                return current
            end
        end
        current = current.Parent
    end

    return seat
end

-- Toca a música e deixa ela presa no carro
local function playMusic(id)
    -- Para o som anterior (se existir)
    if currentSound then
        currentSound:Stop()
        currentSound:Destroy()
        currentSound = nil
    end

    local vehicle = getCurrentVehicle()
    if not vehicle then
        Rayfield:Notify({
            Title = "Aviso",
            Content = "Entre em um carro primeiro!",
            Duration = 3
        })
        return
    end

    lastVehicle = vehicle

    local sound = Instance.new("Sound")
    sound.Name = "LocalCarRadio"
    sound.SoundId = "rbxassetid://" .. tostring(id)
    sound.Volume = 2.5
    sound.Looped = false
    sound.PlaybackSpeed = 1
    sound.RollOffMaxDistance = 90
    sound.RollOffMinDistance = 12
    sound.EmitterSize = 25

    -- Coloca o som em uma parte do carro
    local part = vehicle.PrimaryPart 
        or vehicle:FindFirstChild("Chassis") 
        or vehicle:FindFirstChildWhichIsA("BasePart", true)
        or vehicle:FindFirstChildWhichIsA("VehicleSeat", true)

    if part then
        sound.Parent = part
    else
        sound.Parent = vehicle
    end

    if not sound.IsLoaded then
        sound.Loaded:Wait()
    end

    sound:Play()
    currentSound = sound

    Rayfield:Notify({
        Title = "Tocando no carro",
        Content = "ID: " .. id,
        Duration = 2
    })

    print("[Música] Presa no carro:", vehicle.Name)
    return sound
end

local function stopMusic()
    playlistRunning = false
    if currentSound then
        currentSound:Stop()
        currentSound:Destroy()
        currentSound = nil
    end
end

local function startPlaylist()
    if #MusicList == 0 then
        Rayfield:Notify({
            Title = "Erro",
            Content = "Lista vazia!",
            Duration = 3
        })
        return
    end

    if playlistRunning then return end
    playlistRunning = true
    currentIndex = 1

    local function playNext()
        if not playlistRunning then return end
        if currentIndex > #MusicList then
            currentIndex = 1
        end

        local sound = playMusic(MusicList[currentIndex])
        if not sound then
            playlistRunning = false
            return
        end

        local connection
        connection = sound.Ended:Connect(function()
            connection:Disconnect()
            if playlistRunning then
                currentIndex += 1
                playNext()
            end
        end)
    end

    playNext()
end

local function saveList()
    if writefile then
        writefile(SAVE_FILE, HttpService:JSONEncode(MusicList))
        Rayfield:Notify({
            Title = "Sucesso",
            Content = "Lista salva!",
            Duration = 3
        })
    end
end

-- ========== INTERFACE ==========
local Window = Rayfield:CreateWindow({
    Name = "🎵 Música no Carro | CDT",
    LoadingTitle = "Car Dealership Tycoon",
    LoadingSubtitle = "Som fica no veículo",
    ConfigurationSaving = { Enabled = false },
    KeySystem = false
})

-- ABA ADICIONAR
local TabAdd = Window:CreateTab("Adicionar", 4483362458)

local InputID = TabAdd:CreateInput({
    Name = "ID da Música",
    PlaceholderText = "Cole o ID aqui...",
    RemoveTextAfterFocusLost = false,
    Callback = function() end,
})

TabAdd:CreateButton({
    Name = "➕ Adicionar ID na Lista",
    Callback = function()
        local id = InputID.CurrentValue
        if id and id ~= "" and tonumber(id) then
            table.insert(MusicList, tonumber(id))
            Rayfield:Notify({
                Title = "Adicionado",
                Content = "Total: " .. #MusicList,
                Duration = 3
            })
        else
            Rayfield:Notify({
                Title = "Erro",
                Content = "ID inválido",
                Duration = 3
            })
        end
    end,
})

TabAdd:CreateButton({
    Name = "▶ Testar este ID no carro",
    Callback = function()
        local id = InputID.CurrentValue
        if id and id ~= "" and tonumber(id) then
            playlistRunning = false
            playMusic(tonumber(id))
        end
    end,
})

-- ABA LISTA
local TabList = Window:CreateTab("Lista", 4483362458)

local ListParagraph = TabList:CreateParagraph({
    Title = "IDs (" .. #MusicList .. ")",
    Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID"
})

TabList:CreateButton({
    Name = "🔄 Atualizar",
    Callback = function()
        ListParagraph:Set({
            Title = "IDs (" .. #MusicList .. ")",
            Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID"
        })
    end,
})

TabList:CreateButton({
    Name = "💾 Salvar Lista",
    Callback = function()
        saveList()
    end,
})

TabList:CreateButton({
    Name = "🗑️ Limpar Lista",
    Callback = function()
        MusicList = {}
        ListParagraph:Set({
            Title = "IDs (0)",
            Content = "Lista limpa"
        })
    end,
})

-- ABA PLAYLIST
local TabPlay = Window:CreateTab("Playlist", 4483362458)

TabPlay:CreateButton({
    Name = "▶ Iniciar Playlist",
    Callback = function()
        startPlaylist()
    end,
})

TabPlay:CreateButton({
    Name = "⏹ Parar",
    Callback = function()
        stopMusic()
    end,
})

TabPlay:CreateButton({
    Name = "⏭ Próxima",
    Callback = function()
        if currentSound then
            currentSound:Stop()
        end
        if #MusicList > 0 then
            currentIndex = currentIndex >= #MusicList and 1 or currentIndex + 1
            playMusic(MusicList[currentIndex])
        end
    end,
})

print("✅ Script carregado! O som agora fica no carro mesmo depois de sair.")
