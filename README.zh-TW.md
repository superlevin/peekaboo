# Peekaboo 🫣 - 會看螢幕、會點擊的 Mac 自動化工具

![Peekaboo Banner](assets/peekaboo.png)

[![npm package](https://img.shields.io/badge/npm_package-3.0.0--beta1-brightgreen?logo=npm&logoColor=white&style=flat-square)](https://www.npmjs.com/package/@steipete/peekaboo)
[![授權: MIT](https://img.shields.io/badge/License-MIT-ffd60a?style=flat-square)](https://opensource.org/licenses/MIT)
[![macOS 15.0+ (Sequoia)](https://img.shields.io/badge/macOS-15.0%2B_(Sequoia)-0078d7?logo=apple&logoColor=white&style=flat-square)](https://www.apple.com/macos/)
[![Swift 6.2](https://img.shields.io/badge/Swift-6.2-F05138?logo=swift&logoColor=white&style=flat-square)](https://swift.org/)
[![node >=22](https://img.shields.io/badge/node-%3E%3D22.0.0-2ea44f?logo=node.js&logoColor=white&style=flat-square)](https://nodejs.org/)
[![下載 macOS](https://img.shields.io/badge/Download-macOS-000000?logo=apple&logoColor=white&style=flat-square)](https://github.com/steipete/peekaboo/releases/latest)
[![Homebrew](https://img.shields.io/badge/Homebrew-steipete%2Ftap-b28f62?logo=homebrew&logoColor=white&style=flat-square)](https://github.com/steipete/homebrew-tap)
[![Ask DeepWiki](https://img.shields.io/badge/Ask-DeepWiki-0088cc?style=flat-square)](https://deepwiki.com/steipete/peekaboo)

[English](README.md) | 繁體中文

Peekaboo 為 macOS 帶來高精準度螢幕擷取、AI 分析和完整的 GUI 自動化功能。第 3 版新增了原生代理流程和跨 CLI 及 MCP 伺服器的多螢幕自動化。

> 注意：v3 目前為測試版（3.0.0-beta4），已知有一些問題；詳情請參閱變更日誌。

## 功能特色

- 像素級精準擷取（視窗、螢幕、選單列），可選 Retina 2x 縮放
- 自然語言代理，可串接 Peekaboo 工具（see、click、type、scroll、hotkey、menu、window、app、dock、space）
- 選單與選單列探索，提供結構化 JSON；無需點擊
- 多供應商 AI：GPT-5.1 系列、Claude 4.x、Grok 4-fast（視覺）、Gemini 2.5 和本地 Ollama 模型
- 支援 Claude Desktop 和 Cursor 的 MCP 伺服器，以及原生 CLI；兩者使用相同工具
- 可配置、可測試的工作流程，具有可重現的會話和嚴格類型
- 需要 macOS 螢幕錄製 + 輔助使用權限（請參閱 [docs/permissions.md](docs/permissions.md)）

## 安裝

- macOS 應用程式 + CLI（Homebrew）：
  ```bash
  brew install steipete/tap/peekaboo
  ```
- MCP 伺服器（Node 22+，無需全域安裝）：
  ```bash
  npx -y @steipete/peekaboo
  ```

## 快速開始

```bash
# 以 Retina 比例擷取全螢幕並儲存到桌面
peekaboo image --mode screen --retina --path ~/Desktop/screen.png

# 透過標籤點擊按鈕（一次完成擷取、解析和點擊）
peekaboo see --app Safari --json-output | jq -r '.data.snapshot_id' | read SNAPSHOT
peekaboo click --on "Reload this page" --snapshot "$SNAPSHOT"

# 執行自然語言自動化
peekaboo "開啟備忘錄並建立包含三個項目的待辦事項清單"

# 作為 MCP 伺服器執行（Claude/Cursor）
npx -y @steipete/peekaboo

# Claude Desktop 最小配置片段（開發者 → 編輯配置）：
# {
#   "mcpServers": {
#     "peekaboo": {
#       "command": "npx",
#       "args": ["-y", "@steipete/peekaboo"],
#       "env": {
#         "PEEKABOO_AI_PROVIDERS": "openai/gpt-5.1,anthropic/claude-opus-4"
#       }
#     }
#   }
# }
```

## 指令參考

| 指令 | 主要參數 / 子指令 | 功能說明 |
| --- | --- | --- |
| [see](docs/commands/see.md) | `--app`、`--mode screen/window`、`--retina`、`--json-output` | 擷取並標註 UI，回傳快照 + 元素 ID |
| [click](docs/commands/click.md) | `--on <id/query>`、`--snapshot`、`--wait`、座標 | 透過元素 ID、標籤或座標點擊 |
| [type](docs/commands/type.md) | `--text`、`--clear`、`--delay-ms` | 輸入文字，可調整輸入速度 |
| [press](docs/commands/press.md) | 按鍵名稱、`--repeat` | 特殊按鍵和序列 |
| [hotkey](docs/commands/hotkey.md) | 組合鍵如 `cmd,shift,t` | 修飾鍵組合（cmd/ctrl/alt/shift）|
| [scroll](docs/commands/scroll.md) | `--on <id>`、`--direction up/down`、`--ticks` | 捲動檢視或元素 |
| [swipe](docs/commands/swipe.md) | `--from/--to`、`--duration`、`--steps` | 平滑手勢式拖曳 |
| [drag](docs/commands/drag.md) | `--from/--to`、修飾鍵、Dock/垃圾桶目標 | 在元素/座標之間拖放 |
| [move](docs/commands/move.md) | `--to <id/coords>`、`--screen-index` | 移動游標但不點擊 |
| [window](docs/commands/window.md) | `list`、`move`、`resize`、`focus`、`set-bounds` | 移動/調整大小/聚焦視窗和工作區 |
| [app](docs/commands/app.md) | `launch`、`quit`、`relaunch`、`switch`、`list` | 啟動、結束、重啟、切換應用程式 |
| [space](docs/commands/space.md) | `list`、`switch`、`move-window` | 列出或切換 macOS 工作區 |
| [menu](docs/commands/menu.md) | `list`、`list-all`、`click`、`click-extra` | 列出/點擊應用程式選單和額外項目 |
| [menubar](docs/commands/menubar.md) | `list`、`click` | 透過名稱/索引定位狀態列項目 |
| [dock](docs/commands/dock.md) | `launch`、`right-click`、`hide`、`show`、`list` | 與 Dock 項目互動 |
| [dialog](docs/commands/dialog.md) | `list`、`click`、`input`、`file`、`dismiss` | 操作系統對話框（開啟/儲存等）|
| [image](docs/commands/image.md) | `--mode screen/window/menu`、`--retina`、`--analyze` | 截圖螢幕/視窗/選單列（+分析）|
| [list](docs/commands/list.md) | `apps`、`windows`、`screens`、`menubar`、`permissions` | 列舉應用程式、視窗、螢幕、權限 |
| [tools](docs/commands/tools.md) | `--verbose`、`--json-output`、`--no-sort` | 檢查原生 Peekaboo 工具 |
| [config](docs/commands/config.md) | `init`、`show`、`add`、`login`、`models` | 管理憑證/供應商/設定 |
| [permissions](docs/commands/permissions.md) | `status`、`grant` | 檢查/授予所需的 macOS 權限 |
| [run](docs/commands/run.md) | `.peekaboo.json`、`--output`、`--no-fail-fast` | 執行 `.peekaboo.json` 自動化腳本 |
| [sleep](docs/commands/sleep.md) | `--duration` (ms) | 步驟間的毫秒延遲 |
| [clean](docs/commands/clean.md) | `--all-snapshots`、`--older-than`、`--snapshot` | 清理快照和快取 |
| [agent](docs/commands/agent.md) | `--model`、`--dry-run`、`--resume`、`--max-steps`、audio | 自然語言多步驟自動化 |
| [mcp](docs/commands/mcp.md) | `serve`（預設）| 作為 MCP 伺服器執行 Peekaboo |

## 模型和供應商

- OpenAI：GPT-5.1（預設）和 GPT-4.1/4o 視覺
- Anthropic：Claude 4.x
- xAI：Grok 4-fast 推理 + 視覺
- Google：Gemini 2.5（pro/flash）
- 本地：Ollama（llama3.3、llava 等）

透過 `PEEKABOO_AI_PROVIDERS` 或 `peekaboo config add` 設定供應商。

## 了解更多

- 指令參考：[docs/commands/](docs/commands/)
- 架構：[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
- 從原始碼建置：[docs/building.md](docs/building.md)
- 測試指南：[docs/testing/tools.md](docs/testing/tools.md)
- MCP 設定：[docs/commands/mcp.md](docs/commands/mcp.md)
- 權限：[docs/permissions.md](docs/permissions.md)
- Ollama/本地模型：[docs/ollama.md](docs/ollama.md)
- 代理聊天迴圈：[docs/agent-chat.md](docs/agent-chat.md)
- 服務 API 參考：[docs/service-api-reference.md](docs/service-api-reference.md)

## 開發基礎

- 需求：macOS 15+、Xcode 16+/Swift 6.2。Node 22+ 僅在執行 pnpm 文件/建置輔助腳本時需要（核心 CLI/應用程式/MCP 僅使用 Swift）
- 安裝依賴：`pnpm install` 然後 `pnpm run build:cli` 或 `pnpm run test:safe`
- 檢查/格式化：`pnpm run lint && pnpm run format`

## 授權

MIT
