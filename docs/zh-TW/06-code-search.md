# 第六階段：使用 ripgrep 搜尋程式碼

[上一章](05-edit-file.md) · [學習目錄](README.md) · [原始碼 `code_search_tool.go`](../../code_search_tool.go)

## 這章不是單純疊加第五章

`code_search_tool.go` 註冊的工具包含 `read_file`、`list_files`、`bash` 和 `code_search`，**沒有註冊 `edit_file`**。這個範例的目的是教你新增一個搜尋工具，不代表最終版本自動擁有所有前面示範過的能力。

## 執行與嘗試

請先安裝 [ripgrep](https://github.com/BurntSushi/ripgrep)，確認 `rg --version` 可以執行；這份程式仍包含 `bash` 工具，因此也需要 `bash`。

```bash
go run code_search_tool.go --verbose
```

建議輸入：「搜尋本專案所有 `TODO` 註解，列出檔名與行號。」或「在 Go 檔案中搜尋 `func` 定義。」

## 讀懂 `CodeSearchInput`

| 欄位 | 意義 |
| --- | --- |
| `pattern` | 必填搜尋字串或正規表示式 |
| `path` | 選填搜尋檔案／目錄，預設 `.` |
| `file_type` | 選填 ripgrep 檔案類型（例如 `go`、`js`、`py`） |
| `case_sensitive` | 布林值；未提供時預設 `false`，代表忽略大小寫 |

`CodeSearch()` 建立的基礎參數是：

```bash
rg --line-number --with-filename --color=never --ignore-case TODO .
```

實際執行使用 `exec.Command(args[0], args[1:]...)`，**不是透過 Bash 拼接字串**。這有助於避免將模式字串意外交給 Shell 解釋，但仍應驗證路徑、參數及輸出。

## 三個容易忽略的細節

1. `rg` 找不到符合項目時通常回傳結束碼 1；原始碼會轉換成 `No matches found`，而不是一般工具錯誤。
2. 搜尋結果**超過 50 行**時只回傳前 50 行，避免過量輸出灌入模型的 Context。
3. `file_type` 使用 `rg --type`；傳入值必須是 ripgrep 認得的類型名稱，不保證任意副檔名都有效。

## 練習：把能力合併

先嘗試 `code_search` 找函式，再用 `read_file` 查看所在檔案。

進階練習是讓這個版本也擁有 `edit_file`：對照 [第五階段](05-edit-file.md)，將所需的 `EditFileDefinition`、輸入結構與實作合併，並把新工具註冊進 `NewAgent` 的工具集合。不要只增加 Schema 而忘記實際函式。合併時需留意不同 `.go` 範例有重複的 `main`、`Agent` 等定義，不可直接把全部檔案一起交給 `go run .`。

## 完成後的下一步

- 為所有讀／寫／搜尋工具設計**一致的專案根目錄邊界**。
- 加入指令核准、逾時、輸出上限與對機密檔案的拒絕存取。
- 將重複的 Tool Registry 與 Agent Loop 重構成可測試的套件。
- 為工具增加單元測試：正常結果、缺少檔案、無符合項目、無權限及異常輸入。

**檢查點：** 你已能從零描述一個 Coding Agent 的核心：模型 API、工具定義、執行迴圈、工具結果，以及控制本機副作用所需的安全防護。