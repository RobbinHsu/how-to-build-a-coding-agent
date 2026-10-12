# 第五階段：讓 Agent 建立與修改檔案

[上一章](04-bash.md) · [學習目錄](README.md) · [原始碼 `edit_tool.go`](../../edit_tool.go) · [下一章](06-code-search.md)

## 新增的 `edit_file` 是什麼？

`edit_tool.go` 除了讀取、列出及 Bash 工具，再加入 `edit_file`。它不是任意位置的文字編輯器，而是採用**原字串 `old_str` → 新字串 `new_str`** 的方式修改文字檔。

```bash
go run edit_tool.go --verbose
```

建議在空的測試資料夾執行，並先用 Git 記錄原始檔案狀態。

## 三個輸入欄位

| JSON 欄位 | 用途 |
| --- | --- |
| `path` | 目標檔案路徑 |
| `old_str` | 要精確比對、替換的舊文字 |
| `new_str` | 替換後的文字 |

## 依實際程式碼理解四種情況

1. **建立檔案：** `path` 指向不存在的檔案，且 `old_str` 為空字串，就會呼叫 `createNewFile()`；必要時建立上層目錄。
2. **附加內容：** 檔案存在且 `old_str` 為空字串，就把 `new_str` 接到現有內容後面。
3. **替換內容：** `old_str` 在檔案中**剛好出現一次**時，透過 `strings.Replace(..., 1)` 寫回。
4. **拒絕操作：** 找不到 `old_str`、找到多次、`path` 為空，或 `old_str == new_str`，就會回傳錯誤。

注意：程式用 `os.ReadFile` 和 `os.WriteFile` 操作整份檔案，而非針對局部磁碟資料區塊修改。

## 建議實驗

先要求模型：「建立 `practice/hello.txt`，內容是 `Hello agent`。」接著要求「將 `Hello agent` 改為 `Hello coding agent`」。最後再要求它以 `read_file` 確認內容。

之後自行製造含有兩個相同 `old_str` 的測試檔，觀察工具拒絕修改的錯誤。

## 為什麼需要安全機制？

- 沒有專案根目錄的路徑保護與檔案類型限制。
- 沒有「修改前人工核准」與自動備份。
- `os.WriteFile` 直接寫檔，並未實作原子替換、版本衝突檢查或防止同時寫入。
- 模型可能選錯檔案；即使工具成功回傳 `OK`，也不表示修改符合需求。

可先加入**預覽 diff → 使用者確認 → 寫入 → 讀回驗證**的流程，再考慮讓它編輯真正的專案檔案。

**檢查點：** 能解釋 `old_str` 為什麼要求唯一、空字串的特殊意義，以及為何 Agent 的檔案寫入能力需要防護。