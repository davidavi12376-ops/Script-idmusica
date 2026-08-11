-- Car Dealership Tycoon - Música Local (Rayfield)
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

-- Funções de som
local function getMyCar()
    for _, car in pairs(workspace:FindFirstChild("Cars") and workspace.Cars:GetChildren() or {}) do
        local stats = car:FindFirstChild("Stats")
        if stats and stats:FindFirstChild("Owner") and stats.Owner.Value == LocalPlayer.Name then
            return car
        end
    end
    return nil
end

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
    sound.Looped = false
    sound.RollOffMaxDistance = 50
    sound.RollOffMinDistance = 10

    local car = getMyCar()
    if car then
        local root = car.PrimaryPart or car:FindFirstChild("Chassis") or car:FindFirstChildWhichIsA("BasePart")
        sound.Parent = root or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")) or SoundService
    else
        sound.Parent = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or SoundService
    end

    sound:Play()
    currentSound = sound
    print("[Música] Tocando ID:", id)
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

        sound.Ended:Connect(function()
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
            Content = "Lista salva com sucesso!",
            Duration = 3
        })
    else
        Rayfield:Notify({
            Title = "Erro",
            Content = "Seu executor não suporta salvar arquivos.",
            Duration = 4
        })
    end
end

-- ========== JANELA RAYFIELD ==========
local Window = Rayfield:CreateWindow({
    Name = "🎵 Música Local | CDT",
    LoadingTitle = "Car Dealership Tycoon",
    LoadingSubtitle = "Só você escuta",
    ConfigurationSaving = {
        Enabled = false
    },
    Discord = {
        Enabled = false
    },
    KeySystem = false
})

-- ========== ABA 1: ADICIONAR ==========
local TabAdd = Window:CreateTab("Adicionar", 4483362458)

local InputID = TabAdd:CreateInput({
    Name = "ID da Música",
    PlaceholderText = "Cole o ID aqui...",
    RemoveTextAfterFocusLost = false,
    Callback = function(Text)
        -- só guarda o texto
    end,
})

TabAdd:CreateButton({
    Name = "➕ Adicionar ID na Lista",
    Callback = function()
        local id = InputID.CurrentValue
        if id and id ~= "" and tonumber(id) then
            table.insert(MusicList, tonumber(id))
            Rayfield:Notify({
                Title = "Adicionado!",
                Content = "ID " .. id .. " adicionado. Total: " .. #MusicList,
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

TabAdd:CreateParagraph({
    Title = "Como usar",
    Content = "Cole o ID da música no campo acima e clique em Adicionar. Depois vá na aba Lista e salve."
})

-- ========== ABA 2: LISTA ==========
local TabList = Window:CreateTab("Lista", 4483362458)

local ListParagraph = TabList:CreateParagraph({
    Title = "IDs Salvos",
    Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID na lista ainda."
})

TabList:CreateButton({
    Name = "🔄 Atualizar Lista",
    Callback = function()
        ListParagraph:Set({
            Title = "IDs Salvos (" .. #MusicList .. ")",
            Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID na lista ainda."
        })
    end,
})

TabList:CreateButton({
    Name = "💾 Salvar Lista",
    Callback = function()
        saveList()
        ListParagraph:Set({
            Title = "IDs Salvos (" .. #MusicList .. ")",
            Content = #MusicList > 0 and table.concat(MusicList, ", ") or "Nenhum ID na lista ainda."
        })
    end,
})

TabList:CreateButton({
    Name = "🗑️ Limpar Toda a Lista",
    Callback = function()
        MusicList = {}
        ListParagraph:Set({
            Title = "IDs Salvos (0)",
            Content = "Lista limpa."
        })
        Rayfield:Notify({
            Title = "Lista limpa",
            Content = "Todos os IDs foram removidos.",
            Duration = 3
        })
    end,
})

-- ========== ABA 3: PLAYLIST ==========
local TabPlay = Window:CreateTab("Playlist", 4483362458)

TabPlay:CreateParagraph({
    Title = "Controle da Playlist",
    Content = "A playlist toca todas as músicas da lista em ordem e depois reinicia."
})

TabPlay:CreateButton({
    Name = "▶ Iniciar Playlist",
    Callback = function()
        startPlaylist()
        Rayfield:Notify({
            Title = "Playlist",
            Content = "Iniciada!",
            Duration = 3
        })
    end,
})

TabPlay:CreateButton({
    Name = "⏹ Parar Playlist",
    Callback = function()
        stopMusic()
        Rayfield:Notify({
            Title = "Playlist",
            Content = "Parada.",
            Duration = 3
        })
    end,
})

TabPlay:CreateButton({
    Name = "⏭ Próxima Música",
    Callback = function()
        if playlistRunning and currentSound then
            currentSound:Stop()
            currentIndex += 1
            -- a função playNext vai ser chamada pelo Ended, mas forçamos
            if currentIndex > #MusicList then
                currentIndex = 1
            end
            playMusic(MusicList[currentIndex])
        end
    end,
})

print("✅ Script Rayfield carregado! Arraste a janela normalmente.")
