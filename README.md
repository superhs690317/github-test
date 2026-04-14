# Claude Code × GitHub 連接完整過程紀錄

**日期：** 2026-04-14  
**執行者：** Handsome（GitHub: superhs690317）  
**目標：** 在 Windows 11 上完成 Claude Code 與 GitHub 的完整連接設定

---

## 目錄

1. [環境說明](#1-環境說明)
2. [步驟流程總覽](#2-步驟流程總覽)
3. [詳細過程](#3-詳細過程)
4. [問題與排除紀錄](#4-問題與排除紀錄)
5. [最終環境狀態](#5-最終環境狀態)

---

## 1. 環境說明

| 項目 | 內容 |
|------|------|
| 作業系統 | Windows 11 Home 10.0.26200 |
| Shell | bash (Git Bash / MINGW64) |
| 工作目錄 | `C:\Users\super\Documents\ClaudeCowork\GitHub` |
| 執行工具 | Claude Code 桌面版 |

---

## 2. 步驟流程總覽

| # | 步驟 | 結果 |
|---|------|------|
| 0 | 環境檢查 | ✅ |
| 1 | 安裝 GitHub CLI | ✅（需重啟） |
| 2 | GitHub 帳號登入 | ✅ |
| 3 | 設定 Git 使用者資訊 | ✅ |
| 4 | 建立測試 repo 並推送 | ✅（遇到 push 卡住問題） |
| 5 | 啟用 GitHub Pages | ✅ |
| 6 | 授權 delete_repo 並刪除測試 repo | ✅ |
| 7 | 重建 repo 並上傳完整紀錄 | ✅ |

---

## 3. 詳細過程

### 步驟 0：環境檢查

執行指令：
```bash
git --version
gh --version
ping github.com
git config --global user.name
git config --global user.email
```

結果：

| 項目 | 狀態 | 說明 |
|------|------|------|
| 作業系統 | ✅ | Windows 11 |
| 網路連線 | ✅ | 可連到 github.com，延遲 38ms |
| Git | ✅ | v2.53.0.windows.2 已安裝 |
| GitHub CLI | ❌ | 未安裝，需要安裝 |
| Git 使用者資訊 | ❌ | 從未設定，需要設定 |

---

### 步驟 1：安裝 GitHub CLI

**問題：** 系統未安裝 GitHub CLI（`gh: command not found`）

**解決指令：**
```bash
winget install --id GitHub.cli --accept-source-agreements --accept-package-agreements
```

**安裝結果：** gh v2.89.0 安裝成功

**遇到的問題 #1：安裝後仍然 `gh: command not found`**
- **原因：** Windows 安裝新程式後，PATH 不會在同一個 shell session 內即時更新
- **解決方式：** 重新啟動 Claude Code 桌面版，重開後 gh 指令正常運作

---

### 步驟 2：GitHub 帳號登入

**指令：**
```bash
gh auth login --web --git-protocol https
```

**流程：**
1. 系統產生一次性驗證碼
2. 前往 https://github.com/login/device 輸入驗證碼
3. 在瀏覽器點選「Authorize GitHub CLI」

**驗證碼：** `E062-92A0`

**登入結果：**
```
✓ Logged in to github.com account superhs690317 (keyring)
- Active account: true
- Git operations protocol: https
- Token scopes: 'gist', 'read:org', 'repo'
```

---

### 步驟 3：設定 Git 使用者資訊

**指令：**
```bash
git config --global user.name "Handsome"
git config --global user.email "superhs690317@gmail.com"
```

**確認：**
```bash
git config --global user.name   # → Handsome
git config --global user.email  # → superhs690317@gmail.com
```

---

### 步驟 4：建立測試 Repo 並推送

**建立本地 repo：**
```bash
mkdir ~/Documents/github-test
cd ~/Documents/github-test
git init
# 建立 index.html 測試頁
git add index.html
git commit -m "初始測試頁面"
```

**推送到 GitHub：**
```bash
gh repo create github-test --public --source=. --push
```

**結果：**
```
https://github.com/superhs690317/github-test
* [new branch] HEAD -> master
branch 'master' set up to track 'origin/master'.
```

**遇到的問題 #2：git push 卡住不動**
- **原因：** Windows 上 git HTTPS push 時，Git 嘗試開啟 GUI 憑證視窗等待輸入，但在 Claude Code 環境中無法顯示互動式視窗，導致程序無限期等待
- **診斷方式：** 觀察到背景任務長時間沒有輸出，用 `gh api` 確認 GitHub 上的 commit 未更新
- **解決方式：** 執行 `gh auth setup-git` 設定憑證 helper 後，改用 token 直接帶入 remote URL 推送：
  ```bash
  TOKEN=$(gh auth token)
  git remote set-url origin "https://${TOKEN}@github.com/superhs690317/github-test.git"
  GIT_TERMINAL_PROMPT=0 git push origin master
  git remote set-url origin "https://github.com/superhs690317/github-test.git"
  ```
- **根本原因：** `gh auth setup-git` 雖然設定了 credential helper，但 Windows 上 git 的憑證快取機制仍可能優先觸發 GUI 提示

---

### 步驟 5：啟用 GitHub Pages

**指令：**
```bash
gh api repos/superhs690317/github-test/pages \
  -X POST \
  -f build_type=legacy \
  -f "source[branch]=master" \
  -f "source[path]=/"
```

**結果：** Pages 啟用成功  
**網址：** https://superhs690317.github.io/github-test/

---

### 步驟 6：授權 delete_repo 並刪除測試 repo

**問題：** 執行 `gh repo delete` 時出現 HTTP 403 錯誤

**原因：** 初次登入時的 token 不含 `delete_repo` 權限

**解決：**
```bash
gh auth refresh -h github.com -s delete_repo
```

同樣需要瀏覽器授權，驗證碼：`CCD8-E3B0`

**刪除指令：**
```bash
gh repo delete superhs690317/github-test --yes
```

---

### 步驟 7：重建 Repo 並上傳完整紀錄

重新建立乾淨的 repo，撰寫本完整紀錄後上傳。

**推送方式（同步驟 4 的解決方案）：**
```bash
TOKEN=$(gh auth token)
git remote set-url origin "https://${TOKEN}@github.com/superhs690317/github-test.git"
GIT_TERMINAL_PROMPT=0 git push origin master
git remote set-url origin "https://github.com/superhs690317/github-test.git"
```

---

## 4. 問題與排除紀錄

| # | 問題描述 | 發生時機 | 原因 | 解決方式 |
|---|----------|----------|------|----------|
| 1 | `gh: command not found` | 安裝完 gh 後立即使用 | Windows PATH 不即時更新 | 重啟 Claude Code |
| 2 | `git push` 卡住不動 | 推送到 GitHub 時 | Git GUI 憑證視窗無法在 Claude Code 中顯示 | 用 `gh auth token` 產生 token 帶入 remote URL |
| 3 | `gh repo delete` 回傳 HTTP 403 | 刪除 repo 時 | Token 不含 `delete_repo` scope | `gh auth refresh -s delete_repo` 重新授權 |

---

## 5. 最終環境狀態

```
Git:            v2.53.0.windows.2
GitHub CLI:     v2.89.0
GitHub 帳號:    superhs690317
user.name:      Handsome
user.email:     superhs690317@gmail.com
Protocol:       HTTPS
Token scopes:   gist, read:org, repo, delete_repo
```

---

## 備註：日後 push 的建議做法

若日後遇到 `git push` 卡住，可用以下指令快速推送：

```bash
TOKEN=$(gh auth token)
git remote set-url origin "https://${TOKEN}@github.com/<用戶名>/<repo名>.git"
GIT_TERMINAL_PROMPT=0 git push origin master
git remote set-url origin "https://github.com/<用戶名>/<repo名>.git"
```

或執行 `gh auth setup-git` 重新設定憑證 helper 後再試。
