# 真實事故累積的硬傷清單

動到對應領域**前先讀這份**。每條都是 production 踩過的。

## 1. 位元組 vs 字元（多語系長度誤判）

- 症狀：中文 / CJK / emoji 留言被誤擋成 `content_too_long`，明明遠低於「N 字」上限。
- 成因：用語言內建的 byte 長度（Go `len(string)`、許多語言的 byte length）檢查「N 字」，CJK 一字 3 bytes → 超限；DB 卻用 `char_length`（字元）。
- 修正：應用層也用**字元數**（runes，Go `utf8.RuneCountInString`），與 DB CHECK 同口徑。
- 教訓：任何「長度上限」都要先問清楚是**字元還是位元組**，並讓所有檢查點一致。

## 2. 反向代理 / Tunnel 後拿不到真實 client IP

- 症狀：access log / 來源 IP 稽核 / per-user rate-limit 全部記成內網（docker / proxy）IP。
- 成因：nginx `real_ip` 只信任「CDN 公網網段」，但實際對端是 tunnel / proxy 容器的內網 IP，不在信任清單 → 不採用 `CF-Connecting-IP`，`$remote_addr` 停在內網 IP。
- 修正：`set_real_ip_from` 加入 tunnel / docker 私有網段；並把 `X-Real-IP $remote_addr` 轉發給上游。
- 連帶：把對外埠綁 `127.0.0.1`（只留 tunnel / operator 走），避免 LAN 直連偽造來源 IP header。

## 3. 多 stack 共用 compose project → `--remove-orphans` 互刪（曾使 prod 全掛）

- 症狀：重開機 / 某次 `up` 後，**整個 app stack 容器消失**（連 stopped 都沒有），只剩另一組 stack。公開域名 5xx。
- 成因：app 與 observability 兩組 stack 共用同一個 compose **project 名稱**；其中一組以 `--remove-orphans` 拉起，把「不在自己檔裡」的另一組容器當 orphan 移除。
- 修正：兩組各自獨立 project 名稱；用 override 檔讓附屬 stack 重用既有 volume（保留歷史）+ 接上主 stack 的 external network（讓 Prometheus 仍能 scrape）。**永遠不要**對任一 project 下 `--remove-orphans`。
- 救命：named volume 不會被「移除容器」連帶刪 → 資料還在，重新 `up` 即復原。

## 4. nginx upstream 容器換 IP → 502

- 症狀：recreate 了 api / 上游容器後，nginx 一直回 502。
- 成因：nginx 在啟動時解析 upstream 容器 IP 並快取；容器 recreate 換了 IP，nginx 還用舊的。
- 修正：recreate 上游後**重啟 / reload nginx**（部署腳本應在相依服務被 recreate 時自動 bounce nginx；手動 `up -d` 要自己記得）。

## 5. 靜態站掛在子路徑（reverse proxy 剝前綴）

- 症狀：站台掛 `/docs/`，但點內頁被 301 導到少了 `/docs/` 前綴的網址 → 404 / loop。
- 成因：外層 nginx `proxy_pass http://up/;`（尾斜線）剝掉 `/docs/`；內層對「目錄」自動 301 補尾斜線，那個 Location 是內層路徑（沒前綴）外洩給瀏覽器。
- 修正：靜態站產生器設 `trailingSlash: false`（輸出 `<route>.html`），內層 `try_files $uri $uri.html /404.html` 直接命中、不觸發目錄 301。根路徑用 `return 302 /docs/` 導入；要保留 scheme 時加 `absolute_redirect off;`。

## 6. 第三方 cookie / 嵌入式登入

- 嵌入別站的元件靠 `SameSite=None` 第三方 cookie 維持登入，正被瀏覽器淘汰。
- 規劃時就要考慮 FedCM / Storage Access API / CHIPS（partitioned cookie），或改用非 cookie 的短期 token。

## 7. 部署 metadata 是 release 成功的判斷

- `/readyz` 顯示 `version=dev` 或 `commit/built_at=unknown` → image 不是用正式 metadata build 的，**視為部署失敗**。metadata 要在 build/push 時注入。

## 8. 測試在 async 下要對帳 + 容忍限流

- 同步改 async 後，黑箱測試要**輪詢**直到 worker 落 DB（用 `event_id` / 內容比對），不能假設 POST 後立刻可讀。
- 測試打得比人類快，會撞 rate-limit；功能性測試的 helper 要對 `429` 自動重試（rate-limit 本身另用專屬測試以原始請求驗證）。

## 9. 新增服務 / image role 的權限缺口

- 新 image role 上 registry 前：repo 要先建、**push 身分**與**pull 身分**（常是不同 IAM user）都要被授權涵蓋新 repo，否則 release 會卡在 AccessDenied。

## 10. 磁碟被 docker 塞爆 → DB crash（看似登入 / 應用層 bug）

- 症狀：登入回 `user_upsert_failed`（或任何 DB 呼叫失敗）；DB 日誌 `No space left on device`；`/readyz` 顯示 `postgres down`；反向代理後的 app 用容器名連 DB 卻「解析不到 / server misbehaving」——因為 DB 容器在 crash 迴圈、根本不在 docker 網路上。cache（Redis）同時 MISCONF（無法把快照寫到磁碟而擋寫）。
- 成因：**頻繁 release 累積 docker build cache + 未使用的舊 image tag** 把磁碟塞滿（曾把 prod `/` 撐到 100%）。
- 修正：`docker builder prune -af` + `docker image prune -af`（只清未被任何 running 容器使用的；named volume 不受影響）→ DB 有空間後自動脫離 crash 迴圈；cache 觸發一次成功寫檔（如 `redis-cli BGSAVE`）即清除 MISCONF。
- 預防：**部署成功後自動 `docker image prune -af`**（清掉被取代的舊 tag）＋ **每週 prune cron** ＋ **磁碟用量告警（>85%）**。
- 教訓：容器化服務「看似應用層的錯」要先看**磁碟與資源**（`df -h /`、`docker system df`），再看程式碼。

## 11. 每個 bug 都要補回歸測試

- 修 bug 的流程固定是：**找 root cause → 修 → 在回歸測試加這個 case → 全綠 → tag as release → 部署 → 驗證**。
- 別只修現象；用最能重現 root cause 的輸入（如本清單第 1 條：113 字 CJK / 273 bytes）寫成測試案例。
