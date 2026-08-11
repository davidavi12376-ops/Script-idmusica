-- Car Dealership Tycoon - Música Local (Corrigido)
-- Só você escuta | Delta / Xeno

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local SoundService = game:GetService("SoundService")
local HttpService = game:GetService("HttpService")

local SAVE_FILE = "CDT_MusicList.json"

-- Carrega lista salva
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

-- ==================== FUNÇÃO DE SOM CORRIGIDA ====================
local function playMusic(id)
    -- Para o som anterior
    if currentSound then
        currentSound:Stop()
        currentSound:Destroy()
        currentSound = nil
    end

    local sound = Instance.new("Sound")
    sound.Name = "LocalCarMusic_" .. tostring(id)
    sound.SoundId = "rbxassetid://" .. tostring(id)
    sound.Volume = 2 -- aumentei o volume
    sound.Looped = false
    sound.PlaybackSpeed = 1
    sound.RollOffMaxDistance = 10000
    sound.RollOffMinDistance = 10000

    -- Parent no SoundService (mais confiável pra som local)
    sound.Parent = SoundService

    -- Força o carregamento e toca
    sound.Loaded:Wait() -- espera o áudio carregar
    sound:Play()

    currentSound = sound
    print("[Música] Tocando ID:", id)
    
    Rayfield:Notify({
        Title = "Tocando",
        Content = "ID: " .. id,
        Duration = 2
    })

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
            Content = "A lista de músicas está vazia!",
            Duration = 4
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

        local id = MusicList[currentIndex]
        local sound = playMusic(id)

        -- Quando acabar, toca a próxima
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
    else
        Rayfield:Notify({
            Title = "Erro",
            Content = "Seu executor não suporta writefile",
            Duration = 4
        })
    end
end

-- ========== JANELA ==========
local Window = Rayfield:CreateWindow({
    Name = "🎵 Música Local | CDT",
    LoadingTitle = "Car Dealership Tycoon",
    LoadingSubtitle = "Só você escuta",
    ConfigurationSaving = { Enabled = false },
    KeySystem = false
})

-- ========== ABA 1: ADICIONAR ==========
local TabAdd = Window:CreateTab("Adicionar", 4483362458)

local InputID = TabAdd:CreateInput({
    Name = "ID da Música",
    PlaceholderText = "Cole o ID aqui...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text) end,
})

TabAdd:CreateButton({
    Name = "➕ Adicionar ID na Lista",
    Callback = function()
        local id = InputID.CurrentValue
        if id and id ~= "" and tonumber(id) then
            table.insert(MusicList, tonumber(id))
            Rayfield:Notify({
                Title = "Adicionado!",
                Content = "ID " .. id .. " | Total: " .. #MusicList,
                Duration = 3
            })
        else
            Rayfield:Notify({
                Title = "Erro",
                Content = "ID inválido!",
                Duration = 3
            })
        end
    end,
})

TabAdd:CreateButton({
    Name = "▶ Testar este ID agora",
    Callback = function()
        local id = InputID.CurrentValue
        if id and id ~= "" and tonumber(id) then
            playlistRunning = false
            playMusic(tonumber(id))
        else
            Rayfield:Notify({
                Title = "Erro",
                Content = "Coloque um ID válido primeiro",
                Duration = 3
            })
        end
    end,
})

-- ========== ABA 2: LISTA ==========
local TabList = Window:CreateTab("Lista", 4483362458)

local ListParagraph = TabList:CreateParagraph({
    Title = "IDs na Lista (" .. #MusicList .. ")",
    Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID ainda."
})

TabList:CreateButton({
    Name = "🔄 Atualizar Visualização",
    Callback = function()
        ListParagraph:Set({
            Title = "IDs na Lista (" .. #MusicList .. ")",
            Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID ainda."
        })
    end,
})

TabList:CreateButton({
    Name = "💾 Salvar Lista",
    Callback = function()
        saveList()
        ListParagraph:Set({
            Title = "IDs na Lista (" .. #MusicList .. ")",
            Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID ainda."
        })
    end,
})

TabList:CreateButton({
    Name = "🗑️ Limpar Lista",
    Callback = function()
        MusicList = {}
        ListParagraph:Set({
            Title = "IDs na Lista (0)",
            Content = "Lista limpa."
        })
    end,
})

-- ========== ABA 3: PLAYLIST ==========
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
        Rayfield:Notify({
            Title = "Parado",
            Content = "Música parada.",
            Duration = 2
        })
    end,
})

TabPlay:CreateButton({
    Name = "⏭ Próxima Música",
    Callback = function()
        if currentSound then
            currentSound:Stop()
        end
        if #MusicList > 0 then
            currentIndex = currentIndex + 1
            if currentIndex > #MusicList then
                currentIndex = 1
            end
            playMusic(MusicList[currentIndex])
        end
    end,
})

print("✅ Script corrigido carregado!")
