# Agent 開發指南（繁體中文版）

[English AGENT.md](AGENT.md) | [繁體中文 README](README.zh-TW.md)

本文件是 `AGENT.md` 的中文參考版本。**原始 `AGENT.md` 保留不變**；程式、命令列參數、環境變數及工具名稱均不翻譯。

## 開發環境

專案使用 [devenv](https://devenv.sh/) 與 Nix 建立可重現的開發環境。

## 常用指令

| 指令 | 用途 |
| --- | --- |
| `devenv shell` | 進入 devenv 開發 Shell |
| `devenv test` | 執行環境測試（目前包括 Git 版本檢查等） |
| `go build` | 編譯目前套件；各階段也可用單檔方式執行 |
| `go test ./...` | 執行 Go 測試（如有測試檔案） |
| `go test <package>` | 執行指定套件的測試 |
| `go mod tidy` | 整理並下載 Go 模組相依套件 |
| `hello` | 執行 devenv 定義的問候腳本 |

> 原始英文文件列有 `go run main.go`，但目前儲存庫**沒有 `main.go`**；請使用下方各階段的檔名。

### 各階段的啟動指令

- `go run chat.go`：Claude 基礎對話。
- `go run read.go`：加入檔案讀取。
- `go run list_files.go`：加入目錄瀏覽與檔案讀取。
- `go run bash_tool.go`：加入 Bash 指令執行。
- `go run edit_tool.go`：加入讀取、列出、編輯檔案及 Bash。
- `go run code_search_tool.go`：加入 ripgrep 程式碼搜尋；此版本未註冊 `edit_file`。

### 詳細紀錄（Verbose Logging）

所有 Go 範例都有 `--verbose` 旗標，可列印較詳細的執行資訊：

```bash
go run chat.go --verbose
go run read.go --verbose
go run edit_tool.go --verbose
go run code_search_tool.go --verbose
```

詳細紀錄包含 API 請求、模型資訊、工具名稱與參數、檔案操作、Shell 指令、對話處理及錯誤細節。詳細紀錄使用含時間戳記與原始碼位置的日誌格式，可能包含敏感路徑或輸入內容，分享日誌前請先檢查。

## 程式架構

- **Environment**：以 Nix／devenv 建立開發環境。
- **Shell**：包含 Git、Go 工具鏈及自訂問候腳本。
- **Application**：透過終端機與 Claude 對話的 Go 程式。
- **Agent Loop**：接收訊息、呼叫模型、處理 Tool Use，再回傳 Tool Result，直到模型完成回答。
- **Tool Registry**：`ToolDefinition` 包含工具名稱、說明、輸入 Schema 與執行函式。

## 程式碼風格

- `devenv.nix` 設定遵循 Nix 慣例。
- 使用標準 Git 分支、提交與檢閱流程。
- 維持環境設定可重現。
- 避免為翻譯而更動工具名稱、指令、JSON 欄位及 API 行為。

## 問題排除

### API 請求失敗

使用 `--verbose` 檢查驗證失敗、API 額度或速率限制等錯誤，並確認環境變數 `ANTHROPIC_API_KEY` 已設定。

### 工具操作失敗

從日誌確認失敗的工具、傳入參數及錯誤內容。常見原因包括檔案不存在、存取權限不足及缺少系統指令。

### 回答或執行流程異常

透過詳細紀錄對照模型回應的內容區塊、工具呼叫與後續訊息。這類紀錄是觀察執行流程的資料，不代表可以完整查看模型內部推理。

### 效能問題

檢查 API 請求耗時、回應量，以及工具是否讀取過多檔案或產生過量輸出。

### 環境問題

- 先確認 `ANTHROPIC_API_KEY`。
- 如使用 devenv，重新進入 `devenv shell`。
- 執行 `go mod tidy` 整理套件。
- 第四階段起需要 `bash`；第六階段還需要 `rg`，部分列檔實作依賴 `find`。Windows 可使用 WSL 或相容的 Unix 環境。

## 安全與操作備註

- 終端機按 `Ctrl+C` 可結束聊天程式。
- `bash` 與檔案編輯工具是教學用途，**沒有完善的沙箱與操作確認機制**；不要在重要檔案或機密資料所在目錄直接試驗。
- 請勿提交真實 API Key。
- 原始英文文件提到正常模式主要顯示必要輸出；實際輸出位置仍依不同 Go 範例中的 `log` 設定而定。

更多逐步操作與架構圖，請閱讀 [繁體中文工作坊](README.zh-TW.md)。
