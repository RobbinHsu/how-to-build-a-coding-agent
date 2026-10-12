# Agent 指示

## 開發環境
這個專案使用 [devenv](https://devenv.sh/) 搭配 Nix，建立可重現的開發環境。

## 指令
- `devenv shell` - 進入開發 Shell
- `devenv test` - 執行測試（目前執行 Git 版本檢查）
- `go build` - 建置 Go 專案
- `go run main.go` - 執行聊天應用程式
- `go test ./...` - 執行所有 Go 測試
- `go test <package>` - 執行指定套件的測試
- `go mod tidy` - 下載相依套件
- `hello` - 從開發環境執行自訂問候腳本

### 應用程式指令
- `go run chat.go` - 與 Claude 進行簡單聊天
- `go run read.go` - 具備檔案讀取能力的聊天程式
- `go run list_files.go` - 具備列出與讀取檔案能力的聊天程式
- `go run bash_tool.go` - 具備檔案操作與執行 bash 指令能力的聊天程式
- `go run edit_tool.go` - 具備完整檔案操作能力（讀取、列出、編輯、bash）的聊天程式

### 詳細日誌
所有 Go 應用程式都支援 `--verbose` 旗標，以提供詳細的執行日誌：
- `go run chat.go --verbose` - 啟用詳細日誌進行除錯
- `go run read.go --verbose` - 查看詳細的工具執行與 API 呼叫
- `go run edit_tool.go --verbose` - 除錯檔案操作與工具使用情況

## 架構
- **環境**：使用 devenv 的 Nix 開發環境
- **Shell**：包含 Git、Go 工具鏈及自訂問候腳本
- **結構**：透過 Anthropic API 與 Claude 連線的終端機聊天應用程式

## 程式碼風格指南
- `devenv.nix` 設定遵循 Nix 慣例
- 使用標準 Git 工作流程
- 開發環境設定應可重現

## 疑難排解

### 詳細日誌
在聊天應用程式中除錯時，使用 `--verbose` 旗標取得詳細的執行日誌：

```bash
go run edit_tool.go --verbose
```

**詳細日誌顯示的內容：**
- 對 Claude 的 API 呼叫（模型、時間、成功／失敗）
- 工具執行詳細資訊（呼叫哪些工具、輸入參數、結果）
- 檔案操作（讀取、寫入、列出檔案及其大小／數量）
- Bash 指令執行（執行的指令、輸出、錯誤）
- 對話流程（訊息處理、內容區塊）
- 含有堆疊追蹤的錯誤詳細資訊

**日誌輸出位置：**
- **詳細模式**：包含時間戳記與檔案位置的詳細日誌會寫入 stderr
- **一般模式**：只有必要的輸出會寫入 stdout

**常見疑難排解情境：**
- **API 失敗**：檢查詳細日誌中的驗證錯誤或速率限制
- **工具失敗**：查看究竟是哪個工具失敗，以及原因（找不到檔案、權限錯誤）
- **非預期回應**：查看完整對話流程與 Claude 的推理過程
- **效能問題**：查看 API 呼叫耗時及回應大小

### 環境問題
- 確認已設定 `ANTHROPIC_API_KEY` 環境變數
- 執行 `devenv shell` 以確保開發環境正確
- 使用 `go mod tidy` 確保相依套件已安裝

## 備註
- 需要設定 ANTHROPIC_API_KEY 環境變數
- 聊天應用程式提供與 Claude 對話的簡單終端機介面
- 使用 ctrl-c 結束聊天工作階段
