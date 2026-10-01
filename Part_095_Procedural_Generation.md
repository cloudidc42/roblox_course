# Part 95: Procedural Generation - การสร้างโลกแบบ Procedural

## บทนำ

Procedural Generation คือการสร้างเนื้อหา (แผนที่, dungeon, items) โดยอัลกอริทึมแทนที่จะออกแบบด้วยมือ ทำให้เกมมีความหลากหลายและสามารถ replay ได้ไม่รู้เบื่อ

---

## ส่วนที่ 1: Noise-based Terrain Generation

### 1.1 Simplex Noise สำหรับ Terrain

```lua
-- ModuleScript: NoiseGenerator (ServerStorage)
-- ระบบ noise สำหรับ terrain

-- ใช้ Roblox built-in math.noise (Perlin noise)
local NoiseGenerator = {}

-- สร้าง heightmap จาก noise
function NoiseGenerator:GenerateHeightmap(width, height, config)
    config = config or {}
    local scale = config.scale or 0.01     -- ความละเอียด
    local octaves = config.octaves or 4    -- จำนวน octaves
    local persistence = config.persistence or 0.5
    local lacunarity = config.lacunarity or 2
    local seed = config.seed or math.random(0, 1000)
    local heightMultiplier = config.heightMultiplier or 50
    
    local heightmap = {}
    
    for z = 0, height - 1 do
        heightmap[z] = {}
        for x = 0, width - 1 do
            local amplitude = 1
            local frequency = 1
            local noiseHeight = 0
            
            -- รวม octaves
            for i = 1, octaves do
                local sampleX = (x * scale * frequency) + seed
                local sampleZ = (z * scale * frequency) + seed
                
                local noiseValue = math.noise(sampleX, sampleZ, seed * i)
                noiseHeight = noiseHeight + noiseValue * amplitude
                
                amplitude = amplitude * persistence
                frequency = frequency * lacunarity
            end
            
            -- Normalize และ apply multiplier
            heightmap[z][x] = (noiseHeight + 1) * 0.5 * heightMultiplier
        end
    end
    
    return heightmap
end

-- แปลง heightmap เป็น terrain
function NoiseGenerator:ApplyToTerrain(heightmap, startPos, cellSize)
    local terrain = workspace.Terrain
    startPos = startPos or Vector3.zero
    cellSize = cellSize or 4
    
    for z, row in pairs(heightmap) do
        for x, height in pairs(row) do
            local worldX = startPos.X + x * cellSize
            local worldZ = startPos.Z + z * cellSize
            
            -- กำหนด material ตาม height
            local material
            if height < 5 then
                material = Enum.Material.Sand -- ชายหาด
            elseif height < 15 then
                material = Enum.Material.Grass -- ทุ่งหญ้า
            elseif height < 30 then
                material = Enum.Material.Rock -- หิน
            else
                material = Enum.Material.Snow -- หิมะ
            end
            
            -- Fill terrain
            local pos = Vector3.new(worldX, height / 2, worldZ)
            local size = Vector3.new(cellSize, height, cellSize)
            terrain:FillBlock(CFrame.new(pos), size, material)
        end
    end
end

-- Biome map
function NoiseGenerator:GenerateBiomeMap(width, height, seed)
    seed = seed or math.random(0, 1000)
    local biomes = {}
    
    -- ใช้ noise 2 ชั้น: temperature + moisture
    for z = 0, height - 1 do
        biomes[z] = {}
        for x = 0, width - 1 do
            local temp = math.noise(x * 0.005 + seed, z * 0.005)
            local moisture = math.noise(x * 0.005 + seed * 2, z * 0.005 + seed)
            
            -- แปลงเป็น biome
            local biome
            if temp < -0.3 then
                biome = "tundra"
            elseif temp < 0 then
                if moisture < 0 then
                    biome = "grassland"
                else
                    biome = "forest"
                end
            elseif temp < 0.3 then
                if moisture < -0.2 then
                    biome = "desert"
                elseif moisture < 0.2 then
                    biome = "plains"
                else
                    biome = "rainforest"
                end
            else
                biome = "desert"
            end
            
            biomes[z][x] = biome
        end
    end
    
    return biomes
end

return NoiseGenerator
```

---

## ส่วนที่ 2: Dungeon Generator

### 2.1 BSP Dungeon Generation

```lua
-- ModuleScript: DungeonGenerator (ServerStorage)
-- สร้าง dungeon แบบ Binary Space Partitioning

local DungeonGenerator = {}

-- กำหนดค่า
local MIN_ROOM_SIZE = 8
local MAX_ROOM_SIZE = 20
local MIN_LEAF_SIZE = 12

-- Leaf node สำหรับ BSP
local Leaf = {}
Leaf.__index = Leaf

function Leaf.new(x, y, width, height)
    return setmetatable({
        x = x, y = y,
        width = width, height = height,
        leftChild = nil,
        rightChild = nil,
        room = nil,
    }, Leaf)
end

function Leaf:Split()
    if self.leftChild or self.rightChild then
        return false
    end
    
    -- ตัดสินใจว่าจะแบ่งแนวนอนหรือแนวตั้ง
    local splitHorizontal = math.random() > 0.5
    
    -- ตรวจสอบว่าสามารถแบ่งได้
    if self.width > self.height and self.width / self.height >= 1.25 then
        splitHorizontal = false
    elseif self.height > self.width and self.height / self.width >= 1.25 then
        splitHorizontal = true
    end
    
    local max = (splitHorizontal and self.height or self.width) - MIN_LEAF_SIZE
    
    if max <= MIN_LEAF_SIZE then
        return false
    end
    
    local split = math.random(MIN_LEAF_SIZE, max)
    
    if splitHorizontal then
        self.leftChild = Leaf.new(self.x, self.y, self.width, split)
        self.rightChild = Leaf.new(self.x, self.y + split, self.width, self.height - split)
    else
        self.leftChild = Leaf.new(self.x, self.y, split, self.height)
        self.rightChild = Leaf.new(self.x + split, self.y, self.width - split, self.height)
    end
    
    return true
end

function Leaf:CreateRoom()
    if self.leftChild or self.rightChild then
        -- ไม่ใช่ leaf ไม่ต้องสร้าง room
        return
    end
    
    -- สร้าง room ภายใน leaf
    local roomW = math.random(MIN_ROOM_SIZE, math.min(MAX_ROOM_SIZE, self.width - 2))
    local roomH = math.random(MIN_ROOM_SIZE, math.min(MAX_ROOM_SIZE, self.height - 2))
    local roomX = self.x + math.random(1, self.width - roomW - 1)
    local roomY = self.y + math.random(1, self.height - roomH - 1)
    
    self.room = {
        x = roomX, y = roomY,
        width = roomW, height = roomH,
    }
end

function Leaf:GetRoom()
    if self.room then
        return self.room
    end
    
    if self.leftChild and self.rightChild then
        local leftRoom = self.leftChild:GetRoom()
        local rightRoom = self.rightChild:GetRoom()
        
        if not leftRoom then return rightRoom end
        if not rightRoom then return leftRoom end
        
        return math.random() > 0.5 and leftRoom or rightRoom
    end
    
    return nil
end

-- Dungeon Generator หลัก
function DungeonGenerator:Generate(config)
    config = config or {}
    local mapWidth = config.width or 100
    local mapHeight = config.height or 100
    local maxIterations = config.maxIterations or 5
    
    -- สร้าง map grid
    local grid = {}
    for y = 1, mapHeight do
        grid[y] = {}
        for x = 1, mapWidth do
            grid[y][x] = "wall"
        end
    end
    
    -- BSP Tree
    local root = Leaf.new(0, 0, mapWidth, mapHeight)
    local leaves = {root}
    
    -- Split leaves
    local didSplit = true
    local iterations = 0
    
    while didSplit and iterations < maxIterations do
        didSplit = false
        iterations = iterations + 1
        
        local newLeaves = {}
        for _, leaf in ipairs(leaves) do
            if not leaf.leftChild and not leaf.rightChild then
                if leaf.width > MAX_ROOM_SIZE or leaf.height > MAX_ROOM_SIZE or math.random() > 0.25 then
                    if leaf:Split() then
                        table.insert(newLeaves, leaf.leftChild)
                        table.insert(newLeaves, leaf.rightChild)
                        didSplit = true
                    end
                end
            end
        end
        
        for _, leaf in ipairs(newLeaves) do
            table.insert(leaves, leaf)
        end
    end
    
    -- สร้าง rooms
    local rooms = {}
    root:CreateRoomsRecursive(rooms)
    
    -- วาง rooms บน grid
    for _, room in ipairs(rooms) do
        for y = room.y + 1, room.y + room.height do
            for x = room.x + 1, room.x + room.width do
                if y >= 1 and y <= mapHeight and x >= 1 and x <= mapWidth then
                    grid[y][x] = "floor"
                end
            end
        end
    end
    
    -- เชื่อม rooms ด้วย corridors
    self:_connectRooms(grid, rooms)
    
    return grid, rooms
end

-- เชื่อม rooms ด้วย L-shaped corridors
function DungeonGenerator:_connectRooms(grid, rooms)
    for i = 1, #rooms - 1 do
        local room1 = rooms[i]
        local room2 = rooms[i + 1]
        
        -- Center ของแต่ละ room
        local x1 = room1.x + math.floor(room1.width / 2)
        local y1 = room1.y + math.floor(room1.height / 2)
        local x2 = room2.x + math.floor(room2.width / 2)
        local y2 = room2.y + math.floor(room2.height / 2)
        
        -- วาด L-shaped corridor
        if math.random() > 0.5 then
            self:_drawHorizontalCorridor(grid, x1, x2, y1)
            self:_drawVerticalCorridor(grid, y1, y2, x2)
        else
            self:_drawVerticalCorridor(grid, y1, y2, x1)
            self:_drawHorizontalCorridor(grid, x1, x2, y2)
        end
    end
end

function DungeonGenerator:_drawHorizontalCorridor(grid, x1, x2, y)
    local minX = math.min(x1, x2)
    local maxX = math.max(x1, x2)
    
    for x = minX, maxX do
        if grid[y] and grid[y][x] then
            grid[y][x] = "floor"
            if grid[y-1] and grid[y-1][x] == "wall" then
                grid[y-1][x] = "wall"
            end
            if grid[y+1] and grid[y+1][x] == "wall" then
                grid[y+1][x] = "wall"
            end
        end
    end
end

function DungeonGenerator:_drawVerticalCorridor(grid, y1, y2, x)
    local minY = math.min(y1, y2)
    local maxY = math.max(y1, y2)
    
    for y = minY, maxY do
        if grid[y] and grid[y][x] then
            grid[y][x] = "floor"
        end
    end
end

-- สร้าง dungeon ใน Roblox workspace
function DungeonGenerator:BuildInWorkspace(grid, cellSize, wallHeight, materials)
    cellSize = cellSize or 4
    wallHeight = wallHeight or 8
    materials = materials or {
        wall = Enum.Material.SmoothPlastic,
        floor = Enum.Material.SmoothPlastic,
    }
    
    local dungeonModel = Instance.new("Model")
    dungeonModel.Name = "GeneratedDungeon"
    dungeonModel.Parent = workspace
    
    for y, row in pairs(grid) do
        for x, cellType in pairs(row) do
            local worldX = x * cellSize
            local worldZ = y * cellSize
            
            if cellType == "wall" then
                local wall = Instance.new("Part")
                wall.Size = Vector3.new(cellSize, wallHeight, cellSize)
                wall.Position = Vector3.new(worldX, wallHeight / 2, worldZ)
                wall.Anchored = true
                wall.Material = materials.wall
                wall.BrickColor = BrickColor.new("Dark stone grey")
                wall.Parent = dungeonModel
            elseif cellType == "floor" then
                local floor = Instance.new("Part")
                floor.Size = Vector3.new(cellSize, 1, cellSize)
                floor.Position = Vector3.new(worldX, 0, worldZ)
                floor.Anchored = true
                floor.Material = materials.floor
                floor.BrickColor = BrickColor.new("Medium stone grey")
                floor.Parent = dungeonModel
            end
        end
    end
    
    return dungeonModel
end

return DungeonGenerator
```

---

## ส่วนที่ 3: Loot Table System

```lua
-- ModuleScript: LootTable (ServerStorage)
-- ระบบ loot แบบ weighted random

local LootTable = {}

-- สร้าง loot table ใหม่
function LootTable.new(items)
    local self = setmetatable({}, {__index = LootTable})
    self.items = items or {}
    self.totalWeight = 0
    
    for _, item in ipairs(self.items) do
        self.totalWeight = self.totalWeight + (item.weight or 1)
    end
    
    return self
end

-- Roll สุ่มไอเทม
function LootTable:Roll(luck)
    luck = luck or 1 -- luck modifier
    
    local roll = math.random() * self.totalWeight / luck
    local current = 0
    
    for _, item in ipairs(self.items) do
        current = current + (item.weight or 1)
        if roll <= current then
            return item
        end
    end
    
    return self.items[#self.items]
end

-- Roll หลายครั้ง
function LootTable:RollMultiple(count, luck)
    local results = {}
    for i = 1, count do
        table.insert(results, self:Roll(luck))
    end
    return results
end

-- ตัวอย่าง loot tables
local LOOT_TABLES = {
    -- Common chest
    common = LootTable.new({
        {id = "Gold_Coin", amount = {min=10, max=50}, weight = 50},
        {id = "Health_Potion", amount = {min=1, max=2}, weight = 30},
        {id = "Iron_Sword", amount = 1, weight = 15},
        {id = "Iron_Armor", amount = 1, weight = 5},
    }),
    
    -- Rare chest
    rare = LootTable.new({
        {id = "Gold_Coin", amount = {min=100, max=500}, weight = 40},
        {id = "Health_Potion", amount = {min=2, max=5}, weight = 25},
        {id = "Steel_Sword", amount = 1, weight = 20},
        {id = "Steel_Armor", amount = 1, weight = 10},
        {id = "Magic_Staff", amount = 1, weight = 4},
        {id = "Dragon_Fragment", amount = 1, weight = 1},
    }),
    
    -- Boss chest
    boss = LootTable.new({
        {id = "Gold_Coin", amount = {min=500, max=2000}, weight = 30},
        {id = "Epic_Sword", amount = 1, weight = 20},
        {id = "Dragon_Scale", amount = {min=1, max=3}, weight = 25},
        {id = "Legendary_Item", amount = 1, weight = 5},
        {id = "Pet_Egg", amount = 1, weight = 10},
        {id = "Season_XP_Boost", amount = {min=500, max=1000}, weight = 10},
    }),
}

-- สุ่ม loot จาก table
function LootTable:GenerateLoot(tableType, luck)
    local table_ = LOOT_TABLES[tableType]
    if not table_ then return {} end
    
    local item = table_:Roll(luck)
    if not item then return {} end
    
    -- กำหนดจำนวน
    local amount = 1
    if type(item.amount) == "table" then
        amount = math.random(item.amount.min, item.amount.max)
    elseif type(item.amount) == "number" then
        amount = item.amount
    end
    
    return {id = item.id, amount = amount}
end

return LootTable
```

---

## ส่วนที่ 4: Procedural Quest Generation

```lua
-- ModuleScript: QuestGenerator (ServerStorage)
-- สร้าง quests แบบ procedural

local QuestGenerator = {}

-- Templates สำหรับ quest types
local QUEST_TEMPLATES = {
    kill = {
        type = "kill",
        titleTemplates = {
            "กำจัด {target}",
            "ล่า {target} สุดโหด",
            "ทำลาย {target}",
        },
        descTemplates = {
            "กำจัด {target} จำนวน {count} ตัว",
            "สังหาร {target} ที่กำลังก่อกวนชาวเมือง ({count} ตัว)",
        },
        targets = {
            "Goblin", "Orc", "Skeleton", "Wolf", "Dragon", "Slime"
        },
        countRange = {3, 20},
    },
    
    collect = {
        type = "collect",
        titleTemplates = {
            "เก็บ {item}",
            "ส่ง {item} ให้หมอ",
            "รวบรวม {item}",
        },
        descTemplates = {
            "เก็บ {item} จำนวน {count} ชิ้น",
            "หา {item} {count} ชิ้นจากแผนที่",
        },
        items = {
            "สมุนไพรแดง", "คริสตัลน้ำแข็ง", "ขนนกฟีนิกซ์", "เกล็ดมังกร"
        },
        countRange = {5, 25},
    },
    
    escort = {
        type = "escort",
        titleTemplates = {
            "คุ้มครอง {npc}",
            "พา {npc} ไป {destination}",
        },
        npcs = {"พ่อค้า", "เจ้าหญิง", "นักบวช", "นักวิทยาศาสตร์"},
        destinations = {"หมู่บ้าน", "ปราสาท", "วัด", "ท่าเรือ"},
    },
}

-- สร้าง quest ใหม่
function QuestGenerator:Generate(config)
    config = config or {}
    
    -- เลือก template แบบสุ่ม
    local templateNames = {}
    for name in pairs(QUEST_TEMPLATES) do
        table.insert(templateNames, name)
    end
    
    local templateName = config.type or templateNames[math.random(#templateNames)]
    local template = QUEST_TEMPLATES[templateName]
    
    if not template then return nil end
    
    -- สร้าง quest data
    local quest = {
        id = string.format("QUEST_%d_%d", os.time(), math.random(1000)),
        type = template.type,
        
        -- สุ่ม title และ description
        title = self:_fillTemplate(
            template.titleTemplates[math.random(#template.titleTemplates)],
            template
        ),
        description = self:_fillTemplate(
            template.descTemplates and template.descTemplates[math.random(#template.descTemplates)] or "",
            template
        ),
        
        -- สุ่ม difficulty
        difficulty = config.difficulty or math.random(1, 5),
        
        -- รางวัล (scale ตาม difficulty)
        rewards = self:_generateRewards(config.difficulty or 1),
        
        -- ความต้องการ
        requirements = self:_generateRequirements(template),
        
        -- เวลาจำกัด (optional)
        timeLimit = config.timeLimit,
        
        -- สร้าง timestamp
        createdAt = os.time(),
        expiresAt = config.expiresAt,
    }
    
    return quest
end

-- แทนที่ template variables
function QuestGenerator:_fillTemplate(template, questTemplate)
    local result = template
    
    if questTemplate.targets then
        local target = questTemplate.targets[math.random(#questTemplate.targets)]
        result = result:gsub("{target}", target)
    end
    
    if questTemplate.items then
        local item = questTemplate.items[math.random(#questTemplate.items)]
        result = result:gsub("{item}", item)
    end
    
    if questTemplate.npcs then
        local npc = questTemplate.npcs[math.random(#questTemplate.npcs)]
        result = result:gsub("{npc}", npc)
    end
    
    if questTemplate.destinations then
        local dest = questTemplate.destinations[math.random(#questTemplate.destinations)]
        result = result:gsub("{destination}", dest)
    end
    
    if questTemplate.countRange then
        local count = math.random(questTemplate.countRange[1], questTemplate.countRange[2])
        result = result:gsub("{count}", tostring(count))
    end
    
    return result
end

-- สร้างรางวัล
function QuestGenerator:_generateRewards(difficulty)
    difficulty = difficulty or 1
    
    return {
        coins = math.floor(100 * difficulty * (1 + math.random() * 0.5)),
        xp = math.floor(200 * difficulty * (1 + math.random() * 0.5)),
        gems = difficulty >= 3 and math.floor(5 * difficulty) or 0,
        items = difficulty >= 4 and {"Rare_Chest"} or nil,
    }
end

-- สร้าง requirements
function QuestGenerator:_generateRequirements(template)
    if template.type == "kill" then
        local target = template.targets[math.random(#template.targets)]
        local count = math.random(template.countRange[1], template.countRange[2])
        return {
            type = "kill",
            target = target,
            count = count,
            current = 0,
        }
    elseif template.type == "collect" then
        local item = template.items[math.random(#template.items)]
        local count = math.random(template.countRange[1], template.countRange[2])
        return {
            type = "collect",
            item = item,
            count = count,
            current = 0,
        }
    end
    
    return {}
end

return QuestGenerator
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Terrain Generator
สร้าง terrain ที่:
1. มี biomes หลายอย่าง
2. แม่น้ำและทะเลสาบ
3. ต้นไม้และหิน procedural

### แบบฝึกหัดที่ 2: Dungeon Generator
สร้าง dungeon ที่:
1. ห้องและทางเดิน
2. Spawn enemies ตาม room size
3. Boss room พิเศษ
4. Chest และ traps

### แบบฝึกหัดที่ 3: Item Generator
สร้าง item generator ที่:
1. ชื่อ procedural
2. Stats แบบสุ่ม
3. Rarity system
4. Prefix/Suffix

---

## สรุปบทที่ 95

Procedural Generation ทำให้เกม:

1. **Replayable** - เล่นซ้ำไม่เหมือนกัน
2. **Scalable** - สร้างเนื้อหาได้ไม่จำกัด
3. **Efficient** - ไม่ต้องออกแบบด้วยมือ
4. **Surprising** - ผู้เล่นไม่รู้จะเจออะไร

*บทถัดไป: Part 96 - Advanced AI*
