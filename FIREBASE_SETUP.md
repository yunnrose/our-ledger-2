# Firebase Realtime Database 正式連線設定（GitHub Pages + 匿名登入）

本專案使用 Firebase Authentication Anonymous provider 與 Realtime Database。前端仍部署於 GitHub Pages；Firebase config 由帳本「設定與資料管理」頁面輸入並保存在該瀏覽器的 LocalStorage，不需要提交任何私密金鑰到 GitHub。

## 1. Firebase Console 設定

1. 開啟 Firebase Console，選擇帳本使用的 Firebase 專案。
2. Authentication → Sign-in method → 啟用 **Anonymous**。
3. Authentication → Settings → Authorized domains，加入 GitHub Pages 主機網域，例如 `yunnrose.github.io`。若有自訂網域，也要加入該網域。
4. Realtime Database → 建立或選擇資料庫，確認資料庫網址與 `firebaseConfig.databaseURL` 完全一致。
5. 在 Project settings → General → Your apps → Web app，複製 Firebase config。

## 2. 先部署安全規則（預設拒絕非白名單 UID）

將本專案的 `database.rules.json` 內容貼到 Firebase Console → Realtime Database → Rules。兩個 placeholder 必須替換成兩台裝置登入後顯示的真實 UID：

- `REPLACE_WITH_PEiyun_UID`
- `REPLACE_WITH_BENSON_UID`

在 UID 尚未替換前，資料庫讀寫會被拒絕，這是預期的 fail-closed 行為。不要改成 `auth != null` 作為正式私人帳本規則，否則任何人都能開啟公開網頁、匿名登入並讀寫帳本。

## 3. 取得兩個匿名 UID

1. 開啟 GitHub Pages 帳本網址。
2. 進入「設定與資料管理」→ Firebase Realtime Database 設定。
3. 貼上 Firebase config JSON，按「儲存並連線 Firebase」。
4. 登入成功後，畫面會顯示「匿名登入 UID」。複製佩筠這台裝置的 UID。
5. Benson 在自己的裝置與瀏覽器執行相同步驟，複製另一個 UID。
6. 把兩個 UID 分別貼回 Firebase Rules 的兩個 placeholder，按 Publish。
7. 重新整理兩台裝置上的帳本。UID 應保持不變，因為 Firebase Auth 使用本機持久化登入狀態。

## 4. 匯入既有本機資料

- 若雲端已有 transactions，頁面會以雲端資料為準。
- 若雲端 transactions 為空、但目前瀏覽器有舊的本機交易，設定頁會顯示「將本機舊資料匯入雲端」。
- 按下後會再次檢查雲端 transactions 與 logs 都是空的；只在雲端兩者皆空時才匯入，不會覆蓋已存在的雲端帳本。
- 匯入前仍建議先使用「匯出 JSON 備份」保存一份離線備份。

## 5. 安全限制與風險

- Firebase Web API key 是前端專案識別資訊，不是資料庫密碼；真正的存取控制由 Realtime Database Rules 執行。
- 匿名登入本身不證明使用者是佩筠或 Benson。安全性來自 UID 白名單：只有已登記的兩個匿名 UID 可以讀寫。
- 若清除瀏覽器資料、換瀏覽器、換裝置或匿名帳號遺失，Firebase 可能建立新的 UID。新 UID 不在白名單時會被拒絕；需由資料庫管理者確認後更新 Rules。
- 帳本目前的「我是佩筠／我是 Benson」只是前端操作標籤，不能當成身分驗證。任何獲准的 UID 都能切換標籤並修改共同帳本。
- 任何取得其中一台已授權裝置存取權的人，都可能以該 UID 讀寫帳本。請保護裝置及瀏覽器個人資料。
- 公開 GitHub Pages 的 HTML/JavaScript 可被任何人檢視。不要在前端放 Firebase Admin SDK 私鑰、服務帳戶金鑰或其他伺服器憑證。
- 建議定期匯出 JSON 備份，並在 Firebase Console 檢查用量與異常存取。

## 6. 本次程式調整

- 保留 Firebase Authentication 匿名登入。
- 增加匿名 UID 顯示與複製功能，方便建立 UID 白名單。
- Firebase 連線設定錯誤時顯示明確狀態。
- Firebase 已設定但尚未連線時，不再把新增／修改／刪除默默當成本機成功保存。
- 新瀏覽器不再自動載入示範帳務資料。
- 雲端交易／稽核資料為空時，先保留本機舊資料，提供經二次確認且會檢查雲端是否仍為空的遷移操作。
- 保留既有 `transactions/`、`logs/` 資料路徑與帳本 UI。
