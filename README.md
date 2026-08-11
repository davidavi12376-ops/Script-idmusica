-- Script de Playlist com Abas + Salvamento + Arrastar
-- Car Dealership Tycoon | Só você escuta | Delta / Xeno

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local SoundService = game:GetService("SoundService")
local HttpService = game:GetService("HttpService")
local UserInputService = game:GetService("UserInputService")

-- Arquivo onde salva a lista
local SAVE_FILE = "CDT_MusicList.json"

-- Carrega a lista salva (se existir)
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
        warn("Lista vazia!")
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
        print("✅ Lista salva com sucesso!")
    else
        warn("Seu executor não suporta writefile")
    end
end

-- ========== GUI ==========
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "CDT_MusicTabs"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = game:GetService("CoreGui")

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 340, 0, 320)
MainFrame.Position = UDim2.new(0, 20, 0.5, -160)
MainFrame.BackgroundColor3 = Color3.fromRGB(22, 22, 28)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true -- importante pro arrasto
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 12)
MainCorner.Parent = MainFrame

-- ========== SISTEMA DE ARRASTAR (PC + Mobile) ==========
local dragging = false
local dragStart = nil
local startPos = nil

local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(
        startPos.X.Scale,
        startPos.X.Offset + delta.X,
        startPos.Y.Scale,
        startPos.Y.Offset + delta.Y
    )
end

MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

MainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        if dragging then
            update(input)
        end
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        if dragging then
            update(input)
        end
    end
end)

-- Título (também serve pra arrastar)
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 35)
Title.BackgroundTransparency = 1
Title.Text = "🎵 Música Local - CDT  (arraste)"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 17
Title.Parent = MainFrame

-- Abas
local TabButtons = Instance.new("Frame")
TabButtons.Size = UDim2.new(1, -20, 0, 32)
TabButtons.Position = UDim2.new(0, 10, 0, 38)
TabButtons.BackgroundTransparency = 1
TabButtons.Parent = MainFrame

local function createTabButton(name, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.32, 0, 1, 0)
    btn.Position = UDim2.new(order * 0.34, 0, 0, 0)
    btn.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    btn.Text = name
    btn.TextColor3 = Color3.fromRGB(200, 200, 200)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 13
    btn.Parent = TabButtons

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = btn
    return btn
end

local Tab1 = createTabButton("Adicionar", 0)
local Tab2 = createTabButton("Lista", 1)
local Tab3 = createTabButton("Playlist", 2)

-- Páginas
local Pages = Instance.new("Frame")
Pages.Size = UDim2.new(1, -20, 1, -85)
Pages.Position = UDim2.new(0, 10, 0, 78)
Pages.BackgroundTransparency = 1
Pages.Parent = MainFrame

local function createPage()
    local page = Instance.new("Frame")
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.Visible = false
    page.Parent = Pages
    return page
end

local Page1 = createPage() -- Adicionar
local Page2 = createPage() -- Lista
local Page3 = createPage() -- Playlist

-- ========== PÁGINA 1: ADICIONAR ==========
local AddBox = Instance.new("TextBox")
AddBox.Size = UDim2.new(1, 0, 0, 36)
AddBox.Position = UDim2.new(0, 0, 0, 10)
AddBox.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
AddBox.Text = ""
AddBox.PlaceholderText = "Cole o ID da música..."
AddBox.TextColor3 = Color3.fromRGB(255, 255, 255)
AddBox.PlaceholderColor3 = Color3.fromRGB(140, 140, 140)
AddBox.Font = Enum.Font.Gotham
AddBox.TextSize = 14
AddBox.ClearTextOnFocus = false
AddBox.Parent = Page1

local AddBoxCorner = Instance.new("UICorner")
AddBoxCorner.CornerRadius = UDim.new(0, 6)
AddBoxCorner.Parent = AddBox

local AddButton = Instance.new("TextButton")
AddButton.Size = UDim2.new(1, 0, 0, 38)
AddButton.Position = UDim2.new(0, 0, 0, 55)
AddButton.BackgroundColor3 = Color3.fromRGB(0, 140, 255)
AddButton.Text = "➕ Adicionar ID na Lista"
AddButton.TextColor3 = Color3.fromRGB(255, 255, 255)
AddButton.Font = Enum.Font.GothamBold
AddButton.TextSize = 14
AddButton.Parent = Page1

local AddBtnCorner = Instance.new("UICorner")
AddBtnCorner.CornerRadius = UDim.new(0, 6)
AddBtnCorner.Parent = AddButton

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, 0, 0, 30)
StatusLabel.Position = UDim2.new(0, 0, 0, 105)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Text = "IDs na lista: " .. #MusicList
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 13
StatusLabel.Parent = Page1

-- ========== PÁGINA 2: LISTA ==========
local ListScroll = Instance.new("ScrollingFrame")
ListScroll.Size = UDim2.new(1, 0, 0, 160)
ListScroll.Position = UDim2.new(0, 0, 0, 5)
ListScroll.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
ListScroll.BorderSizePixel = 0
ListScroll.ScrollBarThickness = 4
ListScroll.CanvasSize = UDim2.new(0, 0, 0, 0)
ListScroll.Parent = Page2

local ListCorner = Instance.new("UICorner")
ListCorner.CornerRadius = UDim.new(0, 6)
ListCorner.Parent = ListScroll

local ListLayout = Instance.new("UIListLayout")
ListLayout.Padding = UDim.new(0, 4)
ListLayout.Parent = ListScroll

local SaveButton = Instance.new("TextButton")
SaveButton.Size = UDim2.new(1, 0, 0, 36)
SaveButton.Position = UDim2.new(0, 0, 1, -40)
SaveButton.BackgroundColor3 = Color3.fromRGB(0, 170, 80)
SaveButton.Text = "💾 Salvar Lista"
SaveButton.TextColor3 = Color3.fromRGB(255, 255, 255)
SaveButton.Font = Enum.Font.GothamBold
SaveButton.TextSize = 14
SaveButton.Parent = Page2

local SaveCorner = Instance.new("UICorner")
SaveCorner.CornerRadius = UDim.new(0, 6)
SaveCorner.Parent = SaveButton

-- ========== PÁGINA 3: PLAYLIST ==========
local PlayInfo = Instance.new("TextLabel")
PlayInfo.Size = UDim2.new(1, 0, 0, 30)
PlayInfo.Position = UDim2.new(0, 0, 0, 10)
PlayInfo.BackgroundTransparency = 1
PlayInfo.Text = "Total de músicas: " .. #MusicList
PlayInfo.TextColor3 = Color3.fromRGB(200, 200, 200)
PlayInfo.Font = Enum.Font.Gotham
PlayInfo.TextSize = 14
PlayInfo.Parent = Page3

local PlayButton = Instance.new("TextButton")
PlayButton.Size = UDim2.new(1, 0, 0, 42)
PlayButton.Position = UDim2.new(0, 0, 0, 50)
PlayButton.BackgroundColor3 = Color3.fromRGB(0, 170, 80)
PlayButton.Text = "▶ Iniciar Playlist"
PlayButton.TextColor3 = Color3.fromRGB(255, 255, 255)
PlayButton.Font = Enum.Font.GothamBold
PlayButton.TextSize = 15
PlayButton.Parent = Page3

local PlayCorner = Instance.new("UICorner")
PlayCorner.CornerRadius = UDim.new(0, 6)
PlayCorner.Parent = PlayButton

local StopButton = Instance.new("TextButton")
StopButton.Size = UDim2.new(1, 0, 0, 42)
StopButton.Position = UDim2.new(0, 0, 0, 100)
StopButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
StopButton.Text = "⏹ Parar"
StopButton.TextColor3 = Color3.fromRGB(255, 255, 255)
StopButton.Font = Enum.Font.GothamBold
StopButton.TextSize = 15
StopButton.Parent = Page3

local StopCorner = Instance.new("UICorner")
StopCorner.CornerRadius = UDim.new(0, 6)
StopCorner.Parent = StopButton

-- Função pra atualizar a lista visual
local function updateListVisual()
    for _, child in pairs(ListScroll:GetChildren()) do
        if child:IsA("TextLabel") or child:IsA("TextButton") then
            child:Destroy()
        end
    end

    for i, id in ipairs(MusicList) do
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, -10, 0, 26)
        label.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
        label.Text = "  " .. i .. ".  " .. id
        label.TextColor3 = Color3.fromRGB(220, 220, 220)
        label.Font = Enum.Font.Gotham
        label.TextSize = 13
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = ListScroll

        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, 4)
        c.Parent = label
    end

    ListScroll.CanvasSize = UDim2.new(0, 0, 0, #MusicList * 30)
    StatusLabel.Text = "IDs na lista: " .. #MusicList
    PlayInfo.Text = "Total de músicas: " .. #MusicList
end

updateListVisual()

-- Lógica das abas
local function switchTab(tab)
    Page1.Visible = (tab == 1)
    Page2.Visible = (tab == 2)
    Page3.Visible = (tab == 3)

    Tab1.BackgroundColor3 = (tab == 1) and Color3.fromRGB(0, 140, 255) or Color3.fromRGB(40, 40, 50)
    Tab2.BackgroundColor3 = (tab == 2) and Color3.fromRGB(0, 140, 255) or Color3.fromRGB(40, 40, 50)
    Tab3.BackgroundColor3 = (tab == 3) and Color3.fromRGB(0, 140, 255) or Color3.fromRGB(40, 40, 50)

    Tab1.TextColor3 = (tab == 1) and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(200, 200, 200)
    Tab2.TextColor3 = (tab == 2) and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(200, 200, 200)
    Tab3.TextColor3 = (tab == 3) and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(200, 200, 200)
end

Tab1.MouseButton1Click:Connect(function() switchTab(1) end)
Tab2.MouseButton1Click:Connect(function() switchTab(2) end)
Tab3.MouseButton1Click:Connect(function() switchTab(3) end)

switchTab(1)

-- Botões
AddButton.MouseButton1Click:Connect(function()
    local id = AddBox.Text:gsub("%s+", "")
    if id ~= "" and tonumber(id) then
        table.insert(MusicList, tonumber(id))
        AddBox.Text = ""
        updateListVisual()
        StatusLabel.Text = "Adicionado! Total: " .. #MusicList
        task.wait(1.2)
        StatusLabel.Text = "IDs na lista: " .. #MusicList
    else
        StatusLabel.Text = "ID inválido!"
        task.wait(1.2)
        StatusLabel.Text = "IDs na lista: " .. #MusicList
    end
end)

SaveButton.MouseButton1Click:Connect(function()
    saveList()
    SaveButton.Text = "✅ Salvo!"
    task.wait(1.5)
    SaveButton.Text = "💾 Salvar Lista"
end)

PlayButton.MouseButton1Click:Connect(function()
    startPlaylist()
end)

StopButton.MouseButton1Click:Connect(function()
    stopMusic()
end)

print("✅ Script carregado! Agora você pode arrastar a janela (PC e Mobile).")
