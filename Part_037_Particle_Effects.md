# Part 37: Particle Effects (เอฟเฟกต์อนุภาค)

## บทนำ

Particle Effects หรือเอฟเฟกต์อนุภาค เป็นหนึ่งในเครื่องมือที่ทรงพลังที่สุดในการสร้างภาพที่สวยงามใน Roblox ใช้สร้างไฟ, ควัน, ระเบิด, เวทมนตร์, และเอฟเฟกต์พิเศษต่างๆ ที่ทำให้เกมดูมีชีวิตชีวา

---

## 37.1 ParticleEmitter คืออะไร?

`ParticleEmitter` คือ Instance ที่ปล่อยอนุภาค (particles) ออกมาจาก Part หรือ Attachment

### โครงสร้างพื้นฐาน

```
Part
└── ParticleEmitter
    ├── Rate: number (จำนวนอนุภาคต่อวินาที)
    ├── Lifetime: NumberRange (อายุของอนุภาค)
    ├── Speed: NumberRange (ความเร็ว)
    ├── Size: NumberSequence (ขนาด)
    ├── Color: ColorSequence (สี)
    ├── Transparency: NumberSequence (ความโปร่งใส)
    └── Rotation: NumberRange (การหมุน)
```

### การสร้าง ParticleEmitter ใน Studio

1. เลือก Part ใน Workspace
2. คลิก `+` ใน Explorer
3. ค้นหา `ParticleEmitter`
4. ปรับแต่งใน Properties

---

## 37.2 Properties หลักของ ParticleEmitter

### 37.2.1 Rate และ Lifetime

```lua
local emitter = Instance.new("ParticleEmitter")
emitter.Parent = workspace.MyPart

-- Rate: จำนวนอนุภาคที่ปล่อยต่อวินาที
emitter.Rate = 50  -- ปล่อย 50 อนุภาค/วินาที

-- Lifetime: อายุของอนุภาคแต่ละตัว (วินาที)
-- NumberRange(min, max) - สุ่มระหว่างค่าสองค่า
emitter.Lifetime = NumberRange.new(1, 3)  -- อยู่ได้ 1-3 วินาที

-- หยุดปล่อยอนุภาค
emitter.Enabled = false
```

### 37.2.2 Speed และ SpreadAngle

```lua
-- Speed: ความเร็วของอนุภาค (studs/sec)
emitter.Speed = NumberRange.new(5, 15)  -- ความเร็ว 5-15 studs/sec

-- SpreadAngle: มุมการกระจาย (องศา)
-- (X, Y) = มุมในแกน X และ Y
emitter.SpreadAngle = Vector2.new(45, 45)  -- กระจายออก 45 องศา

-- EmissionDirection: ทิศทางการปล่อย
emitter.EmissionDirection = Enum.NormalId.Top  -- ปล่อยขึ้น
-- Top, Bottom, Front, Back, Left, Right
```

### 37.2.3 Size (NumberSequence)

```lua
-- NumberSequence กำหนดขนาดตลอดอายุอนุภาค
-- Keypoints: เวลา 0 = เกิด, เวลา 1 = ตาย

-- ขนาดคงที่
emitter.Size = NumberSequence.new(1)

-- ขนาดเพิ่มขึ้นแล้วลดลง (ทรงกลม)
emitter.Size = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0),    -- เกิดมาขนาด 0
    NumberSequenceKeypoint.new(0.3, 1),  -- โตขึ้นถึง 1
    NumberSequenceKeypoint.new(1, 0),    -- หดตัวจนหาย
})

-- ขนาดลดลงเรื่อยๆ
emitter.Size = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 2),    -- เริ่มขนาด 2
    NumberSequenceKeypoint.new(1, 0.1),  -- เล็กลงเป็น 0.1
})
```

### 37.2.4 Color (ColorSequence)

```lua
-- สีคงที่
emitter.Color = ColorSequence.new(Color3.fromRGB(255, 100, 0))

-- สีเปลี่ยนจากส้มเป็นเหลือง
emitter.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 50, 0)),   -- ส้มแดง
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 200, 0)), -- เหลือง
    ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 200, 200)), -- เทา
})

-- สีสายรุ้ง
emitter.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),   -- แดง
    ColorSequenceKeypoint.new(0.2, Color3.fromRGB(255, 165, 0)), -- ส้ม
    ColorSequenceKeypoint.new(0.4, Color3.fromRGB(255, 255, 0)), -- เหลือง
    ColorSequenceKeypoint.new(0.6, Color3.fromRGB(0, 255, 0)),   -- เขียว
    ColorSequenceKeypoint.new(0.8, Color3.fromRGB(0, 0, 255)),   -- น้ำเงิน
    ColorSequenceKeypoint.new(1, Color3.fromRGB(148, 0, 211)),   -- ม่วง
})
```

### 37.2.5 Transparency (NumberSequence)

```lua
-- โปร่งใสคงที่ (0 = ทึบ, 1 = โปร่ง)
emitter.Transparency = NumberSequence.new(0.5)

-- Fade in แล้ว fade out
emitter.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 1),    -- เกิดมาโปร่ง
    NumberSequenceKeypoint.new(0.1, 0),  -- ทึบขึ้น
    NumberSequenceKeypoint.new(0.8, 0),  -- ยังทึบอยู่
    NumberSequenceKeypoint.new(1, 1),    -- จางหาย
})
```

### 37.2.6 Rotation และ RotSpeed

```lua
-- Rotation: การหมุนเริ่มต้น (องศา)
emitter.Rotation = NumberRange.new(-180, 180)  -- สุ่มหมุน

-- RotSpeed: ความเร็วการหมุน (องศา/วินาที)
emitter.RotSpeed = NumberRange.new(-90, 90)  -- หมุน -90 ถึง 90 องศา/วินาที
```

### 37.2.7 Acceleration และ Drag

```lua
-- Acceleration: แรงที่กระทำกับอนุภาค
emitter.Acceleration = Vector3.new(0, -10, 0)  -- แรงโน้มถ่วง

-- Drag: แรงต้านทาน (0 = ไม่มี, 10 = ต้านมาก)
emitter.Drag = 2  -- ลดความเร็วเรื่อยๆ

-- LightInfluence: ได้รับอิทธิพลจากแสงในฉาก
emitter.LightInfluence = 1  -- 0 = ไม่รับแสง, 1 = รับแสงเต็มที่

-- LightEmission: ปล่อยแสงออกมาเอง
emitter.LightEmission = 1  -- ทำให้อนุภาคเรืองแสง
```

### 37.2.8 Texture

```lua
-- Texture: รูปของอนุภาค
emitter.Texture = "rbxassetid://6101261690"  -- วงกลม
-- หรือใช้ texture สำเร็จรูป:
-- "rbxassetid://243160943"  -- iskra/spark
-- "rbxassetid://1266188397"  -- snow
-- "rbxassetid://1513844037"  -- star
-- "rbxassetid://6101261690"  -- circle glow
```

---

## 37.3 การ Emit อนุภาคด้วย Script

### Emit() Method

```lua
-- Emit(count): ปล่อยอนุภาคทันทีจำนวน count ตัว
-- ไม่ต้องเปิด Enabled

local part = workspace.ExplosionPart
local emitter = part:FindFirstChildOfClass("ParticleEmitter")

-- ปล่อยอนุภาค 100 ตัวทันที
emitter:Emit(100)
```

### Burst Effect Pattern

```lua
-- Pattern สำหรับเอฟเฟกต์ burst
local function createBurst(position, emitter, count)
    -- ย้าย emitter ไปตำแหน่งที่ต้องการ
    local part = Instance.new("Part")
    part.Anchored = true
    part.Size = Vector3.new(0.1, 0.1, 0.1)
    part.Transparency = 1
    part.CFrame = CFrame.new(position)
    part.Parent = workspace
    
    local e = emitter:Clone()
    e.Parent = part
    
    -- Emit แล้วลบ
    e:Emit(count)
    
    task.delay(e.Lifetime.Max + 0.5, function()
        part:Destroy()
    end)
end
```

---

## 37.4 เอฟเฟกต์ไฟ (Fire Effect)

### วิธีที่ 1: ใช้ Fire Object (สำเร็จรูป)

```lua
-- Fire object สร้างเอฟเฟกต์ไฟอัตโนมัติ
local part = workspace.Campfire

local fire = Instance.new("Fire")
fire.Color = Color3.fromRGB(255, 100, 0)      -- สีไฟ
fire.SecondaryColor = Color3.fromRGB(0, 0, 0) -- สีควัน
fire.Size = 5      -- ขนาดไฟ
fire.Heat = 9      -- ความร้อน (ทำให้ลอยสูง)
fire.Parent = part
```

### วิธีที่ 2: สร้าง Fire ด้วย ParticleEmitter

```lua
local function createCustomFire(parent)
    -- ไฟหลัก
    local fireEmitter = Instance.new("ParticleEmitter")
    fireEmitter.Name = "FireMain"
    fireEmitter.Texture = "rbxassetid://6101261690"
    fireEmitter.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 50, 0)),
        ColorSequenceKeypoint.new(0.3, Color3.fromRGB(255, 150, 0)),
        ColorSequenceKeypoint.new(0.7, Color3.fromRGB(255, 220, 100)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(200, 200, 200)),
    })
    fireEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(0.4, 1.2),
        NumberSequenceKeypoint.new(1, 0),
    })
    fireEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(0.7, 0.5),
        NumberSequenceKeypoint.new(1, 1),
    })
    fireEmitter.Rate = 80
    fireEmitter.Lifetime = NumberRange.new(0.5, 1.5)
    fireEmitter.Speed = NumberRange.new(3, 8)
    fireEmitter.SpreadAngle = Vector2.new(20, 20)
    fireEmitter.EmissionDirection = Enum.NormalId.Top
    fireEmitter.Acceleration = Vector3.new(0, 2, 0)
    fireEmitter.LightEmission = 0.8
    fireEmitter.LightInfluence = 0
    fireEmitter.Rotation = NumberRange.new(-180, 180)
    fireEmitter.RotSpeed = NumberRange.new(-45, 45)
    fireEmitter.Parent = parent
    
    -- ประกายไฟ
    local sparkEmitter = Instance.new("ParticleEmitter")
    sparkEmitter.Name = "Sparks"
    sparkEmitter.Texture = "rbxassetid://243160943"
    sparkEmitter.Color = ColorSequence.new(Color3.fromRGB(255, 200, 50))
    sparkEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(1, 0),
    })
    sparkEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    sparkEmitter.Rate = 20
    sparkEmitter.Lifetime = NumberRange.new(0.3, 0.8)
    sparkEmitter.Speed = NumberRange.new(5, 15)
    sparkEmitter.SpreadAngle = Vector2.new(45, 45)
    sparkEmitter.EmissionDirection = Enum.NormalId.Top
    sparkEmitter.Acceleration = Vector3.new(0, -5, 0)
    sparkEmitter.LightEmission = 1
    sparkEmitter.Parent = parent
    
    -- ควัน
    local smokeEmitter = Instance.new("ParticleEmitter")
    smokeEmitter.Name = "Smoke"
    smokeEmitter.Texture = "rbxassetid://6101261690"
    smokeEmitter.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 50, 50)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(150, 150, 150)),
    })
    smokeEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(1, 3),
    })
    smokeEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.8),
        NumberSequenceKeypoint.new(1, 1),
    })
    smokeEmitter.Rate = 15
    smokeEmitter.Lifetime = NumberRange.new(2, 4)
    smokeEmitter.Speed = NumberRange.new(1, 3)
    smokeEmitter.SpreadAngle = Vector2.new(10, 10)
    smokeEmitter.EmissionDirection = Enum.NormalId.Top
    smokeEmitter.Acceleration = Vector3.new(0.5, 1, 0)
    smokeEmitter.Parent = parent
    
    return {fireEmitter, sparkEmitter, smokeEmitter}
end

-- ใช้งาน
local campfire = workspace.Campfire
createCustomFire(campfire)
```

---

## 37.5 เอฟเฟกต์ระเบิด (Explosion Effect)

### การใช้ Explosion Object

```lua
-- Explosion Object สร้างระเบิดทันที
local explosion = Instance.new("Explosion")
explosion.Position = Vector3.new(0, 5, 0)
explosion.BlastRadius = 15    -- รัศมีระเบิด
explosion.BlastPressure = 5e5 -- แรงระเบิด (ดัน Parts ออก)
explosion.ExplosionType = Enum.ExplosionType.NoCraters -- ไม่สร้างหลุม
explosion.DestroyJointRadiusPercent = 0.5 -- ทำลาย joints ในรัศมี 50%
explosion.Parent = workspace
```

### Custom Explosion ด้วย Particles

```lua
local function createExplosion(position)
    local attachment = Instance.new("Attachment")
    attachment.WorldPosition = position
    attachment.Parent = workspace.Terrain
    
    -- แสงวาบ
    local flash = Instance.new("PointLight")
    flash.Brightness = 100
    flash.Range = 60
    flash.Color = Color3.fromRGB(255, 200, 100)
    flash.Parent = attachment
    
    -- อนุภาคไฟ
    local fireParticles = Instance.new("ParticleEmitter")
    fireParticles.Texture = "rbxassetid://6101261690"
    fireParticles.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 100)),
        ColorSequenceKeypoint.new(0.3, Color3.fromRGB(255, 100, 0)),
        ColorSequenceKeypoint.new(0.7, Color3.fromRGB(100, 100, 100)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(50, 50, 50)),
    })
    fireParticles.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(0.2, 3),
        NumberSequenceKeypoint.new(1, 0),
    })
    fireParticles.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.5, 0.3),
        NumberSequenceKeypoint.new(1, 1),
    })
    fireParticles.Speed = NumberRange.new(10, 30)
    fireParticles.SpreadAngle = Vector2.new(180, 180)
    fireParticles.Lifetime = NumberRange.new(0.5, 1.5)
    fireParticles.Acceleration = Vector3.new(0, -5, 0)
    fireParticles.LightEmission = 0.5
    fireParticles.RotSpeed = NumberRange.new(-180, 180)
    fireParticles.Parent = attachment
    
    -- ควันระเบิด
    local smokeParticles = Instance.new("ParticleEmitter")
    smokeParticles.Texture = "rbxassetid://6101261690"
    smokeParticles.Color = ColorSequence.new(Color3.fromRGB(80, 80, 80))
    smokeParticles.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(1, 8),
    })
    smokeParticles.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(1, 1),
    })
    smokeParticles.Speed = NumberRange.new(5, 15)
    smokeParticles.SpreadAngle = Vector2.new(180, 180)
    smokeParticles.Lifetime = NumberRange.new(2, 4)
    smokeParticles.Acceleration = Vector3.new(0, 3, 0)
    smokeParticles.Drag = 2
    smokeParticles.Parent = attachment
    
    -- Shockwave (วงแหวนกระจาย)
    local shockwave = Instance.new("ParticleEmitter")
    shockwave.Texture = "rbxassetid://6101261690"
    shockwave.Color = ColorSequence.new(Color3.fromRGB(255, 200, 100))
    shockwave.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(0.1, 5),
        NumberSequenceKeypoint.new(1, 0),
    })
    shockwave.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 1),
    })
    shockwave.Speed = NumberRange.new(0, 0)
    shockwave.SpreadAngle = Vector2.new(90, 90)
    shockwave.EmissionDirection = Enum.NormalId.Right
    shockwave.Lifetime = NumberRange.new(0.3, 0.5)
    shockwave.LightEmission = 0.8
    shockwave.Parent = attachment
    
    -- Emit ทันที
    fireParticles:Emit(80)
    smokeParticles:Emit(30)
    shockwave:Emit(20)
    
    -- Flash แสง
    local ts = game:GetService("TweenService")
    ts:Create(flash, TweenInfo.new(0.3, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
        Brightness = 0
    }):Play()
    
    -- ลบหลังจากอนุภาคหมดอายุ
    game:GetService("Debris"):AddItem(attachment, 5)
end

-- ทดสอบ
createExplosion(Vector3.new(0, 5, 0))
```

---

## 37.6 เอฟเฟกต์เวทมนตร์ (Magic Effects)

### Healing Aura

```lua
local function createHealingAura(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    -- Attachment ที่ตัวละคร
    local attachment = Instance.new("Attachment")
    attachment.Name = "HealAura"
    attachment.Parent = hrp
    
    -- อนุภาคใบไม้/ดอกไม้
    local leafEmitter = Instance.new("ParticleEmitter")
    leafEmitter.Texture = "rbxassetid://1513844037"  -- star shape
    leafEmitter.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(100, 255, 100)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(50, 200, 50)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 150, 0)),
    })
    leafEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.2, 0.5),
        NumberSequenceKeypoint.new(0.8, 0.3),
        NumberSequenceKeypoint.new(1, 0),
    })
    leafEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 1),
        NumberSequenceKeypoint.new(0.1, 0),
        NumberSequenceKeypoint.new(0.9, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    leafEmitter.Rate = 30
    leafEmitter.Lifetime = NumberRange.new(1.5, 2.5)
    leafEmitter.Speed = NumberRange.new(2, 5)
    leafEmitter.SpreadAngle = Vector2.new(180, 180)
    leafEmitter.Rotation = NumberRange.new(-180, 180)
    leafEmitter.RotSpeed = NumberRange.new(-90, 90)
    leafEmitter.LightEmission = 0.5
    leafEmitter.Parent = attachment
    
    -- เส้นพลังงาน
    local energyEmitter = Instance.new("ParticleEmitter")
    energyEmitter.Texture = "rbxassetid://6101261690"
    energyEmitter.Color = ColorSequence.new(Color3.fromRGB(150, 255, 150))
    energyEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 0),
    })
    energyEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    energyEmitter.Rate = 60
    energyEmitter.Lifetime = NumberRange.new(0.5, 1)
    energyEmitter.Speed = NumberRange.new(8, 12)
    energyEmitter.EmissionDirection = Enum.NormalId.Top
    energyEmitter.SpreadAngle = Vector2.new(30, 30)
    energyEmitter.LightEmission = 1
    energyEmitter.Parent = attachment
    
    return attachment
end
```

### Magic Spell Projectile

```lua
local function createMagicBall(startPos, targetPos, color)
    local ball = Instance.new("Part")
    ball.Name = "MagicBall"
    ball.Size = Vector3.new(0.5, 0.5, 0.5)
    ball.Shape = Enum.PartType.Ball
    ball.BrickColor = BrickColor.new("White")
    ball.Material = Enum.Material.Neon
    ball.CFrame = CFrame.new(startPos)
    ball.CanCollide = false
    ball.Parent = workspace
    
    -- แสงที่ลูกกลม
    local light = Instance.new("PointLight")
    light.Color = color
    light.Brightness = 5
    light.Range = 15
    light.Parent = ball
    
    -- Trail
    local trail = Instance.new("Trail")
    local a0 = Instance.new("Attachment", ball)
    local a1 = Instance.new("Attachment", ball)
    a0.Position = Vector3.new(0, 0.25, 0)
    a1.Position = Vector3.new(0, -0.25, 0)
    trail.Attachment0 = a0
    trail.Attachment1 = a1
    trail.Color = ColorSequence.new(color)
    trail.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    trail.Lifetime = 0.3
    trail.MinLength = 0
    trail.Parent = ball
    
    -- Particle glow
    local glowEmitter = Instance.new("ParticleEmitter")
    glowEmitter.Color = ColorSequence.new(color)
    glowEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 0),
    })
    glowEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(1, 1),
    })
    glowEmitter.Rate = 50
    glowEmitter.Lifetime = NumberRange.new(0.2, 0.4)
    glowEmitter.Speed = NumberRange.new(0, 2)
    glowEmitter.SpreadAngle = Vector2.new(180, 180)
    glowEmitter.LightEmission = 1
    glowEmitter.Parent = ball
    
    -- เคลื่อนที่ไปยังเป้าหมาย
    local direction = (targetPos - startPos).Unit
    local distance = (targetPos - startPos).Magnitude
    local speed = 50  -- studs/sec
    
    local bodyVelocity = Instance.new("BodyVelocity")
    bodyVelocity.Velocity = direction * speed
    bodyVelocity.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bodyVelocity.Parent = ball
    
    -- ลบหลังจากถึงเป้าหมาย
    local travelTime = distance / speed
    game:GetService("Debris"):AddItem(ball, travelTime + 0.5)
    
    return ball
end
```

---

## 37.7 Trail (รอยทาง)

`Trail` สร้างรอยทิ้งไว้ระหว่าง Attachments สองจุด

```lua
-- LocalScript ใน StarterCharacterScripts
local character = script.Parent
local hrp = character:WaitForChild("HumanoidRootPart")

-- สร้าง Attachments
local attachment0 = Instance.new("Attachment")
attachment0.Position = Vector3.new(0, 1, 0)  -- ด้านบน
attachment0.Parent = hrp

local attachment1 = Instance.new("Attachment")
attachment1.Position = Vector3.new(0, -1, 0)  -- ด้านล่าง
attachment1.Parent = hrp

-- สร้าง Trail
local trail = Instance.new("Trail")
trail.Attachment0 = attachment0
trail.Attachment1 = attachment1

-- Appearance
trail.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 100, 255)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(100, 200, 255)),
})
trail.Transparency = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 0),     -- เริ่มทึบ
    NumberSequenceKeypoint.new(1, 1),     -- หายไป
})
trail.Lifetime = 0.5      -- รอยอยู่นาน 0.5 วินาที
trail.MinLength = 0.1     -- ความยาวขั้นต่ำ
trail.WidthScale = NumberSequence.new({
    NumberSequenceKeypoint.new(0, 1),
    NumberSequenceKeypoint.new(1, 0),
})

-- เปิดใช้ Trail
trail.Enabled = true
trail.Parent = hrp

-- ปิด Trail เมื่อหยุดวิ่ง
local humanoid = character:WaitForChild("Humanoid")
humanoid.Running:Connect(function(speed)
    trail.Enabled = speed > 0.5
end)
```

### Speed Trail Effect

```lua
-- Trail ที่ปรากฏเมื่อวิ่งเร็ว
local function createSpeedTrail(character, color)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    
    -- Attachments หลายจุดเพื่อความกว้าง
    local points = {
        Vector3.new(0.5, 0.8, 0),
        Vector3.new(-0.5, 0.8, 0),
        Vector3.new(0, 0, 0),
    }
    
    local trails = {}
    
    for i = 1, #points - 1 do
        local a0 = Instance.new("Attachment")
        a0.Position = points[i]
        a0.Parent = hrp
        
        local a1 = Instance.new("Attachment")
        a1.Position = points[i + 1]
        a1.Parent = hrp
        
        local t = Instance.new("Trail")
        t.Attachment0 = a0
        t.Attachment1 = a1
        t.Color = ColorSequence.new(color)
        t.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.3),
            NumberSequenceKeypoint.new(1, 1),
        })
        t.Lifetime = 0.3
        t.LightEmission = 0.5
        t.Enabled = false
        t.Parent = hrp
        
        table.insert(trails, t)
    end
    
    -- เปิด/ปิด Trail ตามความเร็ว
    local humanoid = character:FindFirstChild("Humanoid")
    humanoid.Running:Connect(function(speed)
        local enabled = speed > 22  -- เฉพาะตอนวิ่งเร็ว
        for _, t in pairs(trails) do
            t.Enabled = enabled
        end
    end)
end
```

---

## 37.8 Beam (ลำแสง)

`Beam` ลากเส้นระหว่าง Attachments สองจุด เหมาะกับเลเซอร์, สายฟ้า, เชือก

```lua
-- สร้าง Beam ระหว่างสองจุด
local function createBeam(part0, part1, color)
    local attachment0 = Instance.new("Attachment")
    attachment0.Parent = part0
    
    local attachment1 = Instance.new("Attachment")
    attachment1.Parent = part1
    
    local beam = Instance.new("Beam")
    beam.Attachment0 = attachment0
    beam.Attachment1 = attachment1
    
    -- สี
    beam.Color = ColorSequence.new(color)
    
    -- ความโปร่งใส
    beam.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.5, 0.3),
        NumberSequenceKeypoint.new(1, 0),
    })
    
    -- ความกว้าง
    beam.Width0 = 0.5   -- ความกว้างปลาย A
    beam.Width1 = 0.5   -- ความกว้างปลาย B
    
    -- ความโค้ง
    beam.CurveSize0 = 0  -- ตรง (0) หรือโค้ง
    beam.CurveSize1 = 0
    
    -- Texture
    beam.Texture = "rbxassetid://446111271"  -- หรือใช้ default
    beam.TextureLength = 1   -- ความยาว texture tile
    beam.TextureSpeed = 0    -- ความเร็ว scroll texture
    
    -- แสง
    beam.LightEmission = 1
    beam.LightInfluence = 0
    
    beam.Parent = part0
    return beam
end

-- Laser Beam
local laserBeam = createBeam(
    workspace.LaserSource,
    workspace.LaserTarget,
    Color3.fromRGB(255, 0, 0)
)
laserBeam.TextureSpeed = 2  -- เลื่อน texture เหมือนลำแสงไหล
```

### สายฟ้า (Lightning Beam)

```lua
local function createLightning(attachment0, attachment1)
    local beam = Instance.new("Beam")
    beam.Attachment0 = attachment0
    beam.Attachment1 = attachment1
    beam.Color = ColorSequence.new(Color3.fromRGB(150, 150, 255))
    beam.Width0 = 0.2
    beam.Width1 = 0.2
    beam.LightEmission = 0.8
    
    -- ทำให้โค้งงอ (จำลองสายฟ้า)
    beam.CurveSize0 = math.random(-3, 3)
    beam.CurveSize1 = math.random(-3, 3)
    
    beam.Parent = attachment0.Parent
    
    -- อัพเดทความโค้งทุก frame เพื่อให้ดูสั่น
    local runService = game:GetService("RunService")
    local connection = runService.Heartbeat:Connect(function()
        if not beam.Parent then
            -- Disconnect ถ้า beam ถูกลบ
            return
        end
        beam.CurveSize0 = math.random(-3, 3)
        beam.CurveSize1 = math.random(-3, 3)
    end)
    
    return beam, connection
end
```

---

## 37.9 Smoke Object

```lua
-- Smoke Object ง่ายกว่า ParticleEmitter สำหรับควัน
local part = workspace.SmokePart

local smoke = Instance.new("Smoke")
smoke.Color = Color3.fromRGB(100, 100, 100)  -- สีควัน
smoke.Opacity = 0.5      -- ความทึบ
smoke.RiseVelocity = 5   -- ความเร็วลอยขึ้น
smoke.Size = 2           -- ขนาดควัน
smoke.Parent = part

-- เปิด/ปิด
smoke.Enabled = true
```

---

## 37.10 Sparkles Object

```lua
-- Sparkles Object สร้างประกายแวววาว
local part = workspace.TreasureChest

local sparkles = Instance.new("Sparkles")
sparkles.Color = Color3.fromRGB(255, 215, 0)  -- สีทอง
sparkles.SparkleColor = Color3.fromRGB(255, 255, 200)
sparkles.Enabled = true
sparkles.Parent = part
```

---

## 37.11 ระบบ Particle Effect Manager

```lua
-- ModuleScript: ParticleManager
local ParticleManager = {}

-- เก็บ templates
local templates = {}

-- ลงทะเบียน template
function ParticleManager.registerTemplate(name, template)
    templates[name] = template
end

-- เล่น effect ที่ตำแหน่ง
function ParticleManager.playEffect(name, position, duration)
    local template = templates[name]
    if not template then
        warn("ParticleManager: Template '" .. name .. "' not found")
        return
    end
    
    -- สร้าง container
    local container = Instance.new("Part")
    container.Anchored = true
    container.CanCollide = false
    container.Transparency = 1
    container.Size = Vector3.new(0.1, 0.1, 0.1)
    container.CFrame = CFrame.new(position)
    container.Parent = workspace
    
    -- Clone emitters จาก template
    for _, emitter in pairs(template:GetChildren()) do
        if emitter:IsA("ParticleEmitter") then
            local clone = emitter:Clone()
            clone.Parent = container
        end
    end
    
    -- ลบหลังจากหมดเวลา
    local maxLifetime = duration or 3
    for _, emitter in pairs(container:GetChildren()) do
        if emitter:IsA("ParticleEmitter") then
            if emitter.Lifetime.Max > maxLifetime then
                maxLifetime = emitter.Lifetime.Max
            end
        end
    end
    
    -- หยุดปล่อยอนุภาคหลัง duration
    if duration then
        task.delay(duration, function()
            for _, emitter in pairs(container:GetChildren()) do
                if emitter:IsA("ParticleEmitter") then
                    emitter.Enabled = false
                end
            end
        end)
    end
    
    game:GetService("Debris"):AddItem(container, maxLifetime + 1)
    return container
end

-- Burst effect ทันที
function ParticleManager.burstEffect(name, position, count)
    local template = templates[name]
    if not template then return end
    
    local container = Instance.new("Attachment")
    container.WorldPosition = position
    container.Parent = workspace.Terrain
    
    for _, emitter in pairs(template:GetChildren()) do
        if emitter:IsA("ParticleEmitter") then
            local clone = emitter:Clone()
            clone.Enabled = false
            clone.Parent = container
            clone:Emit(count or 30)
        end
    end
    
    -- ค้นหา Lifetime ที่ยาวที่สุด
    local maxLife = 3
    for _, emitter in pairs(template:GetChildren()) do
        if emitter:IsA("ParticleEmitter") then
            maxLife = math.max(maxLife, emitter.Lifetime.Max)
        end
    end
    
    game:GetService("Debris"):AddItem(container, maxLife + 0.5)
end

return ParticleManager
```

---

## 37.12 เอฟเฟกต์สภาพอากาศ

### ฝน (Rain Effect)

```lua
-- LocalScript ใน StarterPlayerScripts
local function createRain()
    local camera = workspace.CurrentCamera
    
    local rainPart = Instance.new("Part")
    rainPart.Name = "RainEmitter"
    rainPart.Anchored = true
    rainPart.CanCollide = false
    rainPart.Transparency = 1
    rainPart.Size = Vector3.new(0.1, 0.1, 0.1)
    rainPart.Parent = workspace
    
    local rain = Instance.new("ParticleEmitter")
    rain.Texture = "rbxassetid://6101261690"
    rain.Color = ColorSequence.new(Color3.fromRGB(150, 200, 255))
    rain.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.05),
        NumberSequenceKeypoint.new(1, 0.05),
    })
    rain.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 1),
    })
    rain.Rate = 500
    rain.Lifetime = NumberRange.new(0.8, 1.2)
    rain.Speed = NumberRange.new(60, 80)
    rain.Rotation = NumberRange.new(0, 0)
    rain.SpreadAngle = Vector2.new(5, 5)
    rain.EmissionDirection = Enum.NormalId.Bottom  -- ตกลง
    rain.Acceleration = Vector3.new(0, -10, 0)
    rain.Parent = rainPart
    
    -- ติดตาม Camera
    local runService = game:GetService("RunService")
    runService.RenderStepped:Connect(function()
        rainPart.CFrame = CFrame.new(
            camera.CFrame.Position + Vector3.new(0, 50, 0)
        )
    end)
    
    return rainPart
end

local rainEffect = createRain()
```

### หิมะ (Snow Effect)

```lua
local function createSnow()
    local camera = workspace.CurrentCamera
    
    local snowPart = Instance.new("Part")
    snowPart.Anchored = true
    snowPart.CanCollide = false
    snowPart.Transparency = 1
    snowPart.Size = Vector3.new(0.1, 0.1, 0.1)
    snowPart.Parent = workspace
    
    local snow = Instance.new("ParticleEmitter")
    snow.Texture = "rbxassetid://1266188397"  -- snow texture
    snow.Color = ColorSequence.new(Color3.fromRGB(255, 255, 255))
    snow.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(0.5, 0.4),
        NumberSequenceKeypoint.new(1, 0),
    })
    snow.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.2),
        NumberSequenceKeypoint.new(0.9, 0.5),
        NumberSequenceKeypoint.new(1, 1),
    })
    snow.Rate = 200
    snow.Lifetime = NumberRange.new(4, 6)
    snow.Speed = NumberRange.new(5, 15)
    snow.Rotation = NumberRange.new(-180, 180)
    snow.RotSpeed = NumberRange.new(-30, 30)
    snow.SpreadAngle = Vector2.new(180, 180)
    snow.EmissionDirection = Enum.NormalId.Bottom
    snow.Acceleration = Vector3.new(1, -5, 1)  -- เยื้องนิดนึง
    snow.Drag = 3  -- ลอยช้าๆ
    snow.Parent = snowPart
    
    local runService = game:GetService("RunService")
    runService.RenderStepped:Connect(function()
        snowPart.CFrame = CFrame.new(
            camera.CFrame.Position + Vector3.new(0, 60, 0)
        )
    end)
    
    return snowPart
end
```

---

## 37.13 เอฟเฟกต์ตัวละคร

### Death Effect

```lua
-- LocalScript ใน StarterCharacterScripts
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")

local function onDied()
    -- สร้างเอฟเฟกต์เมื่อตาย
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local deathPos = hrp.Position
    
    -- อนุภาคระเบิดออก
    local attachment = Instance.new("Attachment")
    attachment.WorldPosition = deathPos
    attachment.Parent = workspace.Terrain
    
    local dustEmitter = Instance.new("ParticleEmitter")
    dustEmitter.Color = ColorSequence.new(Color3.fromRGB(200, 150, 100))
    dustEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 2),
    })
    dustEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(1, 1),
    })
    dustEmitter.Speed = NumberRange.new(5, 15)
    dustEmitter.SpreadAngle = Vector2.new(180, 180)
    dustEmitter.Lifetime = NumberRange.new(1, 2)
    dustEmitter.Acceleration = Vector3.new(0, 5, 0)
    dustEmitter.Parent = attachment
    dustEmitter:Emit(50)
    
    game:GetService("Debris"):AddItem(attachment, 3)
end

humanoid.Died:Connect(onDied)
```

### Level Up Effect

```lua
local function playLevelUpEffect(character)
    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local attachment = Instance.new("Attachment")
    attachment.Parent = hrp
    
    -- วงแหวนพลังงานขยายออก
    local ringEmitter = Instance.new("ParticleEmitter")
    ringEmitter.Texture = "rbxassetid://6101261690"
    ringEmitter.Color = ColorSequence.new(Color3.fromRGB(255, 215, 0))
    ringEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(0.5, 1),
        NumberSequenceKeypoint.new(1, 0),
    })
    ringEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    ringEmitter.Speed = NumberRange.new(10, 20)
    ringEmitter.SpreadAngle = Vector2.new(90, 0)  -- ขยายในแนวนอน
    ringEmitter.EmissionDirection = Enum.NormalId.Right
    ringEmitter.Lifetime = NumberRange.new(0.5, 1)
    ringEmitter.LightEmission = 0.8
    ringEmitter.Parent = attachment
    ringEmitter:Emit(40)
    
    -- ดาวลอยขึ้น
    local starEmitter = Instance.new("ParticleEmitter")
    starEmitter.Texture = "rbxassetid://1513844037"
    starEmitter.Color = ColorSequence.new(Color3.fromRGB(255, 255, 100))
    starEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.2, 0.6),
        NumberSequenceKeypoint.new(1, 0),
    })
    starEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.8, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    starEmitter.Speed = NumberRange.new(5, 15)
    starEmitter.SpreadAngle = Vector2.new(45, 45)
    starEmitter.EmissionDirection = Enum.NormalId.Top
    starEmitter.Lifetime = NumberRange.new(1, 2)
    starEmitter.Rotation = NumberRange.new(-180, 180)
    starEmitter.RotSpeed = NumberRange.new(-90, 90)
    starEmitter.LightEmission = 1
    starEmitter.Parent = attachment
    starEmitter:Emit(30)
    
    game:GetService("Debris"):AddItem(attachment, 3)
end
```

---

## 37.14 Optimization เทคนิค

### Object Pooling สำหรับ Particles

```lua
-- ModuleScript: ParticlePool
local ParticlePool = {}
ParticlePool.__index = ParticlePool

function ParticlePool.new(template, poolSize)
    local self = setmetatable({}, ParticlePool)
    self.template = template
    self.pool = {}
    self.available = {}
    
    -- สร้าง pool ล่วงหน้า
    for i = 1, poolSize do
        local container = Instance.new("Folder")
        container.Name = "ParticleContainer_" .. i
        container.Parent = workspace
        
        local emitters = {}
        for _, child in pairs(template:GetChildren()) do
            if child:IsA("ParticleEmitter") then
                local clone = child:Clone()
                clone.Enabled = false
                clone.Parent = container
                table.insert(emitters, clone)
            end
        end
        
        self.pool[i] = {container = container, emitters = emitters}
        table.insert(self.available, i)
    end
    
    return self
end

function ParticlePool:play(position, duration)
    if #self.available == 0 then
        warn("ParticlePool: ไม่มี particle ว่างใน pool")
        return
    end
    
    local index = table.remove(self.available, 1)
    local item = self.pool[index]
    
    -- ย้ายไปตำแหน่งที่ต้องการ
    if item.container:IsA("Folder") then
        -- หา Part ที่จะย้าย
    end
    
    -- เปิด emitters
    for _, emitter in pairs(item.emitters) do
        emitter.Enabled = true
    end
    
    -- คืน pool หลัง duration
    task.delay(duration or 3, function()
        for _, emitter in pairs(item.emitters) do
            emitter.Enabled = false
        end
        table.insert(self.available, index)
    end)
    
    return item
end

return ParticlePool
```

### เคล็ดลับประหยัด Performance

```lua
-- 1. ปิด Particle เมื่อไม่มองเห็น
local camera = workspace.CurrentCamera

game:GetService("RunService").Heartbeat:Connect(function()
    for _, emitter in pairs(workspace:GetDescendants()) do
        if emitter:IsA("ParticleEmitter") then
            local part = emitter.Parent
            if part:IsA("BasePart") then
                local distance = (camera.CFrame.Position - part.Position).Magnitude
                emitter.Enabled = distance < 200  -- ปิดถ้าไกลเกิน 200
            end
        end
    end
end)

-- 2. ลด Rate เมื่อไกลออกไป
local function updateParticleRates(emitter, baseRate, cameraPart)
    game:GetService("RunService").Heartbeat:Connect(function()
        local dist = (workspace.CurrentCamera.CFrame.Position - cameraPart.Position).Magnitude
        local rate = baseRate * math.max(0, 1 - dist / 100)
        emitter.Rate = rate
    end)
end
```

---

## 37.15 ตัวอย่าง: ระบบ Ability Effects

```lua
-- LocalScript: AbilityEffects
-- ระบบเอฟเฟกต์ที่ใช้กับ ability ต่างๆ

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local hrp = character:WaitForChild("HumanoidRootPart")

-- ============ FIRE ABILITY ============
local function activateFireAbility()
    local attachment = Instance.new("Attachment")
    attachment.Name = "FireAbility"
    attachment.Parent = hrp
    
    -- ไฟรอบตัว
    local auraEmitter = Instance.new("ParticleEmitter")
    auraEmitter.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 50, 0)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(255, 150, 0)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 200, 50)),
    })
    auraEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.3),
        NumberSequenceKeypoint.new(0.5, 0.8),
        NumberSequenceKeypoint.new(1, 0),
    })
    auraEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.5),
        NumberSequenceKeypoint.new(1, 1),
    })
    auraEmitter.Rate = 100
    auraEmitter.Lifetime = NumberRange.new(0.5, 1)
    auraEmitter.Speed = NumberRange.new(3, 8)
    auraEmitter.SpreadAngle = Vector2.new(180, 180)
    auraEmitter.LightEmission = 0.8
    auraEmitter.Acceleration = Vector3.new(0, 5, 0)
    auraEmitter.Parent = attachment
    
    -- แสงไฟ
    local fireLight = Instance.new("PointLight")
    fireLight.Color = Color3.fromRGB(255, 100, 0)
    fireLight.Brightness = 5
    fireLight.Range = 20
    fireLight.Parent = hrp
    
    -- Active flag
    local active = true
    
    -- ดับหลัง 5 วินาที
    task.delay(5, function()
        active = false
        auraEmitter.Enabled = false
        
        TweenService:Create(fireLight, TweenInfo.new(1), {
            Brightness = 0
        }):Play()
        
        game:GetService("Debris"):AddItem(attachment, 2)
        game:GetService("Debris"):AddItem(fireLight, 1.5)
    end)
    
    return attachment
end

-- ============ ICE ABILITY ============
local function activateIceAbility()
    local attachment = Instance.new("Attachment")
    attachment.Name = "IceAbility"
    attachment.Parent = hrp
    
    -- ผลึกน้ำแข็ง
    local crystalEmitter = Instance.new("ParticleEmitter")
    crystalEmitter.Color = ColorSequence.new(Color3.fromRGB(150, 220, 255))
    crystalEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(0.3, 0.5),
        NumberSequenceKeypoint.new(0.7, 0.4),
        NumberSequenceKeypoint.new(1, 0),
    })
    crystalEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.7),
        NumberSequenceKeypoint.new(0.5, 0.3),
        NumberSequenceKeypoint.new(1, 1),
    })
    crystalEmitter.Rate = 80
    crystalEmitter.Lifetime = NumberRange.new(1, 2)
    crystalEmitter.Speed = NumberRange.new(2, 6)
    crystalEmitter.SpreadAngle = Vector2.new(90, 90)
    crystalEmitter.Rotation = NumberRange.new(-180, 180)
    crystalEmitter.RotSpeed = NumberRange.new(-60, 60)
    crystalEmitter.Drag = 5
    crystalEmitter.LightEmission = 0.3
    crystalEmitter.Parent = attachment
    
    -- Frost particles
    local frostEmitter = Instance.new("ParticleEmitter")
    frostEmitter.Color = ColorSequence.new(Color3.fromRGB(200, 240, 255))
    frostEmitter.Size = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0.1),
        NumberSequenceKeypoint.new(1, 0),
    })
    frostEmitter.Transparency = NumberSequence.new({
        NumberSequenceKeypoint.new(0, 0),
        NumberSequenceKeypoint.new(1, 1),
    })
    frostEmitter.Rate = 150
    frostEmitter.Lifetime = NumberRange.new(0.3, 0.8)
    frostEmitter.Speed = NumberRange.new(1, 4)
    frostEmitter.SpreadAngle = Vector2.new(180, 180)
    frostEmitter.LightEmission = 0.5
    frostEmitter.Parent = attachment
    
    return attachment
end

-- ============ THUNDER ABILITY ============
local function activateThunderAbility()
    -- สร้าง Beams สายฟ้าหลายเส้น
    local connections = {}
    
    local function spawnLightning()
        local origin = hrp.Position + Vector3.new(0, 5, 0)
        local targetOffset = Vector3.new(
            math.random(-20, 20),
            math.random(-15, 0),
            math.random(-20, 20)
        )
        local target = origin + targetOffset
        
        -- ลำแสงสายฟ้า
        local part = Instance.new("Part")
        part.Anchored = true
        part.CanCollide = false
        part.Transparency = 1
        part.Size = Vector3.new(0.1, 0.1, 0.1)
        part.CFrame = CFrame.new(origin)
        part.Parent = workspace
        
        local a0 = Instance.new("Attachment", part)
        
        local targetPart = Instance.new("Part")
        targetPart.Anchored = true
        targetPart.CanCollide = false
        targetPart.Transparency = 1
        targetPart.Size = Vector3.new(0.1, 0.1, 0.1)
        targetPart.CFrame = CFrame.new(target)
        targetPart.Parent = workspace
        
        local a1 = Instance.new("Attachment", targetPart)
        
        local beam = Instance.new("Beam")
        beam.Attachment0 = a0
        beam.Attachment1 = a1
        beam.Color = ColorSequence.new(Color3.fromRGB(150, 150, 255))
        beam.Width0 = 0.15
        beam.Width1 = 0.05
        beam.LightEmission = 1
        beam.CurveSize0 = math.random(-5, 5)
        beam.CurveSize1 = math.random(-5, 5)
        beam.Parent = part
        
        -- แสงวาบ
        local flash = Instance.new("PointLight", part)
        flash.Color = Color3.fromRGB(200, 200, 255)
        flash.Brightness = 10
        flash.Range = 40
        
        -- ลบหลังสั้นๆ
        game:GetService("Debris"):AddItem(part, 0.1)
        game:GetService("Debris"):AddItem(targetPart, 0.1)
    end
    
    -- สร้างสายฟ้าทุก 0.2 วินาที นาน 3 วินาที
    local thunderActive = true
    task.spawn(function()
        while thunderActive do
            spawnLightning()
            task.wait(0.15)
        end
    end)
    
    task.delay(3, function()
        thunderActive = false
    end)
end

-- Remote event เพื่อส่งสัญญาณ ability
local remoteEvent = game:GetService("ReplicatedStorage"):WaitForChild("AbilityEvent")
remoteEvent.OnClientEvent:Connect(function(abilityName)
    if abilityName == "Fire" then
        activateFireAbility()
    elseif abilityName == "Ice" then
        activateIceAbility()
    elseif abilityName == "Thunder" then
        activateThunderAbility()
    end
end)
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Campfire
สร้าง Campfire พร้อม:
- ไฟที่กระพริบได้ตามธรรมชาติ
- ควันที่ลอยขึ้น
- ประกายไฟที่กระเด็นออกมา
- แสง PointLight ที่กระพริบ

### แบบฝึกหัดที่ 2: Magic Portal
สร้าง Portal เวทมนตร์ที่มี:
- วงแหวนหมุน (Beam วงกลม)
- อนุภาคพลังงานลอยออกมา
- แสงส่องออกมาจากตรงกลาง
- Trail สวยๆ เมื่อตัวละครเดินผ่าน

### แบบฝึกหัดที่ 3: Weather System
สร้างระบบสภาพอากาศที่:
- เปลี่ยนได้ 4 แบบ: Clear, Rain, Snow, Storm
- มี transition effect เมื่อเปลี่ยน
- ฝนมีเสียงประกอบ
- หิมะสะสมบน Parts ได้

### แบบฝึกหัดที่ 4: Particle Treasure System
สร้างระบบสมบัติที่:
- มี Sparkles เมื่ออยู่ใกล้
- เมื่อเก็บได้มีเอฟเฟกต์ระเบิดทองคำ
- Coins ลอยขึ้นแล้วดูดเข้าหาผู้เล่น (tween)
- Text "+100 Gold" ลอยขึ้นพร้อม fade

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- **ParticleEmitter**: properties ทั้งหมดและการปรับแต่ง
- **NumberSequence/ColorSequence**: กำหนดค่าตลอดอายุอนุภาค
- **Trail**: รอยทิ้งไว้ระหว่างสองจุด
- **Beam**: ลำแสงระหว่าง Attachments
- **Fire, Smoke, Sparkles**: Objects สำเร็จรูป
- **เอฟเฟกต์ต่างๆ**: ไฟ, ระเบิด, เวทมนตร์, สภาพอากาศ
- **Optimization**: Object Pooling, ปิดเมื่อไกล

---

## อ้างอิง
- [ParticleEmitter](https://create.roblox.com/docs/reference/engine/classes/ParticleEmitter)
- [Trail](https://create.roblox.com/docs/reference/engine/classes/Trail)
- [Beam](https://create.roblox.com/docs/reference/engine/classes/Beam)
- [NumberSequence](https://create.roblox.com/docs/reference/engine/datatypes/NumberSequence)
- [ColorSequence](https://create.roblox.com/docs/reference/engine/datatypes/ColorSequence)
