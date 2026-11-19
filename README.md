# Private Pages

把 HTML 放進 `public/`，push 到 `main` 後會由 GitHub Actions 自動部署到 GitHub Pages。

**網站網址**：<https://stella572322.github.io/private-pages/>

## 使用方式

1. 把 HTML 檔放到 `public/` 底下，例如 `public/my-report.html`。
2. commit 並 push 到 `main`。
3. 約一兩分鐘後，即可透過下列網址存取：

```
https://stella572322.github.io/private-pages/my-report.html
```

放在子資料夾的檔案，網址就包含該路徑，例如 `public/team/a.html` → `/private-pages/team/a.html`。

圖片、音檔等附件建議放在 `public/assets/<頁面名稱>/` 底下，頁面內用相對路徑引用，例如 `assets/my-report/cover.jpg`。

## 注意

- 這個 repo 與 Pages 網址是公開的，**不要放機密內容**。
- 部署設定在 [.github/workflows/pages.yml](.github/workflows/pages.yml)；Settings → Pages 的 Source 需設為 **GitHub Actions**。
- 同名檔案再次 push 會直接覆蓋舊版本。

## 用 Claude Code 自動發佈（publish-html skill）

專案內建 skill：[.claude/skills/publish-html/SKILL.md](.claude/skills/publish-html/SKILL.md)。
在這個 repo 目錄下使用 Claude Code，直接把 HTML 檔案交給它即可，例如：

```
/publish-html D:\Downloads\台南美食 景點清單.html
```

或用自然語言：「幫我發佈這個 HTML：<路徑>」。Claude 會自動完成：

1. **改檔名**：轉成網址安全的 `小寫英文_底線_YYYYMMDD.html`（無中文、空白、特殊符號；原檔名有日期就沿用，否則用今天日期；重複則加 `_2`）。
2. **處理附件**：HTML 引用的本機圖片／音檔會放到 `public/assets/<頁面名稱>/` 並改寫路徑。
3. **放進 `public/`**（原始檔不動）。
4. **commit + push 到 `main`**，commit 訊息為 `publish: <檔名>.html`。
5. **等待 GitHub Actions 部署完成並驗證**，最後回覆可直接開啟的網址。

遇到疑似機密內容（API key、密碼、個資）時，會先停下來詢問，因為網址是公開的。

### 覆蓋既有頁面

預設**不會覆蓋**任何既有檔案（撞名時自動加 `_2`）。想更新已發佈的頁面時，必須明確說要覆蓋並指出目標：

```
/publish-html D:\Downloads\new.html --overwrite tainan_food_attractions_20261225.html
```

或用自然語言：「用這個檔案覆蓋 tainan_food_attractions_20261225」、「更新這個頁面：<網址>，內容換成 <路徑>」。

覆蓋時：

- 目標可以是檔名、完整網址或 `public/` 內的相對路徑；找不到或有多個符合時會先問你，不會擅自改成新增。
- 沿用目標原本的檔名，**網址不變**，舊連結照常可用。
- 附件資料夾會同步更新，新版不再引用的舊附件會被刪除（git 歷史仍可還原）。
- commit 訊息為 `update: <檔名>.html`；內容與舊版完全相同時不會產生 commit。
- GitHub Pages 有快取，更新後若看到舊內容，請強制重新整理（Ctrl+F5）。
