---
name: publish-html
description: 把使用者給的任意 HTML 檔案發佈到這個 repo 的 GitHub Pages。自動把檔名改成網址安全且不易撞名的名稱（小寫英數與底線 + 日期），放進 public/，自動 commit、push、等待部署完成，最後回覆可直接開啟的網址。當使用者提供 .html 檔案（路徑、貼上內容或附件）並想要發佈、上線、部署、放到網站上時使用。
---

# publish-html

把任意 HTML 檔案發佈到 `https://stella572322.github.io/private-pages/`。
整個流程自動完成，不需要中途詢問使用者；只有遇到下方「需要停下來確認」的情況才詢問。

## 流程

1. **取得來源檔**：使用者給的路徑，或把貼上的 HTML 內容先寫成暫存檔。確認副檔名為 `.html`（`.htm` 一律改為 `.html`）。
2. **決定新檔名**（見「命名規則」）。
3. **檢查撞名**：`public/` 底下（含子資料夾）若已存在同名檔案，**預設絕不覆蓋**；依規則加上 `_2`、`_3`… 後綴。
   只有使用者明確指定要覆蓋時，才走下方「覆蓋模式」。
4. **處理附件**：掃描 HTML 中的本機相對路徑（`src=`、`href=`、`url(...)`、`<source>` 等，不含 `http(s)://`、`//`、`data:`、`#`、`mailto:`）。
   - 有引用本機檔案 → 複製到 `public/assets/<新檔名不含副檔名>/`，檔名同樣依命名規則清理，並同步改寫 HTML 內的路徑為 `assets/<新檔名不含副檔名>/<檔名>`。
   - 找不到被引用的檔案 → 在最後回報中列出，不要阻擋發佈。
5. **放置檔案**：複製（不是移動）到 `public/<新檔名>.html`，保留使用者的原始檔不動。
6. **Commit**：只 `git add` 這次新增／變更的檔案（`public/<新檔名>.html` 與對應 assets），不要用 `git add -A`。
   - 訊息：`publish: <新檔名>.html`（有附件則 `publish: <新檔名>.html with assets`）；覆蓋模式改用 `update: <檔名>.html`。
   - 結尾加上目前 session 指定的 Co-Authored-By 署名行。
7. **Push**：`git push origin main`。
8. **等待部署**：push 觸發 `.github/workflows/pages.yml`。用背景指令輪詢（本機沒有 `gh`，用公開 API）：
   ```
   until [ "$(curl -s "https://api.github.com/repos/stella572322/private-pages/actions/runs?per_page=1" | grep -m1 '"status"' | grep -o completed)" = completed ]; do sleep 10; done
   curl -s "https://api.github.com/repos/stella572322/private-pages/actions/runs?per_page=1" | grep -E '"(conclusion|html_url)"' | head -2
   ```
   注意：確認抓到的是**這次 push 的 run**（比對 commit SHA 或建立時間），不是上一次的。
9. **驗證**：`curl -s -o /dev/null -w "%{http_code}" <網址>` 應為 200。部署剛完成時若暫時 404，等 10 秒重試幾次。
10. **回覆**：最後的訊息要包含可直接點開的完整網址（獨立一行，不要包在程式碼區塊內），並簡述：原檔名 → 新檔名、附件處理結果。

## 覆蓋模式（使用者明確指定才啟用）

**觸發條件**：使用者的話中明確表達「覆蓋／取代／更新既有頁面」，並指出目標。例如：

- `/publish-html D:\Downloads\new.html --overwrite tainan_food_attractions_20261225.html`
- 「用這個檔案覆蓋 tainan_food_attractions_20261225」
- 「更新這個頁面：https://stella572322.github.io/private-pages/tainan_food_attractions_20261225.html，內容換成 <路徑>」

只說「發佈」、「上傳」、「放上去」**不算**覆蓋；沒有明確指定就一律走一般流程（新檔名、不覆蓋）。

**行為**：

1. **解析目標**：目標可以是檔名（有無 `.html` 皆可）、完整網址，或 `public/` 底下的相對路徑。在 `public/` 內比對：
   - 恰好 1 個符合 → 使用它。
   - 找不到 → **不要改成新增**；列出相近的現有檔名請使用者選，或詢問是否改為新增。
   - 多個符合 → 列出候選，請使用者指定。
   - 目標是 `index.html`、`404.html` 時，只在使用者明確點名該檔時才允許。
2. **保留目標檔名**：新檔案直接覆寫 `public/<目標檔名>`，**不重新命名**，網址維持不變，舊連結照常可用。來源檔本身的檔名只用來判斷內容，不套用命名規則。
3. **附件同步**：若有附件，內容整理到 `public/assets/<目標檔名不含副檔名>/`，並改寫 HTML 內路徑；該資料夾中不再被新版 HTML 引用的舊檔案一併刪除（git 歷史仍保有舊版，可還原）。
4. **覆蓋前先看目標**：覆蓋前用 `git diff --stat` 或檔案大小／標題簡述「舊版 → 新版」的差異，並在最後回覆時告知使用者哪個檔案被覆蓋。若新舊內容完全相同，不要 commit，直接回報「內容無變更」並附上網址。
5. **commit**：`update: <目標檔名>.html`（有附件變動則加 `with assets`）；其餘 push、等待部署、回覆網址的流程與一般流程相同。
6. **驗證**：GitHub Pages 有 CDN 快取，網址剛更新時可能還看到舊版。驗證時用 `curl -s "<網址>?v=<時間戳>"` 與本機檔案內容比對；若仍是舊內容，等 10–30 秒重試。回覆時提醒使用者若瀏覽器看到舊內容，請強制重新整理（Ctrl+F5）。

## 命名規則

最終格式：`<描述>_<YYYYMMDD>.html`，例如 `tainan_food_attractions_20261225.html`。

- **字元**：只允許小寫 `a-z`、數字 `0-9` 與底線 `_`（副檔名前的 `.` 除外）。
  不可出現：空白、中文與其他非 ASCII 字元、`# ? % & + = / \ : ; , ' " ( ) [ ] { } ~ ! @ $ ^ *`、連字號開頭、連續底線。
- **描述部分**：
  - 根據檔名與 HTML 內容（`<title>`、`<h1>`）判斷主題，用 2–5 個英文單字概括，以底線連接。
  - 非英文檔名要翻譯成語意對應的英文，不要用拼音亂碼，也不要用 `index`、`page`、`test`、`untitled`、`new` 這類泛用名稱。
  - 全部轉小寫；原檔名裡的空白、`-`、`.` 一律轉為 `_`；合併連續底線並去除頭尾底線。
- **日期後綴**：
  - 原檔名或內容明確帶有日期（如 `20261225`、`2026-12-25`、`2026_12_25`）→ 正規化為 `YYYYMMDD` 後沿用。
  - 沒有日期 → 使用今天日期（以系統日期為準：`date +%Y%m%d`）。
- **長度**：整個檔名（含副檔名）不超過 60 字元；過長就縮短描述部分。
- **保留名稱**：不可命名為 `index.html`、`404.html`（會覆蓋首頁或錯誤頁）；使用者明確要求才可例外（覆蓋模式明確點名該檔時亦同）。
- **撞名處理**：日期已能避免大部分撞名；若仍重複，加 `_2`、`_3`（放在日期之後），例如 `report_20261225_2.html`。

範例：

| 原檔名 | 新檔名 |
|---|---|
| `台南美食 景點清單.html` | `tainan_food_attractions_<今天日期>.html` |
| `My Report (final) v2.html` | `my_report_final_v2_<今天日期>.html` |
| `旅遊分析-2026-12-25.html` | `travel_analysis_20261225.html` |
| `index.html` | 依內容命名，例如 `team_dashboard_<今天日期>.html` |

## 需要停下來確認的情況

- HTML 內含看起來像機密的內容（API key、token、密碼、私鑰、個資如身分證字號／電話／地址）。repo 與網址是**公開**的，先指出疑似內容並詢問是否仍要發佈。
- 工作目錄不在這個 repo 內，或 `git status` 顯示有與這次發佈無關的未提交變更（只提醒，仍只 commit 本次檔案，不需中止）。
- push 失敗（認證、衝突）→ 回報錯誤，不要 force push。
- 部署 run 失敗 → 回報 Actions 連結與失敗步驟，不要宣稱成功。

## 網址格式

```
https://stella572322.github.io/private-pages/<新檔名>.html
```
