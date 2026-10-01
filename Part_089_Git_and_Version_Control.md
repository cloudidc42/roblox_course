# Part 89: Git and Version Control - การจัดการ Version สำหรับเกม Roblox

## บทนำ

Version Control เป็นสิ่งจำเป็นสำหรับนักพัฒนาทุกคน โดยเฉพาะเมื่อทำงานเป็นทีม ใน Roblox เราจะใช้ Git ร่วมกับ Rojo หรือ Argon เพื่อจัดการ source code ได้อย่างมีประสิทธิภาพ

---

## ส่วนที่ 1: ทำไมต้องใช้ Version Control

**ปัญหาที่เกิดขึ้นโดยไม่มี Version Control:**
- "โค้ดเวอร์ชันเก่าทำงานได้แต่ลบทิ้งไปแล้ว"
- "ไม่รู้ว่าใครแก้ไขอะไรเมื่อไหร่"
- "แก้ bug อันนึงแต่ทำให้อีกอันพัง"
- "ทีมทำงานซ้อนทับกัน"

**ประโยชน์ของ Git:**
- ย้อนกลับไปเวอร์ชันเก่าได้
- ทำงานหลาย feature พร้อมกัน (branches)
- ตรวจสอบว่าใครแก้ไขอะไร (history)
- ทดสอบ feature ใหม่โดยไม่กระทบ production

---

## ส่วนที่ 2: การตั้งค่า Git กับ Roblox

### 2.1 ติดตั้ง Rojo

Rojo เป็นเครื่องมือที่แปลง Roblox Studio project เป็นไฟล์ที่ Git จัดการได้

```bash
# ติดตั้ง Rojo ผ่าน Aftman (package manager)
aftman install rojo-rbx/rojo

# หรือดาวน์โหลดจาก GitHub Releases
# https://github.com/rojo-rbx/rojo/releases
```

### 2.2 โครงสร้างโปรเจกต์

```
my-roblox-game/
├── .git/
├── .gitignore
├── default.project.json    # Rojo config
├── README.md
├── src/
│   ├── client/             # LocalScripts
│   │   ├── Controllers/
│   │   │   ├── ShopController.lua
│   │   │   └── UIController.lua
│   │   └── init.client.lua
│   ├── server/             # Scripts
│   │   ├── Services/
│   │   │   ├── DataService.lua
│   │   │   └── GameService.lua
│   │   └── init.server.lua
│   └── shared/             # ModuleScripts
│       ├── Config/
│       │   ├── GameConfig.lua
│       │   └── ItemConfig.lua
│       └── Modules/
│           ├── MathUtil.lua
│           └── TableUtil.lua
├── assets/                 # รูปภาพ, เสียง (ไม่เก็บใน Git)
└── tests/
    ├── unit/
    └── integration/
```

### 2.3 default.project.json

```json
{
    "name": "MyRobloxGame",
    "tree": {
        "$className": "DataModel",
        
        "ServerScriptService": {
            "$className": "ServerScriptService",
            "GameServer": {
                "$path": "src/server"
            }
        },
        
        "StarterPlayer": {
            "$className": "StarterPlayer",
            "StarterPlayerScripts": {
                "$className": "StarterPlayerScripts",
                "GameClient": {
                    "$path": "src/client"
                }
            }
        },
        
        "ReplicatedStorage": {
            "$className": "ReplicatedStorage",
            "Shared": {
                "$path": "src/shared"
            }
        },
        
        "StarterGui": {
            "$path": "src/gui"
        }
    }
}
```

### 2.4 .gitignore สำหรับ Roblox

```gitignore
# Roblox files
*.rbxl
*.rbxlx
*.rbxm
*.rbxmx

# ยกเว้นถ้าต้องการเก็บ
# !game.rbxl

# Build outputs
build/

# Node modules (ถ้าใช้ npm tools)
node_modules/
npm-debug.log*

# OS files
.DS_Store
Thumbs.db
*.swp
*.swo

# IDE
.vscode/settings.json
.idea/

# Secrets
.env
secrets.lua
config.local.lua

# Rojo state
*.project.json.lock

# Aftman
.aftman/

# Logs
logs/
*.log
```

---

## ส่วนที่ 3: Git Workflow สำหรับ Roblox

### 3.1 GitFlow Workflow

```bash
# โครงสร้าง branches
main          # Production - stable เสมอ
develop       # Development - รวม features ใหม่
feature/*     # Feature branches
hotfix/*      # แก้ bug เร่งด่วนใน production
release/*     # เตรียม release

# ตัวอย่างการสร้าง feature
git checkout develop
git pull origin develop
git checkout -b feature/shop-system

# ทำงาน...
git add src/client/Controllers/ShopController.lua
git commit -m "feat: add shop UI controller"

# Push feature
git push origin feature/shop-system

# Merge กลับ develop
git checkout develop
git merge feature/shop-system --no-ff
git branch -d feature/shop-system
```

### 3.2 Commit Message Convention

```bash
# รูปแบบ: <type>(<scope>): <description>
# 
# Types:
# feat     - feature ใหม่
# fix      - แก้บัค
# docs     - documentation
# style    - format, ไม่เปลี่ยน logic
# refactor - refactor code
# test     - เพิ่ม tests
# chore    - tasks อื่นๆ (config, tools)

# ตัวอย่างที่ดี:
git commit -m "feat(shop): add VIP discount system"
git commit -m "fix(datastore): prevent duplicate purchase receipt"
git commit -m "refactor(inventory): extract item validation logic"
git commit -m "test(economy): add unit tests for coin calculation"
git commit -m "docs: update README with setup instructions"

# ตัวอย่างที่ไม่ดี:
git commit -m "แก้บัค"  # ไม่ชัดเจน
git commit -m "update"  # ไม่มีความหมาย
git commit -m "asdfghjk"  # ไม่มีความหมายเลย
```

---

## ส่วนที่ 4: การใช้ Git สำหรับทีม

### 4.1 Branch Protection Rules (GitHub)

```yaml
# .github/branch-protection.yml
# (ตั้งค่าใน GitHub Settings > Branches)

Protection Rules for 'main':
  - Require pull request before merging
  - Require at least 1 approval
  - Dismiss stale PR approvals when new commits pushed
  - Require status checks to pass
  - Include administrators
  - Restrict who can push (only admin)
```

### 4.2 Pull Request Template

```markdown
<!-- .github/PULL_REQUEST_TEMPLATE.md -->

## คำอธิบาย
อธิบายสิ่งที่เปลี่ยนแปลงในครั้งนี้

## ประเภทของการเปลี่ยนแปลง
- [ ] Bug fix
- [ ] Feature ใหม่
- [ ] Breaking change
- [ ] Documentation update

## วิธีทดสอบ
1. เปิด Studio
2. รัน test suite
3. ทดสอบ feature ด้วยตนเอง

## Checklist
- [ ] โค้ดผ่าน Luau linter
- [ ] เขียน/อัพเดท tests แล้ว
- [ ] ทดสอบบน PC และ Mobile
- [ ] ไม่มี console errors
- [ ] Performance ไม่ลดลง
```

### 4.3 GitHub Actions CI/CD

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    name: Luau Lint
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Aftman
        run: |
          curl -sSf https://raw.githubusercontent.com/LPGhatguy/aftman/main/scripts/install.sh | sh
          echo "$HOME/.aftman/bin" >> $GITHUB_PATH
      
      - name: Install Tools
        run: aftman install
      
      - name: Run Selene Lint
        run: selene src/
      
      - name: Type Check with Luau LSP
        run: |
          luau-lsp analyze --definitions=globalTypes.d.lua src/

  test:
    name: Run Tests
    runs-on: ubuntu-latest
    needs: lint
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Tools
        run: aftman install
      
      - name: Run Tests
        run: run-in-roblox --place game.rbxl --script tests/run_tests.server.lua

  build:
    name: Build Game
    runs-on: ubuntu-latest
    needs: [lint, test]
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Place File
        run: rojo build default.project.json --output game.rbxl
      
      - name: Upload Artifact
        uses: actions/upload-artifact@v3
        with:
          name: game-build
          path: game.rbxl
```

---

## ส่วนที่ 5: Selene - Lua Linter

```yaml
# selene.toml
# กำหนดค่า Selene linter

std = "roblox"
  
[rules]
empty_if = "warn"
global_usage = "warn"
if_same_then_else = "warn"
ifs_same_cond = "deny"
multiple_returns = "allow"
must_use = "deny"
shadowing = "warn"
unbalanced_assignments = "deny"
undefined_variable = "deny"
unused_variable = "warn"
```

```yaml
# .selene.yml
# กำหนด globals ที่ Roblox ใช้

globals:
  - game
  - workspace
  - script
  - require
  - wait
  - task
  - print
  - warn
  - error
```

---

## ส่วนที่ 6: Wally Package Manager

Wally เป็น package manager สำหรับ Roblox Lua

```toml
# wally.toml

[package]
name = "myusername/mygame"
version = "1.0.0"
registry = "https://github.com/UpliftGames/wally-index"
realm = "server"

[dependencies]
ProfileService = "madstudioroblox/profileservice@^1.0.0"
Signal = "sleitnick/signal@^1.5.0"
TableUtil = "sleitnick/tableutil@^2.0.0"
Promise = "evaera/promise@^4.0.0"
Cmdr = "evaera/cmdr@^1.8.4"

[dev-dependencies]
TestEZ = "roblox/testez@^0.4.1"
```

```bash
# ติดตั้ง packages
wally install

# Packages จะอยู่ใน Packages/ folder
```

---

## ส่วนที่ 7: Release Management

### 7.1 Semantic Versioning

```
v1.0.0 - Major.Minor.Patch
  ^- Major: breaking changes
       ^- Minor: new features (backward compatible)
            ^- Patch: bug fixes

ตัวอย่าง:
v1.0.0  - Initial release
v1.1.0  - เพิ่ม shop system
v1.1.1  - แก้บัค shop
v1.2.0  - เพิ่ม VIP system
v2.0.0  - เปลี่ยน data structure ใหม่ (breaking)
```

### 7.2 Changelog

```markdown
# Changelog

## [2.1.0] - 2024-01-15

### เพิ่ม
- ระบบ Daily Quests
- VIP Diamond tier ใหม่
- Animation สำหรับ level up

### แก้ไข
- แก้บัคที่ทำให้ inventory หาย
- แก้ performance issues ใน shop UI

### ปรับปรุง
- ลด load time ลง 30%
- ปรับสมดุล economy

## [2.0.0] - 2024-01-01

### Breaking Changes
- เปลี่ยน data structure ของ inventory
  - Migration จะทำงานอัตโนมัติ

### เพิ่ม
- ระบบ Crafting
- ระบบ Guild
```

### 7.3 Auto-release Script

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Game
        run: rojo build default.project.json --output game.rbxl
      
      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          files: game.rbxl
          generate_release_notes: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## ส่วนที่ 8: Git Commands ที่ใช้บ่อย

```bash
# =========================================
# Commands ที่ใช้ทุกวัน
# =========================================

# ดูสถานะ
git status

# ดู changes
git diff
git diff --staged

# บันทึก changes
git add <file>
git add .
git commit -m "message"

# Push/Pull
git push origin <branch>
git pull origin <branch>

# =========================================
# Branch Management
# =========================================

# สร้าง branch ใหม่
git checkout -b feature/new-feature

# สลับ branch
git checkout develop

# Merge branch
git merge feature/new-feature --no-ff

# ลบ branch
git branch -d feature/new-feature
git push origin --delete feature/new-feature

# =========================================
# Undo Operations
# =========================================

# ยกเลิก changes ใน working directory
git restore <file>

# ยกเลิก staged changes
git restore --staged <file>

# ยกเลิก commit ล่าสุด (keep changes)
git reset HEAD~1

# ยกเลิก commit ล่าสุด (discard changes)
git reset --hard HEAD~1

# สร้าง revert commit
git revert HEAD

# =========================================
# Stash
# =========================================

# เก็บ changes ชั่วคราว
git stash push -m "WIP: shop feature"

# ดู stash list
git stash list

# เอา stash กลับมา
git stash pop

# =========================================
# Inspection
# =========================================

# ดู history
git log --oneline --graph --all

# ดูว่าใครแก้ไขไฟล์
git blame src/server/DataService.lua

# ค้นหาใน history
git log --all -S "findMe" -- "*.lua"

# ดู diff ระหว่าง commits
git diff HEAD~3..HEAD -- src/

# =========================================
# Advanced
# =========================================

# Cherry-pick commit
git cherry-pick <commit-hash>

# Rebase
git rebase develop

# Interactive rebase (รวม commits)
git rebase -i HEAD~3

# Tag
git tag v1.0.0
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin --tags
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Setup Rojo Project
1. ติดตั้ง Rojo
2. สร้าง project structure ใหม่
3. Sync กับ Studio
4. Push ไป GitHub

### แบบฝึกหัดที่ 2: Feature Branch Workflow
1. สร้าง feature branch
2. เพิ่ม feature ใหม่
3. สร้าง Pull Request
4. Code Review และ Merge

### แบบฝึกหัดที่ 3: CI/CD Pipeline
1. ตั้งค่า GitHub Actions
2. Lint โค้ดอัตโนมัติ
3. รัน tests อัตโนมัติ
4. Deploy เมื่อ merge กับ main

---

## สรุปบทที่ 89

Version Control ด้วย Git ช่วยให้:

1. **Safety Net** - ย้อนกลับเมื่อมีปัญหา
2. **Collaboration** - ทำงานเป็นทีมได้ดี
3. **History** - รู้ว่าโค้ดเปลี่ยนอย่างไร
4. **CI/CD** - Automate การทดสอบและ deploy
5. **Code Quality** - Enforce standards ผ่าน lint

*บทถัดไป: Part 90 - Team Development*
