# Private Pages

把 HTML 放進 `public/`，push 到 `main` 後會由 GitHub Actions 自動部署到 GitHub Pages。

**網站網址**：<https://stella572322.github.io/private-pages/>

## 使用 publish-html 發佈頁面

在這個 repo 的資料夾開啟 Claude Code，把 HTML 檔案**拖曳進輸入框**，再輸入下列任一種 prompt，送出即可。完成後 Claude 會回覆可直接開啟的網址。

### 發佈新頁面

```
/publish-html
```

```
幫我發佈這個 HTML
```

```
這是一份新的分析報告，幫我上線，完成後把網址給我
```

也可以不拖曳檔案，直接把 HTML 內容貼在對話裡，再說「幫我發佈這個」。

### 一次發佈多個檔案

一次拖曳多個檔案進輸入框：

```
幫我發佈這幾個 HTML，每個都給我網址
```

### 含圖片或音檔的頁面

把 HTML 和它引用的圖片、音檔一起拖曳進輸入框：

```
幫我發佈這個 HTML，裡面引用的圖片和音檔也要一起上傳
```

### 覆蓋既有頁面

預設不會覆蓋舊檔。要覆蓋時，必須明確說「覆蓋」並指定目標（檔名或網址皆可）。覆蓋後網址不變，若瀏覽器還看到舊內容，請按 Ctrl+F5 重新整理。

```
/publish-html --overwrite my_report_20261120.html
```

```
用這個檔案覆蓋 my_report_20261120
```

```
更新這個頁面 https://stella572322.github.io/private-pages/my_report_20261120.html ，內容換成這個檔案
```

## 注意

- 這個 repo 與 Pages 網址是公開的，**不要放機密內容**。
- 部署設定在 [.github/workflows/pages.yml](.github/workflows/pages.yml)；Settings → Pages 的 Source 需設為 **GitHub Actions**。
- 同名檔案再次 push 會直接覆蓋舊版本。
