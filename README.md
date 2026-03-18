# UX Skills Pack — iPadOS UX Design Agent

快速生成互動式 HTML 原型，透過自然語言描述即可設計、迭代 Demo。
像有一個 UX 設計師幫你了解需求、規劃設計、產出可互動的原型。

## 安裝

### 方法 A：Plugin 安裝（推薦）

```bash
/plugin marketplace add <your-github>/ux-skills-pack
/plugin install ux-skills-pack
```

### 方法 B：手動複製

```bash
cp -r skills/ux-workflow skills/ux-empathize skills/ux-design ~/.claude/skills/
```

安裝後在 Claude App 的 **Customize** 裡會看到 3 個新 skill。

## 包含什麼

| Skill | 做什麼 | 觸發方式 |
|-------|--------|---------|
| **ux-workflow** | 路由 + 設定管理 | `/ux`, `/ux config`, `/ux status` |
| **ux-empathize** | 使用者研究 + 需求分析 | `/ux empathize <描述>` |
| **ux-design** | 設計規格 + HTML 互動原型 | `/ux design <描述>` |

## 能做什麼

### 了解需求（/ux empathize）
像 UX 設計師一樣幫你：
- 辨識目標使用者，建立 Persona
- 發掘痛點（功能性 / 認知性 / 情感性）
- 撰寫問題陳述（POV / HMW）
- 定義成功指標
- 產出 → `ux-brief.md`

### 設計 + 出原型（/ux design）
- 研究設計模式 + 產出替代方案（遵循 Apple HIG）
- 寫設計規格（佈局、元件、互動狀態）
- 產出**可互動的 HTML 原型**（瀏覽器直接打開）
- 快速迭代：說「標題改大」「按鈕改藍色」直接改
- 產出 → `design-spec.md` + `prototype.html`

### 一鍵完成（/ux full）
empathize → design 自動串接

### 跨 Session 接續
- 每個 feature 有版本歷史（`changelog.md`）
- 專案設計摘要（`design-history.yaml`）
- `/ux status` 產生 `CLAUDE.md`，新 Session 自動接續上次進度

## 快速開始

1. 安裝
2. `/ux design 預約日曆頁面`
3. 首次使用會問你：**Web** 還是 **iPad App**？
4. 自動建立設定 → 產出設計規格 + HTML 原型
5. 「間距拉大一點」「配色換暖色」→ 快速迭代

## 設定

首次使用自動建立 `ux-config.yaml`（只需選平台）。之後用自然語言調整：

- `主色改紅色` → 直接改 config token
- `更專業一點` → 自動調整一整組色彩 + 圓角
- `要深色模式` → 自動翻轉色彩

進階設定可參考 `ux-config-example.yaml`。

## 內建設計知識

### references（設計方法論參考文件）
- 使用者研究方法論、人物誌指南、資訊架構基礎
- 視覺設計原則（C.A.R.P.、60-30-10 配色、間距系統）
- 10 種視覺風格庫（Minimalism → Brutalism → 深色模式）
- 線框圖方法論、互動設計模式
- **Apple HIG iPadOS 規範**（佈局、元件、觸控、排版、色彩）

## 支援平台

- **Web** — 響應式 HTML 原型（min 375px）
- **iPad App** — iPad 專屬版面 + Apple HIG + UIKit patterns
