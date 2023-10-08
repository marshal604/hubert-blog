+++
title = '使用 Hugo 建置部落格'
date = 2023-10-07T10:26:52+08:00
draft = false
+++
## QuickStart
---
#### 安裝 hugo
``` bash
brew install hugo
```
#### 創建 hugo 的目錄結構，目錄名稱為 quickstart
```bash
hugo new site quickstart
```
#### 切換當前目錄到 quickstart
```bash
cd quickstart
```
#### 初始化 git
```bash
git init
```
#### Clone Ananke 的 theme 到自己的 theme
```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke
```
#### 設定現在的 theme 是 ananke
```bash
echo "theme = 'ananke'" >> hugo.toml
```
#### 啟動 server
```
hugo server
```
#### 啟動後大概就是長這個樣子
![Image](1.png)

#### 創建第一個頁面

``` bash
hugo new content posts/my-first-post.md
```
#### 確認頁面的資料
```
+++
title: "My First Post"
date: 2023-10-07T10:26:52+08:00
draft: true
+++
```
#### 在底下就可以進行撰寫了，以下是我的文章內容
```MD
+++
title = '使用 Hugo 建置 Blog'
date = 2023-10-07T10:26:52+08:00
draft = false
+++

## QuickStart
---
#### 安裝 hugo
brew install hugo
#### 創建 hugo 的目錄結構，目錄名稱為 quickstart
hugo new site quickstart

```

#### 雖然剛剛新增了頁面，但其實你現在應該看不到這個頁面，所以要先終止原先的 hugo server 使用 development 模式才可以看到 draft 為 true 的頁面
以下兩個都可以
``` bash
hugo server --buildDrafts
```
```bash
hugo server -D
```

#### 接下來你就可以看到剛剛新增的頁面了
首頁
![Image](2.png)
內容
![Image](3.png)

#### 打開 hugo.toml 的檔案，設定網站
這是一開始創建好的資料
```
baseURL = 'https://example.org/'
languageCode = 'en-us'
title = 'My New Hugo Site'
theme = 'ananke'
```
- baseURL 需要調整成 production 用的網址
- languageCode 需要調整成當地語言跟地區
- title 需要調整成 production 用的網站標題
以下是我的調整
```
baseURL = 'https://hubertyang.blog/'
languageCode = 'zh-TW'
title = 'Hubert'
theme = 'ananke'
```
完成圖
![Image](4.png)

## Deploy 網站
---
使用指令將網站打包起來，打包的內容不包含 draft 的資訊，可以在 public 資料夾中看到
```bash
hugo
```
#### Hugo 有提供很多 Deploy 的方式，這邊我會使用 Github 當作這次的示範
創建 Repository，我取名叫做 hubert-blog
![Image](5.png)
推送已經在本地建好的 Repository，marshal604 是我的，你要改成你的
```bash
git remote add origin git@github.com:marshal604/hubert-blog.git
git branch -M master
git push -u origin master
```
如果推送出現
```bash
錯誤: 推送一些引用到 'github.com:marshal604/hubert-blog.git' 失敗
```
請記得先做第一次的 `git commit "init project"`，不然是無法推上去的

#### 接下來到 Settings/Pages 來設定 Github Action
選擇 GitHub Action
![Image](6.png)ｓ
選擇 browse all workflows
![Image](7.png)
輸入 Hugo 並選擇
![Image](8.png)
直接到右邊點擊 Commit Changes，再接著直接 Commit Changes 即可
![Image](9.png)
這個 Commit Changes 會觸發我們剛剛設定的 Workflow，可以到 Action 看
![Image](10.png)
不過因為我沒有把 submodule 的 reference 加進去，所以讀不到 theme
![Image](11.png)
這邊只要在把 submodule 的 commit 補上去即可
```bash
git add theme/ananke
git commit -m "feat: add theme ananke submodule reference"
```
workflow 從紅的變成綠的惹~~
![Image](12.png)
#### 確認部署上去的頁面
點擊部署的網址
![Image](13.png)
大功告成！！
![Image](14.png)
## 設定自己的 domain
---
#### 我是去 Godaddy 找了個喜歡的網域買起來
![Image](15.png)
#### 不過因為我之前有買過 `hubertyang.com` 了，所以打算沿用，這邊可以到帳戶裡的 Domain DNS 做設定
![Image](16.png)
#### 把 `github.io` 指向 `blog.hubertyang.com`
![Image](17.png)
#### 然後到 `github` 設定 custom domain `blog.hubertyang.com`
![Image](18.png)
#### DNS設定需要時間，失敗的話等一下再來嘗試
![Image](19.png)
#### 打開 hugo.toml 更新原先的 base url 為 `https://blog.hubertyang.com`
![Image](20.png)
#### 成功轉址
![Image](21.png)

## 關於 SEO
---
#### 通常我們會建立 sitemap.xml 給搜尋引擎查看，但 hugo 在 build 時已經內建，所以只要打 `hugo` 就可以檢查我們是不是也有 sitemap.xml
![Image](23.png)
#### 到 Google Search Console 登錄網域
到[Google Search Console](https://search.google.com/search-console/about) 點擊立即開始
![Image](24.png)
輸入自己的網域，以我為例是 `blog.hubertyang.com`
![Image](22.png)
接著將操作說明選項選為『任何DNS供應商』，並且選取記錄類型選項選為『CNAME』，接著複製 `CNAME 標籤/主機` 與 `CNAME 目的地/目標`
![Image](25.png)
回到自己的 DNS 註冊商位置，以我為例是 Godaddy，主要是把複製的資料放到 CNAME，這裡有個重點是我把複製的資料後面新增了 `.blog`，因為是單獨給子網域的，要注意，不然會驗證不了的
![Image](26.png)
接著就可以回到 Google Search 點擊驗證
![Image](27.png)
如果失敗的話，可以到 [DNS Checker](https://dnschecker.org/) 查查看是不是 `[google 給的名稱].blog.hubertyang.com` 有沒有存在，反之成功就會看到下圖
![Image](28.png)
#### 將 Sitemap 放到 Google Search Console 上
因為 hugo 有幫我們建立好 sitemap 了，只要到根目錄就可以找到，以我為例就是：`https://blog.hubertyang.com/sitemap.xml`，接著我們就提交吧
![Image](29.png)

## 關於頁面分析
#### 將網站放到 GA 上來分析使用者的瀏覽狀況吧~~
首先到 `https://marketingplatform.google.com/about/analytics/` 點擊 Get Start Today，然後開始註冊帳號，因為我是註冊 `hubertyang.com` 這個網域，所以我帳號就以這個當作一個 group
![Image](30.png)
輸入我要監控的資源名稱，我是要監控 blog，所以我就取名 `blog.hubertyang.com`
![Image](31.png)
技術文章應該就...電腦吧？
![Image](32.png)
這邊是選擇**提升品牌認知**與**檢視使用者行為**，因為主要是推廣我的部落格，而**取得基準報表**就比較是跟同業比較了
![Image](33.png)
選擇網站
![Image](34.png)
輸入網站連結，跟好辨識的名稱
![Image](35.png)
#### 因為我沒有設定 GA 在 Hugo 上，所以會跳出安裝的介面
![Image](36.png)
#### 先把收到的 ID，如 `G-XXXXX` 放到 hugo.toml
```toml
googleAnalytics = 'G-XXXXX'
```
#### 創建 `_internal/google_analytics.html` 把剛剛複製的 Script 放進去
![Image](37.png)
#### 接著放到每個頁面的 head 中
這邊複製 `themes/ananke/layouts/_default/baseof.html` 到 `layouts/_default/baseof.html` 並將 GA 的 html 放到 head 中
```html
<head>
  <!-- 其他的 code -->
  {{ template "_internal/google_analytics.html" . }}
</head>
```
#### 註冊完後，就可以等時間看數據了
![Image](38.png)

## Reference
- https://gohugo.io/getting-started/quick-start
- https://gohugo.io/hosting-and-deployment/hosting-on-github/
- https://gohugo.io/templates/internal/#google-analytics