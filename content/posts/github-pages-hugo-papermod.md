---
title: "實戰教學：用 GitHub Pages 部署 Hugo + PaperMod，打造零成本個人技術部落格"
date: 2025-12-08T15:30:00+08:00
draft: false
categories:
  - 架站實作
  - 技術寫作
tags:
  - ai-工具
  - vibe-coding
  - github-pages
  - papermod

summary: "一步一步從零建立 Hugo + PaperMod 技術部落格，並部署到 GitHub Pages 的完整流程與常見踩坑整理。"

showToc: true
tocOpen: true
---


對做開發的人來說，技術文章最好是放在自己掌控的地方，而不是完全寄託在 Medium 或社群平台。這篇文章會帶你一步步把 Hugo + PaperMod 部落格，部署到 GitHub Pages 上，達到：

- 架站成本 = 0（GitHub Pages 免費）
- 內容掌握度高（Markdown 寫作 + Git 版控）
- 版型清爽、有暗色模式、支援程式碼區塊（PaperMod 主題）

只要你已經有 GitHub 帳號、願意開一下終端機，大概 30–60 分鐘就能把自己的技術部落格跑起來。

---

## 一、為什麼是 Hugo + PaperMod + GitHub Pages？

- **Hugo**：Go 寫的靜態網站產生器，生成速度快、部署簡單，只要丟一堆 HTML/CSS 到 GitHub Pages 就能跑。
- **PaperMod 主題**：設計簡潔，預設就支援：
  - 深 / 淺色模式切換  
  - 程式碼語法高亮  
  - 文章目錄、Tag、分類等技術 blog 需要的元素  
- **GitHub Pages**：不用自己維護伺服器，不用管 Nginx / Apache，只要 push 到指定 branch，就會自動幫你上線。

這組合非常適合：

- 想建立作品集 / 技術筆記
- 不想維運伺服器
- 本來就用 GitHub 管理專案

---

## 二、前置準備

你需要先準備好：

1. GitHub 帳號（免費版就足夠）
2. 安裝好 Git  
3. 安裝好 Hugo（建議擇一方式）
   - macOS：可以用 Homebrew  

     ```bash
     brew install hugo
     ```

4. 一個用來存放專案的資料夾（例如 `~/Projects`）

可以在終端機確認 Hugo 是否安裝成功：

```bash
hugo version
```

有印出版本號就代表 OK。

---

## 三、建立 Hugo 專案並安裝 PaperMod 主題

### 1. 建立新的 Hugo 專案

在你習慣的資料夾底下：

```bash
hugo new site my-blog
cd my-blog
```

這會生成一個 Hugo 專案基本結構。

### 2. 新增 Git Repo

```bash
git init
git add .
git commit -m "Initial Hugo site"
```

這裡先初始化版控，等等處理主題與設定變更會比較好管理。

### 3. 安裝 PaperMod 主題（以 git submodule 方式）

官方推薦的作法通常是把主題當作 submodule：

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git commit -m "Add PaperMod theme"
```

再到 `config` 設定使用這個主題。

---

## 四、設定 Hugo + PaperMod 基礎配置

依 Hugo 版本不同，可能是 `config.toml`、`config.yaml` 或 `config/_default/config.toml`。這裡以 `config.toml` 為例，提供一個「最小可用設定」：

```toml
baseURL = "https://你的github帳號.github.io/"
languageCode = "zh-tw"
title = "我的技術部落格"
theme = "PaperMod"

paginate = 10
enableRobotsTXT = true

[params]
  defaultTheme = "auto" # auto, dark, light
  ShowReadingTime = true
  ShowShareButtons = false
  ShowPostNavLinks = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true

[markup]
  [markup.highlight]
    noClasses = false
```

這裡有兩個重點：

1. `baseURL` 很重要，之後部署到 GitHub Pages 不正常、資源路徑錯誤，80% 都是這裡沒設對。
2. `theme = "PaperMod"` 一定要跟 `themes` 底下的資料夾名稱對到。

---

## 五、本機預覽：確認主題有正確套用

先建立一篇測試文章：

```bash
hugo new posts/hello-world.md
```

這會在 `content/posts/hello-world.md` 生成一個 Markdown 檔，把裡面 content 稍微改一點，確保是發佈狀態：

```markdown
---
title: "Hello World"
date: 2025-12-08T10:00:00+08:00
draft: false
---

這是我的第一篇 Hugo + PaperMod 測試文章。
```

接著啟動本機伺服器：

```bash
hugo server -D
```

在瀏覽器打開 `http://localhost:1313/`，應該就可以看到 PaperMod 的版面，文章列表裡有「Hello World」。

---

## 六、把專案推到 GitHub：建立 Repository

### 1. 在 GitHub 建立 Repository

到 GitHub 新增一個 repo，例如：

- 名稱：`你的github帳號.github.io`（user/organization site 模式），或  
- 名稱：`my-blog`（project site 模式）

初學者最簡單的是用 user site：`username.github.io`，對應的網址會是：

```text
https://username.github.io/
```

### 2. 設定遠端並推送

回到終端機，在 Hugo 專案根目錄：

```bash
git remote add origin https://github.com/你的github帳號/你的repo名稱.git
git branch -M main
git push -u origin main
```

這時候 GitHub 上就有 Hugo 原始碼，但還沒有生成後的靜態網站。

---

## 七、設定 GitHub Pages 自動部署 Hugo

這裡以「用 GitHub Actions 自動編譯 Hugo，輸出到 `gh-pages` branch，再由 GitHub Pages 服務那個 branch」為例。

### 1. 新增 GitHub Actions workflow

在專案建立 `.github/workflows/hugo.yml`：

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: "0.135.0"
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

把 workflow 加入版控並推送：

```bash
git add .github/workflows/hugo.yml
git commit -m "Add Hugo GitHub Pages workflow"
git push
```

### 2. 設定 GitHub Pages 使用 `gh-pages` branch

推上去後，GitHub Actions 應該會自動跑一次，執行成功後會產生 `gh-pages` 分支。

到 GitHub 專案頁面 → Settings → Pages：

- Source 選擇：`Deploy from a branch`
- Branch 選：`gh-pages`，資料夾 `/root`
- 儲存

幾分鐘後，你就可以用瀏覽器打開：

```text
https://你的github帳號.github.io/
```

就會看到 PaperMod 部落格上線。

---

## 八、如何新增文章與日常工作流程

一旦整套流程建立完成，日常寫文與發佈可以固定成一個簡單的步驟：

```bash
# 1. 新增文章
hugo new posts/your-article-slug.md

# 2. 編輯 content（用 VS Code / Neovim / 你習慣的編輯器）

# front matter：
# draft: true 改成 false 或直接用 date 控制發佈時間

# 3. 本機預覽
hugo server -D

# 4. 確認內容沒問題之後
git add .
git commit -m "Add new post: your-article-slug"
git push
```

只要 push 到 `main`，GitHub Actions 就會自動編譯並更新 GitHub Pages，整個部署流程不需要你再手動操作。

---

## 九、常見踩坑與排錯說明

### 1. 網站打開是空白 / CSS 沒載入

可能原因多半跟 `baseURL` 有關：

- 檢查 `config.toml` 的 `baseURL` 是否是 `https://你的github帳號.github.io/`  
- 注意最後的 `/` 不要漏掉。
- 如果是 project site 而不是 user site，例如 `https://你的github帳號.github.io/my-blog/`，那 `baseURL` 要對應完整路徑。

### 2. 主題沒有套用，看起來像 Hugo 預設樣式

通常是這幾個方向：

- 檢查 `theme = "PaperMod"` 是否拼字正確。  
- 檢查 `themes/PaperMod` 資料夾是否存在，submodule 有沒有正確拉下來。  
- 如果有多個 config 檔（例如 `config/_default/config.toml`），要確認修改的是 Hugo 實際讀取的那一個。

### 3. GitHub Actions build 失敗

處理方式：

1. 到 GitHub 專案頁 → Actions → 點進失敗的 workflow。  
2. 看 log 末端的錯誤訊息，常見幾種原因：
   - 指定的 `hugo-version` 太舊或太新，導致某些 config 已不支援。
   - YAML 縮排錯誤。
   - 有多個 output 目錄或自訂 build 指令，與 workflow 的 `publish_dir` 不一致（預設是 `./public`）。

---

## 十、後續可以進階的方向

當 Hugo + PaperMod + GitHub Pages 的基本流程穩定之後，可以考慮幾個進階方向：

- **綁定自訂網域**  
  透過 DNS 把網域 CNAME 指向 `username.github.io`，並在 repo 中加入 `CNAME` 檔案，即可用自己的網域對外。

- **啟用流量分析**  
  可以整合 Google Analytics、Plausible 等工具，追蹤有哪些文章帶來最多流量。

- **RSS、Email 訂閱**  
  利用 Hugo 產生 RSS Feed，或串接第三方 Email 服務，讓讀者可以訂閱你的更新。

- **自訂樣式與元件**  
  在不破壞 PaperMod 結構的前提下，加入自己的 CSS、Shortcodes，讓站點風格更符合個人品牌。

- **多平台部署與備援**  
  除了 GitHub Pages，也可以用相同靜態輸出部署到 Cloudflare Pages、Netlify 等平台，做多處備援或 A/B 測試。

透過這套組合，你可以用最低的維運成本，把技術文章、實作筆記、作品集穩定地放在一個屬於自己的家。

---

有興趣部署透過 vibe coding 製作靜態網頁在 github pages 的朋友
可以參考我這支影片

[![用 GitHub Pages 部署靜態網頁](https://img.youtube.com/vi/LpJ-tq8pwwM/maxresdefault.jpg)](https://www.youtube.com/watch?v=LpJ-tq8pwwM "點我在 YouTube 上觀看完整教學")

https://youtu.be/LpJ-tq8pwwM