---
name: ux-design
description: "UX 設計階段 — 設計規格 + 互動式 HTML 原型。載入專案設定、搜尋設計資料庫、產出設計方案、收斂為 Design Spec，使用者確認後製作互動式 HTML 原型。輸出：design-spec.md + prototype.html。可獨立使用或作為 /ux full 流程的一部分。Triggers: /ux design, design spec, HTML prototype, wireframe, prototype, mockup, UI design, create prototype, design this feature, interactive prototype, design alternatives."
---

# UX Design — Ideate + Spec + HTML Prototype

## Scope

| Do | Don't |
|----|-------|
| Load Config and apply brand colors/fonts/style | No user research (use ux-empathize) |
| Search design references for best practices | No usability validation |
| Generate design alternatives (number fits complexity) | No production code generation |
| Converge into Design Spec | |
| Produce interactive HTML prototype | |

## Shared Context

**Config**: `{project-root}/.claude/ux-config.yaml` — auto-load. If not found, trigger first-time setup:
1. Detect project type (.xcodeproj → iOS, package.json → Web, etc.)
2. Try to extract existing design tokens from source code
3. Ask user key questions (name, platform, style, color, language)
4. Generate config with answers + sensible defaults
5. Never block — fall back to defaults if user skips setup

**Sensible defaults** (when no config and no tokens found):
```yaml
brand:
  colors:
    primary: "#2563EB"
    secondary: "#7C3AED"
    accent: "#F59E0B"
    background: "#FFFFFF"
    surface: "#F9FAFB"
    text_primary: "#111827"
    text_secondary: "#4B5563"
    text_tertiary: "#9CA3AF"
    destructive: "#EF4444"
    success: "#22C55E"
    border: "#E5E7EB"
  typography:
    heading_font: "system-ui"
    body_font: "system-ui"
    base_size: 16
  corner_radius: 8
  style: "minimalism"
```

**Output path**: `{project-root}/.claude/ux-prototypes/{feature-name}/`

**Design History**: `{project-root}/.claude/ux-prototypes/design-history.yaml` — 專案級設計摘要，自動累積。每次設計完成後更新，讓下次設計時能快速掌握專案現況。

Uses Claude's built-in design knowledge. No external scripts required.

**Language**: follow Config `language` field (default zh-TW).

## Input

- **Required**: Feature description OR UX Brief (`ux-brief.md`)
- **Auto-loaded**: Project Config

## Step 0: Confirm Before Proceed

**判斷規模 → 選擇路徑**：

| 規模 | 判斷依據 | 走哪條路 |
|------|---------|---------|
| **微調** | 改顏色、改間距、改排版、改文字、調整現有元件 — 不動結構 | → **快速路徑** |
| **有既有 prototype** | 使用者指定修改現有的 prototype.html | → **快速路徑** |
| **新功能 / 重設計** | 新頁面、新流程、結構性變更 | → **完整路徑** (Step 1-6) |
| **有 UX Brief** | `ux-brief.md` 作為輸入 | → **完整路徑** (Step 1-6) |
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
| 「把訂單區塊移到上面」 | HTML 順序 | 搬移 DOM 區塊位置 |
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
{Nested list with UIKit classes or platform components}

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
{landscape / portrait / multitasking notes}
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

**Structure Rules**:
- Self-contained single .html (inline CSS + vanilla JS, no external deps)
- CSS variables mapped from Config tokens:
  ```css
  :root {
    --color-primary: {config.brand.colors.primary};
    --color-secondary: {config.brand.colors.secondary};
    --color-accent: {config.brand.colors.accent};
    --color-bg: {config.brand.colors.background};
    --color-surface: {config.brand.colors.surface};
    --color-text-primary: {config.brand.colors.text_primary};
    --color-text-secondary: {config.brand.colors.text_secondary};
    --color-text-tertiary: {config.brand.colors.text_tertiary};
    --color-destructive: {config.brand.colors.destructive};
    --color-success: {config.brand.colors.success};
    --color-border: {config.brand.colors.border};
    --font-heading: {config.brand.typography.heading_font};
    --font-body: {config.brand.typography.body_font};
    --font-base-size: {config.brand.typography.base_size}px;
    --radius: {config.brand.corner_radius}px;
  }
  ```

**Viewport**:
- iPad breakpoint: **width < 1100pt = small** (iPad mini 1133×744 landscape ≈ borderline), **width >= 1100pt = large** (iPad Air+ 1194×834, iPad Pro 12.9" 1366×1024)
- iPhone: 393×852
- Web: responsive (min 375)
- For POS/Booking iPad apps: if layout differs between small and large screens, provide both versions in the prototype with a viewport toggle. Not every design needs both — only when the layout significantly changes.

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

## Output

| File | Content |
|------|---------|
| `design-spec.md` | Complete design specification |
| `prototype.html` | Interactive HTML prototype |
| `changelog.md` | Version history + cross-session context (auto-maintained) |

Save to `{project-root}/.claude/ux-prototypes/{feature}/`.

### Version Management — changelog.md

每次設計修改後（完整路徑 Step 6 或快速路徑 Quick Step 3），自動維護 `changelog.md`：

1. 讀取現有 `changelog.md` → 取得目前版號（沒有則從 v1 開始）
2. 在檔案頂部（標題之後）插入新版本區塊（最新在最上面）：

```markdown
# Changelog: {Feature Name}

> 自動維護。每次設計修改後追加新版本區塊（最新在最上面）。

## v{N} — {YYYY-MM-DD} (full|quick)
- **變更**：{具體改了什麼（before → after）}
- **原因**：{為什麼改（來自使用者需求）}
- **待辦**：{這次沒處理、下次要做的事}（沒有則省略此行）
```

3. 完整路徑 v1 額外記錄：
   - **摘要**：初版功能一句話描述
   - **方案**：選擇了哪個替代方案

4. 同步更新 `design-history.yaml` 的 `latest_version` + `last_change`

**跨 Session 接續**：新 Session 收到設計相關請求時：
1. 讀 `changelog.md` 最新 1-2 筆 → 了解目前版本、上次改了什麼、有什麼待辦
2. 呈現給使用者：
   > 「{feature} 目前 v{N}（{date}）
   > 上次：{last_change_summary}
   > 待辦：{pending_items}
   > 要繼續嗎？」
3. 使用者確認 → 接續設計

### Auto-update Design History

每次設計完成（完整路徑或快速路徑），自動更新 `{project-root}/.claude/ux-prototypes/design-history.yaml`：

```yaml
# design-history.yaml — 專案設計摘要（自動維護，勿手動編輯）
project: "Booking"
last_updated: "2026-03-18"

features:
  reservation-time-filter:
    description: "預約時段篩選器"
    status: completed          # completed / iterating
    created: "2026-03-15"
    last_modified: "2026-03-18"
    layout: "右側 drawer，時段格狀選擇"
    key_components: ["時段 grid", "日期選擇", "篩選按鈕"]
    iterations: 3              # 累計修改次數
    latest_version: 3          # 對應 changelog.md 最新版號
    last_change: "間距調整 + loading skeleton"  # 最近一次修改摘要

  member-list:
    description: "會員列表頁"
    status: iterating
    created: "2026-03-17"
    last_modified: "2026-03-18"
    layout: "列表 + 頂部搜尋/篩選"
    key_components: ["搜尋欄", "篩選列", "會員卡片"]
    iterations: 1
    latest_version: 1              # ← 對應 changelog.md 最新版號
    last_change: "初版列表頁"       # ← 最近一次修改摘要
```

**用途**：
- 下次設計新畫面時，自動掌握「專案目前有哪些畫面、長什麼樣」
- 確保新設計的風格、佈局模式與既有畫面一致
- 發現既有畫面需要調整時，可以直接回去改（快速路徑）

## Completion Criteria

### 快速路徑（微調）
- [ ] Config loaded
- [ ] 變更範圍正確（只改了使用者要求的部分）
- [ ] **[Hard constraint]** 顏色來自 Config tokens
- [ ] 變更摘要已列出 (before → after)

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
- [ ] Relevant states covered with toggles
- [ ] HTML matches design-spec.md
