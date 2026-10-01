# Part 47: Object-Oriented Programming พื้นฐานใน Lua

## บทนำ

Object-Oriented Programming (OOP) หรือการเขียนโปรแกรมเชิงวัตถุ เป็นแนวทางการเขียนโค้ดที่จัดระเบียบ code ให้เป็น "วัตถุ" (objects) ที่มีทั้งข้อมูล (properties) และพฤติกรรม (methods) แม้ Lua จะไม่ได้มี class system แบบ built-in แต่เราสามารถสร้าง OOP ได้ผ่าน metatables

## แนวคิดพื้นฐาน OOP

### 4 หลักการหลักของ OOP

1. **Encapsulation** - ซ่อนข้อมูลภายใน
2. **Inheritance** - สืบทอดคุณสมบัติ
3. **Polymorphism** - ทำงานต่างกันตาม context
4. **Abstraction** - ซ่อนความซับซ้อน

## Class พื้นฐานใน Lua

### Class อย่างง่าย

```lua
-- สร้าง "Class" ด้วย table และ metatable

-- ===== Animal Class =====
local Animal = {}
Animal.__index = Animal  -- สำคัญ! ทำให้ method lookup ทำงาน

-- Constructor
function Animal.new(name, species, sound)
    local self = setmetatable({}, Animal)
    
    -- Properties
    self.name = name
    self.species = species
    self.sound = sound
    self.health = 100
    self.isAlive = true
    
    return self
end

-- Methods
function Animal:speak()
    if self.isAlive then
        print(self.name .. " พูดว่า: " .. self.sound .. "!")
    else
        print(self.name .. " ตายไปแล้ว...")
    end
end

function Animal:takeDamage(amount)
    if not self.isAlive then return end
    
    self.health = math.max(0, self.health - amount)
    print(self.name .. " ได้รับ " .. amount .. " damage (HP: " .. self.health .. ")")
    
    if self.health <= 0 then
        self:die()
    end
end

function Animal:heal(amount)
    if not self.isAlive then return end
    self.health = math.min(100, self.health + amount)
    print(self.name .. " ฟื้นฟู " .. amount .. " HP (HP: " .. self.health .. ")")
end

function Animal:die()
    self.isAlive = false
    print("💀 " .. self.name .. " ตายแล้ว!")
end

function Animal:getStatus()
    return string.format(
        "%s (%s) - HP: %d, Status: %s",
        self.name,
        self.species,
        self.health,
        self.isAlive and "มีชีวิต" or "ตาย"
    )
end

-- ===== ทดสอบ =====
local dog = Animal.new("บัดดี้", "สุนัข", "โฮ่ง")
local cat = Animal.new("มิ้ว", "แมว", "เมี้ยว")

dog:speak()
cat:speak()

dog:takeDamage(30)
print(dog:getStatus())

dog:heal(20)
print(dog:getStatus())

dog:takeDamage(100)
print(dog:getStatus())
```

### ทำความเข้าใจ Metatable

```lua
-- อธิบาย setmetatable และ __index

-- เมื่อเรียก self:speak() Lua จะทำแบบนี้:
-- 1. หา "speak" ใน self table -> ไม่เจอ
-- 2. หา metatable ของ self -> พบ Animal
-- 3. หา __index ใน metatable -> Animal (table นั้นเอง)
-- 4. หา "speak" ใน Animal -> เจอ! เรียกใช้

-- ตัวอย่างแสดงการทำงาน
local myObject = {}
local myClass = {
    greet = function(self)
        print("สวัสดี! ฉันชื่อ " .. self.name)
    end
}
myClass.__index = myClass

setmetatable(myObject, myClass)
myObject.name = "ทดสอบ"

myObject:greet()  -- "สวัสดี! ฉันชื่อ ทดสอบ"
-- เหมือนกับ myClass.greet(myObject)

-- Shorthand syntax: self:method() == Class.method(self)
```

## Character Class สำหรับ RPG

```lua
-- ReplicatedStorage/Classes/Character.lua (ModuleScript)
-- Character class สำหรับเกม RPG

local Character = {}
Character.__index = Character

-- ===== Constructor =====
function Character.new(name, characterClass)
    local self = setmetatable({}, Character)
    
    -- ข้อมูลพื้นฐาน
    self.name = name
    self.class = characterClass or "Warrior"
    
    -- Stats
    self.level = 1
    self.experience = 0
    self.expToNextLevel = 100
    
    -- HP/MP
    self.maxHealth = 100
    self.health = 100
    self.maxMana = 50
    self.mana = 50
    
    -- Combat Stats
    self.attack = 10
    self.defense = 5
    self.speed = 10
    self.critChance = 0.1    -- 10%
    self.critMultiplier = 1.5
    
    -- Status
    self.isAlive = true
    self.statusEffects = {}  -- { type, duration, value }
    
    -- Inventory
    self.inventory = {}
    self.equippedItems = {
        weapon = nil,
        armor = nil,
        accessory = nil
    }
    
    -- Abilities
    self.abilities = {}
    self.abilityCooldowns = {}
    
    return self
end

-- ===== Stats Calculation =====

function Character:getTotalAttack()
    local base = self.attack
    
    -- เพิ่มจาก weapon
    if self.equippedItems.weapon then
        base = base + (self.equippedItems.weapon.damage or 0)
    end
    
    -- เพิ่มจาก buffs
    for _, effect in ipairs(self.statusEffects) do
        if effect.type == "attackBuff" then
            base = base + effect.value
        end
    end
    
    return base
end

function Character:getTotalDefense()
    local base = self.defense
    
    if self.equippedItems.armor then
        base = base + (self.equippedItems.armor.defense or 0)
    end
    
    return base
end

-- ===== Combat Methods =====

function Character:attack(target)
    if not self.isAlive then
        print(self.name .. " ตายแล้ว โจมตีไม่ได้")
        return 0
    end
    
    -- คำนวณ damage
    local damage = self:getTotalAttack()
    
    -- Critical hit
    local isCrit = math.random() < self.critChance
    if isCrit then
        damage = math.floor(damage * self.critMultiplier)
    end
    
    -- Target defense
    local finalDamage = math.max(1, damage - target:getTotalDefense())
    
    -- Apply damage
    target:takeDamage(finalDamage, self)
    
    local critText = isCrit and " (💥 Critical!)" or ""
    print(string.format("%s โจมตี %s: %d damage%s", 
        self.name, target.name, finalDamage, critText))
    
    return finalDamage
end

function Character:takeDamage(amount, attacker)
    if not self.isAlive then return end
    
    self.health = math.max(0, self.health - amount)
    
    if self.health <= 0 then
        self:die(attacker)
    end
end

function Character:heal(amount)
    if not self.isAlive then return end
    
    local oldHealth = self.health
    self.health = math.min(self.maxHealth, self.health + amount)
    local healed = self.health - oldHealth
    
    print(self.name .. " ฟื้นฟู " .. healed .. " HP (" .. self.health .. "/" .. self.maxHealth .. ")")
    return healed
end

function Character:useMana(amount)
    if self.mana < amount then
        return false, "Mana ไม่พอ"
    end
    self.mana = self.mana - amount
    return true
end

function Character:regenerateMana(amount)
    self.mana = math.min(self.maxMana, self.mana + amount)
end

function Character:die(killer)
    self.isAlive = false
    self.health = 0
    
    local killerName = killer and killer.name or "สิ่งแวดล้อม"
    print(string.format("💀 %s ถูก %s สังหาร!", self.name, killerName))
end

function Character:revive(healthPercent)
    healthPercent = healthPercent or 0.5
    self.isAlive = true
    self.health = math.floor(self.maxHealth * healthPercent)
    print(self.name .. " ฟื้นคืนชีพแล้ว! HP: " .. self.health)
end

-- ===== Level System =====

function Character:gainExperience(amount)
    if self.level >= 100 then return end  -- Max level
    
    self.experience = self.experience + amount
    
    while self.experience >= self.expToNextLevel do
        self.experience = self.experience - self.expToNextLevel
        self:levelUp()
    end
end

function Character:levelUp()
    self.level = self.level + 1
    self.expToNextLevel = math.floor(100 * (1.5 ^ (self.level - 1)))
    
    -- เพิ่ม stats เมื่อ level up
    self.maxHealth = self.maxHealth + 10
    self.health = self.maxHealth  -- ฟื้น HP เต็ม
    self.maxMana = self.maxMana + 5
    self.mana = self.maxMana
    self.attack = self.attack + 2
    self.defense = self.defense + 1
    
    print(string.format("⭐ %s เลื่อนระดับเป็น Level %d!", self.name, self.level))
    print(string.format("   HP: %d | MP: %d | ATK: %d | DEF: %d",
        self.maxHealth, self.maxMana, self.attack, self.defense))
end

-- ===== Equipment =====

function Character:equip(item)
    local slot = item.slot  -- weapon, armor, accessory
    
    if not slot or not self.equippedItems[slot] ~= nil then
        print("ไม่สามารถสวมใส่ได้")
        return false
    end
    
    -- ถอดของเก่า
    if self.equippedItems[slot] then
        self:unequip(slot)
    end
    
    self.equippedItems[slot] = item
    print(self.name .. " สวม " .. item.name)
    return true
end

function Character:unequip(slot)
    local item = self.equippedItems[slot]
    if item then
        self.equippedItems[slot] = nil
        print(self.name .. " ถอด " .. item.name)
        return item
    end
    return nil
end

-- ===== Status Effects =====

function Character:addStatusEffect(effectType, duration, value)
    -- ตรวจสอบว่ามีอยู่แล้วหรือยัง
    for i, effect in ipairs(self.statusEffects) do
        if effect.type == effectType then
            -- รีเซ็ต duration
            self.statusEffects[i].duration = duration
            return
        end
    end
    
    table.insert(self.statusEffects, {
        type = effectType,
        duration = duration,
        value = value or 0,
        startTime = os.time()
    })
    
    print(self.name .. " ได้รับ status: " .. effectType)
end

function Character:updateStatusEffects(deltaTime)
    for i = #self.statusEffects, 1, -1 do
        local effect = self.statusEffects[i]
        effect.duration = effect.duration - deltaTime
        
        -- ใช้ effect
        if effect.type == "poison" then
            self:takeDamage(effect.value * deltaTime)
        elseif effect.type == "regen" then
            self:heal(effect.value * deltaTime)
        end
        
        -- ลบถ้าหมดเวลา
        if effect.duration <= 0 then
            print(self.name .. " status " .. effect.type .. " หมดแล้ว")
            table.remove(self.statusEffects, i)
        end
    end
end

-- ===== Information =====

function Character:getInfo()
    return {
        name = self.name,
        class = self.class,
        level = self.level,
        health = self.health,
        maxHealth = self.maxHealth,
        mana = self.mana,
        maxMana = self.maxMana,
        attack = self:getTotalAttack(),
        defense = self:getTotalDefense(),
        isAlive = self.isAlive
    }
end

function Character:printStatus()
    print(string.format(
        "=== %s [%s Lv.%d] ===\n" ..
        "HP: %d/%d  MP: %d/%d\n" ..
        "ATK: %d  DEF: %d  SPD: %d",
        self.name, self.class, self.level,
        self.health, self.maxHealth,
        self.mana, self.maxMana,
        self:getTotalAttack(), self:getTotalDefense(), self.speed
    ))
end

return Character
```

### ทดสอบ Character Class

```lua
-- ทดสอบใน Script
local Character = require(game.ReplicatedStorage.Classes.Character)

-- สร้าง characters
local hero = Character.new("อาร์เธอร์", "Warrior")
local enemy = Character.new("กอบลิน", "Monster")

-- ลดค่า enemy
enemy.maxHealth = 50
enemy.health = 50
enemy.attack = 8
enemy.defense = 2

-- แสดงสถานะ
hero:printStatus()
print("---")
enemy:printStatus()
print("---")

-- จำลองการต่อสู้
print("=== เริ่มต่อสู้ ===")
local round = 0

while hero.isAlive and enemy.isAlive do
    round = round + 1
    print("\n-- รอบ " .. round .. " --")
    
    -- hero โจมตี
    hero:attack(enemy)
    
    if enemy.isAlive then
        -- enemy โจมตีกลับ
        enemy:attack(hero)
    end
    
    if round > 20 then  -- safety break
        print("การต่อสู้ยาวเกินไป!")
        break
    end
end

print("\n=== จบการต่อสู้ ===")
if hero.isAlive then
    hero:gainExperience(100)
    hero:printStatus()
else
    print("💀 Hero แพ้!")
end
```

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Monster Class
สร้าง class Monster ที่มี:
- ประเภทศัตรู (Slime, Goblin, Dragon)
- AI พื้นฐาน (เดินหาผู้เล่น)
- Drop items เมื่อตาย
- ระดับความแข็งแกร่ง

### แบบฝึกหัดที่ 2: Item Class
สร้าง class Item ด้วย Weapon, Armor, Consumable subclasses

### แบบฝึกหัดที่ 3: Ability Class
สร้าง class Ability สำหรับ skill ต่างๆ เช่น Fireball, Shield, Heal

## สรุป

OOP ใน Lua ใช้ principles:
1. **setmetatable + __index** = class inheritance
2. **self:method()** = instance method call
3. **Constructor.new()** = สร้าง object ใหม่
4. **table เป็น object** = state ของ object

ในบทถัดไปเราจะเรียนรู้ Inheritance และ Advanced OOP patterns
