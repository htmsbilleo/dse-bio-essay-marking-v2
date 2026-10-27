# DSE Bio Essay 評改助手 v2

呢個 repo 用來開**一個新嘅** Vercel 靜態網站。請唔好覆蓋而家嘅生產網站：

https://dse-bio-essay-marking.vercel.app/

原版同改善版一齊放喺呢個新 project，方便對照。日常請用改善版。

## 兩個頁面

| 網址 | 檔案 | 係咩 |
| --- | --- | --- |
| `/` | `index.html` | **原版**。同而家生產頁係同一份檔（byte 相同），仍然有舊嘅內建 Apps Script／Google Sheet 預設。只供對照。 |
| `/v2/` | `v2/index.html` | **改善版**。冇內建交卷網址、降低 xAI key 外洩、老師 PIN、分頁鍵盤。 |

`/v2`（冇斜線）會由 `vercel.json` 指去 `v2/index.html`。

## 改善版老師要做嘅設定

Apps Script、題庫等設定只存在**該部瀏覽器**。老師 PIN 則係全站共用：任何瀏覽器撳「老師」都要輸入同一個 PIN。學生用自己部手機交卷，就要喺嗰部手機由老師解鎖後貼上並儲存交卷網址；網站唔會自動派發網址。

1. 打開 `/v2/`，撳「老師」，輸入**共用老師 PIN**。任何瀏覽器都用同一組 PIN。PIN 可以包含英文字母、數字同符號，最長 64 個字元。頁面只核對內建加鹽 SHA-256 雜湊，唔會保存 PIN 本身，亦冇「更改 PIN」。清除本機資料唔會重設呢個 PIN。
2. 連續 **5 次** PIN 錯誤會鎖定大約 **45 秒**。
3. 用完撳頁頂「鎖定老師模式」。未解鎖之前，學生模式唔會顯示老師工具。學生模式本身唔使 PIN。
4. 喺「交卷一覽」貼上**你自己**嘅 Apps Script Web App URL，可選再貼 Google Sheet 連結，然後按「儲存交卷設定」。
5. **未儲存 Web App 之前**，學生撳交卷只會寫入該瀏覽器嘅 `localStorage`。狀態列會寫明冇送到 Google Sheet。
6. 可選「課堂權杖」。有填先會放進 POST JSON 嘅 `classroomToken`，方便 Apps Script 核對。權杖、Web App URL、Sheet URL、API key、PIN 明文都**唔好**寫入原始碼，亦**唔好** commit。

xAI：

- 請把 API key 放喺 Apps Script 嘅 **Script Properties**（建議名 `XAI_API_KEY`）。改善版經代理評改時，請求**唔會**帶 `apiKey`。
- 瀏覽器入面嘅 key 預設只放今次分頁（`sessionStorage`）。要寫入 `localStorage` 必須再確認一次。
- 「允許直接呼叫 api.x.ai」預設關閉，開啟前會有共用電腦警告。直接呼叫先會由瀏覽器送出 key。
- 「清除」會清走輸入框、`sessionStorage` 同 `localStorage` 入面呢版儲存嘅 key。

Apps Script 代理應自己讀 Script Properties，忽略 body 入面嘅 key，例如：

```javascript
var props = PropertiesService.getScriptProperties();
var key = props.getProperty("XAI_API_KEY");
var token = props.getProperty("CLASSROOM_TOKEN");
// 用 key 呼叫 xAI；如果設定咗 CLASSROOM_TOKEN，先核對 body.classroomToken
```

## 部署（新 Vercel project）

靜態檔，冇 build。

- 新建一個 Vercel project，連去呢個 repo
- Framework preset：Other
- Build command：留空
- Output / root：repo 根目錄（要同時見到 `index.html` 同 `v2/index.html`）
- 唔好喺舊 project `dse-bio-essay-marking` 改 production，亦唔好將 `dse-bio-essay-marking.vercel.app` 指去呢度

## 秘密

唔好 commit API key、Apps Script URL、Google Sheet URL、課堂權杖、PIN 明文或 `.env`。共用老師 PIN 只以加鹽雜湊放喺改善版頁面；明文唔好寫入檔案、commit 或 PR。其他秘密由老師喺瀏覽器貼上就得。
