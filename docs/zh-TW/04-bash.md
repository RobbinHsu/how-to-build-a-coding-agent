# 第四階段：把 Shell 指令交給 Agent

[上一章](03-list-files.md) · [學習目錄](README.md) · [原始碼 `bash_tool.go`](../../bash_tool.go) · [下一章](05-edit-file.md)

## 新能力與風險

`bash_tool.go` 註冊 `bash` 工具，讓模型可以要求執行指令。這大幅提升 Agent 的能力，也引入明確風險：**示範程式並沒有沙箱、指令白名單或人工確認機制**。

請只在不含機密或重要檔案的隔離練習環境使用；不要把它當作可以安全執行任意命令的正式工具。

## 執行

需要系統能找到 `bash`：

```bash
go run bash_tool.go --verbose
```

在測試儲存庫輸入：「執行 `git status`，告訴我目前有哪些修改。」請避免先要求模型執行任何刪除、上傳或安裝命令。

## 對照原始碼

1. `BashDefinition` 對模型宣告可用工具名稱 `bash`。
2. `BashInput` 定義 `Command string`，JSON 欄位為 `command`。
3. `Bash()` 解析模型送來的 JSON，呼叫 `exec.Command("bash", "-c", bashInput.Command)`。
4. `cmd.CombinedOutput()` 同時取得標準輸出與錯誤輸出。
5. 原始碼對**指令執行失敗**的處理方式，是回傳描述錯誤的字串和 `nil`；因此上層可能把它當成一般工具結果，而不是 `isError=true` 的工具錯誤。這是值得改善的行為。

## 重要觀念

```text
模型提供：{"command":"git status"}
          |
Go 使用 bash -c 啟動外部程序
          |
捕捉輸出並回傳模型
```

和 `read_file` 不同，Shell 可以依目前使用者的系統權限做許多事：修改檔案、啟動程序、存取網路等。**模型寫出指令，不代表這個指令已被驗證安全。**

## 自己動手改

- 在執行指令前顯示命令，要求使用者確認。這能降低誤執行風險，但仍不是安全沙箱。
- 使用 `context.WithTimeout` 與 `exec.CommandContext` 設定逾時；考慮限制輸出量。
- 將非零結束碼轉成明確錯誤，讓 `tool_result` 正確呈現失敗。
- 討論工作目錄、環境變數、權限最小化和網路隔離如何影響安全性。

**Windows 注意：** PowerShell 存在不代表程式內的 `bash` 可執行。可改在 WSL／Linux 完成此章。

**檢查點：** 清楚知道 `exec.Command` 在哪裡執行、具有誰的權限，以及本教學版缺少哪些執行前防護。