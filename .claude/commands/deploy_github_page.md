---
description: 檢查前置需求、將專案推上 GitHub repo，並部署到 GitHub Pages，完成後回報網址
argument-hint: "[repo 名稱，預設為目前資料夾名稱]"
allowed-tools: Bash(git:*), Bash(gh:*), Bash(npm:*), Bash(npx:*), Bash(winget:*), Read, Edit, Write
---

請將此專案推送到 GitHub，並部署到 GitHub Pages。使用者指定的 repo 名稱：`$ARGUMENTS`（若為空，使用目前資料夾名稱）。

**原則：** 每一步先檢查狀態，已完成就跳過。凡是需要互動輸入（登入、瀏覽器授權）的步驟，**停止執行**並請使用者在提示列輸入 `! <指令>` 自行完成，完成後再重新執行 `/deploy_github_page`。不要替使用者猜測帳號或 email。

## 1. 檢查工具

- `git --version`：若沒有 git，請使用者安裝（https://git-scm.com/download/win）後停止。
- `gh --version`：若沒有 GitHub CLI：
  - Windows：詢問使用者是否同意執行 `winget install --id GitHub.cli -e`，安裝後提醒使用者**需重新開啟終端機 / Claude Code** 讓 `gh` 進入 PATH，然後停止。
  - 其他系統：提供 https://cli.github.com 安裝說明後停止。

## 2. 檢查 GitHub 登入

- 執行 `gh auth status`。
- 若尚未登入，停止並請使用者輸入：
  ```
  ! gh auth login --hostname github.com --git-protocol https --web
  ```
- 登入後執行 `gh auth setup-git`，讓 git push 使用 gh 的憑證。
- 用 `gh api user --jq .login` 取得 GitHub 帳號（以下稱 `OWNER`）。

## 3. 檢查 git 身分設定

- 檢查 `git config user.name` 與 `git config user.email`。
- 若任一未設定，**詢問使用者**要使用的名稱與 email（可建議使用 GitHub 提供的 noreply email：`<id>+<OWNER>@users.noreply.github.com`，id 可由 `gh api user --jq .id` 取得），取得同意後用 `git config --global` 設定。

## 4. 初始化本機 repo 並提交

- 若不是 git repo（`git rev-parse --is-inside-work-tree` 失敗），執行 `git init -b main`。
- 確認 `.gitignore` 包含 `node_modules/`、`build/`、`.vercel`；缺少就補上。
- 執行 `git status`，確認沒有要被提交的機密檔（如 `.env`）。有疑慮時先詢問使用者。
- 若有未提交的變更，`git add -A` 並 commit（第一次可用訊息 `Initial commit`；之後用簡短描述變更的訊息）。commit 訊息結尾加上：
  ```
  Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>
  ```

## 5. 建立或連結 GitHub repo

- 決定 repo 名稱 `REPO`（`$ARGUMENTS` 或資料夾名稱），向使用者確認一次名稱。
- 若 `git remote get-url origin` 已存在，直接使用該 remote（從 URL 解析出 `OWNER/REPO`）。
- 否則檢查 `gh repo view OWNER/REPO`：
  - 已存在：`git remote add origin https://github.com/OWNER/REPO.git`
  - 不存在：`gh repo create REPO --public --source=. --remote=origin`
  - 必須是 **public**（免費方案的 GitHub Pages 只支援 public repo）。若使用者要 private，說明需要 GitHub Pro 等付費方案。
- 推送原始碼：`git push -u origin main`

## 6. 建置（GitHub Pages 子路徑）

GitHub Pages 的專案網址是 `https://OWNER.github.io/REPO/`，資源必須用 `/REPO/` 作為 base path，否則頁面會空白。為了不影響 Vercel 部署，**不要修改 `vite.config.ts`**，改用指令參數：

```
npm install            # 若沒有 node_modules
npm run build -- --base=/REPO/
```

建置失敗就回報錯誤並停止。

## 7. 部署到 gh-pages 分支

```
npx --yes gh-pages -d build --nojekyll -m "Deploy to GitHub Pages"
```

這會把 `build/` 推到 `gh-pages` 分支。

## 8. 啟用 GitHub Pages

- 檢查 `gh api repos/OWNER/REPO/pages`：
  - 回傳 404（尚未啟用）：
    ```
    gh api -X POST repos/OWNER/REPO/pages -f "source[branch]=gh-pages" -f "source[path]=/"
    ```
  - 已啟用但來源不是 `gh-pages`：
    ```
    gh api -X PUT repos/OWNER/REPO/pages -f "source[branch]=gh-pages" -f "source[path]=/"
    ```
- 第一次發佈約需 1～2 分鐘。用 `gh api repos/OWNER/REPO/pages/builds/latest --jq .status` 查詢，直到狀態為 `built`（每 15 秒查一次，最多約 3 分鐘；若出現 `errored` 則回報錯誤）。
- 用 `curl -s -o /dev/null -w "%{http_code}" https://OWNER.github.io/REPO/` 確認回傳 200。

## 9. 回報結果

```
✅ 部署成功
GitHub Repo:    https://github.com/OWNER/REPO
Clone URL:      https://github.com/OWNER/REPO.git
GitHub Pages:   https://OWNER.github.io/REPO/
```

若任何步驟失敗，顯示關鍵錯誤訊息、說明可能原因與修正方式，並告訴使用者修正後重新執行 `/deploy_github_page` 即可從中斷處繼續。
