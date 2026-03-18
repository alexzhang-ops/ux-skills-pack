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

## Input

Feature description (one sentence or paragraph).

## Step 0: Confirm Before Proceed

**判斷規模**：
- **小需求**（描述已明確，例如「分析預約日曆的操作流程痛點，店員在 iPad 上每天使用」）→ 直接進 Step 1，不追問
- **模糊需求**（例如「幫我優化 UX 訂單搜尋」）→ 追問釐清

**追問時一次問齊**（What + Why + Who + Where），不分多輪：
> 想先確認：1. 訂單搜尋的哪個環節？2. 目前什麼問題？3. 誰在用？4. 什麼情境？

**釐清後必須確認**：把理解的需求摘要回給使用者，等他說「對」或「OK」才開始。例如：
> 了解，整理一下：
> - 功能：訂單搜尋頁的篩選條件
> - 問題：店員反映篩選太多步驟
> - 使用者：前台店員，iPad 操作
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

- List 1-3 primary user personas
- Each persona: name, role, usage context, tech familiarity
- Mark primary vs secondary users

**Persona 合成原則**：
- 人物誌必須基於**行為模式**（不只是人口統計）— 同一職位可能有截然不同的使用行為
- 每個 persona 需涵蓋：**目標**（想完成什麼）、**行為模式**（怎麼完成）、**挫折點**（什麼阻礙他）、**使用情境**（何時何地使用）
- 區分 primary（核心使用者）/ secondary（偶爾使用）/ edge-case（極端情境）
- **Anti-pattern**：避免「虛構敘事式」persona — 沒有行為依據的角色故事（例如「小明，25 歲，喜歡科技」沒有設計價值）

### Step 2: Discover Pain Points

> **參考**：`references/research-methods.md` — 使用者研究方法論

- Find the key pain points (functional / cognitive / emotional). One precise pain point is more valuable than three forced ones — no quantity floor.
- Each: description, context when it occurs, impact (high/medium/low)
- Note current workaround (if any) and its shortcomings

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
| POV | 使用者需要更快的系統 | 前台店員在尖峰時段需要 3 秒內完成訂單搜尋，因為客人在等待時會失去耐心 |
| HMW | 如何改善搜尋？ | 如何讓店員在不離開當前畫面的情況下快速篩選到目標訂單？ |

### Step 4: Success Metrics

Define measurable metrics covering:
- **Efficiency**: task completion time, number of steps
- **Correctness**: error rate, retry rate
- **Satisfaction**: flow smoothness, learning curve

### Step 5: Constraint Inventory

> **參考**：`references/ia-basics.md` — 資訊架構基礎

- Technical constraints (from Config `platform` + `constraints`)
- Accessibility constraints (WCAG 2.1 AA baseline)
- Brand constraints (from Config `brand.style`)
- Business constraints (user-provided)

**資訊架構思考**（同步盤點）：
- **內容結構**：這個功能涉及哪些資訊類型？如何分組？
- **導航層級**：從入口到完成任務需要幾層？建議 ≤3 層
- **任務頻率**：高頻任務應在最少步驟內可達；低頻任務可以藏深一點
- **既有模式**：與專案其他畫面的導航/佈局模式是否一致？

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
- **Technical**: {from Config}
- **Accessibility**: WCAG 2.1 AA
- **Brand**: {from Config}
- **Business**: {user-provided}

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
- [ ] `changelog.md` initialized (header only)
- [ ] `design-history.yaml` updated with feature entry
