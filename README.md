# UX Skills Pack — UX Design Agent

通用 UI/UX 設計 Agent，理解需求、生成設計規格與可互動的 HTML 原型。
像有一個 UX 設計師幫你了解使用者、規劃設計、產出可 Demo 的原型。任何專案都能用。

## 安裝

### 方法 A：Plugin 安裝（推薦）

```bash
/plugin marketplace add <your-github>/ux-skills-pack
/plugin install ux-skills-pack
```

### 方法 B：手動複製

```bash
cp -r skills/ux-empathize skills/ux-design ~/.claude/skills/
```

安裝後在 Claude App 的 **Customize** 裡會看到 2 個新 skill。

## 包含什麼

| Skill | 做什麼 | 觸發方式 |
|-------|--------|---------|
| **ux-empathize** | 使用者研究 + 需求分析 | `/ux empathize <描述>` |
| **ux-design** | 設計規格 + HTML 互動原型 + 設定管理 | `/ux design <描述>`, `/ux config`, `/ux full <描述>` |

## 能做什麼

### 了解需求（/ux empathize）
像 UX 設計師一樣幫你：
- 辨識目標使用者，建立 Persona
- 發掘痛點（功能性 / 認知性 / 情感性）
- 撰寫問題陳述（POV / HMW）
- 定義成功指標 + 盤點限制條件
- 產出 → `ux-brief.md`

### 設計 + 出原型（/ux design）
- 載入品牌設定（首次使用互動式建立）
- 研究設計模式 + 產出替代方案
- 寫設計規格（佈局、元件、互動狀態）
- 產出**可互動的 HTML 原型**（瀏覽器直接打開）
- 快速迭代：說「標題改大」「按鈕改藍色」直接改
- 產出 → `design-spec.md` + `prototype.html`

### 一鍵完成（/ux full）
empathize → design 自動串接，一句話從研究到出圖。

### 設定管理（/ux config）
- 首次使用互動式建立品牌設定
- 自然語言修改：`主色改紅色` / `更專業一點` / `要深色模式`
- 自動偵測專案類型（.xcodeproj → iOS, package.json → Web）

### 跨 Session 接續
- 每個 feature 有版本歷史（`changelog.md`）
- 專案設計摘要（`design-history.yaml`）
- 新 Session 自動讀取上次進度，問你要接續還是重新開始

## 快速開始

### 在程式專案中使用
```bash
cd ~/Code/my-app    # 有 .git/ 或 .xcodeproj 或 package.json
claude
> /ux design 會員列表頁面
```

### 純設計資料夾（只放 demo）
```bash
mkdir ~/Design/my-app
cd ~/Design/my-app
claude
> /ux design 首頁
# 首次自動問你名稱、平台、風格 → 建立設定 → 開始設計
```

產出結構：
```
my-app/
└── .claude/
    ├── ux-config.yaml              ← 品牌設定
    └── ux-prototypes/
        ├── design-history.yaml     ← 專案全貌
        └── homepage/
            ├── ux-brief.md         ← 研究結果（如果跑了 empathize）
            ├── design-spec.md      ← 設計規格
            ├── prototype.html      ← 互動原型（瀏覽器打開）
            └── changelog.md        ← 版本紀錄
```

迭代：「間距拉大一點」「配色換暖色」→ 快速修改，不重跑流程

## 設定

首次使用時互動式建立 `ux-config.yaml`（自動偵測專案類型 + 詢問偏好）。之後用自然語言調整：

- `主色改紅色` → 直接改 config token
- `更專業一點` → 自動調整一整組色彩 + 圓角（before/after 確認表）
- `要深色模式` → 自動翻轉色彩
- `確定按鈕要實心，取消要空心` → 寫入元件規則

進階設定可參考 `ux-config-example.yaml`。

## 支援平台

由 config `platform.type` 決定，支援：

| 平台 | Viewport | 說明 |
|------|----------|------|
| **Web** | Responsive (min 375px) | 預設平台 |
| **iPad UIKit** | Landscape breakpoint (1100pt) | 含 Apple HIG 規範 |
| **iOS** | 393×852 | iPhone 原型 |
| **Android** | 依 config viewport 設定 | Material Design 參考 |

## 內建設計知識

### references（8 個設計方法論參考文件）

**ux-empathize**（3 個）：
- `persona-guide.md` — 人物誌合成方法論（行為變數、聚類、persona 模板）
- `research-methods.md` — 使用者研究方法（訪談、觀察、任務分析、競品分析）
- `ia-basics.md` — 資訊架構基礎（內容結構、導航、任務頻率）

**ux-design**（5 個）：
- `visual-design-principles.md` — C.A.R.P. 原則、60-30-10 配色、間距系統
- `style-library.md` — 10 種視覺風格模板（Minimalism → Brutalism → 深色模式）
- `wireframing-guide.md` — 線框圖方法論（保真度選擇、方案產出）
- `interaction-patterns.md` — 互動設計模式（狀態機、回饋、漸進式揭露）
- `apple-hig-ipad.md` — Apple HIG iPadOS 規範（佈局、元件、觸控、排版、色彩）
