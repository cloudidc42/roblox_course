# Part 87: Advertising - การโฆษณาและสปอนเซอร์ในเกม Roblox

## บทนำ

การโฆษณาในเกม Roblox มีหลายรูปแบบ ตั้งแต่การโฆษณาเกมของตัวเองไปยังผู้เล่นใหม่ ไปจนถึงการรับสปอนเซอร์จากแบรนด์ภายนอก ในบทนี้เราจะเรียนรู้ทั้งสองด้าน

---

## ส่วนที่ 1: Roblox Advertising Platform

### 1.1 ประเภทโฆษณาที่ Roblox รองรับ

**Immersive Ads (Roblox Official)**
- โฆษณาที่ Roblox ฝัง portal/billboard ให้อัตโนมัติ
- นักพัฒนาได้รับส่วนแบ่งรายได้
- ควบคุม placement ได้

**User Ads**
- โฆษณาเกมของตัวเองในหน้าแรก Roblox
- ใช้ Robux ซื้อ impression/clicks
- มี banner หลายขนาด

**Sponsored Games**
- เกมขึ้นหน้า "Sponsored" ในหน้าหลัก
- ค่าใช้จ่ายสูงกว่า User Ads
- เข้าถึงผู้ชมมากกว่า

### 1.2 การสร้าง Ad Portal (Immersive Ads)

```lua
-- Script: ImmersiveAdsManager (Script ใน ServerScriptService)
-- จัดการ Ad Portals ในเกม

-- Roblox Immersive Ads ทำงานผ่าน AdService
-- ต้องเปิดใช้ใน game settings ก่อน

local AdService = game:GetService("AdService")
local Players = game:GetService("Players")

-- ตรวจสอบว่าผู้เล่นเห็นโฆษณาได้หรือไม่
local function canShowAd(player)
    -- ผู้เล่น Premium ไม่เห็นโฆษณา (optional)
    local MarketplaceService = game:GetService("MarketplaceService")
    
    local success, isPremium = pcall(function()
        return MarketplaceService:GetUserSubscriptionStatusAsync(player, "premium")
    end)
    
    -- ถ้าไม่สามารถตรวจสอบ ให้แสดงโฆษณา
    return true
end

-- Track ad impressions
local adImpressions = {}

local function trackAdImpression(player, adType)
    local userId = player.UserId
    if not adImpressions[userId] then
        adImpressions[userId] = {count = 0, types = {}}
    end
    
    adImpressions[userId].count += 1
    adImpressions[userId].types[adType] = (adImpressions[userId].types[adType] or 0) + 1
end

Players.PlayerRemoving:Connect(function(player)
    adImpressions[player.UserId] = nil
end)
```

---

## ส่วนที่ 2: In-Game Billboard System

```lua
-- ModuleScript: BillboardSystem
-- ระบบป้ายโฆษณาในเกม

local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local BillboardSystem = {}

-- กำหนด billboard ในแผนที่
local BILLBOARD_LOCATIONS = {
    {
        name = "Billboard_1",
        position = Vector3.new(50, 15, 50),
        size = Vector3.new(20, 10, 1),
        facing = Vector3.new(0, 0, 1),
    },
    {
        name = "Billboard_2",
        position = Vector3.new(-50, 15, 0),
        size = Vector3.new(15, 8, 1),
        facing = Vector3.new(1, 0, 0),
    },
    {
        name = "Billboard_3",
        position = Vector3.new(0, 20, -80),
        size = Vector3.new(25, 12, 1),
        facing = Vector3.new(0, 0, -1),
    },
}

-- โฆษณาที่จะหมุนเวียน
local ADS = {
    -- โฆษณาเกมเอง
    {
        type = "self_promo",
        image = "rbxassetid://your_game_banner",
        link = nil,
        duration = 10, -- วินาทีที่แสดง
        title = "ลอง Game Mode ใหม่!",
        description = "Arena PvP เพิ่งเปิดตัว",
    },
    {
        type = "self_promo",
        image = "rbxassetid://vip_banner",
        link = nil,
        duration = 8,
        title = "VIP Pass",
        description = "รับสิทธิ์พิเศษทันที",
    },
}

-- สร้าง billboard
function BillboardSystem:CreateBillboard(config)
    local billboard = Instance.new("Part")
    billboard.Name = config.name
    billboard.Size = config.size
    billboard.Position = config.position
    billboard.CFrame = CFrame.new(config.position) * CFrame.lookAt(Vector3.zero, config.facing)
    billboard.Anchored = true
    billboard.Material = Enum.Material.Neon
    billboard.BrickColor = BrickColor.new("Black")
    billboard.Parent = workspace
    
    -- SurfaceGui สำหรับแสดงโฆษณา
    local surfaceGui = Instance.new("SurfaceGui")
    surfaceGui.Face = Enum.NormalId.Front
    surfaceGui.SizingMode = Enum.SurfaceGuiSizingMode.FixedSize
    surfaceGui.CanvasSize = Vector2.new(1024, 512)
    surfaceGui.Parent = billboard
    
    -- พื้นหลัง
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(1, 0, 1, 0)
    bg.BackgroundColor3 = Color3.fromRGB(10, 10, 20)
    bg.Parent = surfaceGui
    
    -- รูปภาพโฆษณา
    local adImage = Instance.new("ImageLabel")
    adImage.Name = "AdImage"
    adImage.Size = UDim2.new(0.6, 0, 1, 0)
    adImage.BackgroundTransparency = 1
    adImage.Parent = bg
    
    -- ข้อความโฆษณา
    local adTitle = Instance.new("TextLabel")
    adTitle.Name = "AdTitle"
    adTitle.Size = UDim2.new(0.38, 0, 0.4, 0)
    adTitle.Position = UDim2.new(0.62, 0, 0.05, 0)
    adTitle.BackgroundTransparency = 1
    adTitle.TextColor3 = Color3.new(1, 1, 1)
    adTitle.TextSize = 60
    adTitle.Font = Enum.Font.GothamBold
    adTitle.TextWrapped = true
    adTitle.TextXAlignment = Enum.TextXAlignment.Left
    adTitle.Parent = bg
    
    local adDesc = Instance.new("TextLabel")
    adDesc.Name = "AdDesc"
    adDesc.Size = UDim2.new(0.38, 0, 0.35, 0)
    adDesc.Position = UDim2.new(0.62, 0, 0.45, 0)
    adDesc.BackgroundTransparency = 1
    adDesc.TextColor3 = Color3.fromRGB(200, 200, 200)
    adDesc.TextSize = 40
    adDesc.Font = Enum.Font.Gotham
    adDesc.TextWrapped = true
    adDesc.TextXAlignment = Enum.TextXAlignment.Left
    adDesc.Parent = bg
    
    -- เก็บ reference
    billboard._gui = surfaceGui
    billboard._bg = bg
    
    return billboard
end

-- หมุนเวียนโฆษณา
function BillboardSystem:StartRotation(billboard, ads)
    ads = ads or ADS
    local currentAdIndex = 1
    
    local function showNextAd()
        local ad = ads[currentAdIndex]
        
        local image = billboard._bg:FindFirstChild("AdImage")
        local title = billboard._bg:FindFirstChild("AdTitle")
        local desc = billboard._bg:FindFirstChild("AdDesc")
        
        if image then image.Image = ad.image end
        if title then title.Text = ad.title or "" end
        if desc then desc.Text = ad.description or "" end
        
        -- Fade effect
        if image then
            local tween = TweenService:Create(
                image,
                TweenInfo.new(0.5),
                {ImageTransparency = 0}
            )
            tween:Play()
        end
        
        -- ไปโฆษณาถัดไป
        currentAdIndex = (currentAdIndex % #ads) + 1
        
        task.delay(ad.duration, showNextAd)
    end
    
    showNextAd()
end

-- เริ่มระบบ
function BillboardSystem:Initialize()
    for _, config in ipairs(BILLBOARD_LOCATIONS) do
        local billboard = self:CreateBillboard(config)
        self:StartRotation(billboard, ADS)
    end
    print("[Billboard] เริ่มระบบโฆษณา " .. #BILLBOARD_LOCATIONS .. " จุด")
end

return BillboardSystem
```

---

## ส่วนที่ 3: Reward Ads (Video Ads)

```lua
-- LocalScript: RewardAds
-- ระบบดูโฆษณาแลกรางวัล

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local player = Players.LocalPlayer

-- ปุ่มดูโฆษณา
local function createAdRewardButton()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "AdRewardUI"
    screenGui.Parent = player.PlayerGui
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 200, 0, 60)
    button.Position = UDim2.new(0, 10, 0.7, 0)
    button.BackgroundColor3 = Color3.fromRGB(255, 150, 0)
    button.Text = "🎬 ดูโฆษณา\nรับ 50 Gems ฟรี!"
    button.TextColor3 = Color3.new(1, 1, 1)
    button.TextSize = 14
    button.Font = Enum.Font.GothamBold
    button.TextWrapped = true
    button.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = button
    
    -- Cooldown indicator
    local cooldownLabel = Instance.new("TextLabel")
    cooldownLabel.Name = "Cooldown"
    cooldownLabel.Size = UDim2.new(1, 0, 0, 20)
    cooldownLabel.Position = UDim2.new(0, 0, 1, 2)
    cooldownLabel.BackgroundTransparency = 1
    cooldownLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
    cooldownLabel.TextSize = 12
    cooldownLabel.Font = Enum.Font.Gotham
    cooldownLabel.Text = "ดูได้ทุก 30 นาที"
    cooldownLabel.Parent = button
    
    -- Cooldown timer
    local lastWatched = 0
    local COOLDOWN = 1800 -- 30 นาที
    
    local function updateCooldown()
        local elapsed = os.time() - lastWatched
        local remaining = COOLDOWN - elapsed
        
        if remaining > 0 then
            button.BackgroundColor3 = Color3.fromRGB(100, 100, 100)
            button.Text = "🎬 ดูโฆษณา\n(รอ " .. math.ceil(remaining / 60) .. " นาที)"
            button.Active = false
        else
            button.BackgroundColor3 = Color3.fromRGB(255, 150, 0)
            button.Text = "🎬 ดูโฆษณา\nรับ 50 Gems ฟรี!"
            button.Active = true
        end
    end
    
    -- อัพเดท cooldown ทุกวินาที
    task.spawn(function()
        while screenGui.Parent do
            task.wait(1)
            updateCooldown()
        end
    end)
    
    -- ดูโฆษณา
    button.MouseButton1Click:Connect(function()
        if os.time() - lastWatched < COOLDOWN then return end
        
        -- Simulate watching an ad
        -- ในความเป็นจริง ต้องใช้ Roblox Ad service API
        
        button.Active = false
        button.Text = "กำลังโหลด..."
        
        -- Simulate ad duration
        task.delay(5, function()
            -- ส่ง request ไปยัง server เพื่อให้รางวัล
            local remotes = ReplicatedStorage:FindFirstChild("Remotes")
            if remotes then
                local adRewardEvent = remotes:FindFirstChild("AdReward")
                if adRewardEvent then
                    adRewardEvent:FireServer("watched_ad")
                end
            end
            
            lastWatched = os.time()
            updateCooldown()
        end)
    end)
    
    return button
end

createAdRewardButton()
```

---

## ส่วนที่ 4: Sponsor Integration System

```lua
-- ModuleScript: SponsorSystem
-- ระบบจัดการสปอนเซอร์

local TweenService = game:GetService("TweenService")

local SponsorSystem = {}

-- ข้อมูลสปอนเซอร์ (จะได้รับจากสปอนเซอร์จริง)
local SPONSORS = {
    -- ตัวอย่าง (ไม่ใช่สปอนเซอร์จริง)
    {
        id = "SPONSOR_001",
        name = "ExampleBrand",
        logoImage = "rbxassetid://sponsor_logo_1",
        bannerImage = "rbxassetid://sponsor_banner_1",
        color = Color3.fromRGB(0, 100, 255),
        
        -- งาน event พิเศษ
        event = {
            name = "ExampleBrand Cup",
            description = "แข่งขัน sponsored by ExampleBrand",
            reward = {coins = 5000, badge = "SponsorBadge"},
        },
        
        -- ตำแหน่งโฆษณา
        placements = {"Billboard_1", "SpawnArea"},
        
        -- ระยะเวลา
        startDate = os.time(),
        endDate = os.time() + (7 * 24 * 3600), -- 1 สัปดาห์
    },
}

-- ตรวจสอบว่า sponsor event active
function SponsorSystem:IsActive(sponsorId)
    for _, sponsor in ipairs(SPONSORS) do
        if sponsor.id == sponsorId then
            local now = os.time()
            return now >= sponsor.startDate and now <= sponsor.endDate
        end
    end
    return false
end

-- ใช้ branding ของ sponsor
function SponsorSystem:ApplyBranding(sponsorId)
    local sponsor = nil
    for _, s in ipairs(SPONSORS) do
        if s.id == sponsorId then
            sponsor = s
            break
        end
    end
    
    if not sponsor or not self:IsActive(sponsorId) then return end
    
    -- อัพเดท billboards
    for _, placement in ipairs(sponsor.placements) do
        local billboard = workspace:FindFirstChild(placement)
        if billboard then
            local surfaceGui = billboard:FindFirstChildOfClass("SurfaceGui")
            if surfaceGui then
                local bg = surfaceGui:FindFirstChild("Frame")
                if bg then
                    local image = bg:FindFirstChild("AdImage")
                    if image then
                        image.Image = sponsor.bannerImage
                    end
                end
            end
        end
    end
    
    print(string.format("[Sponsor] ใช้ branding ของ %s", sponsor.name))
end

-- สร้าง Sponsored Event
function SponsorSystem:CreateSponsoredEvent(sponsorId)
    local sponsor = nil
    for _, s in ipairs(SPONSORS) do
        if s.id == sponsorId then
            sponsor = s
            break
        end
    end
    
    if not sponsor or not sponsor.event then return end
    
    -- สร้าง event ในเกม
    local event = sponsor.event
    print(string.format("[Sponsor] เริ่ม event: %s", event.name))
    
    -- ประกาศให้ผู้เล่นทราบ
    local Players = game:GetService("Players")
    for _, player in ipairs(Players:GetPlayers()) do
        local remotes = game.ReplicatedStorage:FindFirstChild("Remotes")
        if remotes then
            local announceEvent = remotes:FindFirstChild("Announce")
            if announceEvent then
                announceEvent:FireClient(player, {
                    title = "🎉 " .. event.name,
                    message = event.description,
                    duration = 10,
                })
            end
        end
    end
end

return SponsorSystem
```

---

## ส่วนที่ 5: Cross-Promotion System

```lua
-- ModuleScript: CrossPromotion
-- ระบบโปรโมตเกมอื่นในเครือ

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")

local CrossPromotion = {}

-- เกมในเครือที่จะโปรโมต
local PARTNER_GAMES = {
    {
        id = 123456789, -- PlaceId ของเกมคู่
        name = "Partner Game: Adventure World",
        description = "RPG สุดมัน! มากกว่า 50 dungeon",
        icon = "rbxassetid://partner_game_icon",
        reward = {
            -- รางวัลเมื่อผู้เล่นไปเล่นเกมคู่แล้วกลับมา
            coins = 500,
            title = "Explorer",
        },
    },
}

-- แสดง popup โปรโมตเกม
function CrossPromotion:ShowPromo(player, gameIndex)
    local game = PARTNER_GAMES[gameIndex or 1]
    if not game then return end
    
    local remotes = game.ReplicatedStorage:FindFirstChild("Remotes")
    if remotes then
        local promoEvent = remotes:FindFirstChild("ShowCrossPromo")
        if promoEvent then
            promoEvent:FireClient(player, game)
        end
    end
end

-- Teleport ผู้เล่นไปเกมคู่
function CrossPromotion:TeleportToPartner(player, gameId)
    -- ให้รางวัลก่อน teleport
    local ProfileManager = require(game.ServerStorage.ProfileManager)
    local data = ProfileManager:GetData(player)
    if data then
        data.PendingCrossPromoReward = gameId
    end
    
    -- Teleport
    local success = pcall(function()
        TeleportService:Teleport(gameId, player)
    end)
    
    if not success then
        warn("[CrossPromo] Teleport ล้มเหลว")
    end
end

return CrossPromotion
```

---

## ส่วนที่ 6: Metrics สำหรับ Advertisers

```lua
-- Script: AdMetrics
-- ติดตาม metrics สำหรับโฆษณา

local DataStoreService = game:GetService("DataStoreService")
local adMetricsStore = DataStoreService:GetDataStore("AdMetrics")

local AdMetrics = {}

-- บันทึก ad view
function AdMetrics:TrackView(adId, playerId)
    local key = string.format("Ad_%s_%s", adId, os.date("%Y-%m-%d"))
    
    pcall(function()
        adMetricsStore:UpdateAsync(key, function(data)
            data = data or {
                adId = adId,
                date = os.date("%Y-%m-%d"),
                views = 0,
                uniqueViewers = {},
                clicks = 0,
            }
            
            data.views = data.views + 1
            
            if not table.find(data.uniqueViewers, playerId) then
                table.insert(data.uniqueViewers, playerId)
            end
            
            return data
        end)
    end)
end

-- บันทึก ad click
function AdMetrics:TrackClick(adId, playerId)
    local key = string.format("Ad_%s_%s_clicks", adId, os.date("%Y-%m-%d"))
    
    pcall(function()
        adMetricsStore:IncrementAsync(key, 1)
    end)
end

-- คำนวณ CTR (Click Through Rate)
function AdMetrics:GetCTR(adId, date)
    local viewKey = string.format("Ad_%s_%s", adId, date)
    local clickKey = string.format("Ad_%s_%s_clicks", adId, date)
    
    local views = 0
    local clicks = 0
    
    local success1, viewData = pcall(function()
        return adMetricsStore:GetAsync(viewKey)
    end)
    
    local success2, clickCount = pcall(function()
        return adMetricsStore:GetAsync(clickKey)
    end)
    
    if success1 and viewData then views = viewData.views end
    if success2 then clicks = clickCount or 0 end
    
    if views == 0 then return 0 end
    return (clicks / views) * 100
end

return AdMetrics
```

---

## ส่วนที่ 7: Best Practices สำหรับการโฆษณา

### การโฆษณาเกมของตัวเอง

**1. Thumbnails ที่ดึงดูด**
- ขนาด 1920x1080 pixels
- แสดง gameplay จริง
- มีข้อความที่อ่านง่าย
- ใช้สีสดใส

**2. Game Description**
- อธิบาย gameplay ชัดเจน
- ใส่ keywords ที่ค้นหาง่าย
- อัพเดทสม่ำเสมอ

**3. Video Trailers**
- 30-60 วินาที
- แสดงส่วนที่น่าตื่นเต้น
- มี hook ใน 3 วินาทีแรก

### การดึงดูดสปอนเซอร์

**1. สิ่งที่ต้องมีก่อนหาสปอนเซอร์**
```
✅ ผู้เล่น concurrent 1000+ คน
✅ Active community
✅ Regular updates
✅ Clean content (ไม่มี inappropriate content)
✅ Professional portfolio
```

**2. วิธีหาสปอนเซอร์**
```
- ติดต่อ Roblox Advertising ผ่าน creator portal
- Social media (Twitter/YouTube) ที่แสดง stats
- Gaming influencer networks
- Brand partnerships ผ่าน community managers
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Billboard Rotation
สร้างระบบ billboard ที่:
1. หมุนโฆษณา 5 อัน
2. มี fade transition
3. Track viewer metrics

### แบบฝึกหัดที่ 2: Reward Ad UI
สร้าง UI ดูโฆษณาที่:
1. แสดง countdown
2. Reward หลายอย่าง (coins, gems, boost)
3. จำกัด 3 ครั้ง/วัน

### แบบฝึกหัดที่ 3: Game Thumbnail Generator
เรียนรู้การสร้าง thumbnail ที่ดี:
1. ออกแบบ thumbnail ใน Canva/Photoshop
2. A/B test thumbnails ต่างกัน
3. วัดผลด้วย click rate

---

## สรุปบทที่ 87

การโฆษณาที่ดีในเกม Roblox ต้องสมดุล:

1. **Non-intrusive** - ไม่รบกวน gameplay
2. **Value Exchange** - ดูโฆษณาแลกรางวัล
3. **Brand Safety** - เลือกสปอนเซอร์ที่เหมาะสม
4. **Metrics-driven** - วัดผลและปรับปรุง

*บทถัดไป: Part 88 - Testing and Debugging*
