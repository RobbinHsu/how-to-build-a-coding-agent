# 第二階段：第一次 Tool Use——讀取檔案

[上一章](01-chat.md) · [學習目錄](README.md) · [原始碼 `read.go`](../../read.go) · [下一章](03-list-files.md)

## 新增了什麼？

`read.go` 在聊天迴圈加入第一個工具 `read_file`。Claude 不會直接存取磁碟，而是回傳要求執行工具的 `tool_use` 內容；Go 程式實際執行 `os.ReadFile`，再把結果傳回 Claude。

## 執行與練習

```bash
go run read.go --verbose
```

輸入：「讀取 `riddle.txt` 並解釋謎題。」接著嘗試讀取不存在的 `missing.txt`，比較成功與失敗流程。

## 依序閱讀這五處

1. `ReadFileDefinition`：工具名稱是 `read_file`，並提供英文用途說明、`InputSchema` 和 `Function: ReadFile`。
2. `ReadFileInput`：以 `Path string` 與 JSON tag `json:"path"` 表示模型應傳入的參數。
3. `GenerateSchema[ReadFileInput]()`：利用 `jsonschema` 從 Go struct 產生工具的輸入 Schema。
4. `runInference()`：將工具清單轉成 Anthropic API 的 `Tools` 參數，讓模型知道可以使用哪些工具。
5. `Agent.Run()`：處理 `message.Content` 中的 `tool_use`，按工具名稱尋找函式，執行之後以 `anthropic.NewToolResultBlock(toolUse.ID, ..., isError)` 回報。工具結果再包成 `anthropic.NewUserMessage(toolResults...)`，發出後續推論請求。

這裡最值得觀察的，是 **`tool_use.ID` 必須對應同一筆工具結果**。模型一個回應也可能包含多個工具呼叫，所以程式會收集 `toolResults`。

## 訊息循環的重點

```text
使用者：「讀取 riddle.txt」
  -> Claude: tool_use(name=read_file, input={"path":"riddle.txt"})
  -> Go: ReadFile(...)
  -> 使用者角色訊息：tool_result(tool_use_id=..., content=...)
  -> Claude: 文字回答
```

這是概念示意，不是原始 API 封包的完整內容。工具執行錯誤時 `isError` 會是 `true`。

## 練習與限制

- 嘗試讀取存在與不存在的檔案，觀察回傳給模型的結果。
- 修改工具的 `Description`，思考它如何影響模型決定是否呼叫工具。
- 研究 `ReadFile()` 對非 JSON、空路徑、絕對路徑或 `../` 路徑的處理。

**安全提醒：** 這份教學程式沒有把讀檔範圍限制在專案內；部分 JSON 解析錯誤也以 `panic` 處理。若要做實用 Agent，應加入路徑範圍驗證、大小限制與完整錯誤處理。

**檢查點：** 你能指出 `tool_use` 是模型的要求，`ReadFile()` 才是真正讀取本機檔案的函式。