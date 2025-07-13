---
title: "使用 Hugo + Netlify 建立部落格記錄"
date: 2025-07-13T22:30:00+08:00
draft: false
tags: ["Hugo", "Netlify", "部落格", "靜態網站"]
categories: ["技術筆記"]
description: "記錄使用 Hugo 靜態網站生成器搭配 Netlify 建立部落格的完整流程，包含常見問題解決方案"
---

## 前置準備

### 1. 安裝 Hugo

```bash
# macOS
brew install hugo

# Windows (使用 Chocolatey)
choco install hugo-extended

# Linux (Ubuntu/Debian)
sudo apt install hugo
```

### 2. 建立 Hugo 網站

```bash
hugo new site my-blog
cd my-blog
```

### 3. 安裝主題 (以 PaperMod 為例)

```bash
git init
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

## 設定檔配置

### hugo.toml 基本設定

```toml
baseURL = 'http://localhost:1313'  # 本地開發用
languageCode = 'zh-tw'
title = 'My Blog'
theme = 'PaperMod'

[params]
env = "production"
title = "My Blog"
description = "My Hugo Blog"
ShowReadingTime = true
ShowShareButtons = true
ShowPostNavLinks = true
ShowBreadCrumbs = true

[params.homeInfoParams]
Title = "歡迎來到我的部落格 👋"
Content = "這裡分享我的學習心得和技術筆記"

[[menu.main]]
identifier = "home"
name = "首頁"
url = "/"
weight = 10

[[menu.main]]
identifier = "posts"
name = "文章"
url = "/posts/"
weight = 20

[[menu.main]]
identifier = "tags"
name = "標籤"
url = "/tags/"
weight = 30
```

## 建立文章

### 新增文章

```bash
hugo new posts/test-post.md
```

### 文章格式範例

```markdown
---
title: "測試文章"
date: 2025-07-13
draft: false
tags: ["Hugo"]
---

## 開始使用 Hugo

Hugo 是一個靜態網站生成器，這意味著它可以將純文本文件轉換為靜態網站。這些文件通常是用 Markdown 寫的，這是一種輕量級的標記語言，非常適合寫作。
```

### 本地測試

```bash
hugo server -D

# 測試後先 init commit
git add .
git commit -m "Initial Hugo site setup"
```

## Netlify 部署設定

### 關鍵：建立 netlify.toml

在專案根目錄建立 `netlify.toml` 檔案：

```toml
[build]
  command = "hugo --minify --baseURL $DEPLOY_PRIME_URL"
  publish = "public"

[build.environment]
  HUGO_VERSION = "0.148.1"
  HUGO_EXTENDED = "true"
  NODE_VERSION = "18"

[context.production]
  command = "hugo --minify --baseURL $URL"

[context.production.environment]
  HUGO_ENV = "production"

[context.deploy-preview]
  command = "hugo --buildFuture --baseURL $DEPLOY_PRIME_URL"

[context.branch-deploy]
  command = "hugo --buildFuture --baseURL $DEPLOY_PRIME_URL"

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
```

## 部署流程

### 1. 推送到 Git

```bash
git add .
git commit -m "Add netlify config and first post"
git push origin main
```

### 2. 連接 Netlify 部署

1. 登入 [Netlify](https://netlify.com/)
2. 點擊 "New site from Git"
3. 選擇 Git 提供商（GitHub/GitLab/Bitbucket）
4. 選擇你的 repository
5. 建置設定會自動從 `netlify.toml` 讀取
6. 點擊 "Deploy site"

## 常見問題與解決方案

### 問題 1：Hugo command not found

**現象：** Netlify 建置時出現 `hugo: command not found`

**原因：** Netlify 建置環境預設沒有安裝 Hugo

**解決方案：** 建立 `netlify.toml` 並指定 Hugo 版本（如上面設定）


### 問題 2：文章連結指向 localhost

**現象：** 部署後點擊文章會跳轉到 `localhost:1313`

**原因：** `hugo.toml` 中的 `baseURL` 設定為 localhost

**解決方案：** 使用 `netlify.toml` 中的環境變數動態設定 baseURL：

```toml
[context.production]
  command = "hugo --minify --baseURL $URL"
```

## 總結

使用 Hugo + Netlify 建立部落格的關鍵要點：

1. **netlify.toml 是必要的** - 解決 Hugo 版本和 baseURL 問題
2. **文章記得設定 `draft: false`** - 才會在生產環境顯示
3. **善用 Netlify 的環境變數** - 讓本地開發和生產環境都能正常運作

整體來說，Hugo 的建置速度很快，搭配 Netlify 的自動部署功能，可以很輕鬆地維護一個靜態部落格！