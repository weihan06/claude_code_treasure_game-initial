---
description: 將專案建置並部署到 Vercel（production），完成後回報網址
allowed-tools: Bash(npm:*), Bash(npx:*), Bash(vercel:*), Read, Write, Edit
---

請將此專案部署到 Vercel production，並在最後回報部署網址。依照以下步驟執行：

## 1. 檢查 Vercel CLI 與登入狀態

- 執行 `npx vercel --version` 確認 CLI 可用。
- 執行 `npx vercel whoami` 確認已登入。
  - 若尚未登入，**停止執行**，請使用者在提示列輸入 `! npx vercel login` 自行完成互動式登入後再重新執行 `/deploy_vercel`。

## 2. 確認 Vercel 設定

本專案的 `vite build` 輸出到 `./build`（不是 Vite 預設的 `./dist`），Vercel 預設會找不到輸出目錄。

- 若專案根目錄沒有 `vercel.json`，建立以下內容：

```json
{
  "framework": "vite",
  "buildCommand": "npm run build",
  "outputDirectory": "build"
}
```

- 若已存在，確認 `outputDirectory` 為 `build`。

## 3. 本機先建置驗證

- 若沒有 `node_modules`，先執行 `npm install`。
- 執行 `npm run build`，確認建置成功。若失敗，回報錯誤並停止，不要部署。

## 4. 部署到 production

- 執行 `npx vercel --prod --yes`
  - `--yes` 會在第一次部署時自動建立並連結 Vercel 專案（產生 `.vercel/` 目錄）。
- 部署可能需要 1～2 分鐘，請等待指令完成。

## 5. 回報結果

部署成功後，從指令輸出中擷取網址，用以下格式回報：

```
✅ 部署成功
Production 網址: https://<專案名稱>.vercel.app
本次部署網址:   https://<本次部署的唯一網址>
```

若部署失敗，顯示關鍵錯誤訊息並說明可能的原因與修正方式。
