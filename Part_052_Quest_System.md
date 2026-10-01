# Part 52: ระบบ Quest/Mission สมบูรณ์

## บทนำ

Quest System หรือระบบภารกิจเป็นหนึ่งในฟีเจอร์ที่สำคัญที่สุดในการสร้าง engagement ให้ผู้เล่น ภารกิจที่ดีจะทำให้ผู้เล่นมีเป้าหมายและแรงจูงใจในการเล่น

## ประเภทของ Quest

1. **Main Quest** - เนื้อเรื่องหลัก
2. **Side Quest** - เนื้อเรื่องย่อย
3. **Daily Quest** - ภารกิจประจำวัน
4. **Weekly Quest** - ภารกิจประจำสัปดาห์
5. **Repeatable Quest** - ทำซ้ำได้

## โครงสร้างข้อมูล Quest

```lua
-- ตัวอย่างโครงสร้าง Quest
local questExample = {
    id = "quest_001",
    name = "นักรบมือใหม่",
    description = "พิสูจน์ตัวเองด้วยการเอาชนะศัตรู",
    type = "main",
    
    -- เงื่อนไขรับ Quest
    requirements = {
        level = 1,
        completedQuests = {},
    },
    
    -- วัตถุประสงค์
    objectives = {
        {
            id = "kill_slimes",
            type = "kill",
            target = "Slime",
            required = 5,
            current = 0,
            description = "ฆ่า Slime 5 ตัว"
        },
        {
            id = "collect_items",
            type = "collect",
            target = "slime_jelly",
            required = 3,
            current = 0,
            description = "เก็บ Slime Jelly 3 ชิ้น"
        }
    },
    
    -- รางวัล
    rewards = {
        coins = 200,
        experience = 150,
        items = {
            { id = "potion_hp", quantity = 3 }
        }
    },
    
    -- สถานะ
    status = "available",  -- available, active, completed, failed
    startTime = nil,
    completedTime = nil,
    
    -- พิเศษ
    timeLimit = nil,  -- nil = ไม่มีกำหนดเวลา
    repeatable = false
}
```

## Quest Database

```lua
-- ServerStorage/Quests/QuestDatabase.lua (ModuleScript)

local QuestDatabase = {}

local questData = {
    -- ===== Main Quests =====
    {
        id = "main_001",
        name = "🗡️ การเดินทางเริ่มต้น",
        description = "ออกเดินทางเพื่อพิสูจน์ตัวเองในโลกกว้าง",
        type = "main",
        category = "story",
        
        requirements = {
            level = 1,
        },
        
        objectives = {
            {
                id = "talk_to_elder",
                type = "talk",
                target = "ElderNPC",
                required = 1,
                description = "พูดคุยกับผู้อาวุโส"
            },
            {
                id = "kill_slimes",
                type = "kill",
                target = "Slime",
                required = 5,
                description = "ฆ่า Slime 5 ตัว"
            },
            {
                id = "return_to_elder",
                type = "talk",
                target = "ElderNPC",
                required = 1,
                description = "รายงานกลับผู้อาวุโส",
                unlockAfter = "kill_slimes"  -- ต้องทำหลัง kill_slimes
            }
        },
        
        rewards = {
            experience = 200,
            coins = 100,
            items = {
                { id = "sword_basic", quantity = 1 }
            }
        }
    },
    
    {
        id = "main_002",
        name = "⚔️ ภัยคุกคามจากป่า",
        description = "Goblin กำลังโจมตีหมู่บ้าน!",
        type = "main",
        category = "story",
        
        requirements = {
            level = 5,
            completedQuests = { "main_001" }
        },
        
        objectives = {
            {
                id = "kill_goblins",
                type = "kill",
                target = "Goblin",
                required = 10,
                description = "ฆ่า Goblin 10 ตัว"
            },
            {
                id = "kill_goblin_chief",
                type = "kill",
                target = "GoblinChief",
                required = 1,
                description = "กำจัดหัวหน้า Goblin",
                unlockAfter = "kill_goblins"
            }
        },
        
        rewards = {
            experience = 500,
            coins = 300,
            items = {
                { id = "armor_leather", quantity = 1 },
                { id = "potion_hp", quantity = 5 }
            }
        }
    },
    
    -- ===== Daily Quests =====
    {
        id = "daily_001",
        name = "💰 นักสะสมรายวัน",
        description = "เก็บเหรียญให้ครบตามเป้า",
        type = "daily",
        category = "daily",
        repeatable = true,
        resetTime = "daily",
        
        requirements = { level = 1 },
        
        objectives = {
            {
                id = "collect_coins",
                type = "collect_coins",
                required = 1000,
                description = "เก็บเหรียญ 1,000 เหรียญ"
            }
        },
        
        rewards = {
            experience = 100,
            coins = 200,
            gems = 1
        }
    },
    
    {
        id = "daily_002",
        name = "☠️ นักล่า",
        description = "ฆ่าศัตรูให้ครบ",
        type = "daily",
        repeatable = true,
        resetTime = "daily",
        
        requirements = { level = 3 },
        
        objectives = {
            {
                id = "kill_enemies",
                type = "kill",
                target = "any",
                required = 20,
                description = "ฆ่าศัตรู 20 ตัว"
            }
        },
        
        rewards = {
            experience = 150,
            coins = 100
        }
    },
    
    -- ===== Side Quests =====
    {
        id = "side_001",
        name = "💊 นักเก็บยา",
        description = "ช่วยหมอหาส่วนผสมยา",
        type = "side",
        timeLimit = 3600,  -- 1 ชั่วโมง
        
        requirements = { level = 2 },
        
        objectives = {
            {
                id = "gather_herbs",
                type = "gather",
                target = "HerbPlant",
                required = 10,
                description = "เก็บสมุนไพร 10 ต้น"
            }
        },
        
        rewards = {
            experience = 120,
            coins = 80,
            items = {
                { id = "potion_hp", quantity = 5 },
                { id = "potion_mp", quantity = 5 }
            }
        }
    }
}

-- สร้าง lookup
local questLookup = {}
for _, quest in ipairs(questData) do
    questLookup[quest.id] = quest
end

function QuestDatabase.getQuest(id)
    return questLookup[id]
end

function QuestDatabase.getAllQuests()
    return questData
end

function QuestDatabase.getQuestsByType(questType)
    local results = {}
    for _, quest in ipairs(questData) do
        if quest.type == questType then
            table.insert(results, quest)
        end
    end
    return results
end

function QuestDatabase.getAvailableQuests(playerData)
    local available = {}
    
    for _, quest in ipairs(questData) do
        if QuestDatabase.canAcceptQuest(playerData, quest.id) then
            table.insert(available, quest)
        end
    end
    
    return available
end

function QuestDatabase.canAcceptQuest(playerData, questId)
    local quest = questLookup[questId]
    if not quest then return false end
    
    -- ตรวจสอบ level
    if quest.requirements.level and playerData.level < quest.requirements.level then
        return false
    end
    
    -- ตรวจสอบว่าทำแล้วหรือยัง
    if not quest.repeatable then
        local completed = playerData.completedQuests or {}
        for _, id in ipairs(completed) do
            if id == questId then return false end
        end
    end
    
    -- ตรวจสอบ prerequisite quests
    if quest.requirements.completedQuests then
        local completed = playerData.completedQuests or {}
        local completedSet = {}
        for _, id in ipairs(completed) do
            completedSet[id] = true
        end
        
        for _, required in ipairs(quest.requirements.completedQuests) do
            if not completedSet[required] then
                return false
            end
        end
    end
    
    return true
end

return QuestDatabase
```

## Quest Manager (Server)

```lua
-- ServerStorage/Quests/QuestManager.lua (ModuleScript)

local QuestDatabase = require(script.Parent.QuestDatabase)

local QuestManager = {}

-- Deep copy
local function deepCopy(t)
    if type(t) ~= "table" then return t end
    local copy = {}
    for k, v in pairs(t) do
        copy[k] = deepCopy(v)
    end
    return copy
end

-- รับ Quest
function QuestManager.acceptQuest(playerData, questId)
    -- ตรวจสอบ
    if not QuestDatabase.canAcceptQuest(playerData, questId) then
        return false, "ไม่สามารถรับ Quest นี้ได้"
    end
    
    -- ตรวจสอบว่ากำลังทำอยู่หรือยัง
    local activeQuests = playerData.activeQuests or {}
    for _, activeQuest in ipairs(activeQuests) do
        if activeQuest.id == questId then
            return false, "กำลังทำ Quest นี้อยู่แล้ว"
        end
    end
    
    -- ตรวจสอบ max active quests
    if #activeQuests >= 10 then
        return false, "รับ Quest ได้สูงสุด 10 Quest พร้อมกัน"
    end
    
    -- สร้าง quest instance
    local questTemplate = QuestDatabase.getQuest(questId)
    local questInstance = deepCopy(questTemplate)
    questInstance.status = "active"
    questInstance.startTime = os.time()
    questInstance.timeRemaining = questTemplate.timeLimit
    
    -- Reset progress
    for _, objective in ipairs(questInstance.objectives) do
        objective.current = 0
    end
    
    -- เพิ่มใน active
    if not playerData.activeQuests then
        playerData.activeQuests = {}
    end
    table.insert(playerData.activeQuests, questInstance)
    
    return true, string.format("รับ Quest '%s' แล้ว!", questTemplate.name)
end

-- อัพเดท progress
function QuestManager.updateProgress(playerData, eventType, eventData)
    local activeQuests = playerData.activeQuests or {}
    local completedQuests = {}
    
    for i, quest in ipairs(activeQuests) do
        if quest.status == "active" then
            local updated = false
            
            for _, objective in ipairs(quest.objectives) do
                if objective.current >= objective.required then
                    continue  -- ทำสำเร็จแล้ว
                end
                
                -- ตรวจสอบว่า objective ถูก unlock แล้วหรือยัง
                if objective.unlockAfter then
                    local parentDone = false
                    for _, obj in ipairs(quest.objectives) do
                        if obj.id == objective.unlockAfter and 
                           obj.current >= obj.required then
                            parentDone = true
                            break
                        end
                    end
                    if not parentDone then continue end
                end
                
                -- Match event กับ objective
                local matched = false
                
                if eventType == "kill" and objective.type == "kill" then
                    if objective.target == "any" or objective.target == eventData.enemy then
                        matched = true
                    end
                elseif eventType == "collect" and objective.type == "collect" then
                    if objective.target == eventData.itemId then
                        matched = true
                    end
                elseif eventType == "talk" and objective.type == "talk" then
                    if objective.target == eventData.npcId then
                        matched = true
                    end
                elseif eventType == "gather" and objective.type == "gather" then
                    if objective.target == eventData.objectId then
                        matched = true
                    end
                elseif eventType == "collect_coins" and objective.type == "collect_coins" then
                    matched = true
                    objective.current = math.min(
                        objective.required,
                        objective.current + (eventData.amount or 0)
                    )
                end
                
                if matched and eventType ~= "collect_coins" then
                    objective.current = math.min(
                        objective.required,
                        objective.current + (eventData.count or 1)
                    )
                    updated = true
                end
            end
            
            -- ตรวจสอบว่าทำสำเร็จหรือยัง
            if QuestManager.isQuestCompleted(quest) then
                quest.status = "completed"
                quest.completedTime = os.time()
                table.insert(completedQuests, quest)
            end
        end
    end
    
    return completedQuests
end

-- ตรวจสอบว่า Quest สำเร็จหรือยัง
function QuestManager.isQuestCompleted(quest)
    for _, objective in ipairs(quest.objectives) do
        if objective.current < objective.required then
            return false
        end
    end
    return true
end

-- รับรางวัล Quest
function QuestManager.claimReward(playerData, questId)
    local activeQuests = playerData.activeQuests or {}
    local questIndex = nil
    local questData = nil
    
    for i, quest in ipairs(activeQuests) do
        if quest.id == questId and quest.status == "completed" then
            questIndex = i
            questData = quest
            break
        end
    end
    
    if not questIndex then
        return false, "Quest ยังไม่สำเร็จหรือรับรางวัลไปแล้ว"
    end
    
    local template = QuestDatabase.getQuest(questId)
    if not template then return false, "ไม่พบ Quest" end
    
    -- ให้รางวัล
    local rewards = template.rewards
    local rewardMessages = {}
    
    if rewards.experience then
        playerData.experience = (playerData.experience or 0) + rewards.experience
        table.insert(rewardMessages, "+" .. rewards.experience .. " EXP")
    end
    
    if rewards.coins then
        playerData.coins = (playerData.coins or 0) + rewards.coins
        table.insert(rewardMessages, "+" .. rewards.coins .. " 💰")
    end
    
    if rewards.gems then
        playerData.gems = (playerData.gems or 0) + rewards.gems
        table.insert(rewardMessages, "+" .. rewards.gems .. " 💎")
    end
    
    if rewards.items then
        if not playerData.inventory then
            playerData.inventory = {}
        end
        for _, reward in ipairs(rewards.items) do
            table.insert(playerData.inventory, {
                id = reward.id,
                quantity = reward.quantity
            })
            table.insert(rewardMessages, "+" .. reward.id .. " x" .. reward.quantity)
        end
    end
    
    -- ลบออกจาก active
    table.remove(activeQuests, questIndex)
    
    -- เพิ่มใน completed
    if not playerData.completedQuests then
        playerData.completedQuests = {}
    end
    
    if not template.repeatable then
        table.insert(playerData.completedQuests, questId)
    end
    
    return true, "รับรางวัลแล้ว: " .. table.concat(rewardMessages, ", ")
end

-- อัพเดท time limit
function QuestManager.updateTimers(playerData, deltaTime)
    local activeQuests = playerData.activeQuests or {}
    local failedQuests = {}
    
    for _, quest in ipairs(activeQuests) do
        if quest.status == "active" and quest.timeRemaining then
            quest.timeRemaining = quest.timeRemaining - deltaTime
            
            if quest.timeRemaining <= 0 then
                quest.status = "failed"
                table.insert(failedQuests, quest)
            end
        end
    end
    
    return failedQuests
end

-- ดึงข้อมูล Quest สำหรับแสดง UI
function QuestManager.getQuestDisplayData(quest)
    local display = {
        id = quest.id,
        name = quest.name,
        description = quest.description,
        type = quest.type,
        status = quest.status,
        timeRemaining = quest.timeRemaining,
        objectives = {}
    }
    
    for _, obj in ipairs(quest.objectives) do
        table.insert(display.objectives, {
            id = obj.id,
            description = obj.description,
            current = obj.current,
            required = obj.required,
            completed = obj.current >= obj.required
        })
    end
    
    return display
end

return QuestManager
```

## Quest UI (Client)

```lua
-- LocalScript ใน StarterGui
-- แสดง Quest Log

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer

-- ===== Quest Log UI =====
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "QuestGui"
screenGui.Parent = player.PlayerGui

-- Quest Tracker (แสดงบนหน้าจอตลอดเวลา)
local tracker = Instance.new("Frame")
tracker.Size = UDim2.new(0, 280, 0, 200)
tracker.Position = UDim2.new(1, -290, 0, 100)
tracker.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
tracker.BackgroundTransparency = 0.3
tracker.Parent = screenGui
Instance.new("UICorner", tracker).CornerRadius = UDim.new(0, 10)

local trackerStroke = Instance.new("UIStroke")
trackerStroke.Color = Color3.fromRGB(255, 200, 0)
trackerStroke.Thickness = 1
trackerStroke.Parent = tracker

local trackerTitle = Instance.new("TextLabel")
trackerTitle.Size = UDim2.new(1, 0, 0, 25)
trackerTitle.Text = "📋 Quest"
trackerTitle.TextSize = 14
trackerTitle.Font = Enum.Font.GothamBold
trackerTitle.TextColor3 = Color3.fromRGB(255, 200, 0)
trackerTitle.BackgroundTransparency = 1
trackerTitle.Parent = tracker

local trackerContent = Instance.new("Frame")
trackerContent.Size = UDim2.new(1, -10, 1, -30)
trackerContent.Position = UDim2.new(0, 5, 0, 28)
trackerContent.BackgroundTransparency = 1
trackerContent.Parent = tracker

local trackerLayout = Instance.new("UIListLayout")
trackerLayout.Padding = UDim.new(0, 3)
trackerLayout.Parent = trackerContent

-- Quest Log Window
local questLog = Instance.new("Frame")
questLog.Size = UDim2.new(0, 600, 0, 450)
questLog.Position = UDim2.new(0.5, -300, 0.5, -225)
questLog.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
questLog.Visible = false
questLog.Parent = screenGui
Instance.new("UICorner", questLog).CornerRadius = UDim.new(0, 16)

-- ฟังก์ชันอัพเดท Quest Tracker
local function updateQuestTracker(activeQuests)
    -- ลบ entries เก่า
    for _, child in ipairs(trackerContent:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end
    
    if #activeQuests == 0 then
        local noQuest = Instance.new("TextLabel")
        noQuest.Size = UDim2.new(1, 0, 0, 30)
        noQuest.Text = "ไม่มี Quest กำลังทำ"
        noQuest.TextSize = 12
        noQuest.TextColor3 = Color3.new(0.5, 0.5, 0.5)
        noQuest.BackgroundTransparency = 1
        noQuest.Parent = trackerContent
        return
    end
    
    -- แสดงสูงสุด 3 quests
    local showCount = math.min(3, #activeQuests)
    
    for i = 1, showCount do
        local quest = activeQuests[i]
        
        local questEntry = Instance.new("Frame")
        questEntry.Size = UDim2.new(1, 0, 0, 0)  -- จะปรับ auto
        questEntry.BackgroundTransparency = 1
        questEntry.Parent = trackerContent
        
        local entryLayout = Instance.new("UIListLayout")
        entryLayout.Parent = questEntry
        
        local questName = Instance.new("TextLabel")
        questName.Size = UDim2.new(1, 0, 0, 18)
        questName.Text = "▶ " .. quest.name
        questName.TextSize = 12
        questName.Font = Enum.Font.GothamBold
        questName.TextColor3 = Color3.fromRGB(255, 215, 0)
        questName.TextXAlignment = Enum.TextXAlignment.Left
        questName.BackgroundTransparency = 1
        questName.Parent = questEntry
        
        -- แสดง objectives ที่ยังไม่เสร็จ
        for _, obj in ipairs(quest.objectives) do
            if obj.current < obj.required then
                local objLabel = Instance.new("TextLabel")
                objLabel.Size = UDim2.new(1, 0, 0, 15)
                objLabel.Text = string.format("  • %s (%d/%d)", 
                    obj.description, obj.current, obj.required)
                objLabel.TextSize = 10
                objLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
                objLabel.TextXAlignment = Enum.TextXAlignment.Left
                objLabel.BackgroundTransparency = 1
                objLabel.Parent = questEntry
                
                -- Progress bar
                local progressBg = Instance.new("Frame")
                progressBg.Size = UDim2.new(1, -10, 0, 4)
                progressBg.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
                Instance.new("UICorner", progressBg).CornerRadius = UDim.new(1,0)
                progressBg.Parent = questEntry
                
                local progress = Instance.new("Frame")
                local pct = math.min(1, obj.current / obj.required)
                progress.Size = UDim2.new(pct, 0, 1, 0)
                progress.BackgroundColor3 = Color3.fromRGB(0, 200, 0)
                Instance.new("UICorner", progress).CornerRadius = UDim.new(1,0)
                progress.Parent = progressBg
                
                break  -- แสดงแค่ objective แรกที่ยังไม่เสร็จ
            end
        end
        
        questEntry.Size = UDim2.new(1, 0, 0, entryLayout.AbsoluteContentSize.Y)
    end
    
    tracker.Size = UDim2.new(0, 280, 0, trackerLayout.AbsoluteContentSize.Y + 35)
end

-- ===== Quest Notification =====
local function showQuestNotification(message, notifType)
    local notif = Instance.new("Frame")
    notif.Size = UDim2.new(0, 300, 0, 60)
    notif.Position = UDim2.new(0.5, -150, 0, -70)
    notif.BackgroundColor3 = notifType == "complete" and 
        Color3.fromRGB(0, 150, 0) or Color3.fromRGB(50, 50, 200)
    notif.Parent = screenGui
    Instance.new("UICorner", notif).CornerRadius = UDim.new(0, 10)
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -20, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.Text = message
    label.TextSize = 14
    label.Font = Enum.Font.GothamBold
    label.TextColor3 = Color3.new(1,1,1)
    label.TextWrapped = true
    label.BackgroundTransparency = 1
    label.Parent = notif
    
    -- Animate in
    TweenService:Create(notif, TweenInfo.new(0.5, Enum.EasingStyle.Back), {
        Position = UDim2.new(0.5, -150, 0, 10)
    }):Play()
    
    -- Animate out
    task.delay(3, function()
        TweenService:Create(notif, TweenInfo.new(0.5), {
            Position = UDim2.new(0.5, -150, 0, -70)
        }):Play()
        task.delay(0.5, function()
            notif:Destroy()
        end)
    end)
end

-- ===== Remote Events =====
local questUpdateRE = ReplicatedStorage.Events:WaitForChild("Quest_Update")

questUpdateRE.OnClientEvent:Connect(function(updateType, data)
    if updateType == "accepted" then
        showQuestNotification("📋 รับ Quest: " .. data.name, "new")
    elseif updateType == "completed" then
        showQuestNotification("✅ Quest สำเร็จ: " .. data.name, "complete")
    elseif updateType == "failed" then
        showQuestNotification("❌ Quest ล้มเหลว: " .. data.name, "fail")
    elseif updateType == "progress" then
        -- อัพเดท tracker ด้วย active quests ใหม่
        updateQuestTracker(data.activeQuests)
    end
end)

-- เปิด/ปิด Quest Log
local questBtn = Instance.new("TextButton")
questBtn.Size = UDim2.new(0, 50, 0, 50)
questBtn.Position = UDim2.new(0, 70, 0.5, -25)
questBtn.Text = "📋"
questBtn.TextSize = 28
questBtn.BackgroundColor3 = Color3.fromRGB(50, 100, 200)
Instance.new("UICorner", questBtn).CornerRadius = UDim.new(1,0)
questBtn.Parent = screenGui

questBtn.MouseButton1Click:Connect(function()
    questLog.Visible = not questLog.Visible
end)
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Quest Chain
สร้าง quest chain ที่ต่อเนื่องกัน (ทำ quest A แล้วได้รับ quest B)

### แบบฝึกหัดที่ 2: Timed Quest
สร้าง quest ที่มีเวลาจำกัดพร้อม countdown timer

### แบบฝึกหัดที่ 3: Daily Reset
สร้างระบบที่ reset daily quests ทุกเที่ยงคืน

## สรุป

Quest System ที่ดีต้องมี:
- **Quest Database** ที่ชัดเจน
- **Progress Tracking** ที่ accurate
- **Reward System** ที่น่าดึงดูด
- **UI/UX ที่ดี** - tracker, log, notifications
- **Flexibility** - รองรับ quest หลายประเภท
