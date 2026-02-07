# Windows 平台相容性分析

## 執行摘要

**Peekaboo 目前無法在 Windows 上運行**，因為它深度依賴 macOS 專屬的系統 API 和框架。本文件分析了移植到 Windows 的技術挑戰，並提供可能的替代方案。

## 關鍵的 macOS 依賴

### 1. 螢幕擷取技術

| macOS API | 用途 | Windows 等效 |
|-----------|------|--------------|
| `ScreenCaptureKit` | 高效能螢幕錄製和視窗擷取 | Windows Graphics Capture API (Windows 10 1803+) |
| `CGWindowListCreateImage` | 視窗和螢幕截圖 | BitBlt/PrintWindow (Win32) 或 Graphics Capture |
| `CGDisplayCreateImage` | 顯示器擷取 | Desktop Duplication API |

**影響**：需要完全重寫螢幕擷取層
- 檔案：`Core/PeekabooCore/Sources/PeekabooCapture/*.swift`
- 程式碼量：~2000 行

### 2. 輔助功能和自動化

| macOS API | 用途 | Windows 等效 |
|-----------|------|--------------|
| `AXUIElement` | UI 元素檢查和操作 | UI Automation (UIA) |
| `CGEvent` | 滑鼠和鍵盤事件合成 | SendInput/mouse_event/keybd_event |
| `AXObserver` | 監聽輔助功能通知 | UI Automation 事件處理器 |
| `NSAccessibility` | 輔助功能協定 | IAccessible/IUIAutomation |

**影響**：需要完全重寫自動化引擎
- 檔案：`AXorcist/*.swift`、`Core/PeekabooAutomation/*.swift`
- 程式碼量：~5000+ 行
- 複雜度：非常高（不同的 API 設計理念）

### 3. 應用程式和視窗管理

| macOS API | 用途 | Windows 等效 |
|-----------|------|--------------|
| `NSRunningApplication` | 應用程式生命週期管理 | Process API + WMI |
| `CGWindowListCopyWindowInfo` | 列舉視窗 | EnumWindows/FindWindow |
| `Spaces API（私有）` | macOS 工作區管理 | Virtual Desktops（Windows 10+）|
| `NSWorkspace` | 應用程式啟動和管理 | ShellExecute/CreateProcess |

**影響**：中等重寫
- 檔案：`Core/PeekabooCore/Sources/PeekabooAppManagement/*.swift`
- 程式碼量：~1500 行

### 4. 作業系統整合

| 功能 | macOS 實作 | Windows 挑戰 |
|------|-----------|-------------|
| 選單列擷取 | 直接存取選單列 | 無等效概念（工作列 ≠ 選單列）|
| Dock 互動 | Dock API | 工作列（不同的 API）|
| 系統對話框 | NSOpenPanel/NSSavePanel | Common Dialogs（不同架構）|
| 權限系統 | TCC（透明度、同意與控制）| UAC + 隱私設定（不同模型）|

## 技術障礙

### 1. 程式語言和執行環境

- **當前**：100% Swift，使用 Swift Package Manager
- **Windows 挑戰**：
  - Swift on Windows 仍在發展中（官方支援有限）
  - 許多 macOS 框架在 Windows 上不存在
  - 需要移植到 C#/.NET 或使用 C++ 與 Win32 API

### 2. 依賴的 macOS 框架（167 個引用）

```swift
import AppKit          // UI 和應用程式框架
import Cocoa           // macOS UI 框架
import ScreenCaptureKit // 螢幕擷取
import ApplicationServices // 輔助功能和事件
import CoreGraphics    // 圖形和視窗 API
import IOKit           // 硬體互動
```

這些在 Windows 上**都不存在**。

### 3. 架構差異

| 面向 | macOS | Windows |
|------|-------|---------|
| UI 框架 | AppKit/SwiftUI | WPF/WinUI/Win32 |
| 輔助功能 | AX API（樹狀結構導向）| UIA（模式導向）|
| 權限 | 細粒度應用程式權限 | UAC + 應用程式功能 |
| 封裝 | .app 套件 | .exe + DLL |
| 分發 | Homebrew、.dmg、App Store | MSI、MSIX、Store |

## 移植工作量估計

### 完全移植（原生 Windows 應用程式）

| 元件 | 工作量 | 複雜度 |
|------|-------|--------|
| 螢幕擷取 | 4-6 週 | 高 |
| 自動化引擎 | 8-12 週 | 非常高 |
| 應用程式管理 | 3-4 週 | 中 |
| CLI 介面 | 2-3 週 | 中 |
| UI（如果移植 Mac 應用程式）| 6-8 週 | 高 |
| 測試與除錯 | 4-6 週 | 高 |
| **總計** | **27-39 週**（~6-9 個月）| - |

### 部分移植（僅 CLI，基本功能）

- 螢幕擷取 + 點擊/輸入：~8-12 週
- 省略：選單列、Dock、工作區、進階視窗管理

## 替代方案

### 方案 1：跨平台抽象層（建議用於概念驗證）

```
┌─────────────────────────────────────┐
│     Peekaboo Core Logic             │
│     (Protocol-based)                │
├─────────────────────────────────────┤
│  Platform Abstraction Layer         │
├───────────────┬─────────────────────┤
│  macOS 實作   │   Windows 實作      │
│  (當前)       │   (新增)            │
└───────────────┴─────────────────────┘
```

**優點**：
- 共享核心邏輯
- 可以逐步開發

**缺點**：
- 大量的介面設計工作
- 功能對等性挑戰（macOS 有 Windows 沒有的功能）

### 方案 2：遠端協定（跨平台相容）

Peekaboo 作為 macOS 伺服器，Windows 客戶端遠端連接：

```
┌──────────────┐      網路      ┌──────────────┐
│   Windows    │ ◄──────────► │   macOS      │
│   客戶端      │   (HTTP/WS)    │   Peekaboo   │
│   (控制)     │                │   伺服器      │
└──────────────┘                └──────────────┘
```

**優點**：
- 無需移植核心功能
- Windows 使用者可以遠端控制 Mac
- 利於雲端部署

**缺點**：
- 需要 macOS 機器
- 網路延遲
- 安全性考量

### 方案 3：雙重開發（獨立專案）

建立獨立的 Windows 版本：
- 不同的程式碼庫
- 不同的技術棧（C#/.NET）
- 相似的使用者介面和功能

**優點**：
- 最佳化平台專屬功能
- 無跨平台妥協

**缺點**：
- 雙倍的開發和維護成本
- 功能不對等風險

### 方案 4：專注於 MCP 協定（推薦短期）

Peekaboo 保持為 macOS 專用，但：
- 強化 MCP 伺服器功能
- Windows 使用者透過 MCP 客戶端連接
- 支援遠端 Mac 自動化

**優點**：
- 最小的額外工作
- 充分利用現有的 MCP 生態系統
- 透過 Claude Desktop/Cursor 等工具已有跨平台介面

**缺點**：
- Windows 使用者仍需存取 macOS 機器

## Windows 專屬工具（替代方案）

如果需要 Windows 自動化，可考慮這些成熟工具：

| 工具 | 描述 | 授權 |
|------|------|------|
| **Playwright** | 瀏覽器自動化（跨平台）| Apache 2.0 |
| **PyAutoGUI** | Python 圖形自動化庫 | BSD |
| **AutoHotkey** | Windows 腳本和自動化 | GPL |
| **Sikuli** | 視覺化自動化（螢幕圖像）| MIT |
| **UI Automation** | Microsoft 官方 API（.NET）| 免費 |
| **WinAppDriver** | Windows 應用程式 UI 測試 | MIT |

## 建議

### 短期（0-6 個月）
1. **維持 macOS 專用** - 專注於核心功能和穩定性
2. **強化 MCP 支援** - 讓 Windows 使用者可以遠端使用
3. **記錄遠端設定** - 提供 Windows → Mac 的設定指南
4. **在 README 中明確說明** - 避免期望錯配

### 中期（6-12 個月）
1. **評估市場需求** - Windows 使用者有多少？
2. **探索合作夥伴** - 是否有 Windows 自動化專家有興趣？
3. **設計抽象層** - 如果決定移植，先設計介面

### 長期（12+ 個月）
1. **決策點**：完全移植 vs. 混合方法 vs. 維持 macOS 專用
2. 如果移植，考慮：
   - 尋找贊助或資金
   - 組建 Windows 專業團隊
   - 採用階段性方法（先 CLI，再 GUI）

## 結論

**直接答案**：Peekaboo **目前無法**在 Windows 上運行，移植需要顯著的工程投資（6-9 個月全職開發）。

**推薦路徑**：
1. 保持 macOS 專用，強化 MCP 伺服器
2. 為 Windows 使用者提供遠端連接選項
3. 根據需求評估未來的移植工作

**關鍵考量**：
- macOS 和 Windows 的自動化 API 根本上不同
- 完全功能對等極具挑戰性
- 維持兩個平台需要持續的資源投入
