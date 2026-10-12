# 第一階段：先做出能對話的 Agent

[回到學習目錄](README.md) · [原始碼 `chat.go`](../../chat.go) · [下一章：讀取檔案](02-read-file.md)

## 這一章要解決什麼？

先把「從終端機讀取訊息 → 傳給 Claude API → 顯示回覆」的最短路徑跑通。這個版本還不是會自己操作電腦的 Coding Agent，因為**沒有任何工具**。

## 執行

```bash
go run chat.go --verbose
```

可以輸入：「請用三句話說明什麼是 Agent。」結束時按 `Ctrl+C`。

## 從原始碼找四個重點

1. `main()` 用 `flag.Bool("verbose", ...)` 處理旗標，再用 `anthropic.NewClient()` 建立 API Client；SDK 會從環境設定讀取驗證資訊。
2. `bufio.NewScanner(os.Stdin)` 每次取得一行使用者文字；`getUserMessage` 把輸入取得方式交給 `Agent`。
3. `Agent.Run()` 建立 `conversation := []anthropic.MessageParam{}`，將每次輸入轉成 `anthropic.NewUserMessage(...)`，再把模型回應的 `message.ToParam()` 留在對話紀錄中。
4. `runInference()` 透過 `a.client.Messages.New(...)` 送出訊息；目前範例設定 `anthropic.ModelClaudeOpus4_6` 與 `MaxTokens: 1024`。可用模型與 API 費用仍須以 Anthropic 帳戶設定為準。

關鍵是**保存並重送對話紀錄**。若每次都只送目前輸入，模型就無法從請求中取得先前的聊天內容。

## 觀察實驗

1. 先問「我接下來會告訴你一個代號：ALPHA。」
2. 再問「我剛才給的代號是什麼？」
3. 使用 `--verbose` 觀察 `conversation length` 如何成長。
4. 改讀 `runInference()`：確認每次請求都帶入 `Messages: conversation`。

## 自己動手改

把 `MaxTokens` 改為不同的正整數，觀察短回答與長回答受到的影響。修改模型參數時，先確認 SDK 支援的常數及你帳戶可使用的模型。這一階段不要急著加入工具。

**檢查點：** 你能區分「程式的輸入迴圈」與「模型的一次 API 推論」，也知道聊天紀錄是你的應用程式自己管理的。

**下一章：** 讓模型不只回答文字，還能要求本機程式讀取指定檔案。