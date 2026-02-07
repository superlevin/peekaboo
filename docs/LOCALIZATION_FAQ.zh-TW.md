# 繁體中文化與 Windows 平台評估 - 常見問題解答 (FAQ)

## 關於繁體中文化

### Q1: 我可以在哪裡找到繁體中文文件？
**A:** 所有繁體中文文件都已創建：
- 主要說明：[README.zh-TW.md](../README.zh-TW.md)
- 本地化策略：[LOCALIZATION_STRATEGY.zh-TW.md](LOCALIZATION_STRATEGY.zh-TW.md)
- Windows 平台分析：[WINDOWS_PLATFORM_ANALYSIS.zh-TW.md](WINDOWS_PLATFORM_ANALYSIS.zh-TW.md)
- 實作摘要：[LOCALIZATION_IMPLEMENTATION_SUMMARY.md](LOCALIZATION_IMPLEMENTATION_SUMMARY.md)

在主 README 頂部有語言選擇連結。

### Q2: 應用程式本身會顯示繁體中文嗎？
**A:** 目前還不會。此 PR 僅包含文件翻譯。應用程式介面（CLI 和 Mac 應用程式）仍為英文，因為所有 950+ 個字串都是硬編碼在原始碼中。

### Q3: 如何讓應用程式顯示繁體中文？
**A:** 需要實作程式碼層級的本地化：
1. 建立 `.lproj` 目錄結構（~1 週）
2. 重構 CLI 字串以使用 `String(localized:)`（~2-3 週）
3. 重構 Mac 應用程式 UI 字串（~1-2 週）
4. 測試和品質保證（~1 週）

**總計**：約 7-10 週的開發工作。完整計劃請參閱 [LOCALIZATION_STRATEGY.md](LOCALIZATION_STRATEGY.md)。

### Q4: 為什麼只做文件翻譯？
**A:** 文件翻譯提供立即價值且無需程式碼變更：
- ✅ 零風險（不影響現有功能）
- ✅ 快速完成（2-3 週 vs 7-10 週）
- ✅ 讓中文使用者能夠理解和使用 Peekaboo
- ✅ 為未來的程式碼本地化奠定基礎

程式碼本地化可以根據社群需求和回饋來決定是否實作。

### Q5: 未來會支援哪些語言？
**A:** 建議優先順序：
1. ✅ 繁體中文（已完成文件）
2. 日文（類似工作量）
3. 韓文（類似工作量）
4. 簡體中文（可重用大部分繁體中文工作）

## 關於 Windows 平台

### Q6: Peekaboo 可以在 Windows 上運行嗎？
**A:** **不行**，目前無法在 Windows 上運行。Peekaboo 深度依賴 macOS 專屬的系統 API 和框架。

### Q7: 為什麼不能在 Windows 上運行？
**A:** 主要技術障礙：
- **167 個 macOS 框架引用**：ScreenCaptureKit、AXUIElement、AppKit、CoreGraphics 等
- **無 Windows 對等物**：這些 API 在 Windows 上不存在
- **架構根本不同**：macOS 和 Windows 的自動化 API 設計理念完全不同
- **語言限制**：Swift on Windows 仍在發展中

詳細技術分析請參閱 [WINDOWS_PLATFORM_ANALYSIS.zh-TW.md](WINDOWS_PLATFORM_ANALYSIS.zh-TW.md)。

### Q8: 移植到 Windows 需要多少工作量？
**A:** 完整移植估計：
| 元件 | 工作量 |
|------|-------|
| 螢幕擷取 | 4-6 週 |
| 自動化引擎 | 8-12 週 |
| 應用程式管理 | 3-4 週 |
| CLI 介面 | 2-3 週 |
| UI（如果移植）| 6-8 週 |
| 測試與除錯 | 4-6 週 |
| **總計** | **27-39 週**（約 6-9 個月）|

這還不包括持續維護兩個平台的成本。

### Q9: 有替代方案嗎？
**A:** 是的，推薦方法：

1. **保持 macOS 專用**（推薦）
   - 強化 MCP 伺服器功能
   - Windows 使用者可以遠端連接到 Mac
   - 利用現有的 Claude Desktop/Cursor 等跨平台工具

2. **遠端協定**
   - Peekaboo 作為 macOS 伺服器
   - Windows 客戶端透過網路連接
   - 適合雲端部署

3. **使用 Windows 原生工具**
   - PyAutoGUI、AutoHotkey、UI Automation、WinAppDriver
   - 專為 Windows 設計的成熟工具

### Q10: Windows 使用者該怎麼辦？
**A:** 三個選項：
1. **存取 Mac 機器**：透過 MCP 伺服器遠端使用 Peekaboo
2. **虛擬機**：在 Windows 上執行 macOS 虛擬機（受限於授權和效能）
3. **替代工具**：使用專為 Windows 設計的自動化工具

## 關於本地化策略

### Q11: 文件中提到的 5 個階段是什麼？
**A:**
- **階段 1**：文件本地化（✅ 已完成 - 本 PR）
- **階段 2**：建立基礎設施（~1 週）
- **階段 3**：CLI 本地化（~2-3 週）
- **階段 4**：Mac 應用程式本地化（~1-2 週）
- **階段 5**：工具和自動化（~1 週）

### Q12: 需要翻譯多少個字串？
**A:** 估計約 950+ 個字串：
- CLI 指令：~500 個
- 錯誤訊息：~200 個
- Mac 應用程式 UI：~150 個
- 代理輸出：~100 個

### Q13: 如何維護翻譯？
**A:** 建議的工作流程：
1. 新增字串時使用 `String(localized:)`
2. 執行字串提取腳本
3. 更新所有語言的 .strings 檔案
4. 在 PR 中標記需要翻譯
5. 每次發布前驗證翻譯完整性

每次發布週期約需 4-6 小時維護時間。

## 技術問題

### Q14: 為什麼所有字串都是硬編碼的？
**A:** Peekaboo 最初設計為英文單一語言應用程式。在建立時沒有預見國際化需求，因此所有字串直接寫在程式碼中。這在初期開發時簡化了工作，但現在需要重構以支援多語言。

### Q15: 使用哪種本地化 API？
**A:** 建議：
- **CLI**：`String(localized:defaultValue:comment:)` (Swift 6.2+)
- **Mac 應用程式**：SwiftUI 的 `String(localized:)` 或 `.xcstrings` 目錄
- **錯誤訊息**：在 `LocalizedError` 實作中使用 `String(localized:)`

### Q16: 可以使用機器翻譯嗎？
**A:** 可以作為初稿：
1. 使用 AI（Claude/GPT）生成初始翻譯
2. **必須**進行人工審查
3. 特別注意技術術語和上下文
4. 測試實際使用情境

機器翻譯可以節省時間，但品質控制至關重要。

## 貢獻和維護

### Q17: 我可以幫忙翻譯嗎？
**A:** 當然！如果專案決定實作程式碼層級的本地化，歡迎貢獻：
1. 參閱 [LOCALIZATION_STRATEGY.md](LOCALIZATION_STRATEGY.md) 了解架構
2. 遵循術語標準化指南
3. 測試你的翻譯
4. 提交 PR 並說明變更

### Q18: 翻譯品質如何保證？
**A:** 多層次方法：
- 使用標準化術語表
- 人工審查所有翻譯
- 在實際使用情境中測試
- 社群回饋和修正
- 定期更新和改進

### Q19: 如果發現翻譯錯誤怎麼辦？
**A:** 請：
1. 在 GitHub 上開 issue
2. 說明哪個檔案和行
3. 提供正確的翻譯
4. 解釋為什麼目前的翻譯不正確

我們會盡快審查和更新。

## 未來計劃

### Q20: 接下來會做什麼？
**A:** 建議優先順序：
1. 收集社群回饋
2. 評估對程式碼本地化的需求
3. 如果需求高，實作階段 2-5
4. 持續翻譯新的文件

### Q21: 什麼時候會支援其他語言？
**A:** 取決於：
- 社群需求
- 可用資源
- 維護能力

繁體中文是第一個本地化語言，為未來的語言建立了模板和流程。

### Q22: 有翻譯貢獻指南嗎？
**A:** 目前文件翻譯已完成。如果未來實作程式碼本地化，將會建立：
- 翻譯者指南
- 術語表
- 樣式指南
- 測試檢查清單

## 更多資訊

**完整文件**：
- 英文：[LOCALIZATION_STRATEGY.md](LOCALIZATION_STRATEGY.md)
- 繁體中文：[LOCALIZATION_STRATEGY.zh-TW.md](LOCALIZATION_STRATEGY.zh-TW.md)
- Windows 分析（英文）：[WINDOWS_PLATFORM_ANALYSIS.md](WINDOWS_PLATFORM_ANALYSIS.md)
- Windows 分析（繁體中文）：[WINDOWS_PLATFORM_ANALYSIS.zh-TW.md](WINDOWS_PLATFORM_ANALYSIS.zh-TW.md)
- 實作摘要：[LOCALIZATION_IMPLEMENTATION_SUMMARY.md](LOCALIZATION_IMPLEMENTATION_SUMMARY.md)

**問題或建議**？請在 GitHub 上開 issue！
