# ตอนที่ 16: Math Library ใน Lua

## บทนำ

Math Library ของ Lua มีฟังก์ชันคณิตศาสตร์ที่จำเป็นสำหรับการพัฒนาเกม ตั้งแต่การคำนวณพื้นฐาน ตรีโกณมิติ จนถึงการสุ่มตัวเลข การเข้าใจ Math Library จะช่วยให้คุณสร้างระบบต่างๆ ในเกมได้อย่างถูกต้อง

---

## 16.1 ค่าคงที่ทางคณิตศาสตร์

```lua
print(math.pi)      -- 3.1415926535898  (π)
print(math.huge)    -- inf  (อนันต์)
print(-math.huge)   -- -inf
print(math.maxinteger)  -- 9223372036854775807 (max int64)
print(math.mininteger)  -- -9223372036854775808 (min int64)
```

---

## 16.2 ฟังก์ชัน Rounding

```lua
-- floor: ปัดลง
print(math.floor(3.7))   -- 3
print(math.floor(-3.2))  -- -4
print(math.floor(5.0))   -- 5

-- ceil: ปัดขึ้น
print(math.ceil(3.2))    -- 4
print(math.ceil(-3.7))   -- -3
print(math.ceil(5.0))    -- 5

-- ปัดปกติ (Lua ไม่มี round แต่จำลองได้)
local function round(n)
    return math.floor(n + 0.5)
end

print(round(3.4))   -- 3
print(round(3.5))   -- 4
print(round(-3.5))  -- -3

-- ปัดทศนิยม n ตำแหน่ง
local function roundTo(n, decimals)
    local factor = 10 ^ decimals
    return math.floor(n * factor + 0.5) / factor
end

print(roundTo(3.14159, 2))  -- 3.14
print(roundTo(2.567, 1))    -- 2.6

-- modf: แยกส่วนจำนวนเต็มและทศนิยม
local intPart, fracPart = math.modf(3.75)
print(intPart, fracPart)   -- 3.0  0.75

local intPart2, fracPart2 = math.modf(-3.75)
print(intPart2, fracPart2) -- -3.0  -0.75
```

### การใช้งานจริงใน Roblox

```lua
-- ระบบ damage ที่ต้องเป็นจำนวนเต็ม
local function calculateDamage(baseDamage, multiplier)
    return math.floor(baseDamage * multiplier)
end

-- ระบบ timer
local function getFormattedTime(totalSeconds)
    totalSeconds = math.floor(totalSeconds)  -- ตัดทศนิยม
    local minutes = math.floor(totalSeconds / 60)
    local seconds = totalSeconds % 60
    return string.format("%02d:%02d", minutes, seconds)
end

-- ราคาที่ปัดขึ้น
local function getItemCost(basePrice, taxRate)
    local total = basePrice * (1 + taxRate)
    return math.ceil(total)  -- ปัดขึ้นเสมอ
end

print(getItemCost(99, 0.07))  -- 106
```

---

## 16.3 ฟังก์ชัน min, max, abs

```lua
-- min: ค่าน้อยที่สุด
print(math.min(5, 3, 8, 1, 9))    -- 1
print(math.min(10, 20))            -- 10

-- max: ค่ามากที่สุด
print(math.max(5, 3, 8, 1, 9))    -- 9
print(math.max(10, 20))            -- 20

-- abs: ค่าสัมบูรณ์
print(math.abs(-5))    -- 5
print(math.abs(5))     -- 5
print(math.abs(-3.14)) -- 3.14
```

### การใช้งานจริง

```lua
-- clamp: จำกัดค่าในช่วง
local function clamp(value, minVal, maxVal)
    return math.max(minVal, math.min(maxVal, value))
end

-- ใช้กับ health
local function applyHealing(current, amount, maxHealth)
    return clamp(current + amount, 0, maxHealth)
end

-- ใช้กับ movement speed
local function applySpeedModifier(baseSpeed, modifier)
    local newSpeed = baseSpeed * modifier
    return clamp(newSpeed, 5, 50)  -- speed ระหว่าง 5-50
end

print(applyHealing(80, 50, 100))   -- 100 (ไม่เกิน max)
print(applyHealing(80, -90, 100))  -- 0 (ไม่ต่ำกว่า 0)
print(applySpeedModifier(16, 5))   -- 50 (ไม่เกิน max speed)
print(applySpeedModifier(16, 0.1)) -- 5 (ไม่ต่ำกว่า min speed)

-- distance ระหว่าง 2 จุด 2D
local function distance2D(x1, y1, x2, y2)
    local dx = math.abs(x2 - x1)
    local dy = math.abs(y2 - y1)
    return math.sqrt(dx^2 + dy^2)
end
```

---

## 16.4 ฟังก์ชัน Power และ Root

```lua
-- sqrt: รากที่สอง
print(math.sqrt(4))    -- 2.0
print(math.sqrt(2))    -- 1.4142135623731
print(math.sqrt(100))  -- 10.0

-- pow ผ่าน ^ operator
print(2 ^ 10)   -- 1024
print(3 ^ 3)    -- 27

-- exp: e^x
print(math.exp(1))  -- 2.718... (e)
print(math.exp(2))  -- 7.389...

-- log: logarithm
print(math.log(1))          -- 0 (ln(1))
print(math.log(math.exp(1))) -- 1 (ln(e))
print(math.log(100, 10))    -- 2 (log base 10)
print(math.log(8, 2))       -- 3 (log base 2)
```

### การใช้งานจริง

```lua
-- ระบบ exponential growth (EXP ที่ต้องการต่อ level)
local function expRequired(level, baseExp, growthRate)
    return math.floor(baseExp * (growthRate ^ (level - 1)))
end

print("EXP ที่ต้องการต่อ level:")
for level = 1, 10 do
    print(string.format("Level %2d: %5d EXP", level, expRequired(level, 100, 1.2)))
end

-- ระยะ 3D
local function distance3D(pos1, pos2)
    return (pos2 - pos1).Magnitude  -- Roblox มี .Magnitude ให้อยู่แล้ว
end

-- ถ้าคำนวณเอง:
local function distance3DManual(x1, y1, z1, x2, y2, z2)
    return math.sqrt((x2-x1)^2 + (y2-y1)^2 + (z2-z1)^2)
end

-- ระบบ damage falloff ตามระยะ
local function calculateRangedDamage(baseDamage, distance, maxRange)
    if distance > maxRange then return 0 end
    local falloff = 1 - (distance / maxRange) ^ 2
    return math.floor(baseDamage * falloff)
end

print(calculateRangedDamage(100, 0, 100))    -- 100 (ระยะ 0)
print(calculateRangedDamage(100, 50, 100))   -- 75  (ระยะ 50%)
print(calculateRangedDamage(100, 100, 100))  -- 0   (ระยะสูงสุด)
```

---

## 16.5 ฟังก์ชัน Trigonometry

```lua
-- ฟังก์ชัน trig ทำงานด้วย radians
-- 1 degree = π/180 radians
-- 1 radian = 180/π degrees

-- แปลงองศา <-> radians
local function toRad(degrees)
    return degrees * (math.pi / 180)
end

local function toDeg(radians)
    return radians * (180 / math.pi)
end

-- Sin, Cos, Tan
print(math.sin(0))           -- 0
print(math.sin(math.pi/2))   -- 1
print(math.cos(0))           -- 1
print(math.cos(math.pi))     -- -1
print(math.tan(math.pi/4))   -- 1

-- Inverse trig
print(math.asin(1))          -- π/2 ≈ 1.5708
print(math.acos(1))          -- 0
print(math.atan(1))          -- π/4 ≈ 0.7854

-- atan2: คำนวณมุมระหว่างสองจุด
local angle = math.atan(3, 4)  -- Lua 5.3+: atan(y, x)
print(toDeg(angle))  -- ≈ 36.87 องศา
```

### การใช้งานจริงใน Roblox

```lua
-- สร้างวงกลม
local function createCircleOfParts(center, radius, count)
    local parts = {}
    
    for i = 1, count do
        local angle = (i / count) * (2 * math.pi)
        local x = center.X + math.cos(angle) * radius
        local z = center.Z + math.sin(angle) * radius
        
        local part = Instance.new("Part")
        part.Size = Vector3.new(1, 1, 1)
        part.Position = Vector3.new(x, center.Y, z)
        part.Anchored = true
        part.BrickColor = BrickColor.new("Bright red")
        part.Parent = game.Workspace
        
        table.insert(parts, part)
    end
    
    return parts
end

-- ทิศทางจากตำแหน่งหนึ่งไปอีกตำแหน่ง
local function getAngleBetween(from, to)
    local dx = to.X - from.X
    local dz = to.Z - from.Z
    return math.atan(dz, dx)
end

-- หมุน object ไปหา target
local function lookAt(part, targetPos)
    local angle = getAngleBetween(part.Position, targetPos)
    part.CFrame = CFrame.new(part.Position) * CFrame.Angles(0, -angle, 0)
end

-- Projectile trajectory
local function getProjectilePos(origin, angle, speed, time)
    local vx = speed * math.cos(toRad(angle))
    local vy = speed * math.sin(toRad(angle))
    local g = -196.2  -- Roblox gravity
    
    return Vector3.new(
        origin.X + vx * time,
        origin.Y + vy * time + 0.5 * g * time^2,
        origin.Z
    )
end

-- Orbit: ทำให้ object วนรอบจุดศูนย์กลาง
local function getOrbitPosition(center, radius, speed, time)
    local angle = speed * time
    return Vector3.new(
        center.X + math.cos(angle) * radius,
        center.Y,
        center.Z + math.sin(angle) * radius
    )
end

-- ตัวอย่าง: ดาวเทียมวนรอบ Planet
local function orbitDemo(planet, satellite)
    local startTime = os.clock()
    local ORBIT_RADIUS = 20
    local ORBIT_SPEED = 1  -- radians per second
    
    game:GetService("RunService").Heartbeat:Connect(function()
        local t = os.clock() - startTime
        satellite.Position = getOrbitPosition(
            planet.Position, ORBIT_RADIUS, ORBIT_SPEED, t
        )
    end)
end
```

---

## 16.6 ฟังก์ชัน Random

```lua
-- ตั้ง seed (สำคัญมาก!)
math.randomseed(os.time())  -- ใช้เวลาปัจจุบันเป็น seed

-- random(): ตัวเลขทศนิยมระหว่าง 0-1
print(math.random())  -- เช่น 0.37452...

-- random(n): จำนวนเต็มระหว่าง 1-n
print(math.random(6))   -- 1-6 (ลูกเต๋า)
print(math.random(100)) -- 1-100

-- random(m, n): จำนวนเต็มระหว่าง m-n
print(math.random(50, 100))  -- 50-100
print(math.random(-10, 10))  -- -10-10
```

### Weighted Random

```lua
-- สุ่มแบบ weighted (บางอย่างมีโอกาสมากกว่า)
local function weightedRandom(options)
    -- options = {{item, weight}, ...}
    local totalWeight = 0
    for _, option in ipairs(options) do
        totalWeight = totalWeight + option.weight
    end
    
    local roll = math.random() * totalWeight
    local cumulative = 0
    
    for _, option in ipairs(options) do
        cumulative = cumulative + option.weight
        if roll <= cumulative then
            return option.item
        end
    end
    
    return options[#options].item
end

-- ตัวอย่าง: drop system
local lootTable = {
    {item = "ทอง", weight = 50},      -- 50% โอกาส
    {item = "ยาแดง", weight = 30},    -- 30% โอกาส
    {item = "ดาบ", weight = 15},      -- 15% โอกาส
    {item = "เกราะ", weight = 4},     -- 4% โอกาส
    {item = "ของหายาก", weight = 1}, -- 1% โอกาส
}

-- ทดสอบ 1000 ครั้ง
local results = {}
for i = 1, 1000 do
    local item = weightedRandom(lootTable)
    results[item] = (results[item] or 0) + 1
end

print("ผลการทดสอบ 1000 ครั้ง:")
for item, count in pairs(results) do
    print(string.format("%-12s: %d (%.1f%%)", item, count, count/10))
end
```

### Shuffle Array

```lua
-- Fisher-Yates shuffle
local function shuffle(arr)
    local shuffled = {}
    for i, v in ipairs(arr) do
        shuffled[i] = v
    end
    
    for i = #shuffled, 2, -1 do
        local j = math.random(i)
        shuffled[i], shuffled[j] = shuffled[j], shuffled[i]
    end
    
    return shuffled
end

local deck = {"A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"}
local shuffled = shuffle(deck)
print(table.concat(shuffled, ", "))

-- Random point ใน Circle
local function randomPointInCircle(cx, cz, radius)
    local angle = math.random() * 2 * math.pi
    local r = radius * math.sqrt(math.random())  -- uniform distribution
    return cx + r * math.cos(angle), cz + r * math.sin(angle)
end

-- Gaussian random (Bell curve)
local function gaussianRandom(mean, stdDev)
    -- Box-Muller transform
    local u1 = math.random()
    local u2 = math.random()
    local z = math.sqrt(-2 * math.log(u1)) * math.cos(2 * math.pi * u2)
    return mean + stdDev * z
end

-- ใช้สำหรับ damage variation
local function getDamageWithVariance(baseDamage)
    local damage = gaussianRandom(baseDamage, baseDamage * 0.1)
    return math.max(1, math.floor(damage))
end

-- ทดสอบ
for i = 1, 5 do
    print("Damage: " .. getDamageWithVariance(100))
end
```

---

## 16.7 ฟังก์ชันอื่นๆ

```lua
-- fmod: floating point modulo
print(math.fmod(7.5, 2.5))   -- 0.0
print(math.fmod(7.8, 2.0))   -- 1.8

-- type: ตรวจสอบว่าเป็น int หรือ float
print(math.type(1))      -- integer
print(math.type(1.0))    -- float
print(math.type("1"))    -- ไม่ใช่ number (error ใน Lua 5.3+)

-- tointeger: แปลงเป็น int
print(math.tointeger(5.0))   -- 5
print(math.tointeger(5.5))   -- nil (ไม่สามารถแปลงได้)
```

---

## 16.8 Vector Mathematics

ใน Roblox, Vector3 มีฟังก์ชันคณิตศาสตร์ built-in:

```lua
-- Vector3 operations
local v1 = Vector3.new(1, 2, 3)
local v2 = Vector3.new(4, 5, 6)

-- บวก ลบ
local sum = v1 + v2       -- Vector3(5, 7, 9)
local diff = v2 - v1      -- Vector3(3, 3, 3)

-- คูณ/หารด้วย scalar
local scaled = v1 * 2     -- Vector3(2, 4, 6)
local halved = v2 / 2     -- Vector3(2, 2.5, 3)

-- Magnitude (ความยาว)
local magnitude = v1.Magnitude  -- sqrt(1+4+9) = sqrt(14) ≈ 3.742

-- Unit vector (ความยาว = 1)
local unit = v1.Unit  -- v1 / magnitude

-- Dot product
local dot = v1:Dot(v2)  -- 1*4 + 2*5 + 3*6 = 32
print("Dot: " .. dot)

-- Cross product
local cross = v1:Cross(v2)  -- perpendicular vector
print("Cross: " .. tostring(cross))

-- Lerp (linear interpolation)
local lerped = v1:Lerp(v2, 0.5)  -- halfway between v1 and v2
print("Lerp 50%: " .. tostring(lerped))  -- (2.5, 3.5, 4.5)

-- Distance
local distance = (v2 - v1).Magnitude
print("Distance: " .. distance)
```

### การใช้ Vector Math ในเกม

```lua
-- ระบบ AI: ตรวจสอบว่าศัตรูมองเห็น player หรือไม่
local function canSeeTarget(enemy, target, fovAngle, maxDistance)
    local enemyPos = enemy.Position
    local targetPos = target.Position
    
    -- ตรวจสอบระยะ
    local distance = (targetPos - enemyPos).Magnitude
    if distance > maxDistance then
        return false, "ไกลเกินไป"
    end
    
    -- ตรวจสอบมุมการมองเห็น
    local toTarget = (targetPos - enemyPos).Unit
    local enemyForward = enemy.CFrame.LookVector
    
    local dotProduct = toTarget:Dot(enemyForward)
    local angle = math.deg(math.acos(dotProduct))
    
    if angle > fovAngle / 2 then
        return false, "อยู่นอก FOV"
    end
    
    return true, "มองเห็น"
end

-- ระบบ knockback
local function calculateKnockback(attackDir, power, distance)
    local direction = attackDir.Unit
    return direction * power * (1 / distance)
end

-- Smooth camera
local function smoothLookAt(camera, target, smoothing)
    local currentCF = camera.CFrame
    local targetCF = CFrame.new(camera.CFrame.Position, target.Position)
    camera.CFrame = currentCF:Lerp(targetCF, smoothing)
end
```

---

## 16.9 ตัวอย่างโปรเจกต์: Procedural Generation

```lua
-- Procedural Terrain Generator ง่ายๆ

-- Noise function (Perlin-like simplification)
local function noise(x, y, seed)
    seed = seed or 0
    local n = math.sin(x * 127.1 + y * 311.7 + seed * 74.7) * 43758.5453
    return n - math.floor(n)  -- fractional part (0-1)
end

-- Smooth noise
local function smoothNoise(x, y, seed)
    local corners = (noise(x-1, y-1, seed) + noise(x+1, y-1, seed) +
                    noise(x-1, y+1, seed) + noise(x+1, y+1, seed)) / 16
    local sides = (noise(x-1, y, seed) + noise(x+1, y, seed) +
                  noise(x, y-1, seed) + noise(x, y+1, seed)) / 8
    local center = noise(x, y, seed) / 4
    return corners + sides + center
end

-- Octave noise สำหรับ terrain ที่ดูเป็นธรรมชาติ
local function octaveNoise(x, y, octaves, persistence, scale, seed)
    local total = 0
    local frequency = 1
    local amplitude = 1
    local maxValue = 0
    
    for i = 1, octaves do
        total = total + smoothNoise(x * frequency / scale, y * frequency / scale, seed) * amplitude
        maxValue = maxValue + amplitude
        amplitude = amplitude * persistence
        frequency = frequency * 2
    end
    
    return total / maxValue
end

-- สร้าง terrain
local function generateTerrain(width, depth, seed)
    print("กำลังสร้าง terrain " .. width .. "x" .. depth)
    
    for x = 1, width do
        for z = 1, depth do
            local height = octaveNoise(x, z, 4, 0.5, 50, seed)
            height = math.floor(height * 20) + 1  -- 1-21 blocks สูง
            
            -- เลือก material ตามความสูง
            local material
            if height <= 3 then
                material = Enum.Material.Sand
            elseif height <= 8 then
                material = Enum.Material.Grass
            elseif height <= 15 then
                material = Enum.Material.Rock
            else
                material = Enum.Material.Ice
            end
            
            local part = Instance.new("Part")
            part.Size = Vector3.new(4, height * 2, 4)
            part.Position = Vector3.new(x * 4, height, z * 4)
            part.Anchored = true
            part.Material = material
            part.TopSurface = Enum.SurfaceType.Smooth
            part.Parent = game.Workspace
        end
        
        -- yield เพื่อไม่ให้ติด
        if x % 10 == 0 then
            task.wait()
        end
    end
    
    print("สร้าง terrain เสร็จแล้ว!")
end

-- ทดสอบ (ขนาดเล็กเพื่อทดสอบ)
-- generateTerrain(20, 20, os.time())
```

---

## 16.10 แบบฝึกหัด

### แบบฝึกหัดที่ 1: Combat Math

```lua
-- คำนวณ:
-- 1. ความน่าจะเป็น critical hit โดยใช้ luck stat
-- 2. ความเสียหายหลัง dodge calculation
-- 3. ระยะเวลาที่ใช้กำจัดศัตรู (DPS calculation)

local function critChance(luck)
    -- luck 0-100, base crit = 5%, max = 50%
    -- เติมโค้ด
end

local function calculateDodge(accuracy, evasion)
    -- hit chance = accuracy / (accuracy + evasion)
    -- เติมโค้ด
end

local function timeToKill(dps, enemyHealth)
    -- เติมโค้ด
end
```

### แบบฝึกหัดที่ 2: Geometry

```lua
-- คำนวณ:
-- 1. พื้นที่ polygon
-- 2. จุดที่อยู่ในวงกลมหรือไม่
-- 3. จุดตัดของเส้นตรงสองเส้น

local function isPointInCircle(px, pz, cx, cz, radius)
    -- เติมโค้ด
end

local function distanceToLine(px, pz, x1, z1, x2, z2)
    -- เติมโค้ด
end
```

### แบบฝึกหัดที่ 3: Simulation

```lua
-- สร้าง simple physics simulation:
-- - กระสุนที่ยิงออกไป
-- - คำนวณตำแหน่งทุก frame
-- - แสดง trajectory

local function simulateProjectile(startPos, velocity, time)
    -- เติมโค้ด
end
```

---

## สรุป

| ฟังก์ชัน | การใช้งาน |
|----------|----------|
| `math.floor/ceil` | ปัดลง/ขึ้น |
| `math.round` | ปัดปกติ (จำลอง) |
| `math.min/max` | ค่าต่ำสุด/สูงสุด |
| `math.abs` | ค่าสัมบูรณ์ |
| `math.sqrt` | รากที่สอง |
| `math.random` | สุ่มตัวเลข |
| `math.sin/cos/tan` | ตรีโกณมิติ |
| `math.pi` | ค่า π |
| `math.huge` | อนันต์ |
| `math.modf` | แยกส่วนเต็มและทศนิยม |

### บทถัดไป

ในบทที่ 17 เราจะเรียนเรื่อง **Events and Signals** - ระบบ event ของ Roblox
