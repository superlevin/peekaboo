# Peekaboo 國際化（i18n）與本地化（l10n）策略

## 概述

本文件說明 Peekaboo 專案的國際化和本地化實作策略，重點關注繁體中文支援，同時建立可擴展的框架以支援未來的語言。

## 當前狀態

### 現有架構
- **無本地化基礎設施**：所有字串都是硬編碼在 Swift 原始碼中
- **無 .strings 檔案**：未建立任何語言資源檔案
- **無 .lproj 目錄**：沒有本地化資料夾結構
- **零 NSLocalizedString 使用**：未使用任何本地化 API

### 字串分佈

| 位置 | 字串類型 | 數量估計 | 檔案範例 |
|------|---------|---------|---------|
| CLI 指令 | 說明文字、旗標、幫助訊息 | ~500 | `*+CommanderMetadata.swift` |
| 錯誤訊息 | 錯誤描述、建議 | ~200 | `ErrorTypes.swift` |
| Mac 應用程式 | UI 標籤、按鈕、設定 | ~150 | `SettingsWindow.swift` |
| 代理輸出 | 系統訊息、狀態 | ~100 | `AgentMessages.swift` |
| 文件 | README、文件 | ~50 個檔案 | `docs/*.md` |

**總計**：~950+ 個需要本地化的字串

## 建議的實作策略

### 階段 1：文件本地化（立即，無程式碼變更）

✅ **已完成**：
- [x] 繁體中文 README (`README.zh-TW.md`)
- [x] Windows 平台分析文件（繁體中文）
- [x] 本地化策略文件（本文件）

🔄 **進行中**：
- [ ] 翻譯關鍵文件：
  - `ARCHITECTURE.md` → `ARCHITECTURE.zh-TW.md`
  - `docs/building.md` → `docs/building.zh-TW.md`
  - `docs/permissions.md` → `docs/permissions.zh-TW.md`
  - 指令文件：`docs/commands/*.md` → `docs/commands/*.zh-TW.md`

### 階段 2：基礎設施建立（需程式碼變更）

#### 2.1 建立本地化框架

```
Apps/CLI/Sources/Resources/
├── zh-TW.lproj/
│   ├── Localizable.strings      # 一般字串
│   ├── Commands.strings         # CLI 指令字串
│   └── Errors.strings           # 錯誤訊息
├── en.lproj/                    # 英文（基礎語言）
│   ├── Localizable.strings
│   ├── Commands.strings
│   └── Errors.strings
└── ja.lproj/                    # 未來：日文
    └── ...
```

#### 2.2 Swift 本地化 API 選擇

**選項 A：傳統 NSLocalizedString**（推薦用於 CLI）
```swift
// 之前
let message = "No displays available for capture."

// 之後
let message = NSLocalizedString(
    "error.no_displays",
    comment: "Error when no displays are available"
)
```

**選項 B：Swift 6.2+ String(localized:)**（推薦用於 Mac 應用程式）
```swift
// 之前
Text("Welcome to Peekaboo!")

// 之後
Text(String(localized: "welcome.title", 
            comment: "Welcome screen title"))
```

**選項 C：結構化字串目錄**（未來，Swift 5.9+）
```swift
// 使用 .xcstrings 檔案（Xcode 15+）
// 自動生成的類型安全 API
Text(Strings.welcome.title)
```

### 階段 3：CLI 本地化（優先）

#### 3.1 指令描述

**現有結構**（Commander 框架）：
```swift
extension WindowListCommand: Describable {
    static var commandName: String { "list" }
    static var commandHelp: String { 
        "List all windows across all applications" 
    }
    // ...
}
```

**本地化實作**：
```swift
extension WindowListCommand: Describable {
    static var commandName: String { "list" }
    static var commandHelp: String { 
        String(localized: "command.window.list.help",
               defaultValue: "List all windows across all applications",
               comment: "Window list command help text")
    }
    // ...
}
```

#### 3.2 錯誤訊息

**現有模式**：
```swift
enum CaptureError: LocalizedError {
    case noDisplaysAvailable
    
    var errorDescription: String? {
        switch self {
        case .noDisplaysAvailable:
            return "No displays available for capture."
        }
    }
}
```

**本地化實作**：
```swift
enum CaptureError: LocalizedError {
    case noDisplaysAvailable
    
    var errorDescription: String? {
        switch self {
        case .noDisplaysAvailable:
            return String(localized: "error.capture.no_displays",
                         defaultValue: "No displays available for capture.",
                         comment: "Error when no displays found")
        }
    }
}
```

### 階段 4：Mac 應用程式本地化

#### 4.1 SwiftUI 介面

**之前**：
```swift
struct SettingsWindow: View {
    var body: some View {
        TabView {
            GeneralSettingsView()
                .tabItem {
                    Label("General", systemImage: "gear")
                }
            PermissionsSettingsView()
                .tabItem {
                    Label("Permissions", systemImage: "lock.shield")
                }
        }
    }
}
```

**之後**：
```swift
struct SettingsWindow: View {
    var body: some View {
        TabView {
            GeneralSettingsView()
                .tabItem {
                    Label(String(localized: "settings.general"), 
                          systemImage: "gear")
                }
            PermissionsSettingsView()
                .tabItem {
                    Label(String(localized: "settings.permissions"), 
                          systemImage: "lock.shield")
                }
        }
    }
}
```

#### 4.2 XIB/Storyboard（如果使用）

在 Xcode 中：
1. 選擇專案 → Info 標籤
2. Localizations → 新增 "Chinese (Traditional)"
3. 為每個 .xib/.storyboard 啟用本地化
4. 翻譯生成的 .strings 檔案

### 階段 5：工具和自動化

#### 5.1 字串提取

**建立腳本**：`scripts/extract-strings.sh`
```bash
#!/bin/bash
# 從 Swift 檔案提取所有 NSLocalizedString 呼叫
find Apps/ Core/ -name "*.swift" -exec \
    genstrings -o Apps/CLI/Sources/Resources/en.lproj {} +
```

#### 5.2 翻譯驗證

**建立腳本**：`scripts/validate-translations.sh`
```bash
#!/bin/bash
# 檢查遺失的翻譯鍵
for lang in zh-TW ja; do
    comm -23 \
        <(grep -o '"[^"]*"' en.lproj/Localizable.strings | sort) \
        <(grep -o '"[^"]*"' ${lang}.lproj/Localizable.strings | sort)
done
```

#### 5.3 機器翻譯輔助（初稿）

使用 AI 進行初始翻譯，然後人工審查：
```bash
# 使用 Claude/GPT 翻譯 .strings 檔案
# 需要人工審查技術術語和上下文
```

## 繁體中文特定考量

### 字型和渲染
- ✅ macOS 內建優秀的中文字型支援（蘋方-繁）
- ✅ SwiftUI/AppKit 自動處理 CJK 字元
- ⚠️ 確保文字容器有足夠空間（中文可能較長）

### 術語標準化

| 英文術語 | 繁體中文 | 注意事項 |
|---------|---------|---------|
| Screen Capture | 螢幕擷取 | 不使用「截圖」|
| Accessibility | 輔助使用 | macOS 官方翻譯 |
| Automation | 自動化 | 標準用語 |
| Snapshot | 快照 | 技術用語 |
| CLI | 命令列介面 或 CLI | 保留縮寫或翻譯 |
| Agent | 代理 | AI 上下文中 |
| Window | 視窗 | 繁體寫法（非「窗口」）|
| Menu Bar | 選單列 | macOS 介面用語 |
| Dock | Dock | 保持原文（macOS 特有）|

### 文化適應
- **日期格式**：民國年 vs. 西元年（建議使用系統設定）
- **排版**：支援直排（目前 Peekaboo 不需要）
- **標點符號**：使用全形標點（，。！？）

## 測試策略

### 測試檢查清單
- [ ] 系統語言設為繁體中文時，應用程式顯示中文
- [ ] 所有 CLI 指令幫助文字正確本地化
- [ ] 錯誤訊息以正確語言顯示
- [ ] UI 元素不會因文字長度而截斷
- [ ] 混合語言環境正確退回到預設語言
- [ ] JSON 輸出保持英文（API 相容性）

### 測試指令
```bash
# 設定系統語言
defaults write NSGlobalDomain AppleLanguages -array "zh-TW"

# 測試 CLI
peekaboo --help
peekaboo see --help
peekaboo capture invalid-param  # 應顯示中文錯誤

# 恢復英文
defaults write NSGlobalDomain AppleLanguages -array "en-US"
```

## 維護工作流程

### 新增新字串時
1. 使用 `String(localized:)` 而非硬編碼
2. 提供清晰的註解說明上下文
3. 執行 `scripts/extract-strings.sh`
4. 更新所有語言的 .strings 檔案
5. 在 PR 中標記需要翻譯

### 發布前檢查
1. 執行 `scripts/validate-translations.sh`
2. 手動審查新增的翻譯
3. 測試所有支援的語言
4. 更新 CHANGELOG（多語言）

## 成本估計

### 初始實作（繁體中文）

| 任務 | 工作量 | 優先級 |
|------|-------|--------|
| 文件翻譯 | 2-3 週 | ✅ 高 |
| 建立本地化基礎設施 | 1 週 | 高 |
| CLI 字串本地化 | 2-3 週 | 高 |
| Mac 應用程式字串本地化 | 1-2 週 | 中 |
| 測試和品質保證 | 1 週 | 高 |
| **總計** | **7-10 週** | - |

### 維護（每個發布週期）
- 新字串翻譯：2-4 小時
- 測試：1-2 小時
- **總計**：每次發布 ~4-6 小時

## 建議行動計劃

### 立即（本 PR）
✅ 1. 新增繁體中文 README
✅ 2. 建立此策略文件
✅ 3. 新增 Windows 平台分析文件

### 短期（下 2-4 週）
📋 4. 翻譯核心文件（建置、權限、架構）
📋 5. 翻譯 CLI 指令文件
📋 6. 在主 README 中新增語言選擇連結

### 中期（1-3 個月）
⏳ 7. 建立 .lproj 結構和 .strings 檔案
⏳ 8. 重構 CLI 錯誤訊息以使用本地化
⏳ 9. 重構 CLI 指令描述以使用本地化
⏳ 10. 新增本地化測試

### 長期（3-6 個月）
🔮 11. 完整的 Mac 應用程式本地化
🔮 12. 新增其他語言（日文、韓文、簡體中文）
🔮 13. 建立翻譯貢獻指南
🔮 14. 自動化翻譯工作流程

## 替代方法

### 最小化方法（僅文件）
**優點**：
- 零程式碼變更
- 快速實作
- 無維護負擔

**缺點**：
- 應用程式本身仍為英文
- 使用者體驗不一致

### 社群驅動翻譯
**使用工具**：
- Crowdin
- Weblate
- POEditor

**優點**：
- 社群參與
- 支援多語言
- 版本控制整合

**缺點**：
- 需要設定和維護
- 品質控制挑戰

## 結論

Peekaboo 的繁體中文本地化是可行的，但需要有組織的方法：

1. **立即價值**：文件翻譯（無程式碼變更）
2. **短期投資**：CLI 本地化（中等工作量）
3. **長期目標**：完整應用程式本地化（顯著投資）

**建議起點**：完成文件本地化，然後根據社群回饋和需求評估進一步的工作。
