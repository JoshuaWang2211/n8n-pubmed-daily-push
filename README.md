# 🧬 PubMed Daily Push – 資料庫文獻每日精選推播（n8n Workflow）

這是一個建構在 **n8n** 上的每日資料庫文獻推播流程。  
在此 Workflow 中，會依多個主題從 **PubMed** 抓取最新論文，隨機挑選文章後擷取標題與摘要，再交給 AI 轉寫成適合一般大眾閱讀的中文說明，最後推送到 **LINE**。

同時，本流程也作為範例：  
1. 你可以自行將 PubMed API 替換成 **任意資料庫**（如 arXiv、CrossRef、bioRxiv、CNKI、Semantic Scholar），只需修改 API 來源與解析邏輯即可。
2. 不一定要使用 **LINE** 推播，也可以使用 **Email、Discord、Telegram** 等其他通訊方式來推播。

---

## ✨ 功能特色

### 🔍 自動擷取 PubMed 最新論文
- 依主題（如「immunology」「cardiology」「oncology」等）抓取每日最新文章。
- 每個主題隨機挑一篇，避免每天推播相同內容。

### 📘 AI 轉寫大眾版摘要
- 擷取論文 `title`、`abstract`、`authors`、`journal`。
- 透過 OpenAI（例如 GPT-5.1 或任意 AI 模型，BYOK/model）進行摘要整理並重寫為中文，讓內容更適合一般讀者。

### 🤖 LINE 推播通知
- 支援多位使用者推送或群發。
- 使用 `$env.LINE_CHANNEL_TOKEN`、`$env.LINE_USER_ID_1/2/...` 從環境變數讀取，不會洩漏資訊。

### ⏰ 每日自動觸發
- 預設每天 12:00 推播，可自行修改排程。

---

## 🖼️ 截圖

[![Workflow 截圖](/workflow.png)](/workflow.png)
---
[![LINE 輸出截圖](/line.png)](/line.png)

---

## 🧱 Workflow 節點架構

以下為此 workflow 的主要流程：

### 1️⃣ Schedule Trigger（每日固定時間）
- 預設每天中午 12:00 觸發。
- 可自由設定為 06:00 / 12:00 / 晚上等，一天一次或多次都可以，每次都會重新擷取最新文獻。

---

### 2️⃣ 設定主題列表（Code Node）
你可以在這裡自由新增 PubMed 主題：

```js
return [
  { topic: "immunology" },
  { topic: "oncology" },
  { topic: "cardiology" },
  { topic: "metabolism" }
];
```

若要改成其他資料庫，只要將每個 topic 改為你想查詢的關鍵字即可。

---

### 3️⃣ 迴圈（Split In Batches）
- 每次處理一個主題。
- 將該主題傳給 PubMed 搜尋 API。

---

### 4️⃣ PubMed Search
使用 NCBI E-utilities（esearch）：

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi
```

參數：
- term（搜尋關鍵字）
- api_key = `{{$env.NCBI_API_KEY}}`
- email = `{{$env.NCBI_EMAIL}}`

---

### 5️⃣ PubMed Fetch
使用 efetch 從 PubMed 取回實際文章資訊：

```
https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi
```

解析 XML 內容後取得：
- Title
- Abstract
- Authors
- Journal
- Publication Date

---

### 6️⃣ AI 重寫文案（OpenAI Chat）
請模型將醫學摘要轉寫成：
- 一般大眾可理解
- 中文
- 口語化但不失準確

---

### 7️⃣ 組裝 LINE 訊息（Code）
透過環境變數取得收件者：

```js
const userIds = [
  $env.LINE_USER_ID_1,
  $env.LINE_USER_ID_2
].filter(Boolean);
```

---

### 8️⃣ 推送 LINE
使用 HTTP Request：

header：

```json
{
  "Authorization": "={{$env.LINE_CHANNEL_TOKEN}}",
  "Content-Type": "application/json"
}
```

---

## 🔧 如何改成你自己的專屬版本

### ✔️ 改成其他資料庫（最簡方式）
只需修改兩步：

#### **步驟 A：更改 API URL**
例如改成 arXiv：

```
https://export.arxiv.org/api/query?search_query=all:AI
```

或 Semantic Scholar：

```
https://api.semanticscholar.org/graph/v1/paper/search
```

#### **步驟 B：修改資料解析邏輯**
把 PubMed 的 XML 解析改成對應的 JSON/XML 欄位即可。

---

## 🛠️ 安全性：需要自行補上的環境變數

請務必自行設定以下環境變數（n8n → Settings → Environment Variables）：

```
LINE_CHANNEL_TOKEN=
LINE_USER_ID_1=
LINE_USER_ID_2=
NCBI_API_KEY=
NCBI_EMAIL=
```

---

## 📬 聯絡作者

- GitHub：@JoshuaWang2211

---

本 workflow 由 Joshua Wang 設計，是一個可擴充、可複用的資料庫資訊推播系統。  
你可以照此結構/邏輯打造屬於你自己的資料庫推播系統。