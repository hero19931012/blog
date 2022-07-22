---
title: 設定 GitHub SSH 連線，並在同一台電腦設定不同 GitHub 帳號
description:
date: 2022-07-22
scheduled: 2022-07-22
tags: ssh, git, GitHub
layout: layouts/post.njk
---

## 前言

SSH key 用來與 GitHub 連線，不用像網頁登入需要輸入帳號密碼。

由於工作與個人在 GitHub 使用不同的帳號，且公司有提供筆電，即使只在家使用也自由度大增，有時想做點自己的小專案，會需要在不同帳號間轉換 (其實是很懶不想一直換電腦)，所以參考別人的方法完成設定 SSH 連線與切換帳號，完成後以我自己的角度來記錄這篇心得。

## 步驟

1. 打開 terminal 透過`ssh-keygen` 搭配 `RSA` 演算法產生一對金鑰 (官方文件用的是 `ed25519`，`RSA` 對較舊的系統相容性較好，詳細可閱讀參考文件)

    ```bash
    cd ~/.ssh
    ssh-keygen -t rsa -C "userName@address"
    ```

    中間會詢問要存放的路徑與檔名，通常預設路徑在會 `/Users/{username}/.ssh/id_rsa`

    ```bash
    >Enter a file in which to save the key (/Users/username/.ssh/id_rsa): [Press enter]
    ```
    - username 是目前的使用者名稱 (由於剛剛執行過 `cd ~/.ssh`，可以直接用 `pwd` 取得當前路徑)
    - `id_rsa` 是根據目前使用的演算法產生
    - 不輸入直接 Enter 套用預設值

    然後提示輸入 passphrase (不輸入也可，直接 Enter)

    ```bash
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```

2. 接著會產生 2 支檔案：`id_rsa` 與 `id_rsa.pub`，接下來把 `id_rsa.pub` 的內容貼到 GitHub 上新增一把公鑰

    Settings > SSH and GPG keys

    ![](https://i.imgur.com/QG7envj.png)

    New SSH keys

    ![](https://i.imgur.com/kkNtkQA.png)

    取一個名字然後把 `id_rsa.pub` 的內容貼上來

3. 回到 terminal，把私鑰 `id_rsa` 加入 `ssh-agent`

    ```bash
    ssh-add ~/.ssh/id_rsa
    ```

4. 由於我們需要存取 2 個帳號，所以再把 1-3 步再做一次

    金鑰名字另外取，例如 `id_rsa_personal`，再去另一個 GitHub 帳號添加一次公鑰

5. 用 vim 打開 ~/.ssh/config

    如果沒有這個檔案可以先執行 `touch config`

    ```bash
    vim config
    ```

    輸入下面內容

    ```bash
    #Default GitHub
    Host github.com                      # 代號
     HostName github.com                 # IP or domain name
     User git                            # username => git@github.com[]
     IdentityFile ~/.ssh/id_rsa          # 指定的金鑰

    #New GitHub
    Host github-personal
     HostName github.com
     User git
     IdentityFile ~/.ssh/id_rsa_personal
    ```

    意思是針對不同的 host 使用不同的 key 進行 SSH 連線，

6. 測試連線

    ```bash
    $ssh -T git@github.com
    Hi User1! You’ve successfully authenticated, but GitHub does not provide shell access.

    $ssh -T git@github-personal
    Hi User2! You’ve successfully authenticated, but GitHub does not provide shell access.
    ```

7. Clone repository

    選擇 SSH 的方式 clone 就可以了

    ![](https://i.imgur.com/p2vdSxz.png)
    
    ```bash
    git clone git@github.com:{username}/{repository}
    git clone git@github-personal:{username}/{repository}
    ```

## Reference

1. [如何在一台電腦使用多個 Git 帳號](https://medium.com/@hyWang/%E5%A6%82%E4%BD%95%E5%9C%A8%E4%B8%80%E5%8F%B0%E9%9B%BB%E8%85%A6%E4%BD%BF%E7%94%A8%E5%A4%9A%E5%80%8Bgit%E5%B8%B3%E8%99%9F-907c8eadbabf)
1. [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
1. [Testing your SSH connection](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/testing-your-ssh-connection)
1. [選擇 SSH key 的加密演算法](https://medium.com/@honglong/%E9%81%B8%E6%93%87-ssh-key-%E7%9A%84%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95-70ca45c94d8e)
1. [增進 SSH 使用效率 - ssh_config](https://chusiang.gitbooks.io/working-on-gnu-linux/content/20.ssh_config.html)
1. [How to setup SSH config ：使用 SSH 設定檔簡化指令與連線網址](https://medium.com/%E6%B5%A6%E5%B3%B6%E5%A4%AA%E9%83%8E%E7%9A%84%E6%B0%B4%E6%97%8F%E7%BC%B8/how-to-setup-ssh-config-%E4%BD%BF%E7%94%A8-ssh-%E8%A8%AD%E5%AE%9A%E6%AA%94-74ad46f99818)
1. [Using the SSH Config File](https://linuxize.com/post/using-the-ssh-config-file/)