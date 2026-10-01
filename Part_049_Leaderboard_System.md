# Part 49: ระบบ Leaderboard สมบูรณ์

## บทนำ

Leaderboard หรือตารางคะแนนเป็นฟีเจอร์ที่พบในเกมเกือบทุกประเภท ช่วยสร้าง engagement และการแข่งขัน ในบทนี้เราจะสร้างระบบ Leaderboard ที่สมบูรณ์ ทั้ง in-game leaderstats และ Global Leaderboard ด้วย OrderedDataStore

## ประเภทของ Leaderboard

1. **Session Leaderboard** - คะแนนในเกม session นั้นๆ รีเซ็ตทุก round
2. **Daily Leaderboard** - คะแนนสูงสุดในวันนี้
3. **Weekly/Monthly** - คะแนนในช่วงเวลา
4. **All-Time** - คะแนนสูงสุดตลอดกาล

## ระบบ Leaderstats (In-Game)

```lua
-- ServerScriptService/LeaderboardSystem.lua (Script)
-- ระบบ leaderstats ที่แสดงในหน้าต่างผู้เล่น

local Players = game:GetService("Players")
local DataStoreService = game:GetService("DataStoreService")

-- ===== การตั้งค่า Leaderboard =====
local LEADERBOARD_CONFIG = {
    -- แสดงค่าอะไรใน leaderstats
    stats = {
        { name = "💰 เหรียญ", valueName = "coins", valueType = "IntValue" },
        { name = "⭐ ระดับ", valueName = "level", valueType = "IntValue" },
        { name = "🏆 ชัยชนะ", valueName = "wins", valueType = "IntValue" },
        { name = "☠️ ฆ่าศัตรู", valueName = "kills", valueType = "IntValue" }
    }
}

-- ข้อมูลผู้เล่นในหน่วยความจำ
local playerStats = {}

-- ===== ตั้งค่า Leaderstats =====
local function setupLeaderstats(player)
    -- สร้าง leaderstats folder
    local leaderstats = Instance.new("Folder")
    leaderstats.Name = "leaderstats"
    leaderstats.Parent = player
    
    -- สร้างค่าแต่ละตัว
    local statsValues = {}
    
    for _, statConfig in ipairs(LEADERBOARD_CONFIG.stats) do
        local value = Instance.new(statConfig.valueType)
        value.Name = statConfig.name
        value.Value = 0
        value.Parent = leaderstats
        statsValues[statConfig.valueName] = value
    end
    
    return statsValues
end

-- ===== โหลดข้อมูล =====
local dataStore = DataStoreService:GetDataStore("PlayerStats_v1")

local function loadPlayerStats(player)
    local key = "Stats_" .. player.UserId
    
    local success, data = pcall(function()
        return dataStore:GetAsync(key)
    end)
    
    if success and data then
        return data
    else
        return {
            coins = 0,
            level = 1,
            wins = 0,
            kills = 0,
            deaths = 0,
            experience = 0,
            playTime = 0
        }
    end
end

local function savePlayerStats(player)
    local stats = playerStats[player.UserId]
    if not stats then return end
    
    local key = "Stats_" .. player.UserId
    
    pcall(function()
        dataStore:SetAsync(key, {
            coins = stats.coins,
            level = stats.level,
            wins = stats.wins,
            kills = stats.kills,
            deaths = stats.deaths,
            experience = stats.experience,
            playTime = stats.playTime
        })
    end)
end

-- ===== Event Handlers =====
Players.PlayerAdded:Connect(function(player)
    -- โหลดข้อมูล
    local data = loadPlayerStats(player)
    
    -- ตั้งค่า leaderstats
    local statsValues = setupLeaderstats(player)
    
    -- เก็บข้อมูลใน memory
    playerStats[player.UserId] = {
        -- Raw data
        coins = data.coins,
        level = data.level,
        wins = data.wins,
        kills = data.kills,
        deaths = data.deaths,
        experience = data.experience,
        playTime = data.playTime,
        joinTime = os.time(),
        
        -- References to IntValues
        _values = statsValues
    }
    
    -- อัพเดทค่าเริ่มต้น
    local stats = playerStats[player.UserId]
    if statsValues["coins"] then statsValues["coins"].Value = stats.coins end
    if statsValues["level"] then statsValues["level"].Value = stats.level end
    if statsValues["wins"] then statsValues["wins"].Value = stats.wins end
    if statsValues["kills"] then statsValues["kills"].Value = stats.kills end
end)

Players.PlayerRemoving:Connect(function(player)
    -- บันทึก play time
    local stats = playerStats[player.UserId]
    if stats then
        stats.playTime = stats.playTime + (os.time() - stats.joinTime)
        savePlayerStats(player)
    end
    
    playerStats[player.UserId] = nil
end)

-- ===== Public API =====
local LeaderboardSystem = {}

function LeaderboardSystem.addCoins(player, amount)
    local stats = playerStats[player.UserId]
    if not stats then return end
    
    stats.coins = math.max(0, stats.coins + amount)
    
    if stats._values["coins"] then
        stats._values["coins"].Value = stats.coins
    end
    
    return stats.coins
end

function LeaderboardSystem.addKill(player)
    local stats = playerStats[player.UserId]
    if not stats then return end
    
    stats.kills = stats.kills + 1
    
    if stats._values["kills"] then
        stats._values["kills"].Value = stats.kills
    end
    
    -- ให้ EXP สำหรับการฆ่า
    LeaderboardSystem.addExperience(player, 25)
    
    return stats.kills
end

function LeaderboardSystem.addWin(player)
    local stats = playerStats[player.UserId]
    if not stats then return end
    
    stats.wins = stats.wins + 1
    
    if stats._values["wins"] then
        stats._values["wins"].Value = stats.wins
    end
    
    LeaderboardSystem.addExperience(player, 100)
    LeaderboardSystem.addCoins(player, 50)
    
    return stats.wins
end

function LeaderboardSystem.addExperience(player, amount)
    local stats = playerStats[player.UserId]
    if not stats then return end
    
    stats.experience = stats.experience + amount
    
    -- Check level up
    local expRequired = math.floor(100 * (1.5 ^ (stats.level - 1)))
    
    local levelsGained = 0
    while stats.experience >= expRequired do
        stats.experience = stats.experience - expRequired
        stats.level = stats.level + 1
        expRequired = math.floor(100 * (1.5 ^ (stats.level - 1)))
        levelsGained = levelsGained + 1
        
        -- Level up rewards
        LeaderboardSystem.addCoins(player, stats.level * 10)
    end
    
    if levelsGained > 0 then
        if stats._values["level"] then
            stats._values["level"].Value = stats.level
        end
        print(string.format("⭐ %s เลื่อนระดับ -> Level %d!", player.Name, stats.level))
    end
    
    return levelsGained
end

function LeaderboardSystem.getStats(player)
    return playerStats[player.UserId]
end

game:BindToClose(function()
    for _, player in ipairs(Players:GetPlayers()) do
        savePlayerStats(player)
    end
    task.wait(2)
end)

return LeaderboardSystem
```

## Global Leaderboard ด้วย OrderedDataStore

```lua
-- ServerScriptService/GlobalLeaderboard.lua (ModuleScript)
-- ระบบ Global Leaderboard

local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local GlobalLeaderboard = {}

-- OrderedDataStores สำหรับแต่ละ category
local leaderboards = {
    coins = DataStoreService:GetOrderedDataStore("GlobalCoins"),
    wins = DataStoreService:GetOrderedDataStore("GlobalWins"),
    kills = DataStoreService:GetOrderedDataStore("GlobalKills"),
    level = DataStoreService:GetOrderedDataStore("GlobalLevel")
}

-- Cache สำหรับ username
local usernameCache = {}

local function getUsernameFromId(userId)
    if usernameCache[userId] then
        return usernameCache[userId]
    end
    
    local success, username = pcall(function()
        return Players:GetNameFromUserIdAsync(userId)
    end)
    
    if success then
        usernameCache[userId] = username
        return username
    end
    
    return "Player_" .. userId
end

-- อัพเดทคะแนนใน leaderboard
function GlobalLeaderboard.updateScore(userId, category, score)
    local lb = leaderboards[category]
    if not lb then
        warn("Leaderboard category ไม่พบ: " .. category)
        return
    end
    
    local key = tostring(userId)
    
    local success, err = pcall(function()
        lb:SetAsync(key, math.floor(score))
    end)
    
    if not success then
        warn("อัพเดท Global Leaderboard ล้มเหลว: " .. err)
    end
end

-- ดึงอันดับสูงสุด
function GlobalLeaderboard.getTopPlayers(category, count)
    local lb = leaderboards[category]
    if not lb then return {} end
    
    count = math.min(count or 10, 100)
    
    local success, pages = pcall(function()
        return lb:GetSortedAsync(true, count)  -- true = descending
    end)
    
    if not success then
        warn("ดึง Leaderboard ล้มเหลว")
        return {}
    end
    
    local results = {}
    local currentPage = pages:GetCurrentPage()
    
    for rank, entry in ipairs(currentPage) do
        local userId = tonumber(entry.key)
        local username = userId and getUsernameFromId(userId) or "Unknown"
        
        table.insert(results, {
            rank = rank,
            userId = userId,
            username = username,
            score = entry.value
        })
    end
    
    return results
end

-- ดึงอันดับของผู้เล่นคนนี้
function GlobalLeaderboard.getPlayerRank(userId, category)
    local lb = leaderboards[category]
    if not lb then return nil end
    
    local key = tostring(userId)
    
    -- ต้องดึงทุก page เพื่อหาอันดับ (ช้า แนะนำใช้ Cache)
    -- ในระบบจริงควรเก็บ rank แยกต่างหาก
    local success, score = pcall(function()
        return lb:GetAsync(key)
    end)
    
    if success and score then
        return score  -- คืนคะแนน (Roblox ไม่มี built-in rank query)
    end
    
    return nil
end

return GlobalLeaderboard
```

## UI Leaderboard - แสดงบนหน้าจอ

```lua
-- LocalScript ใน StarterGui/LeaderboardGui
-- แสดง Leaderboard บนหน้าจอ

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player.PlayerGui

-- RemoteFunction เพื่อขอข้อมูล leaderboard
local getLeaderboardRF = ReplicatedStorage:WaitForChild("Functions"):WaitForChild("GetLeaderboard")

-- ===== สร้าง UI =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "LeaderboardGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

-- Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 350, 0, 450)
mainFrame.Position = UDim2.new(1, -370, 0.5, -225)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
mainFrame.BackgroundTransparency = 0.1
mainFrame.Visible = false
mainFrame.Parent = screenGui

-- Rounded corners
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = mainFrame

-- Border
local stroke = Instance.new("UIStroke")
stroke.Color = Color3.fromRGB(255, 215, 0)
stroke.Thickness = 2
stroke.Parent = mainFrame

-- Title Bar
local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 50)
titleBar.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
titleBar.BackgroundTransparency = 0.1
titleBar.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 12)
titleCorner.Parent = titleBar

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 1, 0)
titleLabel.Text = "🏆 LEADERBOARD"
titleLabel.TextSize = 20
titleLabel.TextColor3 = Color3.fromRGB(15, 15, 25)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.BackgroundTransparency = 1
titleLabel.Parent = titleBar

-- Category Tabs
local tabFrame = Instance.new("Frame")
tabFrame.Size = UDim2.new(1, -20, 0, 35)
tabFrame.Position = UDim2.new(0, 10, 0, 55)
tabFrame.BackgroundTransparency = 1
tabFrame.Parent = mainFrame

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 5)
tabLayout.Parent = tabFrame

local categories = {"💰 เหรียญ", "🏆 ชัยชนะ", "☠️ ฆ่า"}
local categoryIds = {"coins", "wins", "kills"}
local selectedCategory = "coins"
local tabButtons = {}

for i, catName in ipairs(categories) do
    local tab = Instance.new("TextButton")
    tab.Size = UDim2.new(0, 100, 1, 0)
    tab.Text = catName
    tab.TextSize = 12
    tab.Font = Enum.Font.Gotham
    tab.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    tab.TextColor3 = Color3.new(0.7, 0.7, 0.7)
    
    local tabCorner = Instance.new("UICorner")
    tabCorner.CornerRadius = UDim.new(0, 6)
    tabCorner.Parent = tab
    
    tab.Parent = tabFrame
    tabButtons[categoryIds[i]] = tab
    
    tab.MouseButton1Click:Connect(function()
        selectedCategory = categoryIds[i]
        
        -- อัพเดท tab appearance
        for catId, btn in pairs(tabButtons) do
            if catId == selectedCategory then
                btn.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
                btn.TextColor3 = Color3.fromRGB(15, 15, 25)
            else
                btn.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
                btn.TextColor3 = Color3.new(0.7, 0.7, 0.7)
            end
        end
        
        loadLeaderboard(selectedCategory)
    end)
end

-- Scroll Frame สำหรับรายการ
local scrollFrame = Instance.new("ScrollingFrame")
scrollFrame.Size = UDim2.new(1, -20, 1, -110)
scrollFrame.Position = UDim2.new(0, 10, 0, 100)
scrollFrame.BackgroundTransparency = 1
scrollFrame.ScrollBarThickness = 4
scrollFrame.ScrollBarImageColor3 = Color3.fromRGB(255, 215, 0)
scrollFrame.Parent = mainFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 5)
listLayout.Parent = scrollFrame

-- ===== ฟังก์ชัน =====
local function createPlayerRow(rank, username, score, isCurrentPlayer)
    local row = Instance.new("Frame")
    row.Size = UDim2.new(1, 0, 0, 45)
    row.BackgroundColor3 = isCurrentPlayer and 
        Color3.fromRGB(50, 40, 0) or 
        Color3.fromRGB(25, 25, 40)
    
    local rowCorner = Instance.new("UICorner")
    rowCorner.CornerRadius = UDim.new(0, 8)
    rowCorner.Parent = row
    
    if isCurrentPlayer then
        local rowStroke = Instance.new("UIStroke")
        rowStroke.Color = Color3.fromRGB(255, 215, 0)
        rowStroke.Thickness = 1
        rowStroke.Parent = row
    end
    
    -- Rank
    local rankLabel = Instance.new("TextLabel")
    rankLabel.Size = UDim2.new(0, 40, 1, 0)
    rankLabel.Position = UDim2.new(0, 5, 0, 0)
    rankLabel.BackgroundTransparency = 1
    rankLabel.TextColor3 = rank <= 3 and 
        Color3.fromRGB(255, 215, 0) or 
        Color3.new(0.8, 0.8, 0.8)
    rankLabel.Font = Enum.Font.GothamBold
    rankLabel.TextSize = 16
    
    if rank == 1 then
        rankLabel.Text = "🥇"
    elseif rank == 2 then
        rankLabel.Text = "🥈"
    elseif rank == 3 then
        rankLabel.Text = "🥉"
    else
        rankLabel.Text = "#" .. rank
    end
    
    rankLabel.Parent = row
    
    -- Username
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, -120, 1, 0)
    nameLabel.Position = UDim2.new(0, 50, 0, 0)
    nameLabel.Text = username
    nameLabel.TextXAlignment = Enum.TextXAlignment.Left
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.new(1, 1, 1)
    nameLabel.Font = isCurrentPlayer and Enum.Font.GothamBold or Enum.Font.Gotham
    nameLabel.TextSize = 14
    nameLabel.Parent = row
    
    -- Score
    local scoreLabel = Instance.new("TextLabel")
    scoreLabel.Size = UDim2.new(0, 80, 1, 0)
    scoreLabel.Position = UDim2.new(1, -90, 0, 0)
    scoreLabel.BackgroundTransparency = 1
    scoreLabel.TextColor3 = Color3.fromRGB(255, 215, 0)
    scoreLabel.Font = Enum.Font.GothamBold
    scoreLabel.TextSize = 16
    
    -- Format score
    if score >= 1000000 then
        scoreLabel.Text = string.format("%.1fM", score/1000000)
    elseif score >= 1000 then
        scoreLabel.Text = string.format("%.1fK", score/1000)
    else
        scoreLabel.Text = tostring(score)
    end
    
    scoreLabel.Parent = row
    
    return row
end

local function loadLeaderboard(category)
    -- Clear existing rows
    for _, child in ipairs(scrollFrame:GetChildren()) do
        if child:IsA("Frame") then
            child:Destroy()
        end
    end
    
    -- Loading indicator
    local loadingLabel = Instance.new("TextLabel")
    loadingLabel.Size = UDim2.new(1, 0, 0, 40)
    loadingLabel.Text = "⏳ กำลังโหลด..."
    loadingLabel.TextSize = 14
    loadingLabel.TextColor3 = Color3.new(0.7, 0.7, 0.7)
    loadingLabel.BackgroundTransparency = 1
    loadingLabel.Parent = scrollFrame
    
    -- โหลดข้อมูล
    task.spawn(function()
        local success, data = pcall(function()
            return getLeaderboardRF:InvokeServer(category, 20)
        end)
        
        loadingLabel:Destroy()
        
        if success and data then
            for _, entry in ipairs(data) do
                local isMe = entry.userId == player.UserId
                local row = createPlayerRow(
                    entry.rank,
                    entry.username,
                    entry.score,
                    isMe
                )
                row.Parent = scrollFrame
            end
            
            -- อัพเดท canvas size
            scrollFrame.CanvasSize = UDim2.new(0, 0, 0, listLayout.AbsoluteContentSize.Y + 10)
        else
            local errorLabel = Instance.new("TextLabel")
            errorLabel.Size = UDim2.new(1, 0, 0, 40)
            errorLabel.Text = "❌ โหลดไม่สำเร็จ"
            errorLabel.TextSize = 14
            errorLabel.TextColor3 = Color3.fromRGB(255, 100, 100)
            errorLabel.BackgroundTransparency = 1
            errorLabel.Parent = scrollFrame
        end
    end)
end

-- ===== เปิด/ปิด Leaderboard =====
local isOpen = false
local openButton = Instance.new("TextButton")
openButton.Size = UDim2.new(0, 40, 0, 40)
openButton.Position = UDim2.new(1, -55, 0.5, -20)
openButton.Text = "🏆"
openButton.TextSize = 20
openButton.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
openButton.Parent = screenGui

local openCorner = Instance.new("UICorner")
openCorner.CornerRadius = UDim.new(1, 0)
openCorner.Parent = openButton

openButton.MouseButton1Click:Connect(function()
    isOpen = not isOpen
    
    if isOpen then
        mainFrame.Visible = true
        loadLeaderboard(selectedCategory)
        
        -- Animate in
        mainFrame.Position = UDim2.new(1, 0, 0.5, -225)
        TweenService:Create(mainFrame, TweenInfo.new(0.3), {
            Position = UDim2.new(1, -370, 0.5, -225)
        }):Play()
    else
        -- Animate out
        TweenService:Create(mainFrame, TweenInfo.new(0.3), {
            Position = UDim2.new(1, 0, 0.5, -225)
        }):Play()
        task.delay(0.3, function()
            mainFrame.Visible = false
        end)
    end
end)
```

## Server: GetLeaderboard RemoteFunction

```lua
-- ServerScriptService/LeaderboardServer.lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local getLeaderboardRF = ReplicatedStorage:WaitForChild("Functions"):WaitForChild("GetLeaderboard")

local leaderboards = {
    coins = DataStoreService:GetOrderedDataStore("GlobalCoins"),
    wins = DataStoreService:GetOrderedDataStore("GlobalWins"),
    kills = DataStoreService:GetOrderedDataStore("GlobalKills")
}

local usernameCache = {}

local function getUsername(userId)
    if usernameCache[userId] then
        return usernameCache[userId]
    end
    
    local ok, name = pcall(function()
        return Players:GetNameFromUserIdAsync(userId)
    end)
    
    local username = ok and name or ("Player_" .. userId)
    usernameCache[userId] = username
    return username
end

getLeaderboardRF.OnServerInvoke = function(player, category, count)
    local lb = leaderboards[category or "coins"]
    if not lb then return {} end
    
    count = math.min(count or 10, 50)
    
    local ok, pages = pcall(function()
        return lb:GetSortedAsync(true, count)
    end)
    
    if not ok then return {} end
    
    local results = {}
    for rank, entry in ipairs(pages:GetCurrentPage()) do
        local userId = tonumber(entry.key)
        local username = userId and getUsername(userId) or "Unknown"
        
        table.insert(results, {
            rank = rank,
            userId = userId,
            username = username,
            score = entry.value
        })
    end
    
    return results
end
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Session Leaderboard
สร้าง leaderboard ที่ reset ทุก round (ไม่ใช้ DataStore)

### แบบฝึกหัดที่ 2: Daily Reset
สร้าง daily leaderboard ที่ reset ทุกเที่ยงคืน

### แบบฝึกหัดที่ 3: Achievement Points
เพิ่ม achievement system ที่ให้คะแนนสะสม

## สรุป

ระบบ Leaderboard ที่ดีต้องมี:
- **Leaderstats** สำหรับ in-game display
- **OrderedDataStore** สำหรับ global ranking
- **UI ที่สวยงาม** ดึงดูดผู้เล่น
- **Real-time updates** เมื่อคะแนนเปลี่ยน
- **Performance** ไม่ดึงข้อมูลบ่อยเกิน
