# Apple Human Interface Guidelines — iPadOS 設計規範

> 供 ux-design skill 參考。設計 iPad App 原型時，應遵循以下 Apple HIG 規範以確保原生體驗。

## 核心設計原則

| 原則 | 說明 | 實踐 |
|------|------|------|
| **Clarity** | 文字清晰可讀、圖示精確傳達意義、裝飾不干擾功能 | SF Symbols、清晰的層級、足夠的對比 |
| **Deference** | UI 輔助內容、不喧賓奪主 | 半透明材質、輕薄 UI chrome、內容優先 |
| **Depth** | 利用層級和動畫建立空間感 | Sheet、Popover、視差效果 |

---

## 佈局與導航

### iPad 導航模式

| 模式 | 適用場景 | HIG 規範 |
|------|---------|---------|
| **Sidebar** | 主導航（≥3 個頂層區域） | 寬度 320pt 預設、可收合、支援 `UISplitViewController` |
| **Tab Bar** | 底部 Tab（≤5 個） | iPad 上 Tab Bar 在底部或側邊均可 |
| **Toolbar** | 頁面級操作 | 固定在頂部或底部，使用 SF Symbols |

### Split View（iPad 核心佈局）

```
┌──────────┬─────────────────────────────┐
│          │                             │
│ Sidebar  │      Detail / Content       │
│ (320pt)  │                             │
│          │                             │
│ Primary  │      Secondary              │
│          │                             │
└──────────┴─────────────────────────────┘
```

- **兩欄模式**（Sidebar + Detail）：最常見的 iPad 佈局
- **三欄模式**（Sidebar + Supplementary + Detail）：郵件、檔案管理類
- Sidebar 在 compact width 時自動隱藏為 overlay
- 支援 `columnVisibility` 控制欄位顯示

### Multitasking 支援

| 模式 | 寬度 | 設計要求 |
|------|------|---------|
| Full Screen | 全螢幕 | 標準佈局 |
| Split View 1/2 | ≈507pt (11") | 需適配窄螢幕，隱藏 Sidebar |
| Split View 1/3 | ≈320pt | 極窄，類似 iPhone 佈局 |
| Slide Over | ≈320pt | 浮動視窗 |

**設計要點**：
- 所有 iPad App **必須**支援 Multitasking（Apple 審核要求）
- 使用 Auto Layout / Size Classes 適配不同寬度
- Compact width 時 Sidebar 變為 overlay 或隱藏

---

## 元件規範

### Navigation Bar

| 屬性 | 規範 |
|------|------|
| 高度 | 44pt（standard）/ 96pt（large title） |
| Large Title | 首頁級頁面用、往下滑動時收合為 standard |
| Back Button | 系統自動提供、顯示上一頁標題（可自訂） |
| 右側按鈕 | ≤3 個，使用 SF Symbols |

### Bar Buttons & Controls

| 元件 | 最小尺寸 | 說明 |
|------|---------|------|
| Bar Button Item | 44×44pt 觸控區域 | 圖示建議 22×22pt |
| Segmented Control | 高度 32pt | ≤5 個選項 |
| Search Bar | 高度 36pt | 支援 scope buttons、tokens |

### Buttons

| 類型 | HIG 用途 | 樣式 |
|------|---------|------|
| **Filled** | 主要操作（Primary CTA） | 實心背景 + 白字 |
| **Tinted** | 次要操作 | 淺色底 + 品牌色文字 |
| **Gray** | 輔助操作 | 灰色底 |
| **Plain** | 低優先級 / 導航 | 僅文字（品牌色） |
| **Destructive** | 危險操作 | 紅色 |

**按鈕間距**：相鄰按鈕至少 8pt 間距、觸控區域 ≥44×44pt

### Lists（UITableView / UICollectionView）

| 樣式 | 適用 |
|------|------|
| **Inset Grouped** | iPad 預設推薦 — 圓角卡片分組 |
| **Plain** | 長列表、搜尋結果 |
| **Sidebar** | 導航列表 |

**Cell 高度**：最小 44pt、建議 52-72pt（iPad 有更多空間）

### Sheets & Modals

| 類型 | 大小 | 適用 |
|------|------|------|
| **Sheet (.formSheet)** | 540×620pt 預設 | 表單、設定、簡短任務 |
| **Fullscreen** | 全螢幕 | 沉浸式內容（相機、編輯器） |
| **Popover** | 自適應 | iPad 獨有！選單、選擇器、確認 |
| **Page Sheet** | 寬度填滿 | 長內容、多步驟流程 |

**iPad 優先用 Popover**：不要用 Action Sheet（那是 iPhone 的），iPad 上用 Popover 取代。

---

## 觸控與互動

### 觸控目標

| 規範 | 值 |
|------|-----|
| 最小觸控區域 | **44×44pt** |
| 建議觸控區域 | 48×48pt |
| 按鈕間最小間距 | 8pt |

### 手勢

| 手勢 | 系統標準用途 | 設計注意 |
|------|------------|---------|
| Tap | 選擇、觸發 | 提供即時視覺回饋 |
| Long Press | Context Menu | 使用 `UIContextMenuInteraction` |
| Swipe (leading/trailing) | 快捷操作（刪除、標記） | 提供可見替代操作 |
| Pinch | 縮放 | 地圖、圖片、PDF |
| Pan | 滾動、拖曳 | 遵循系統慣性滾動 |
| **Edge Swipe** (系統保留) | 返回上一頁（左邊緣） | 不要覆蓋！ |

### Pointer / Trackpad（iPad 獨有）

| 行為 | 規範 |
|------|------|
| Hover 效果 | 按鈕/連結 hover 時自動顯示高亮圓角矩形 |
| Cursor 自適應 | 游標自動吸附到按鈕形狀 |
| 右鍵選單 | 等同 Long Press Context Menu |

**設計要點**：iPad 同時支援觸控和 Pointer，設計時兩者都要考慮。

### Keyboard Shortcuts

- 支援外接鍵盤是 iPad App 的加分項
- `⌘+主要操作` 是標準模式（例如 `⌘N` 新增、`⌘S` 儲存）
- 使用 `UIKeyCommand` 註冊
- 長按 `⌘` 顯示快捷鍵列表

---

## 排版

### SF Pro（系統字型）

| 用途 | 字型 | Text Style |
|------|------|-----------|
| 大標題 | SF Pro Bold, 34pt | `.largeTitle` |
| 標題 | SF Pro Bold, 28pt | `.title1` |
| 副標題 | SF Pro Regular, 22pt | `.title2` |
| 段落標題 | SF Pro Semibold, 20pt | `.title3` |
| 內文 | SF Pro Regular, 17pt | `.body` |
| 次要文字 | SF Pro Regular, 15pt | `.subheadline` |
| 說明文字 | SF Pro Regular, 13pt | `.footnote` |
| 小字 | SF Pro Regular, 12pt | `.caption1` |

**Dynamic Type**：必須支援！使用 `UIFont.preferredFont(forTextStyle:)` 而非固定字級。

### 字重使用

| 字重 | 用途 |
|------|------|
| **Bold** | 標題、重要數字 |
| **Semibold** | 段落標題、強調文字 |
| **Medium** | 按鈕文字、Tab 標籤 |
| **Regular** | 內文、一般文字 |

---

## 色彩

### 系統色（System Colors）

| 色彩 | Light Mode | Dark Mode | 用途 |
|------|-----------|-----------|------|
| **systemBlue** | #007AFF | #0A84FF | 連結、選中、CTA |
| **systemGreen** | #34C759 | #30D158 | 成功、完成 |
| **systemRed** | #FF3B30 | #FF453A | 錯誤、刪除、警告 |
| **systemOrange** | #FF9500 | #FF9F0A | 警告、注意 |
| **systemYellow** | #FFCC00 | #FFD60A | 星標、亮點 |
| **systemGray** | #8E8E93 | #8E8E93 | 次要、禁用 |

### 語義色（Semantic Colors）

| Token | 說明 |
|-------|------|
| `label` | 主要文字 |
| `secondaryLabel` | 次要文字 |
| `tertiaryLabel` | 輔助文字 |
| `systemBackground` | 主背景 |
| `secondarySystemBackground` | 次要背景（grouped table） |
| `separator` | 分隔線 |
| `systemFill` | 填充（搜尋欄、輸入框背景） |

**原則**：使用語義色而非固定色值，自動適配 Dark Mode。

### Dark Mode

- **必須支援**（Apple 審核建議）
- 使用語義色自動適配
- 背景層級：`systemBackground` → `secondarySystemBackground` → `tertiarySystemBackground`
- Elevation 越高越亮（與 Light Mode 相反）

---

## 圖示

### SF Symbols

- Apple 提供 **5000+** 向量圖示
- 自動適配 Dynamic Type 字級
- 支援多色、階層色、調色盤模式
- **優先使用 SF Symbols**，只在沒有合適的時候自訂

| 配置 | 說明 |
|------|------|
| Rendering Mode | Monochrome / Hierarchical / Palette / Multicolor |
| Weight | 與文字字重匹配 |
| Scale | Small / Medium / Large（與文字字級對應） |

---

## 間距與格線

### iPad 間距規範

| 位置 | 值 |
|------|-----|
| Safe Area（頂部） | 24pt（無 NavigationBar 時） |
| Safe Area（底部） | 20pt（無 TabBar 時） |
| 水平 margin | 20pt（compact）/ 20pt（regular） |
| Cell 內 padding | 16pt（leading/trailing） |
| Section 間距 | 20-35pt |
| 元素間距 | 8pt（緊密）/ 16pt（標準）/ 24pt（寬鬆） |

### Readable Content Guide

- iPad 寬螢幕上，長文內容不應填滿整個寬度
- 使用 `readableContentGuide` 自動限制最大閱讀寬度（≈672pt）
- 列表、表單優先使用 Inset Grouped 樣式

---

## 動畫與轉場

### 系統標準動畫

| 轉場 | 時長 | 曲線 |
|------|------|------|
| Push（導航推進） | 0.35s | `curveEaseInOut` |
| Present Sheet | 0.3s | `spring(duration: 0.3)` |
| Dismiss | 0.25s | `curveEaseIn` |
| Fade In/Out | 0.2s | `curveEaseInOut` |

### 動畫原則

- **有意義**：動畫需傳達空間關係（從哪來、到哪去）
- **快速**：不超過 0.5s，不要讓使用者等待
- **可中斷**：使用者應能在動畫中繼續操作
- **一致**：相同類型的轉場使用相同動畫

---

## Drag & Drop（iPad 特色功能）

| 行為 | 規範 |
|------|------|
| 發起拖曳 | Long Press → 元素浮起 → 可拖曳 |
| 多選拖曳 | 拖曳中可 Tap 其他項目加入 |
| 放置目標 | 高亮顯示可放置區域 |
| 跨 App | iPad 支援跨 App Drag & Drop |

**設計要點**：列表、卡片、圖片等內容建議支援 Drag & Drop，這是 iPad 的差異化功能。

---

## 無障礙（Accessibility）

| 類別 | 要求 |
|------|------|
| VoiceOver | 所有互動元素有 `accessibilityLabel` |
| Dynamic Type | 使用 Text Styles、支援最大字級 |
| Color | 不依賴色彩作為唯一辨識（加 icon/文字） |
| Motion | 尊重「減少動態效果」設定 |
| Contrast | ≥4.5:1（一般文字）、≥3:1（大文字） |

---

## Checklist — 設計 iPad 原型時的快速檢查

- [ ] 使用 Split View 佈局（Sidebar + Detail）？
- [ ] 支援 Multitasking（不同寬度適配）？
- [ ] 觸控目標 ≥44×44pt？
- [ ] 使用語義色 / SF Pro / SF Symbols？
- [ ] Popover 取代 Action Sheet？
- [ ] Inset Grouped List 而非 Plain？
- [ ] 支援 Pointer hover 效果？
- [ ] 危險操作有確認對話框？
- [ ] 空狀態有引導？
- [ ] 動畫 ≤0.5s 且有意義？
