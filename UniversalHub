
--[[
    UNIVERSAL HUB v0.2
    Client Utility Interface
    For authorised testing in your own or permitted Roblox experience.

    Features:
    - Player list, search, teleport, loop teleport, spectate
    - Item search, full paths, teleport
    - Saved positions and coordinate display
    - Script Library reference list
    - Draggable, mobile-friendly, minimisable interface

    No remote invocation, cooldown bypass, or server manipulation.
]]

--==================================================
-- SERVICES
--==================================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--==================================================
-- CONFIGURATION
--==================================================

local CONFIG = {
    Title = "UNIVERSAL HUB",
    Version = "v0.2",

    Background = Color3.fromRGB(20, 22, 29),
    Panel = Color3.fromRGB(29, 32, 42),
    Secondary = Color3.fromRGB(39, 43, 55),
    Accent = Color3.fromRGB(110, 130, 255),
    Text = Color3.fromRGB(240, 242, 255),
    Muted = Color3.fromRGB(155, 161, 180),
    Success = Color3.fromRGB(90, 190, 130),
    Danger = Color3.fromRGB(210, 90, 100),

    PlayerLoopInterval = 1,
    ItemLoopInterval = 1,

    -- Change this to a specific folder name if desired.
    -- Example: "Items" scans Workspace.Items when it exists.
    ItemFolderName = "Items",

    -- Maximum number of item results displayed.
    MaxItemResults = 150
}

local TAB_NAMES = {
    "Introduction",
    "Players",
    "Items",
    "Areas",
    "Script Library"
}

--==================================================
-- CLEAN UP PREVIOUS VERSION
--==================================================

local Existing = PlayerGui:FindFirstChild("UniversalHub")
if Existing then
    Existing:Destroy()
end

--==================================================
-- STATE
--==================================================

local SelectedPlayer = nil
local SelectedItem = nil

local PlayerLoopEnabled = false
local ItemLoopEnabled = false
local Spectating = false

local SavedPositions = {}
local SelectedSavedPosition = nil

local PlayerButtons = {}
local ItemButtons = {}
local SavedPositionButtons = {}

local OriginalCameraSubject = nil

--==================================================
-- GENERAL HELPERS
--==================================================

local function Create(class, properties, parent)
    local object = Instance.new(class)

    for property, value in pairs(properties) do
        object[property] = value
    end

    object.Parent = parent
    return object
end

local function Corner(parent, radius)
    Create("UICorner", {
        CornerRadius = UDim.new(0, radius or 8)
    }, parent)
end

local function Padding(parent, amount)
    Create("UIPadding", {
        PaddingTop = UDim.new(0, amount),
        PaddingBottom = UDim.new(0, amount),
        PaddingLeft = UDim.new(0, amount),
        PaddingRight = UDim.new(0, amount)
    }, parent)
end

local function MakeLabel(parent, text, height, size, color)
    return Create("TextLabel", {
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 0, height or 28),
        Font = Enum.Font.GothamMedium,
        Text = text or "",
        TextSize = size or 13,
        TextColor3 = color or CONFIG.Text,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        TextWrapped = true
    }, parent)
end

local function MakeButton(parent, text, callback, height)
    local button = Create("TextButton", {
        Size = UDim2.new(1, 0, 0, height or 38),
        BackgroundColor3 = CONFIG.Secondary,
        BorderSizePixel = 0,
        Text = text,
        TextColor3 = CONFIG.Text,
        Font = Enum.Font.GothamMedium,
        TextSize = 12,
        TextWrapped = true,
        AutoButtonColor = true
    }, parent)

    Corner(button, 8)

    if callback then
        button.Activated:Connect(callback)
    end

    return button
end

local function MakeBox(parent, placeholder, height)
    local box = Create("TextBox", {
        Size = UDim2.new(1, 0, 0, height or 38),
        BackgroundColor3 = CONFIG.Secondary,
        BorderSizePixel = 0,
        Text = "",
        PlaceholderText = placeholder,
        PlaceholderColor3 = CONFIG.Muted,
        TextColor3 = CONFIG.Text,
        Font = Enum.Font.Gotham,
        TextSize = 12,
        ClearTextOnFocus = false
    }, parent)

    Corner(box, 8)
    Padding(box, 8)

    return box
end

local function MakeList(parent, height)
    local list = Create("ScrollingFrame", {
        Size = UDim2.new(1, 0, 0, height),
        BackgroundColor3 = CONFIG.Background,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        CanvasSize = UDim2.new(),
        AutomaticCanvasSize = Enum.AutomaticSize.Y
    }, parent)

    Corner(list, 8)
    Padding(list, 6)

    Create("UIListLayout", {
        Padding = UDim.new(0, 5),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, list)

    return list
end

local function ClearList(list)
    for _, child in ipairs(list:GetChildren()) do
        if child:IsA("GuiObject") and not child:IsA("UIListLayout")
            and not child:IsA("UIPadding") then
            child:Destroy()
        end
    end
end

local function GetCharacter()
    return LocalPlayer.Character
end

local function GetRoot(character)
    if not character then return nil end
    return character:FindFirstChild("HumanoidRootPart")
end

local function GetHumanoid(character)
    if not character then return nil end
    return character:FindFirstChildOfClass("Humanoid")
end

local function LocalTeleport(targetCFrame)
    local character = GetCharacter()

    if not character or not targetCFrame then
        return false
    end

    local root = GetRoot(character)
    if not root then
        return false
    end

    character:PivotTo(targetCFrame)
    return true
end

local function GetFullPath(object)
    if not object then return "Unknown" end

    local path = object.Name
    local current = object.Parent

    while current and current ~= game do
        path = current.Name .. "." .. path
        current = current.Parent
    end

    return path
end

local function GetItemRoot()
    local configured = workspace:FindFirstChild(CONFIG.ItemFolderName)

    if configured then
        return configured
    end

    return workspace
end

--==================================================
-- MAIN INTERFACE
--==================================================

local ScreenGui = Create("ScreenGui", {
    Name = "UniversalHub",
    ResetOnSpawn = false,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling
}, PlayerGui)

local Main = Create("Frame", {
    Name = "Main",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromScale(0.82, 0.78),
    BackgroundColor3 = CONFIG.Background,
    BorderSizePixel = 0
}, ScreenGui)

Corner(Main, 14)

Create("UISizeConstraint", {
    MinSize = Vector2.new(300, 320),
    MaxSize = Vector2.new(900, 720)
}, Main)

--==================================================
-- HEADER
--==================================================

local Header = Create("Frame", {
    Name = "Header",
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundColor3 = CONFIG.Panel,
    BorderSizePixel = 0
}, Main)

Corner(Header, 12)

local HeaderTitle = MakeLabel(
    Header,
    CONFIG.Title .. "  " .. CONFIG.Version,
    36,
    14
)

HeaderTitle.Position = UDim2.fromOffset(14, 5)
HeaderTitle.Size = UDim2.new(1, -60, 0, 36)

local MinimizeButton = Create("TextButton", {
    AnchorPoint = Vector2.new(1, 0.5),
    Position = UDim2.new(1, -9, 0.5, 0),
    Size = UDim2.fromOffset(34, 30),
    BackgroundColor3 = CONFIG.Secondary,
    Text = "—",
    TextColor3 = CONFIG.Text,
    TextSize = 18,
    Font = Enum.Font.GothamBold
}, Header)

Corner(MinimizeButton, 8)

--==================================================
-- TAB BAR
--==================================================

local TabBar = Create("ScrollingFrame", {
    Position = UDim2.fromOffset(8, 55),
    Size = UDim2.new(1, -16, 0, 42),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 2,
    ScrollingDirection = Enum.ScrollingDirection.X,
    AutomaticCanvasSize = Enum.AutomaticSize.X,
    CanvasSize = UDim2.new()
}, Main)

Create("UIListLayout", {
    FillDirection = Enum.FillDirection.Horizontal,
    Padding = UDim.new(0, 6),
    SortOrder = Enum.SortOrder.LayoutOrder
}, TabBar)

--==================================================
-- CONTENT AREA
--==================================================

local Content = Create("Frame", {
    Position = UDim2.fromOffset(10, 103),
    Size = UDim2.new(1, -20, 1, -113),
    BackgroundColor3 = CONFIG.Panel,
    BorderSizePixel = 0
}, Main)

Corner(Content, 10)

local Pages = {}
local TabButtons = {}

local function CreatePage(name)
    local page = Create("ScrollingFrame", {
        Name = name,
        Size = UDim2.new(1, 0, 1, 0),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 3,
        CanvasSize = UDim2.new(),
        AutomaticCanvasSize = Enum.AutomaticSize.Y,
        Visible = false
    }, Content)

    Padding(page, 12)

    Create("UIListLayout", {
        Padding = UDim.new(0, 9),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, page)

    Pages[name] = page
    return page
end

local function AddTab(name, order)
    local button = Create("TextButton", {
        Name = name,
        Size = UDim2.fromOffset(125, 34),
        BackgroundColor3 = CONFIG.Secondary,
        Text = name,
        TextColor3 = CONFIG.Muted,
        TextSize = 12,
        Font = Enum.Font.GothamMedium,
        LayoutOrder = order
    }, TabBar)

    Corner(button, 8)

    local page = CreatePage(name)
    TabButtons[name] = button

    button.Activated:Connect(function()
        for tabName, tabPage in pairs(Pages) do
            tabPage.Visible = tabName == name
        end

        for tabName, tabButton in pairs(TabButtons) do
            tabButton.BackgroundColor3 =
                tabName == name and CONFIG.Accent or CONFIG.Secondary

            tabButton.TextColor3 =
                tabName == name and CONFIG.Text or CONFIG.Muted
        end

        Pages[name].CanvasPosition = Vector2.zero
    end)

    return page
end

--==================================================
-- TAB 1: INTRODUCTION
--==================================================

local Intro = AddTab(TAB_NAMES[1], 1)

MakeLabel(Intro, "Welcome to Universal Hub", 35, 20)

MakeLabel(
    Intro,
    "A collection of client-side utilities for organised gameplay testing.",
    50,
    13,
    CONFIG.Muted
)

MakeLabel(Intro, "Current version: " .. CONFIG.Version, 25, 12)

MakeLabel(
    Intro,
    "Use these features only in experiences where you have permission to test.",
    55,
    12,
    CONFIG.Muted
)

MakeLabel(
    Intro,
    "This hub does not execute server commands or bypass game protections.",
    50,
    12,
    CONFIG.Muted
)

--==================================================
-- TAB 2: PLAYERS
--==================================================

local PlayersPage = AddTab(TAB_NAMES[2], 2)

MakeLabel(PlayersPage, "Player Display", 30, 18)

local PlayerSearch = MakeBox(
    PlayersPage,
    "Search player name or display name..."
)

local PlayerStatus = MakeLabel(
    PlayersPage,
    "Select a player from the list.",
    28,
    11,
    CONFIG.Muted
)

local PlayerList = MakeList(PlayersPage, 170)

local function StopPlayerLoop()
    PlayerLoopEnabled = false
end

local function StopSpectating()
    Spectating = false

    local camera = workspace.CurrentCamera

    if camera then
        local humanoid = GetHumanoid(GetCharacter())

        if humanoid then
            camera.CameraSubject = humanoid
        elseif OriginalCameraSubject then
            camera.CameraSubject = OriginalCameraSubject
        end
    end
end

local function RefreshPlayers()
    ClearList(PlayerList)
    table.clear(PlayerButtons)

    local query = string.lower(PlayerSearch.Text)

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local matches =
                query == ""
                or string.find(string.lower(player.Name), query, 1, true)
                or string.find(string.lower(player.DisplayName), query, 1, true)

            if matches then
                local button = MakeButton(
                    PlayerList,
                    player.DisplayName .. "  (@" .. player.Name .. ")",
                    function()
                        SelectedPlayer = player
                        PlayerStatus.Text = "Selected: " .. player.Name

                        for _, otherButton in ipairs(PlayerButtons) do
                            otherButton.BackgroundColor3 = CONFIG.Secondary
                        end

                        button.BackgroundColor3 = CONFIG.Accent
                    end,
                    34
                )

                table.insert(PlayerButtons, button)
            end
        end
    end

    if #PlayerButtons == 0 then
        MakeLabel(PlayerList, "No matching players found.", 30, 11, CONFIG.Muted)
    end
end

PlayerSearch:GetPropertyChangedSignal("Text"):Connect(RefreshPlayers)

Players.PlayerAdded:Connect(RefreshPlayers)
Players.PlayerRemoving:Connect(function(player)
    if SelectedPlayer == player then
        SelectedPlayer = nil
        StopPlayerLoop()
        StopSpectating()
        PlayerStatus.Text = "Selected player left."
    end

    RefreshPlayers()
end)

MakeButton(PlayersPage, "Refresh Player List", RefreshPlayers)

MakeButton(PlayersPage, "Teleport to Selected Player", function()
    if not SelectedPlayer or not SelectedPlayer.Parent then
        PlayerStatus.Text = "Select an available player first."
        return
    end

    local targetRoot = GetRoot(SelectedPlayer.Character)

    if targetRoot then
        LocalTeleport(targetRoot.CFrame * CFrame.new(0, 0, 4))
        PlayerStatus.Text = "Moved to selected player's location."
    else
        PlayerStatus.Text = "Selected player has no available character."
    end
end)

local PlayerLoopButton

PlayerLoopButton = MakeButton(
    PlayersPage,
    "Start Loop Teleport",
    function()
        PlayerLoopEnabled = not PlayerLoopEnabled

        PlayerLoopButton.Text = PlayerLoopEnabled
            and "Stop Loop Teleport"
            or "Start Loop Teleport"

        if PlayerLoopEnabled then
            task.spawn(function()
                while PlayerLoopEnabled do
                    if SelectedPlayer and SelectedPlayer.Parent then
                        local targetRoot = GetRoot(SelectedPlayer.Character)

                        if targetRoot then
                            LocalTeleport(
                                targetRoot.CFrame * CFrame.new(0, 0, 4)
                            )
                        end
                    end

                    task.wait(CONFIG.PlayerLoopInterval)
                end
            end)
        end
    end
)

MakeButton(PlayersPage, "Spectate Selected Player", function()
    if not SelectedPlayer or not SelectedPlayer.Parent then
        PlayerStatus.Text = "Select an available player first."
        return
    end

    local humanoid = GetHumanoid(SelectedPlayer.Character)

    if not humanoid then
        PlayerStatus.Text = "Target character is unavailable."
        return
    end

    local camera = workspace.CurrentCamera

    if camera then
        if not Spectating then
            OriginalCameraSubject = camera.CameraSubject
        end

        Spectating = true
        camera.CameraSubject = humanoid
        PlayerStatus.Text = "Spectating: " .. SelectedPlayer.Name
    end
end)

MakeButton(PlayersPage, "Stop Spectating", function()
    StopSpectating()
    PlayerStatus.Text = "Camera restored."
end)

--==================================================
-- TAB 3: ITEMS
--==================================================

local ItemsPage = AddTab(TAB_NAMES[3], 3)

MakeLabel(ItemsPage, "Item Display", 30, 18)

MakeLabel(
    ItemsPage,
    "Search BaseParts by name. Full paths help distinguish duplicate names.",
    45,
    11,
    CONFIG.Muted
)

local ItemSearch = MakeBox(
    ItemsPage,
    "Type an item name..."
)

local ItemStatus = MakeLabel(
    ItemsPage,
    "No item selected.",
    32,
    11,
    CONFIG.Muted
)

local ItemList = MakeList(ItemsPage, 190)

local function RefreshItems()
    ClearList(ItemList)
    table.clear(ItemButtons)
    SelectedItem = nil

    local query = string.lower(ItemSearch.Text)
    local root = GetItemRoot()
    local count = 0

    for _, object in ipairs(root:GetDescendants()) do
        if object:IsA("BasePart") then
            local fullPath = GetFullPath(object)

            local matches =
                query == ""
                or string.find(string.lower(object.Name), query, 1, true)
                or string.find(string.lower(fullPath), query, 1, true)

            if matches then
                count += 1

                if count > CONFIG.MaxItemResults then
                    break
                end

                local button = MakeButton(
                    ItemList,
                    object.Name .. "\n" .. fullPath,
                    function()
                        SelectedItem = object
                        ItemStatus.Text = "Selected: " .. fullPath

                        for _, otherButton in ipairs(ItemButtons) do
                            otherButton.BackgroundColor3 = CONFIG.Secondary
                        end

                        button.BackgroundColor3 = CONFIG.Accent
                    end,
                    48
                )

                table.insert(ItemButtons, button)
            end
        end
    end

    if count == 0 then
        MakeLabel(ItemList, "No matching BaseParts found.", 30, 11, CONFIG.Muted)
    elseif count > CONFIG.MaxItemResults then
        ItemStatus.Text = "Result limit reached. Refine your search."
    end
end

ItemSearch:GetPropertyChangedSignal("Text"):Connect(RefreshItems)

MakeButton(ItemsPage, "Refresh Item List", RefreshItems)

MakeButton(ItemsPage, "Teleport to Selected Item", function()
    if not SelectedItem or not SelectedItem.Parent then
        ItemStatus.Text = "Select an available item first."
        return
    end

    LocalTeleport(SelectedItem.CFrame * CFrame.new(0, 3, 0))
    ItemStatus.Text = "Moved to selected item's location."
end)

local ItemLoopButton

ItemLoopButton = MakeButton(
    ItemsPage,
    "Start Item Loop",
    function()
        ItemLoopEnabled = not ItemLoopEnabled

        ItemLoopButton.Text = ItemLoopEnabled
            and "Stop Item Loop"
            or "Start Item Loop"

        if ItemLoopEnabled then
            task.spawn(function()
                while ItemLoopEnabled do
                    if SelectedItem and SelectedItem.Parent then
                        LocalTeleport(
                            SelectedItem.CFrame * CFrame.new(0, 3, 0)
                        )
                    end

                    task.wait(CONFIG.ItemLoopInterval)
                end
            end)
        end
    end
)

MakeButton(ItemsPage, "Stop Item Loop", function()
    ItemLoopEnabled = false
    ItemLoopButton.Text = "Start Item Loop"
    ItemStatus.Text = "Item loop stopped."
end)

MakeLabel(
    ItemsPage,
    "Note: The item scanner searches BaseParts under the configured root. It does not identify whether an object is a collectible or whether the server will accept an interaction.",
    65,
    11,
    CONFIG.Muted
)

--==================================================
-- TAB 4: AREAS
--==================================================

local AreasPage = AddTab(TAB_NAMES[4], 4)

MakeLabel(AreasPage, "Area Display", 30, 18)

local CoordinateStatus = MakeLabel(
    AreasPage,
    "Position: unavailable",
    48,
    11,
    CONFIG.Muted
)

local CoordinateBox = MakeBox(
    AreasPage,
    "Coordinates will appear here...",
    42
)

local function GetPositionText()
    local root = GetRoot(GetCharacter())

    if not root then
        return "Character position unavailable."
    end

    local p = root.Position

    return string.format(
        "X: %.2f, Y: %.2f, Z: %.2f",
        p.X,
        p.Y,
        p.Z
    )
end

MakeButton(AreasPage, "Refresh Current Coordinates", function()
    local positionText = GetPositionText()

    CoordinateStatus.Text = positionText
    CoordinateBox.Text = positionText
end)

MakeLabel(AreasPage, "Save Current Position", 28, 15)

local PositionNameBox = MakeBox(
    AreasPage,
    "Enter a name for this position..."
)

local SavedList = MakeList(AreasPage, 150)

local function RefreshSavedPositions()
    ClearList(SavedList)
    table.clear(SavedPositionButtons)

    for index, entry in ipairs(SavedPositions) do
        local button = MakeButton(
            SavedList,
            entry.Name .. "\n" .. entry.PositionText,
            function()
                SelectedSavedPosition = index

                for _, otherButton in ipairs(SavedPositionButtons) do
                    otherButton.BackgroundColor3 = CONFIG.Secondary
                end

                button.BackgroundColor3 = CONFIG.Accent
            end,
            48
        )

        table.insert(SavedPositionButtons, button)
    end

    if #SavedPositions == 0 then
        MakeLabel(SavedList, "No saved positions yet.", 30, 11, CONFIG.Muted)
    end
end

MakeButton(AreasPage, "Save Current Position", function()
    local root = GetRoot(GetCharacter())

    if not root then
        CoordinateStatus.Text = "Character position unavailable."
        return
    end

    local name = PositionNameBox.Text

    if name == "" then
        name = "Position " .. tostring(#SavedPositions + 1)
    end

    table.insert(SavedPositions, {
        Name = name,
        CFrame = root.CFrame,
        PositionText = GetPositionText()
    })

    PositionNameBox.Text = ""
    RefreshSavedPositions()

    CoordinateStatus.Text = "Saved: " .. name
end)

MakeButton(AreasPage, "Teleport to Selected Saved Position", function()
    if not SelectedSavedPosition then
        CoordinateStatus.Text = "Select a saved position first."
        return
    end

    local entry = SavedPositions[SelectedSavedPosition]

    if entry then
        LocalTeleport(entry.CFrame)
        CoordinateStatus.Text = "Moved to: " .. entry.Name
    end
end)

MakeButton(AreasPage, "Delete Selected Position", function()
    if not SelectedSavedPosition then
        CoordinateStatus.Text = "Select a saved position first."
        return
    end

    table.remove(SavedPositions, SelectedSavedPosition)
    SelectedSavedPosition = nil

    RefreshSavedPositions()
    CoordinateStatus.Text = "Saved position deleted."
end)

MakeLabel(
    AreasPage,
    "Saved positions are stored in memory only. They will reset when the script is restarted.",
    48,
    11,
    CONFIG.Muted
)

--==================================================
-- TAB 5: SCRIPT LIBRARY
--==================================================

local LibraryPage = AddTab(TAB_NAMES[5], 5)

MakeLabel(LibraryPage, "Script Library", 30, 18)

MakeLabel(
    LibraryPage,
    "A reference collection for your trusted scripts. Add entries in the table below.",
    48,
    11,
    CONFIG.Muted
)

-- Add your own entries here.
-- These are references only. URLs are not automatically executed.

local LibraryEntries = {
    {
        Name = "Example Script",
        URL = "https://github.com/username/repository"
    },

    {
        Name = "Another Example",
        URL = "https://github.com/username/another-repository"
    }
}

local LibrarySearch = MakeBox(
    LibraryPage,
    "Search library..."
)

local LibraryList = MakeList(LibraryPage, 260)

local function RefreshLibrary()
    ClearList(LibraryList)

    local query = string.lower(LibrarySearch.Text)
    local count = 0

    for _, entry in ipairs(LibraryEntries) do
        if string.find(string.lower(entry.Name), query, 1, true)
            or string.find(string.lower(entry.URL), query, 1, true) then

            count += 1

            local button = MakeButton(
                LibraryList,
                entry.Name .. "\n" .. entry.URL,
                function()
                    -- Selectable reference display.
                    -- No URL execution is performed.
                    CoordinateStatus.Text = "Library selected: " .. entry.Name
                end,
                48
            )
        end
    end

    if count == 0 then
        MakeLabel(LibraryList, "No matching library entries.", 30, 11, CONFIG.Muted)
    end
end

LibrarySearch:GetPropertyChangedSignal("Text"):Connect(RefreshLibrary)

RefreshLibrary()

MakeLabel(
    LibraryPage,
    "To add a script, edit LibraryEntries near the top of the Script Library section. Change Name and URL for each entry.",
    70,
    11,
    CONFIG.Muted
)

--==================================================
-- TAB INITIALISATION
--==================================================

TabButtons["Introduction"].BackgroundColor3 = CONFIG.Accent
TabButtons["Introduction"].TextColor3 = CONFIG.Text
Pages["Introduction"].Visible = true

--==================================================
-- MINIMISE / RESTORE
--==================================================

local MiniButton = Create("TextButton", {
    Name = "MiniButton",
    AnchorPoint = Vector2.new(0.5, 0.5),
    Position = UDim2.fromScale(0.5, 0.5),
    Size = UDim2.fromOffset(56, 56),
    BackgroundColor3 = CONFIG.Accent,
    Text = "UH",
    TextColor3 = CONFIG.Text,
    Font = Enum.Font.GothamBold,
    TextSize = 16,
    Visible = false
}, ScreenGui)

Corner(MiniButton, 100)

MinimizeButton.Activated:Connect(function()
    Main.Visible = false
    MiniButton.Visible = true
end)

MiniButton.Activated:Connect(function()
    Main.Visible = true
    MiniButton.Visible = false
end)

--==================================================
-- DRAG SUPPORT: MOUSE AND TOUCH
--==================================================

local function MakeDraggable(handle, target)
    local dragging = false
    local dragStart
    local startPosition
    local activeInput

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then

            dragging = true
            activeInput = input
            dragStart = input.Position
            startPosition = target.Position
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not dragging then return end

        local matches =
            input == activeInput
            or input.UserInputType == Enum.UserInputType.MouseMovement

        if matches then
            local delta = input.Position - dragStart

            target.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if input == activeInput
            or input.UserInputType == Enum.UserInputType.MouseButton1 then

            dragging = false
            activeInput = nil
        end
    end)
end

MakeDraggable(Header, Main)
MakeDraggable(MiniButton, MiniButton)

--==================================================
-- RESPONSIVE SCREEN SIZE
--==================================================

local Camera = workspace.CurrentCamera

local function UpdateLayout()
    Camera = workspace.CurrentCamera
    if not Camera then return end

    local viewport = Camera.ViewportSize

    if viewport.X < 700 then
        Main.Size = UDim2.new(0.94, 0, 0.84, 0)
    else
        Main.Size = UDim2.new(0.82, 0, 0.78, 0)
    end
end

UpdateLayout()

if Camera then
    Camera:GetPropertyChangedSignal("ViewportSize"):Connect(UpdateLayout)
end

--==================================================
-- RESPAWN HANDLING
--==================================================

LocalPlayer.CharacterAdded:Connect(function(character)
    task.wait(0.5)

    if Spectating then
        StopSpectating()
    end
end)

--==================================================
-- INITIALISE
--==================================================

RefreshPlayers()
RefreshItems()
RefreshSavedPositions()

print("[Universal Hub] " .. CONFIG.Version .. " initialized.")
