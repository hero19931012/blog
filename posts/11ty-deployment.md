---
title: 11ty 部落格部署紀錄
description:
date: 2022-02-20
scheduled: 2022-02-20
tags: 11ty
layout: layouts/post.njk
---

受到 [「星巴哥文章推薦」開發全記錄 — 從 Hexo 到 11ty](https://pse.is/3vf88q) 與 [除了 hexo，也可以考慮用 eleventy 來寫技術部落格 - Huli](https://pse.is/3uztdr) 的啟發，決定動手架一個想了很久但都沒做的個人部落格。

### 環境

透過 [google/eleventy-high-performance-blog](https://github.com/google/eleventy-high-performance-blog) 產生靜態網站，部署在 GitHub Pages 上。

### 步驟

#### 1. 建立 repository

用 [high performance blog template](https://github.com/google/eleventy-high-performance-blog) 當作 template 製作一個自己的 repo，然後 clone 到 local

#### 2. 設定 GitHub Actions

參考 [Eleventy and Github pages](https://www.linkedin.com/pulse/eleventy-github-pages-lea-tortay/) 設定 repo 在 commit 之後要執行的 actions。在編輯器中打開 `.github/workflows/build-and-test.yaml`，加入一個 deploy 的 step:

```yaml
# Runs build and test
name: CI

on:
  push:
	 branches: [main]
  pull_request:
	 branches: [main]

jobs:
  build:
	 runs-on: ubuntu-latest

	 strategy:
		matrix:
		  node-version: [12.x, 14.x]

	 steps:
		- uses: actions/checkout@v2
		- name: Use Node.js ${{ matrix.node-version }}
		  uses: actions/setup-node@v1
		  with:
			 node-version: ${{ matrix.node-version }}
		- run: npm install
		- run: npm run build-ci

		# 加人下面部署的 action
		- name: Deploy
		  uses: peaceiris/actions-gh-pages@v3
		  with:
			 deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
			 publish_dir: ./_site
```

`secrets.ACTIONS_DEPLOY_KEY` 待會會在 GitHub 上面設定。

#### 3. 設定全域 path prefix

在 GitHub Pages 上部署，會有一個屬於自己的 domain `username.github.io`，gh-pages 透過增加一層 subdomain 來區分不同的 repo，如果要訪問一個路徑 `path`，必須用 `username.github.io/repo-name/path`，才能正確存取，因此需要將專案裡所有的 link 加上一個 prefix。

打開 `.eleventy.js`，找到最下面 return 的部分，可以看到 subdirectory 的說明，把原本被註解的 `pathPrefix: "/"` 取消註解，改成 repo name 就可以了。

```javascript
// If your site lives in a different subdirectory, change this.
// Leading or trailing slashes are all normalized away, so don’t worry about those.

// If you don’t have a subdirectory, use "" or "/" (they do the same thing)
// This is only used for link URLs (it does not affect your file structure)
// Best paired with the `url` filter: https://www.11ty.io/docs/filters/url/

// You can also pass this in on the command line using `--pathprefix`
pathPrefix: "/repo-name"
```

#### 4. 產生與設定 Deploy key

終端機中用下面的指令產生 2 支檔案：**gh-pages.pub** (public key) 和 **gh-pages** (private key)

```bash
ssh-keygen -t rsa -b 4096 -C "$(git config user.email)" -f gh-pages -N ""
```

進入 repo Settings 頁面

![settings-tab]({{ '/img/11ty-deployment/settings-tab.png' | url }})

選擇 Security > Deploy Keys > Add deploy key

![deploy-key]({{ '/img/11ty-deployment/deploy-key.png' | url }})

填入 title 與 key (複製 gh-pages.pub 的內容)，然後勾選 `Allow write access`，允許修改權限

![add-deploy-key]({{ '/img/11ty-deployment/add-deploy-key.png' | url }})

接著到 Security > Secrets > Actions，點選 New repository secret，將 Name 填入 ACTIONS_DEPLOY_KEY，key 則複製 gh-pages 的內容貼上

![add-secret]({{ '/img/11ty-deployment/add-secret.png' | url }})

#### 5. Commit 並推上 GitHub

回到本機，先把剛剛的修改 commit，然後 git push 上 Github

#### 6. 確認 Action 有正常執行

檢查 repo Actions 頁面，可以看到所有正在執行或之前已經跑好的 workflows，並會根據 commit 來命名，這邊會看到 2 個分別是 CI 與 CodeQL 的 workflow，是 eleventy-high-performance-blog 內建的

![default-actions]({{ '/img/11ty-deployment/default-actions.png' | url }})

這 2 個跑好後會出現第 3 個名為 pages-build-deployment 的 workflow，是將網站部署到 gh-pages 的 workflow

![deploy-action]({{ '/img/11ty-deployment/deploy-action.png' | url }})

點進去可以看 log 了解現在在什麼階段，完成後在 deploy 的區塊下會有部署完的網址，沒意外是 `https://username.github.io/repo-name`

![deploy-action-detail]({{ '/img/11ty-deployment/deploy-action-detail.png' | url }})

#### 7. 部署完成

網址列輸入該網址，沒出現 404 而是以下畫面的話就是部署成功了

![deploy-success]({{ '/img/11ty-deployment/deploy-success.png' | url }})

### 後記

這篇記錄只提到如何透過 GitHub Pages 搭配 GitHub Actions 來部署 11ty 網站，沒提到設計與客製頁面的開發是由於我只想先快速簡單弄一個可以放筆記的平台，所以目前都還沒客製自己的頁面，之後有需要會慢慢新增。

然後部署在 GitHub Pages 上有一個已知但不知道如何解決的問題，就是圖片在 markdown 裡面沒辦法套用 path prefix 的設定，也就會抓不到圖，猜測如果有買 domain 應該就可以解決這個問題，但我沒有 domain，所以目前只能 workaround，就是利用 11ty 的 markdown 會先經過 liquid template engine 編譯的設定，直接在 markdown 的圖片路徑上使用 filter，寫起來如下 (不含 backslash)：

```md
![demo](\{\{ '/img/first-post/abc.png' | url \}\})
```

缺點就是不太好看，也沒辦法 preview (還是可以用 npm run watch 在瀏覽器裡 preview 啦)，我研究了一整天，想要找到能夠對 markdown 裡面的 img 路徑加上 prefix 的方法，但是沒找到，甚至還想說自己寫個 markdown-it 的 plugin 好了，後來覺得太麻煩作罷，就先接受這樣子了。

以上就是粗略的 11ty 部落格架設記錄。

### References

[Eleventy and Github pages | LinkedIn](https://www.linkedin.com/pulse/eleventy-github-pages-lea-tortay/)

[How to Deploy Eleventy to GitHub Pages With GitHub Actions | rockyourcode](https://www.rockyourcode.com/how-to-deploy-eleventy-to-github-pages-with-github-actions/)

[google/eleventy-high-performance-blog: A high performance blog template for the 11ty static site generator. (github.com)](https://github.com/google/eleventy-high-performance-blog)
