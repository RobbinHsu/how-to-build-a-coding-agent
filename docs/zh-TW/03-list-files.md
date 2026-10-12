# 第三階段：讓 Agent 探索目錄

[上一章](02-read-file.md) · [學習目錄](README.md) · [原始碼 `list_files.go`](../../list_files.go) · [下一章](04-bash.md)

## 為什麼需要第二個工具？

上一章只能讀取已知路徑。現在加入 `list_files`，讓模型先找出有哪些檔案，再決定要讀取哪一個。這也是多工具 Agent 的第一個簡單範例。

```bash
go run list_files.go --verbose
```

可以問：「列出目前目錄的檔案，並找出哪個檔案包含謎題。」

## 從哪裡讀？

- `NewAgent(...)`：觀察工具清單如何同時包含 `ReadFileDefinition` 與 `ListFilesDefinition`。
- `ListFilesInput`：`path` 為選填欄位。未提供時，函式預設從目前工作目錄 `"."` 開始。
- `ListFiles()`：使用 `filepath.Walk` 遞迴走訪，利用 `filepath.Rel` 將路徑改為相對表示；目錄名稱會附上 `/`。
- 結果用 `json.Marshal(files)` 輸出 JSON 陣列字串，由 Agent 再傳給模型。
- 此示範會略過 `.devenv` 目錄，但沒有通用的隱藏檔、敏感檔或大型目錄排除規則。

## 一次呼叫與多次呼叫

模型可能先要求 `list_files`，看到檔案清單後，再要求 `read_file`。這不是你的 Go 程式事先寫死的兩步；程式只提供工具並回傳結果，**由模型決定後續要用哪個工具**。

```text
Claude: list_files({"path":"."})
Go:     ["README.md","riddle.txt",...]
Claude: read_file({"path":"riddle.txt"})
Go:     檔案內容
Claude: 整理後回答使用者
```

以上僅為簡化示意，實際工具呼叫順序由模型決定。

## 動手練習

1. 對根目錄、指定子目錄、不存在的目錄分別要求列檔。
2. 在測試資料夾建立兩層子資料夾，確認列檔行為是**遞迴**，不是只列第一層。
3. 找到 `filepath.Walk` 的回呼函式，思考如何增添最大深度、忽略目錄或最大回傳筆數。

**實務注意：** 列出整棵目錄樹可能消耗大量時間與 Token；沒有經過限制時，也可能讓模型看到不該出現的檔名。後續應增加工作目錄邊界、排除清單與結果上限。

**檢查點：** 能說明 Tool Registry 如何同時管理不同工具，以及模型如何利用工具結果決定下一次操作。