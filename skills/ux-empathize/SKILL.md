---
name: ux-empathize
description: "UX 同理心階段 — 使用者研究 + 問題定義。辨識目標使用者、發掘關鍵痛點、撰寫 POV/HMW 問題陳述、定義成功指標、盤點限制條件。輸出：ux-brief.md。可獨立使用或作為 /ux full 流程的一部分。Triggers: /ux empathize, user research, pain points, persona, user needs, POV statement, HMW statement, UX brief, understand users."
---

# UX Empathize — User Research + Problem Definition

## Scope

| Do | Don't |
|----|-------|
| Identify target users (role, goal, context) | No wireframes |
| Find key pain points (quality over quantity) | No style/color choices |
| Write POV / HMW problem statements | No Design Spec |
| Define success metrics (completion rate, error rate, time) | No HTML |
| List constraints (tech/a11y/brand) | No code |

## Shared Context

**Config**: `{project-root}/.claude/ux-config.yaml` — auto-load for constraints + platform info. If not found, still proceed (empathize doesn't require config).

**Output path**: `{project-root}/.claude/ux-prototypes/{feature-name}/ux-brief.md`

**Feature name**: auto-generated from description (kebab-case, e.g., `reservation-calendar`).

Uses Claude's built-in UX research knowledge. No external scripts required.

**Language**: follow Config `language` field (default zh-TW).

## Output Length Guide

整份 ux-brief.md 控制在 **150-250 行**。寧可精準簡潔，不要冗長。每個 Step 的篇幅建議：
- Persona：20-30 行（1-2 人，每人 3-4 行描述）
- Pain Points：15-25 行（2-4 個，每個 3 行）
- POV + HMW：10 行
- Metrics：10 行
- Constraints：10-15 行
- UX Concept：40-60 行（flow + wireframe + states 加總）

## Input

Feature description (one sentence or paragraph).

## Step 0: Confirm Before Proceed

**判斷規模**：
- **小需求**（描述已明確，例如「分析會員列表的操作流程痛點，使用者在桌機上每天使用」）→ 直接進 Step 1，不追問
- **模糊需求**（例如「幫我優化 UX 搜尋功能」）→ 追問釐清

**追問時一次問齊**（What + Why + Who + Where），不分多輪：
> 想先確認：1. 搜尋功能的哪個環節？2. 目前什麼問題？3. 誰在用？4. 什麼情境？

**釐清後必須確認**：把理解的需求摘要回給使用者，等他說「對」或「OK」才開始。例如：
> 了解，整理一下：
> - 功能：搜尋頁的篩選條件
> - 問題：使用者反映篩選太多步驟
> - 使用者：一般用戶，桌機操作
>
> 這樣理解正確嗎？確認後我開始分析。

**使用者確認** → Step 1 | **使用者修正** → 更新理解 → 再確認

## Progress Tracking

At the start, create the feature directory and `progress.yaml`:
```yaml
feature: "{feature-name}"
description: "{user's description}"
started_at: "{now}"
updated_at: "{now}"
stages:
  empathize:
    status: in_progress
    step: 1
    step_name: "identify_users"
  design:
    status: pending
pending_action: "Identify target users"
```
Update `step` + `step_name` + `updated_at` after each step. On completion, set `status: completed`.

## Process

### Step 1: Identify Users

> **參考**：`references/persona-guide.md` — 完整的人物誌合成方法論

- List 1-2 primary user personas（通常 1 個 primary + 1 個 secondary 就夠，不要超過 3 個）
- Each persona: name, role, usage context, tech familiarity
- Mark primary vs secondary users
- **簡潔原則**：每個 persona 的行為描述控制在 3-4 行內，不要寫小說

**Persona 合成原則**：
- 人物誌必須基於**行為模式**（不只是人口統計）— 同一職位可能有截然不同的使用行為
- 每個 persona 需涵蓋：**目標**（想完成什麼）、**行為模式**（怎麼完成）、**挫折點**（什麼阻礙他）、**使用情境**（何時何地使用）
- 區分 primary（核心使用者）/ secondary（偶爾使用）/ edge-case（極端情境）
- **Anti-pattern**：避免「虛構敘事式」persona — 沒有行為依據的角色故事（例如「小明，25 歲，喜歡科技」沒有設計價值）

### Step 2: Discover Pain Points

> **參考**：`references/research-methods.md` — 使用者研究方法論

- Find 2-3 key pain points (functional / cognitive / emotional). One precise pain point is more valuable than three forced ones.
- Each: one-line description, context, impact (high/medium/low)
- Workaround: one sentence if any, skip if none

**研究方法論指引**：
- **開放式提問**：用「描述一下你上次…的過程」「遇到什麼困難？」而非「你覺得這個好不好用？」
- **避免引導性問題**（Leading Questions）：「這個功能是不是很難用？」→ 改問「你怎麼完成這個任務的？」
- **觀察 vs 意見**：使用者說的和做的常不同 — 行為觀察比口頭意見更可靠
- **痛點需有佐證**：每個痛點應有具體場景描述或行為觀察支持，避免純推測
- **三層痛點分類**：功能性（做不到）→ 認知性（想不通）→ 情感性（感到挫折）

### Step 3: Problem Statement

- **POV** (Point of View): `[User] needs [need], because [insight]`
- **HMW** (How Might We): `How might we [opportunity]?` (2-3 statements)
- HMW should not be too broad (unfocused) or too narrow (limits creativity)

**收斂方法論**：
- POV 必須具體且可行動 — 「使用者需要更好的體驗」太空泛，沒有設計指引
- HMW 的平衡點：太廣（「如何改善所有操作？」）→ 沒有焦點；太窄（「如何在按鈕旁加說明文字？」）→ 限制創意

**好的 vs 不好的範例**：
| | ❌ 不好的 | ✅ 好的 |
|---|---|---|
| POV | 使用者需要更快的系統 | 業務人員在尖峰時段需要 3 秒內完成搜尋，因為客戶在等待時會失去耐心 |
| HMW | 如何改善搜尋？ | 如何讓使用者在不離開當前畫面的情況下快速篩選到目標項目？ |

### Step 4: Success Metrics

Define measurable metrics covering:
- **Efficiency**: task completion time, number of steps
- **Correctness**: error rate, retry rate
- 3-5 個指標就夠，每個一行，不需要長篇解釋

### Step 5: Constraint Inventory

> **參考**：`references/ia-basics.md` — 資訊架構基礎

- Technical constraints (from Config `platform` + `constraints`, if available)
- Accessibility constraints (WCAG 2.1 AA baseline)
- Brand constraints (from Config `brand.style`, if available)
- Business constraints (user-provided)

**資訊架構思考**（同步盤點）：
- **內容結構**：這個功能涉及哪些資訊類型？如何分組？
- **導航層級**：從入口到完成任務需要幾層？建議 ≤3 層
- **任務頻率**：高頻任務應在最少步驟內可達；低頻任務可以藏深一點
- **既有模式**：與專案其他畫面的導航/佈局模式是否一致？

## Step 5.5: UX Concept（體驗概念 — 三件套）

研究完成後，用文字呈現**完整的體驗概念**，讓使用者在進入 design 階段前就能對齊方向。UX 不只是畫面，還有流程和狀態，所以需要三個東西一起看：

### A. Task Flow（使用者怎麼走）

用 ASCII flow diagram 畫出**主要任務流程** — 從使用者的角度，不是系統的角度。

```
{觸發點} → {動作 1} → {動作 2} → {完成}
                         ↓ 異常
                    {錯誤處理} → {回到哪一步}
```

**原則**：
- 每個節點是使用者的**動作**或**決策**，不是系統內部邏輯
- 標註哪個 pain point 在哪一步被解決
- 分支 = 使用者的決策點或系統的異常處理
- 複雜流程標註預估步驟數（呼應 Step 4 的 success metrics）

**範例**：
```
客人報姓名
    → 店員觸碰搜尋框（1 tap）     ← 解決 Pain 1：入口直達
    → 輸入關鍵字（即時結果）       ← 解決 Pain 2：不需送出
    → 看到結果列表（含電話末碼）   ← 解決 Pain 3：一屏辨識
    → 點選確認 → 完成
         ↓ 找不到
    系統提示「試試電話後四碼」→ 重新輸入
         ↓ 仍找不到
    「新增會員」CTA → 快速註冊
```

### B. Concept Wireframe（畫面長什麼樣）

用 ASCII 畫出**概念版面**，標註各區域解決什麼問題。

```
┌─────────────────────────────────────────────┐
│ [Title Bar]                          [btn]  │
├──────────┬──────────────────────────────────┤
│          │                                  │
│  List    │    Detail / Content              │
│          │                                  │
├──────────┴──────────────────────────────────┤
│ [Status Bar / Actions]                      │
└─────────────────────────────────────────────┘
```
- 用 `[ ]` 標註元件
- 用 `← Pain 1` 旁註對應的 pain point
- 如果 Config 有 `platform.type`，畫出對應的 viewport 比例
- 複雜功能可畫 2 個方向讓使用者選

### C. Key States（關鍵狀態清單）

列出這個功能**會遇到的所有狀態**，確保設計階段不遺漏。

```
| 狀態 | 觸發條件 | 使用者看到什麼 | 設計重點 |
|------|---------|--------------|---------|
| 正常 | 有資料 | 列表 + 內容 | 主路徑 |
| 載入中 | API 請求中 | skeleton / spinner | 回饋即時性 |
| 空結果 | 搜尋/篩選無匹配 | 空狀態 + 引導 CTA | 告訴使用者下一步 |
| 錯誤 | 網路/伺服器問題 | 錯誤訊息 + 重試 | 可恢復性 |
| 首次使用 | 無歷史資料 | onboarding / 引導 | 降低學習門檻 |
```

**控制數量**：只列實際會發生的狀態，通常 4-6 種就夠。不要列超過 8 種 — 太多反而失去焦點。

### 呈現後確認

三件套畫完後，問使用者：
> 「以上是研究結果和體驗概念：
> - **流程**：{N} 步完成核心任務
> - **版面**：{簡述佈局方向}
> - **狀態**：需要處理 {N} 種狀態
>
> 這個方向 OK 嗎？確認後進入 design 階段，會做成完整互動原型。」

## Data Search (optional)

Uses Claude's built-in UX research knowledge to identify relevant patterns and best practices.

## Output Template

Save to `{project-root}/.claude/ux-prototypes/{feature}/ux-brief.md`:

```markdown
# UX Brief: {Feature Name}

## Target Users
| Persona | Goal | Context | Tech Familiarity |
|---------|------|---------|-----------------|
| ... | ... | ... | ... |

## Pain Points
1. **{Pain Point}** — {Description}
   - Context: {when it occurs}
   - Impact: {high/medium/low}
   - Current workaround: {yes/no, shortcomings}

2. **{Pain Point}** — ...

## Problem Statement
**POV**: {User} needs {need}, because {insight}

**HMW**:
- How might we {opportunity 1}?
- How might we {opportunity 2}?

## Success Metrics
| Metric | Baseline | Target |
|--------|----------|--------|
| Completion time | — | — |
| Error rate | — | — |
| Steps count | — | — |

## Constraints
- **Technical**: {from Config or user-provided}
- **Accessibility**: WCAG 2.1 AA
- **Brand**: {from Config or user-provided}
- **Business**: {user-provided}

## UX Concept

### Task Flow
{ASCII flow diagram — 主要任務流程 + 異常分支，標註各步驟解決的 pain point}

### Concept Wireframe
{ASCII 概念線框圖 — 標註各區域對應的 pain point / HMW}

### Key States
| 狀態 | 觸發條件 | 使用者看到什麼 | 設計重點 |
|------|---------|--------------|---------|
| ... | ... | ... | ... |

## UX Priority
{High / Medium / Low} — {one sentence rationale}
```

## Step 6: Initialize Version Tracking

儲存 `ux-brief.md` 後，初始化版本紀錄：

1. **建立 `changelog.md`**（只有標題，尚無版本紀錄 — 等 design 階段才會有 v1）：
```markdown
# Changelog: {Feature Name}

> 自動維護。每次設計修改後追加新版本區塊（最新在最上面）。
```

2. **更新 `design-history.yaml`**：新增 feature 條目：
```yaml
  {feature}:
    description: "{一句話描述}"
    status: in_progress
    created: "{today}"
    last_modified: "{today}"
    latest_version: 0          # 尚無設計版本
    last_change: "empathize 完成"
```

## Completion Criteria

- [ ] User personas identified (at least 1)
- [ ] Key pain points discovered, each with context + impact level (quality over quantity — no minimum)
- [ ] POV is specific and actionable
- [ ] HMW neither too broad nor too narrow
- [ ] Success metrics are measurable
- [ ] Constraints inventoried (tech + a11y + brand)
- [ ] UX Concept complete: task flow + concept wireframe + key states
- [ ] Each pain point mapped to a specific flow step or UI region
- [ ] User confirmed direction before proceeding to design
- [ ] `changelog.md` initialized (header only)
- [ ] `design-history.yaml` updated with feature entry
