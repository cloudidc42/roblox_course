# Part 90: Team Development - การทำงานในทีมพัฒนาเกม

## บทนำ

การพัฒนาเกมเป็นทีมต้องการทักษะการจัดการที่ดี ทั้งด้านการสื่อสาร การแบ่งงาน และการใช้เครื่องมือร่วมกัน ในบทนี้เราจะเรียนรู้วิธีทำงานเป็นทีมอย่างมีประสิทธิภาพ

---

## ส่วนที่ 1: โครงสร้างทีม

### 1.1 บทบาทในทีม

**Lead Developer**
- วางแผนสถาปัตยกรรมระบบ
- Code review
- ตัดสินใจด้านเทคนิค

**Gameplay Developer**
- ระบบ gameplay หลัก
- Level design logic
- Game balance

**UI/UX Developer**
- สร้าง GUI
- Animation และ visual effects
- User experience

**Backend Developer**
- DataStore systems
- Server-side logic
- Security

**3D Artist**
- Models, terrain
- Lighting
- Visual effects

**Sound Designer**
- เสียงประกอบ
- ดนตรี

**QA Tester**
- Testing
- Bug reports
- User experience testing

### 1.2 Team Size และ Game Scale

```
Solo (1 คน):
- เกมขนาดเล็ก
- Casual games
- Portfolio projects

Small Team (2-4 คน):
- Indie games
- 3-6 เดือน development

Medium Team (5-10 คน):
- Full-featured games
- 6-12 เดือน development

Large Team (10+ คน):
- AAA Roblox games
- 1+ ปี development
- ต้องการ Project Manager
```

---

## ส่วนที่ 2: การสื่อสารในทีม

### 2.1 Communication Channels

```
Discord Server Structure:
├── 📢 announcements
├── 💬 general
├── 🛠️ development
│   ├── dev-general
│   ├── dev-progress
│   ├── bugs-and-issues
│   └── code-review
├── 🎨 art-and-design
├── 🎵 audio
├── 📋 planning
│   ├── feature-requests
│   └── roadmap
├── 🔒 private
│   ├── admin-chat
│   └── finance
└── 🎮 playtesting
```

### 2.2 Daily Standup Format

```
ทุกเช้า 10:00 น. (Discord voice/text):
1. ทำอะไรไปเมื่อวานนี้?
2. วันนี้จะทำอะไร?
3. มีอุปสรรคอะไรบ้าง?

ตัวอย่าง:
"เมื่อวาน: ทำระบบ VIP shop เสร็จ
วันนี้: เขียน tests สำหรับ shop system
ปัญหา: DataStore API ให้ error แปลกๆ ขอให้ช่วยดูด้วย"
```

---

## ส่วนที่ 3: Project Management

### 3.1 Trello/Notion Board

```
Board: MyGame Development

Lists:
📋 Backlog
  - Feature: Pet System
  - Feature: Guild System
  - Bug: Inventory lag on mobile
  
🔥 Sprint (This Week)
  - Feature: Daily Login Rewards
  - Fix: Shop UI not closing
  
💻 In Progress
  - VIP System (Assigned: Dev1)
  - New map area (Assigned: Artist1)
  
👀 Code Review
  - DataStore refactor (PR #45)
  
✅ Done
  - Basic shop
  - Character customization
```

### 3.2 Sprint Planning

```lua
-- ตัวอย่าง Sprint Planning Template

local SPRINT = {
    number = 12,
    startDate = "2024-01-15",
    endDate = "2024-01-28",
    
    goals = {
        "Launch VIP system",
        "Fix top 5 reported bugs",
        "Improve shop UI",
    },
    
    tasks = {
        {
            id = "TASK-045",
            title = "สร้าง VIP tier comparison UI",
            assignee = "Dev1",
            storyPoints = 5,
            status = "todo",
        },
        {
            id = "TASK-046",
            title = "ระบบ daily VIP rewards",
            assignee = "Dev2",
            storyPoints = 3,
            status = "in_progress",
        },
        {
            id = "TASK-047",
            title = "VIP exclusive area",
            assignee = "Dev1",
            storyPoints = 8,
            status = "todo",
        },
    },
    
    velocity = 32, -- story points เฉลี่ยต่อ sprint
}
```

---

## ส่วนที่ 4: Code Collaboration

### 4.1 Code Review Guidelines

```lua
-- Code Review Checklist

-- ✅ Functionality
-- - โค้ดทำสิ่งที่ต้องการได้หรือไม่?
-- - Edge cases ถูกจัดการหรือไม่?
-- - Error handling เพียงพอหรือไม่?

-- ✅ Code Quality
-- - อ่านง่ายและเข้าใจได้?
-- - มี comment อธิบายส่วนที่ซับซ้อน?
-- - ชื่อตัวแปรและฟังก์ชันชัดเจน?

-- ✅ Performance
-- - มี bottleneck ที่ชัดเจนหรือไม่?
-- - Memory leaks?
-- - Network calls ที่ไม่จำเป็น?

-- ✅ Security
-- - Validate input ทุก RemoteEvent?
-- - ไม่ trust client?
-- - ไม่ expose sensitive data?

-- ✅ Testing
-- - มี tests สำหรับ logic ใหม่?
-- - Tests ผ่านทั้งหมด?
```

### 4.2 Coding Standards

```lua
-- Coding Standards สำหรับทีม
-- บันทึกไว้ใน CODING_STANDARDS.md

-- ==========================================
-- 1. Naming Conventions
-- ==========================================

-- Classes/Modules: PascalCase
local DataManager = {}
local ShopSystem = {}

-- Functions: camelCase
local function calculateDamage(attacker, defender)
end

local function getPlayerData(player)
end

-- Variables: camelCase  
local maxHealth = 100
local currentLevel = 5

-- Constants: UPPER_SNAKE_CASE
local MAX_PLAYERS = 50
local SAVE_INTERVAL = 60
local DATA_STORE_NAME = "PlayerData_v1"

-- Private (convention): underscore prefix
local _privateData = {}
local function _internalHelper()
end

-- ==========================================
-- 2. Module Structure
-- ==========================================

-- ModuleScript template
local ModuleName = {}

-- Private variables
local _config = {}
local _cache = {}

-- Private functions
local function _validate(value)
    return type(value) == "number" and value > 0
end

-- Public API
function ModuleName:initialize(config)
    _config = config or {}
end

function ModuleName:doSomething(param)
    if not _validate(param) then
        return false, "Invalid parameter"
    end
    -- implementation
    return true
end

return ModuleName

-- ==========================================
-- 3. Error Handling
-- ==========================================

-- ✅ ดี: Handle errors properly
local function safeFetchData(key)
    local success, result = pcall(function()
        return dataStore:GetAsync(key)
    end)
    
    if not success then
        warn("[DataStore] Failed to fetch " .. key .. ": " .. tostring(result))
        return nil
    end
    
    return result
end

-- ❌ ไม่ดี: ไม่ handle errors
local function badFetchData(key)
    return dataStore:GetAsync(key) -- อาจ error!
end

-- ==========================================
-- 4. Comments
-- ==========================================

-- ✅ Comment อธิบาย "ทำไม" ไม่ใช่ "อะไร"

-- ✅ ดี
-- ใช้ exponential backoff เพื่อหลีกเลี่ยง rate limiting
local delay = RETRY_DELAY * (2 ^ (attempt - 1))
task.wait(delay)

-- ❌ ไม่ดี
-- คำนวณ delay
local delay = RETRY_DELAY * (2 ^ (attempt - 1))
task.wait(delay)

-- ==========================================
-- 5. Function Size
-- ==========================================

-- ✅ ดี: ฟังก์ชันทำหน้าที่เดียว (Single Responsibility)
local function validateCoinAmount(amount)
    return type(amount) == "number" and amount > 0 and amount <= 1000000
end

local function deductCoins(playerData, amount)
    if playerData.Coins < amount then
        return false, "Insufficient coins"
    end
    playerData.Coins = playerData.Coins - amount
    return true
end

local function purchaseItem(player, itemId, cost)
    local data = DataManager:GetData(player)
    if not data then return false, "No player data" end
    
    if not validateCoinAmount(cost) then return false, "Invalid cost" end
    
    local success, err = deductCoins(data, cost)
    if not success then return false, err end
    
    InventorySystem:AddItem(player, itemId, 1)
    return true
end
```

---

## ส่วนที่ 5: Asset Management

### 5.1 Asset Naming Convention

```
-- รูปภาพ
icons/          -> icon_[name]_[size].png
thumbnails/     -> thumb_[name].png
ui/             -> ui_[component]_[state].png
characters/     -> char_[name]_[part].png

-- เสียง
sfx/            -> sfx_[action].mp3
music/          -> mus_[name]_[mood].mp3
voice/          -> vo_[character]_[line].mp3

-- Models
models/         -> mdl_[category]_[name].rbxm
```

### 5.2 Shared Asset Repository

```lua
-- ModuleScript: AssetConfig (ReplicatedStorage)
-- กำหนด Asset IDs ทั้งหมด

local Assets = {}

Assets.Icons = {
    -- UI Icons
    coins = "rbxassetid://1234567890",
    gems = "rbxassetid://1234567891",
    vip = "rbxassetid://1234567892",
    settings = "rbxassetid://1234567893",
    close = "rbxassetid://1234567894",
    
    -- Item Icons
    sword_basic = "rbxassetid://2345678901",
    sword_fire = "rbxassetid://2345678902",
    armor_basic = "rbxassetid://2345678903",
    potion_health = "rbxassetid://2345678904",
}

Assets.Sounds = {
    -- UI Sounds
    button_click = "rbxassetid://3456789012",
    purchase_success = "rbxassetid://3456789013",
    level_up = "rbxassetid://3456789014",
    
    -- Combat Sounds
    sword_swing = "rbxassetid://4567890123",
    sword_hit = "rbxassetid://4567890124",
    
    -- Ambient Sounds
    wind = "rbxassetid://5678901234",
    water = "rbxassetid://5678901235",
}

Assets.Models = {
    -- Characters
    hero_base = "rbxassetid://6789012345",
    
    -- Environment
    tree_pine = "rbxassetid://7890123456",
    rock_large = "rbxassetid://7890123457",
}

Assets.Animations = {
    idle = "rbxassetid://8901234567",
    walk = "rbxassetid://8901234568",
    run = "rbxassetid://8901234569",
    jump = "rbxassetid://8901234570",
    attack = "rbxassetid://8901234571",
    death = "rbxassetid://8901234572",
}

return Assets
```

---

## ส่วนที่ 6: Remote Work Tools

### 6.1 Roblox Collaboration Features

```lua
-- Team Create ใน Studio
-- เปิดใช้ผ่าน: File > Game Settings > Permissions > Enable Studio Access to API Services

-- Collaborative Editing:
-- 1. เปิด Team Create (View > Team Create)
-- 2. เชิญ collaborators
-- 3. ทำงานได้พร้อมกัน real-time

-- หมายเหตุ:
-- - ทุกคนต้องมี Group membership (สำหรับ group games)
-- - การเปลี่ยนแปลงบันทึกอัตโนมัติ
-- - มี rollback history
```

### 6.2 Work Assignment System

```lua
-- ModuleScript: TaskTracker (ใช้สำหรับ internal tracking)

local TaskTracker = {}

local tasks = {
    pending = {},
    inProgress = {},
    completed = {},
}

function TaskTracker:AddTask(task)
    table.insert(tasks.pending, {
        id = math.random(100000, 999999),
        title = task.title,
        description = task.description,
        assignee = task.assignee,
        priority = task.priority or "medium",
        tags = task.tags or {},
        createdAt = os.time(),
        estimatedHours = task.estimatedHours,
    })
end

function TaskTracker:StartTask(taskId, developerId)
    -- ย้ายงานจาก pending ไป inProgress
    for i, task in ipairs(tasks.pending) do
        if task.id == taskId then
            task.startedAt = os.time()
            task.developer = developerId
            table.insert(tasks.inProgress, task)
            table.remove(tasks.pending, i)
            return true
        end
    end
    return false
end

function TaskTracker:CompleteTask(taskId)
    for i, task in ipairs(tasks.inProgress) do
        if task.id == taskId then
            task.completedAt = os.time()
            task.actualHours = (task.completedAt - task.startedAt) / 3600
            table.insert(tasks.completed, task)
            table.remove(tasks.inProgress, i)
            return true
        end
    end
    return false
end

return TaskTracker
```

---

## ส่วนที่ 7: Revenue Sharing

### 7.1 วิธีแบ่งรายได้ในทีม

**Model 1: Percentage Split**
```
Lead Developer: 40%
Gameplay Dev: 20%
UI Developer: 20%
Artist: 10%
Sound Designer: 5%
Misc: 5%
```

**Model 2: Hourly Rate**
```
ติดตามชั่วโมงการทำงาน
แบ่งตามสัดส่วนชั่วโมง
เหมาะเมื่อ contribution ไม่เท่ากัน
```

**Model 3: Milestone-based**
```
ผู้ร่วมพัฒนาในแต่ละ milestone ได้รับส่วนแบ่ง
เหมาะสำหรับ freelancers
```

### 7.2 GroupFund Management

```lua
-- Groups ใน Roblox มี Group Funds
-- รายได้เข้า Group Account

-- วิธีแบ่งรายได้:
-- 1. ไปที่ Group > Configure Group > Revenue
-- 2. ตั้งค่าการจ่าย Robux ให้สมาชิก
-- 3. หรือใช้ API ในการจ่ายอัตโนมัติ

-- ตัวอย่าง Revenue Distribution (Pseudocode)
local function distributeRevenue(totalRevenue)
    local shares = {
        ["LeadDev"] = 0.40,
        ["GameplayDev"] = 0.20,
        ["UIdev"] = 0.20,
        ["Artist"] = 0.10,
        ["SoundDesigner"] = 0.05,
        ["Reserve"] = 0.05, -- กองทุนสำรอง
    }
    
    local distribution = {}
    for role, percentage in pairs(shares) do
        distribution[role] = math.floor(totalRevenue * percentage)
    end
    
    return distribution
end
```

---

## ส่วนที่ 8: Conflict Resolution

### 8.1 Code Conflicts

```bash
# เมื่อเกิด merge conflict

# 1. ดูว่า conflict อยู่ที่ไหน
git status

# 2. เปิดไฟล์ที่มี conflict
# <<<<<<< HEAD
# local config = {speed = 16}  <- ของเรา
# =======
# local config = {speed = 20}  <- ของเขา
# >>>>>>> feature/speed-boost

# 3. ตัดสินใจว่าจะใช้อันไหน หรือรวมกัน
# local config = {speed = 20}  <- ตัดสินใจใช้ค่าใหม่

# 4. Mark resolved
git add src/config.lua
git commit -m "fix: resolve speed config conflict"
```

### 8.2 Design Conflicts

```
วิธีแก้ปัญหา design ที่ไม่ตรงกัน:

1. Present ทั้งสองแนวคิดให้ทีมฟัง
2. ออกเสียง (majority vote)
3. ถ้า tie - Lead Developer ตัดสิน
4. บันทึกเหตุผลการตัดสินใจไว้

หมายเหตุ: พยายามเลือกแนวทางที่เป็นประโยชน์ต่อผู้เล่นมากที่สุด
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Team Setup
1. สร้าง Discord server สำหรับทีม
2. กำหนด role ของสมาชิก
3. ตั้งค่า channel structure
4. เขียน Code Standards document

### แบบฝึกหัดที่ 2: Sprint Planning
1. สร้าง Trello/Notion board
2. ระบุ backlog items
3. เลือก sprint goals
4. แบ่งงาน

### แบบฝึกหัดที่ 3: Code Review
1. สร้าง PR สำหรับ feature
2. ทำ code review ให้กัน
3. แก้ไขตาม feedback
4. Merge เมื่อ approve

---

## สรุปบทที่ 90

การทำงานเป็นทีมที่ดีต้องการ:

1. **Clear Roles** - ทุกคนรู้หน้าที่ของตัวเอง
2. **Communication** - สื่อสารอย่างสม่ำเสมอ
3. **Standards** - มาตรฐานที่ทุกคนเข้าใจ
4. **Tools** - เครื่องมือที่เหมาะสม
5. **Respect** - เคารพความคิดเห็นของกัน

*บทถัดไป: Part 91 - Publishing and Marketing*
