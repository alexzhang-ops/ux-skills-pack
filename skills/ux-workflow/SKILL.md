---
name: ux-workflow
description: "UX 工作流路由 + 專案設定管理。觸發：/ux, /ux full, /ux config, /ux status, UX workflow, 改品牌色, 改風格, 改字型。路由到 2 個獨立 UX skill：ux-empathize, ux-design。管理專案設定 (ux-config.yaml)。用 /ux full 依序串接 empathize→design。/ux status 產生 CLAUDE.md 供跨 Session 使用。Triggers: /ux, /ux full, /ux config, /ux status, change brand color, change style, change font."
---

# UX Workflow — Router + Config Manager

**iPadOS 專用 UX Design Agent** — Routes to 2 independent UX skills. Each skill works standalone — this router handles `/ux full` chaining, `/ux config` management, and `/ux status` for cross-session context.

## Architecture

```
/ux <command> <description>
     │
     ├── empathize  → ux-empathize skill (standalone)
     ├── design     → ux-design skill (standalone)
     ├── full       → empathize → design (sequential)
     ├── config     → View/edit project config (this skill)
     └── status     → Generate/update CLAUDE.md (this skill)
```

## Routing

| Command | Delegates to |
|---------|-------------|
| `/ux empathize <desc>` | **ux-empathize** skill |
| `/ux design <desc>` | **ux-design** skill |
| `/ux full <desc>` | Run ux-empathize → ux-design sequentially |
| `/ux config` | Show current config (handled here) |
| `/ux status` | Generate/update project CLAUDE.md (handled here) |
| (natural language) | Edit config — e.g., "按鈕改白色", "主色改紅色" |

## Project Config

Config stores design style and brand information **inside the project** at `{project-root}/.claude/ux-config.yaml`.

### Auto-Detection（新專案 vs 接續）

1. Detect project root (where `.git/` or `.xcodeproj` or `package.json` lives)
2. Look for `{project-root}/.claude/ux-config.yaml`
3. **Found** → auto-load config，讀 `design-history.yaml` 掌握全貌 → 接續現有專案
4. **Not found** → 新專案流程：
   - 只問一個問題：「這個專案是 **Web** 還是 **iPad App**？」
   - 使用者選擇後 → 套用對應平台預設 + 共用色彩預設
   - 自動建立 `{project-root}/.claude/ux-config.yaml` 並存檔
   - 其餘設定之後可用 `/ux config` 調整

### Sensible Defaults

**共用色彩預設**（兩個平台共用）：
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
  corner_radius: 8
  style: "minimalism"
language: "zh-TW"
```

**Web 預設**：
```yaml
platform:
  type: "Web"
brand:
  typography:
    heading_font: "system-ui"
    body_font: "system-ui"
    base_size: 16
```

**iPad App 預設**：
```yaml
platform:
  type: "iPad App"
  min_version: "iPadOS 15"
brand:
  typography:
    heading_font: "PingFangTC-Semibold"
    body_font: "PingFangTC-Regular"
    base_size: 16
  viewport:
    breakpoint: 1100
    small: "iPad mini landscape (1133×744)"
    large: "iPad Air/Pro 11\" (1194×834) ~ iPad Pro 12.9\" (1366×1024)"
```

### Config Management

**`/ux config`** — Show current config summary:
```
Brand: Booking (minimalism)
Primary: #F2971B | Secondary: #4099FF | Accent: #E68A0D
Text: #181D27 / #414651 / #535862
Background: #FFFFFF | Surface: #FAFAFA
Font: PingFangTC-Semibold / PingFangTC-Regular @ 16pt
Corner: 8px | Platform: iPad UIKit
```

**Config editing is natural language only**:

**判斷規模**：
- **明確請求**（例如「主色改成 #FF6600」「圓角改 12」）→ 直接改，顯示 before/after 確認
- **模糊請求**（例如「我要修改配色 會員畫面」）→ 先顯示目前值，問要改哪個

**修改前必須確認**：顯示即將變更的 token 和 before/after 值，等使用者同意才寫入 config。

Examples:

Component-level (map to nearest token):
- "按鈕改白色" → update `brand.colors.surface` or button-specific token
- "標題改大" → update `brand.typography.base_size`
- "背景太亮了" → update `brand.colors.background`
- "文字看不清楚" → check contrast, update text/background colors

Token-level (direct mapping):
- "主色改紅色" → update `brand.colors.primary`
- "換 glassmorphism 風格" → update `brand.style`
- "圓角改成 12" → update `brand.corner_radius`
- "要深色模式" → update to dark palette + `dark-mode` style

Style-level (auto-cascade — 一句話觸發一整組調整):
- "整體感覺太冷" → 自動調整 primary/secondary/accent 為暖色調 + surface 微調
- "更專業一點" → 自動降低彩度、收緊 corner_radius、調整 text 對比
- "太花了" → 自動減少 accent 使用、style 改 `minimalism`、簡化色彩層次
- "換 glassmorphism 風格" → 自動調整 style + background 加透明度 + corner_radius 加大 + 加 blur 提示
- "要深色模式" → 自動翻轉整組色彩（bg↔text、surface 變深、border 變暗、保持 primary）
- "更活潑" → 自動提高 accent 彩度、加大 corner_radius、建議更鮮豔的 secondary
- "更簡潔" → 自動減少色彩數量、加大留白、style 改 `minimalism`

**Auto-cascade 規則**：
當使用者描述的是風格方向（非具體 token），自動產出**完整的 before/after 對照表**，一次呈現所有會連動的 token 變化：

```
風格調整：「更專業一點」
┌─────────────────┬──────────┬──────────┐
│ Token           │ Before   │ After    │
├─────────────────┼──────────┼──────────┤
│ style           │ minimal  │ minimal  │
│ primary         │ #F2971B  │ #D4850F  │ ← 降低彩度
│ secondary       │ #4099FF  │ #2B7ACC  │ ← 降低彩度
│ accent          │ #E68A0D  │ #C47A0B  │ ← 降低彩度
│ corner_radius   │ 8        │ 6        │ ← 收緊
│ text_primary    │ #181D27  │ #0F1319  │ ← 加深對比
└─────────────────┴──────────┴──────────┘
確認這些調整嗎？
```

使用者可以逐項接受/修改，確認後一次寫入 config。

Component-rule level (定義元件行為規範，寫入 config 的 `button_styles` / `component_rules`):
- "確定按鈕要實心，取消要空心" → 更新 `button_styles.confirm.bg` = primary, `button_styles.cancel.bg` = white + border
- "卡片要有陰影" → 更新 `shadow` 設定 or 新增 `component_rules.card.shadow: true`
- "輸入框聚焦要橘色邊框" → 更新 `component_rules.input.focus_border` = primary
- "導航列固定在上方" → 記錄 `component_rules.nav.position: fixed-top`

這類規則直接寫入 `ux-config.yaml`，後續 `ux-design` 產 HTML 時自動套用。

**Interpretation rules**:
1. Component mention → find closest config token
2. Component behavior rule → write to `button_styles` or `component_rules` section
3. Ambiguous → show current values, ask which to change
4. Style direction → auto-cascade 產出完整對照表
5. Can't map to token or rule (e.g., "加個 icon") → note as design feedback for next `/ux design`
6. Always show before/after values

After config change: remind user existing prototypes won't auto-update — re-run `/ux design` to apply.

## Progress Tracking (crash recovery)

Every feature has `{project-root}/.claude/ux-prototypes/{feature}/progress.yaml`. When any `/ux` command is received:

1. Scan for `in_progress` stages
2. If found → show user what was interrupted, ask continue or new
3. Continue → resume from `pending_action`

### 跨 Session 設計接續

當收到設計相關請求且指定了現有 feature：
1. 讀 `{feature}/changelog.md` 最新 1-2 筆 → 了解目前版本、上次改了什麼、有什麼待辦
2. 讀 `design-history.yaml` 取得專案全貌（所有 feature 的 `latest_version` + `last_change`）
3. 呈現目前版本 + 待辦事項 → 問使用者要接續還是重新開始：
   > 「{feature} 目前 v{N}（{date}）
   > 上次：{last_change_summary}
   > 待辦：{pending_items}
   > 要接續待辦、做新修改、還是重新開始？」
4. 如果使用者未指定 feature，列出 `design-history.yaml` 中所有 `status: iterating` 的 feature，讓使用者選擇

## Project CLAUDE.md Generation

`/ux status` 產生或更新 `{project-root}/CLAUDE.md`，讓新 Session 立即掌握專案 UX 狀態。

### 行為
1. 讀取 `ux-config.yaml` → 專案名稱、平台、品牌資訊
2. 讀取 `design-history.yaml` → feature 清單
3. 產出 / 覆寫 `{project-root}/CLAUDE.md`
4. 回報：「CLAUDE.md 已更新，包含 {N} 個 feature。」

### CLAUDE.md 模板

```markdown
# {project.name} — UX Design Context

## Platform
{platform.type}（{platform details}）

## Brand
- Primary: {primary} | Secondary: {secondary} | Accent: {accent}
- Font: {heading_font} / {body_font} @ {base_size}pt
- Style: {style} | Corner: {corner_radius}px

## Features
{For each feature in design-history.yaml:}
- **{feature}** ({status}) — {description} | v{latest_version} | {last_change}

## UX Commands
- `/ux empathize <描述>` — 使用者研究 + 痛點分析
- `/ux design <描述>` — 設計規格 + HTML 原型
- `/ux full <描述>` — empathize → design 一鍵完成
- `/ux config` — 查看 / 修改品牌設定
- `/ux status` — 重新產生此檔案
```

### 自動觸發
- 設計完成後（full path Step 6 或 quick path Step 3）自動更新 CLAUDE.md
- 使用者手動呼叫 `/ux status` 時更新

## Communication Style

- Language: follow Config `language` field (default zh-TW)
- Format: Markdown tables + ASCII wireframes
- Tone: professional but friendly, explain "why" behind decisions
