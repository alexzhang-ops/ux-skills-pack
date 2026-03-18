---
name: ux-design
description: "UX 設計階段 — 設計規格 + 互動式 HTML 原型 + 專案設定管理。載入專案設定、搜尋設計資料庫、產出設計方案、收斂為 Design Spec，使用者確認後製作互動式 HTML 原型。支援自然語言修改品牌設定。用 /ux full 依序串接 empathize→design。輸出：design-spec.md + prototype.html。Triggers: /ux design, /ux full, /ux config, design spec, HTML prototype, wireframe, prototype, mockup, UI design, create prototype, design this feature, interactive prototype, design alternatives, change brand color, change style, change font."
---

# UX Design — Ideate + Spec + HTML Prototype + Config Management

## Scope

| Do | Don't |
|----|-------|
| Load Config and apply brand colors/fonts/style | No user research (use ux-empathize) |
| Search design references for best practices | No usability validation |
| Generate design alternatives (number fits complexity) | No production code generation |
| Converge into Design Spec | |
| Produce interactive HTML prototype | |
| Manage project config (ux-config.yaml) | |
| Chain empathize → design via `/ux full` | |

## Shared Context

**Config**: `{project-root}/.claude/ux-config.yaml` — auto-load（詳見下方 Config 管理章節）。

**Output path**: `{project-root}/.claude/ux-prototypes/{feature-name}/`

**Design History**: `{project-root}/.claude/ux-prototypes/design-history.yaml` — 專案級設計摘要，自動累積。每次設計完成後更新，讓下次設計時能快速掌握專案現況。

Uses Claude's built-in design knowledge. No external scripts required.

**Language**: follow Config `language` field (default zh-TW).

---

## Config 管理

### Config 位置

`{project-root}/.claude/ux-config.yaml`

### 載入邏輯

1. 偵測 project root（依序檢查，找到即停）：
   a. 當前目錄有 `.claude/ux-config.yaml` → 當前目錄就是 root（純設計資料夾）
   b. 當前目錄有 `.git/` 或 `.xcodeproj` 或 `package.json` → 該目錄是 root
   c. 往上層目錄找，直到找到以上任一標記
   d. 都找不到 → 以當前工作目錄為 root
2. 尋找 `{project-root}/.claude/ux-config.yaml`
3. **找到** → auto-load config，讀 `design-history.yaml` 掌握全貌 → 接續現有專案
4. **找不到** → 首次 setup：
   a. 嘗試偵測專案類型（`.xcodeproj` → iPad UIKit, `Package.swift` → iOS, `package.json` → Web, `build.gradle` → Android；都沒有 → 純設計專案）
   b. 如果是程式專案，嘗試從原始碼提取現有 design tokens（CSS variables, color constants, theme files）
   c. 問使用者（**一次問齊**）：專案名稱、平台、風格偏好
   d. 用答案 + 通用預設 生成 config，存到 `{project-root}/.claude/ux-config.yaml`
   e. 如果使用者跳過 → 直接用通用預設（藍色系，見 `ux-config-example.yaml`）

### `/ux config` — 查看/編輯設定

**查看**：顯示目前 config 摘要：
```
Brand: {product.name} ({brand.style})
Primary: {primary} | Secondary: {secondary} | Accent: {accent}
Text: {text_primary} / {text_secondary} / {text_tertiary}
Background: {background} | Surface: {surface}
Font: {heading_font} / {body_font} @ {base_size}pt
Corner: {corner_radius}px | Platform: {platform.type}
```

### Config 自然語言編輯

**判斷規模**：
- **明確請求**（例如「主色改成 #FF6600」「圓角改 12」）→ 直接改，顯示 before/after 確認
- **模糊請求**（例如「我要修改配色」）→ 先顯示目前值，問要改哪個

**修改前必須確認**：顯示即將變更的 token 和 before/after 值，等使用者同意才寫入 config。

**元件層級**（對應到最近的 token）：
- 「按鈕改白色」→ update `brand.colors.surface` or button-specific token
- 「標題改大」→ update `brand.typography.base_size`
- 「背景太亮了」→ update `brand.colors.background`
- 「文字看不清楚」→ check contrast, update text/background colors

**Token 層級**（直接對應）：
- 「主色改紅色」→ update `brand.colors.primary`
- 「換 glassmorphism 風格」→ update `brand.style`
- 「圓角改成 12」→ update `brand.corner_radius`
- 「要深色模式」→ update to dark palette + `dark-mode` style

**風格層級**（auto-cascade — 一句話觸發一整組調整）：
- 「整體感覺太冷」→ 自動調整 primary/secondary/accent 為暖色調 + surface 微調
- 「更專業一點」→ 自動降低彩度、收緊 corner_radius、調整 text 對比
- 「太花了」→ 自動減少 accent 使用、style 改 `minimalism`、簡化色彩層次
- 「更活潑」→ 自動提高 accent 彩度、加大 corner_radius、建議更鮮豔的 secondary
- 「更簡潔」→ 自動減少色彩數量、加大留白、style 改 `minimalism`

**Auto-cascade 規則**：
當使用者描述的是風格方向（非具體 token），自動產出**完整的 before/after 對照表**，一次呈現所有會連動的 token 變化：

```
風格調整：「更專業一點」
┌─────────────────┬──────────┬──────────┐
│ Token           │ Before   │ After    │
├─────────────────┼──────────┼──────────┤
│ style           │ minimal  │ minimal  │
│ primary         │ #2563EB  │ #1E4FCB  │ ← 降低彩度
│ secondary       │ #7C3AED  │ #5B2DB0  │ ← 降低彩度
│ corner_radius   │ 8        │ 6        │ ← 收緊
│ text_primary    │ #111827  │ #0A0F1A  │ ← 加深對比
└─────────────────┴──────────┴──────────┘
確認這些調整嗎？
```

使用者可以逐項接受/修改，確認後一次寫入 config。

**元件行為規範**（寫入 config 的 `button_styles` / `component_rules`）：
- 「確定按鈕要實心，取消要空心」→ 更新 `button_styles`
- 「卡片要有陰影」→ 更新 `component_rules.card.shadow`
- 「輸入框聚焦要藍色邊框」→ 更新 `component_rules.input.focus_border`

**Interpretation rules**:
1. Component mention → find closest config token
2. Component behavior rule → write to `button_styles` or `component_rules` section
3. Ambiguous → show current values, ask which to change
4. Style direction → auto-cascade 產出完整對照表
5. Can't map to token or rule (e.g., "加個 icon") → note as design feedback for next `/ux design`
6. Always show before/after values

After config change: remind user existing prototypes won't auto-update — re-run `/ux design` to apply.

---

## `/ux full` 串接邏輯

收到 `/ux full <描述>` 時：
1. 先觸發 ux-empathize skill 完成研究 → 產出 `ux-brief.md`
2. 自動讀取 `ux-brief.md` 作為 design 的輸入
3. 進入完整路徑 Step 1-7

---

## Input

- **Required**: Feature description OR UX Brief (`ux-brief.md`)
- **Auto-loaded**: Project Config

## Step 0: Confirm Before Proceed

**判斷規模 → 選擇路徑**：

| 規模 | 判斷依據 | 走哪條路 |
|------|---------|---------|
| **微調** | 改顏色、改間距、改排版、改文字、調整現有元件 — 不動結構 | → **快速路徑** |
| **有既有 prototype** | 使用者指定修改現有的 prototype.html | → **快速路徑** |
| **新功能 / 重設計** | 新頁面、新流程、結構性變更 | → **完整路徑** (Step 1-7) |
| **有 UX Brief** | `ux-brief.md` 作為輸入 | → **完整路徑** (Step 1-7) |
| **模糊需求** | 例如「我要調整 UI 會員畫面」 | → 追問釐清後，再判斷走哪條路 |

**追問時一次問齊**（What + Why + Scope），不分多輪：
> 想先確認：1. 會員畫面要調整哪個部分？2. 目前什麼問題？3. 調整幅度？（微調 vs 重設計）

**釐清後必須確認**：摘要回給使用者，等他同意才動手。例如：
> 了解，整理一下：
> - 目標：會員詳情頁的資訊排版
> - 問題：資訊太密集，找不到重點
> - 範圍：重新排版 + 資訊分組，不改功能
>
> 這樣理解正確嗎？確認後我開始設計。

**使用者確認** → Step 1 | **使用者修正** → 更新理解 → 再確認

---

## 快速路徑（Demo 迭代 / 微調）

**核心原則：改什麼就動什麼，不碰其他部分，不重跑整個流程。**

適用場景（這些都走快速路徑，不追問、不產方案、不要確認 Gate）：
- 改顏色 / 配色調整
- 改排版 / 區塊順序調整
- 改間距 / 字級 / 圓角
- 改文字內容
- 調整元件樣式
- 新增 / 移除某個區塊
- 修改互動行為

### Quick Step 1: 讀取現有檔案

- 載入 `ux-config.yaml` 取得 brand tokens
- 讀取 `design-history.yaml` 掌握專案現有畫面（如果存在）
- **讀取既有的** `prototype.html`（和 `design-spec.md` 如果有的話）
- 讀取 `changelog.md` 最新 1-2 筆 → 了解目前版本、上次改了什麼、有什麼待辦
- 如果找不到既有 prototype → 提示使用者先跑完整路徑

**跨 Session 接續**：如果 `changelog.md` 存在且有待辦事項，主動提示：
> 「{feature} 目前 v{N}（{date}）
> 上次：{last_change}
> 待辦：{pending_items}
> 要繼續處理待辦，還是有新的修改？」

### Quick Step 2: 精準修改

直接修改 `prototype.html` 中對應的部分：

| 使用者說 | 改什麼 | 怎麼改 |
|---------|--------|--------|
| 「標題改大」 | CSS `font-size` | 只改對應的 CSS rule |
| 「卡片間距拉大」 | CSS `gap` / `margin` | 只改 spacing 值 |
| 「把區塊移到上面」 | HTML 順序 | 搬移 DOM 區塊位置 |
| 「按鈕改藍色」 | CSS `background` | 用 Config token (--color-secondary) |
| 「加一個篩選列」 | HTML + CSS | 插入新元件，套用現有風格 |
| 「loading 狀態要有 skeleton」 | HTML + JS | 在對應的 state toggle 加入 skeleton |
| 「整體配色太冷，換暖一點」 | 多個 CSS variables | 調整 Config 對應的色票，一次改 |

**原則**：
- **只改受影響的行**，不重新產生整個 HTML
- `design-spec.md` 同步更新受影響的段落（如果有的話），不重寫整份
- **[Hard constraint]** 顏色仍必須來自 Config tokens
- 如果修改涉及新增的顏色需求，先建議對應的 Config token，確認後再改

### Quick Step 3: 輸出變更摘要 + 更新版本紀錄

簡短列出改了什麼，然後更新版本紀錄：

> ✅ 已更新 prototype.html (v{N+1})
> - 標題字級：24px → 32px
> - 卡片間距：8px → 16px
> - 重新整理好了，刷新瀏覽器即可看到變更。

**更新 `changelog.md`**：在標題之後插入新版本區塊（最新在最上面）：
```markdown
## v{N+1} — {YYYY-MM-DD} (quick)
- **變更**：{before → after 清單}
- **原因**：{使用者需求摘要}
- **待辦**：{這次沒處理、下次要做的事}（沒有則省略此行）
```

**更新 `design-history.yaml`**：同步更新該 feature 的 `latest_version`、`last_change`、`iterations`、`last_modified`。

---

## Progress Tracking

Update `progress.yaml`:
```yaml
design:
  status: in_progress
  step: 1
  step_name: "load_config"
```
Update after each step. On completion, set `status: completed`.

## Process

### Step 1: Load Config + Design History

Auto-detect project → load Config → confirm brand colors, fonts, style, platform.

**讀取 Design History**：如果 `design-history.yaml` 存在，讀取已完成的設計清單，了解專案現有畫面。這能確保新設計與現有風格一致。如果發現現有設計有問題，也可以回頭修改。

**讀取 Changelog（迭代時）**：如果是對現有 feature 重設計（非全新功能），讀取 `changelog.md` 最新 1-2 筆，了解設計演進脈絡。提示使用者目前版本和待辦事項，確認要在此基礎上重設計。

### Step 2: Research Design Patterns (optional)

> **參考**：`references/visual-design-principles.md` + `references/style-library.md`

Use design knowledge to identify:
- Best practices for the feature type (e.g., filter patterns, list layouts, form flows)
- Platform-appropriate patterns based on `config.platform.type`
- Accessibility considerations

No external scripts required — use Claude's built-in design knowledge + Config tokens.

**視覺設計研究指引**：
- **風格方向判斷**：根據產品類型（B2B/B2C）+ 品牌調性（專業/活潑）+ 使用情境（辦公/休閒）選擇風格方向
- **色彩配比**：主色 60% / 輔色 30% / 強調色 10%（60-30-10 法則）
- **排版層級**：標題 → 副標 → 內文 → 輔助文字，每級至少 2px 差距 + 粗細/色彩變化
- **間距系統**：以 4pt 為基準的 grid 系統（4/8/12/16/24/32/48），一致的留白策略
- **視覺掃描**：內容排列遵循 F-pattern（列表頁）或 Z-pattern（著陸頁）

### Step 3: Generate Alternatives

> **參考**：`references/wireframing-guide.md`

Generate as many alternatives as the complexity warrants — a simple tweak may need only 1, a complex feature may benefit from several. No upper or lower limit.

Each alternative:
- **Name** + one-sentence description
- **Visual presentation** — use the most effective format: ASCII wireframe, structured list, text description, or combination
- **Interaction model** (gesture, navigation, feedback)
- **User need mapping** (how this addresses the described problem)
- **Pros and cons**

Select one with rationale.

**線框圖方法論**：
- **保真度選擇**：Low（驗證結構合理性）→ Mid（驗證互動流程）→ High（接近最終視覺）— 此階段通常用 Mid
- **複雜度判斷**：B2B dashboard、多角色流程、數據密集介面 → 需要 ≥2 個替代方案
- **每個方案必須回答「Why」**：為什麼這個佈局能解決使用者的問題？
- **Anti-pattern**：過度精緻的低保真線框圖 — 讓利益關係人以為是最終設計，反而限制修改空間

### Step 4: Write Design Spec + Save

Write and **save `design-spec.md` to disk immediately**.

**設計決策記錄**：每個設計選擇需記錄「Why」— 為什麼選這個佈局、配色、互動方式。如果有 ux-brief，標註哪個 pain point 被哪個設計元素解決。

```markdown
# Design Spec: {Feature Name}

## Design Tokens (from Config)
| Token | Value |
|-------|-------|

## Layout
{ASCII wireframe — annotated with dimensions}

## View Hierarchy
{Nested list with platform-appropriate components}

## Constraint Table
| View | Leading | Top | Width | Height | Notes |
|------|---------|-----|-------|--------|-------|

## Interaction States
| Component | Normal | Pressed | Disabled | Loading | Error |
|-----------|--------|---------|----------|---------|-------|

## Animation
| Trigger | Type | Duration | Curve |
|---------|------|----------|-------|

## Platform Adaptation
{Viewport / orientation / responsive notes based on config.platform.type}
```

### Step 5: Confirmation Gate — MUST STOP HERE

**Hard gate. Do NOT proceed to HTML without user confirmation.**

Present summary:
1. Selected alternative name, layout approach, main components
2. Design Tokens being applied
3. Interaction States per screen
4. Screen list the HTML prototype will contain
5. Ask: "Design Spec 已儲存。以上規格是否正確？確認後我會開始製作 HTML 互動原型。"

**User confirms** → Step 6 | **Feedback** → update spec → re-confirm | **Stop** → end here

### Step 6: Produce HTML Interactive Prototype

**使用模板**：讀取 `assets/prototype-template.html` 作為起點。模板已包含：
- iPad frame (1024×768) + status bar + sidebar + page header
- 完整 iOS-native CSS（card、dialog、sheet、toast、segmented control、stats bar、form inputs）
- Base JS utilities（showScreen、showToast、showDialog、hideDialog、showSheet、switchTab、setDevState）

**Agent 只需要填入 `{{placeholder}}` 區塊**：
- `{{PAGE_TITLE}}` — 頁面標題
- `{{SIDEBAR_ITEMS}}` — sidebar nav items（標記哪個 active）
- `{{SCREENS}}` — 主要畫面內容（`.screen` divs）
- `{{DIALOGS}}` — 對話框 / action sheet HTML
- `{{DEV_BUTTONS}}` — 底部狀態切換按鈕
- `{{CUSTOM_CSS}}` — 頁面專屬 CSS（放在 style 結尾）
- `{{CUSTOM_JS}}` — 頁面專屬 JS（放在 script 結尾）

**不要重寫模板已有的 CSS/JS**。只寫新增的部分。這能將 HTML 產出從 ~800 行降到 ~300-400 行。

**Structure Rules**:
- 基於模板產出 self-contained single .html
- 模板的 CSS class 名稱直接使用，不要另起新名（例如用 `.bk-card` 不要自創 `.card-item`）
- CSS variables mapped from Config tokens:
  ```css
  :root {
    /* 品牌橘 */
    --brand: {config.brand.colors.brand};
    --brand-light: {config.brand.colors.brand_light};
    --brand-dark: {config.brand.colors.brand_dark};
    /* 灰 */
    --gray-900: {config.brand.colors.gray_900};
    --gray-600: {config.brand.colors.gray_600};
    --gray-300: {config.brand.colors.gray_300};
    /* 紅 */
    --red: {config.brand.colors.red};
    --red-light: {config.brand.colors.red_light};
    /* 功能色 */
    --blue: {config.brand.colors.blue};
    --green: {config.brand.colors.green};
    --warning: {config.brand.colors.warning};
    /* 背景 */
    --white: {config.brand.colors.white};
    --surface: {config.brand.colors.surface};
    --sidebar-bg: {config.brand.colors.sidebar};
    /* 文字 */
    --label: {config.brand.colors.label};
    --placeholder: {config.brand.colors.placeholder};
    /* 字型 + 圓角 */
    --font: {config.brand.typography.font};
    --font-sm: {config.brand.typography.size.sm}px;
    --font-md: {config.brand.typography.size.md}px;
    --font-lg: {config.brand.typography.size.lg}px;
    --radius: {config.brand.corner_radius}px;
    --spacing: {config.brand.spacing}px;
  }
  ```
- **元件規格**：讀取 config `components` 區塊，套用 sidebar width、page header height、card radius/shadow/padding、dialog width 等精確值。不要自己猜元件尺寸。
- **HTML 精簡原則**：控制在 **600-900 行**內。mock data 5-6 筆就夠。狀態切換用 4 個 toggle（normal / loading / empty / error）。CSS 共用 class 盡量復用，避免重複定義相似樣式。JS 用事件委派（event delegation）減少重複 handler。

**Viewport（由 Config `platform.type` 決定）**：

| Platform | Viewport | 說明 |
|----------|----------|------|
| **Web** | Responsive (min 375px) | 預設。Mobile-first, 無固定寬度 |
| **iPad UIKit** | Landscape breakpoint | small <1100pt / large >=1100pt。預設橫向。如果 small/large 版面差異顯著，在 prototype 提供 viewport 切換 |
| **iOS** | 393×852 | iPhone 15 尺寸 |
| **Android** | 依 config viewport 設定 | 或預設 393×873 |

**Interactions**: click → state change, nav → page switching, forms → validation feedback, mock data only.

**State Coverage**: cover all states the feature actually needs (e.g. normal, loading, empty, error, success) with toggle buttons. Be thorough but don't force states that don't apply.

> **參考**：`references/interaction-patterns.md`

> **參考**：`references/apple-hig-ipad.md` — iPad App 設計時必須遵循 Apple HIG 規範

**互動設計原則**：
- **回饋即時性**：按鈕點擊 <100ms 回饋（視覺變化）、載入 <1s 顯示 skeleton screen
- **操作可逆性**：危險操作需確認對話框（刪除、離開未儲存頁面）
- **漸進式揭露**：不一次展示所有選項，依需求展開（例如「更多篩選」按鈕）
- **空狀態設計**：不只是「無資料」文字，要有插圖 + 引導下一步行動的 CTA
- **錯誤預防 > 錯誤修復**：用 disabled 狀態、即時驗證預防錯誤，而非事後彈錯誤訊息

### Step 7: Version Tracking

**更新 `changelog.md`**：在標題之後插入新版本區塊（最新在最上面）：

```markdown
# Changelog: {Feature Name}

> 自動維護。每次設計修改後追加新版本區塊（最新在最上面）。

## v{N} — {YYYY-MM-DD} (full|quick)
- **變更**：{具體改了什麼（before → after）}
- **原因**：{為什麼改（來自使用者需求）}
- **待辦**：{這次沒處理、下次要做的事}（沒有則省略此行）
```

完整路徑 v1 額外記錄：
- **摘要**：初版功能一句話描述
- **方案**：選擇了哪個替代方案

**更新 `design-history.yaml`**：同步更新該 feature 的 `latest_version`、`last_change`、`iterations`、`last_modified`。

---

## 跨 Session 接續

```
機制 1：changelog.md（per feature）
- 記錄每次修改的 before → after
- 新 session 讀取最新 1-2 筆 → 呈現目前狀態 + 待辦

機制 2：design-history.yaml（per project）
- 記錄所有 feature 的 status, latest_version, last_change
- 新 session 讀取 → 掌握專案全貌
```

**載入流程（新 Session 收到設計請求）**：
1. 讀 `design-history.yaml` → 列出現有 feature
2. 如果指定現有 feature → 讀 `changelog.md` 最新紀錄
3. 呈現：「{feature} 目前 v{N}，上次：{summary}，待辦：{items}」
4. 問：「要接續還是重新開始？」
5. 如果使用者未指定 feature，列出 `design-history.yaml` 中所有 `status: iterating` 的 feature，讓使用者選擇

---

## Auto-update Design History

每次設計完成（完整路徑或快速路徑），自動更新 `{project-root}/.claude/ux-prototypes/design-history.yaml`：

```yaml
# design-history.yaml — 專案設計摘要（自動維護，勿手動編輯）
project: "{config.project.name}"
last_updated: "{today}"

features:
  {feature-name}:
    description: "{一句話描述}"
    status: completed          # completed / iterating
    created: "{date}"
    last_modified: "{today}"
    layout: "{佈局摘要}"
    key_components: ["{元件1}", "{元件2}"]
    iterations: {N}            # 累計修改次數
    latest_version: {N}        # 對應 changelog.md 最新版號
    last_change: "{最近一次修改摘要}"
```

**用途**：
- 下次設計新畫面時，自動掌握「專案目前有哪些畫面、長什麼樣」
- 確保新設計的風格、佈局模式與既有畫面一致
- 發現既有畫面需要調整時，可以直接回去改（快速路徑）

---

## Progress Tracking (crash recovery)

Every feature has `{project-root}/.claude/ux-prototypes/{feature}/progress.yaml`. When any `/ux` command is received:

1. Scan for `in_progress` stages
2. If found → show user what was interrupted, ask continue or new
3. Continue → resume from `pending_action`

---

## Output

| File | Content |
|------|---------|
| `design-spec.md` | Complete design specification |
| `prototype.html` | Interactive HTML prototype |
| `changelog.md` | Version history + cross-session context (auto-maintained) |

Save to `{project-root}/.claude/ux-prototypes/{feature}/`.

## Completion Criteria

### 快速路徑（微調）
- [ ] Config loaded
- [ ] 變更範圍正確（只改了使用者要求的部分）
- [ ] **[Hard constraint]** 顏色來自 Config tokens
- [ ] 變更摘要已列出 (before → after)
- [ ] changelog.md updated
- [ ] design-history.yaml updated

### 完整路徑（新功能 / 重設計）

**Phase 1 — Spec (Steps 1-4)**:
- [ ] Config loaded, brand tokens applied
- [ ] Alternatives generated with selection rationale (quantity fits complexity)
- [ ] design-spec.md saved with Layout + View Hierarchy + Interaction States

**Gate (Step 5)**:
- [ ] Summary presented, user explicitly confirmed

**Phase 2 — HTML (Step 6)**:
- [ ] HTML opens and works in browser
- [ ] **[Hard constraint]** CSS variables map to Config tokens — no custom colors
- [ ] Viewport matches `config.platform.type`
- [ ] Relevant states covered with toggles
- [ ] HTML matches design-spec.md

**Phase 3 — Version (Step 7)**:
- [ ] changelog.md updated (v1 includes summary + selected alternative)
- [ ] design-history.yaml updated

## Communication Style

- Language: follow Config `language` field (default zh-TW)
- Format: Markdown tables + ASCII wireframes
- Tone: professional but friendly, explain "why" behind decisions
