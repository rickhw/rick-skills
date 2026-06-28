# API 與執行期慣例

## 寫入 async、讀取 cache（核心原則）

End User / Customer 的寫入 API（`POST` / `PUT` / `PATCH` / `DELETE`）一律 **async**：

- API 端做**同步的唯讀預檢**（欄位 / 長度 / 權限 / 封鎖 / 父子關係…）給即時錯誤回饋，通過後**入列 queue** 回 **`202 Accepted` + `event_id`**。
- 由 **worker 作為 authoritative writer** 落 DB、更新 cache、發通知。
- 前端 **optimistic UI**：送出後立即顯示，下次讀取用 `event_id` 對帳去重。
- 控制面 / 低頻的管理動作（審核、封鎖等）可維持同步，但要與既有慣例一致。

讀取 API 一律有 **cache 設計**（read-through + 失效策略）。

### 冪等（async 寫入必做）

worker 可能重投（crash 後 redelivery），每種寫入都要冪等：

- **建立**：audit / 主表用 `event_id` UNIQUE，`ON CONFLICT DO NOTHING`；只有「真的新建」才做 cache 計數等副作用。
- **編輯**：用 `last_event_id` 欄位或 audit 去重，重投不重複套用（避免 edit_count 被加兩次）。
- **刪除**：靠狀態轉移天生冪等（只從特定狀態轉一次），副作用（如計數 -1）只在真的轉移時做。

## OpenAPI / 文件

- 提供機器可讀 OpenAPI（`/api/openapi.yaml`）+ 一個 Swagger UI 頁；後台放連結。
- 人類可讀的 `API-spec.md` 與 OpenAPI 並存，標注哪份是機器可讀來源。
- spec 手寫時，用 validator（如 `swagger-cli validate`）把關；改 API 記得同步。

## 慣例

- **分頁**：用 keyset（cursor）而非 offset，回 `next_cursor`；大量資料（萬筆級）要壓測。
- **錯誤碼**：回穩定的機器可讀字串（`{"error":"content_too_long"}`），不要只靠 HTTP status。
- **長度 / 數量限制**：應用層的檢查要與 DB CHECK **同一個口徑**（字元就都用字元，見 operational-pitfalls 的「位元組 vs 字元」）。
- **存取控制 fail-closed**：跨租戶 / 營運者後台的 allowlist 空值要**全拒**（安全），不要 fail-open。

## 觀測性（正常路徑也要寫 log）

- **每個 HTTP 請求**一行結構化 access log（method / path / status / 耗時 / request_id / 真實 client IP）——不要只有錯誤才 log。健康探針可排除以免洗版。
- **每則背景訊息**寫「received」+「processed」（queue / routing_key / message_id / 耗時）。
- 指標（RPS / 5xx / latency p99 / queue depth / 連線狀態）進 Prometheus + Grafana；錯誤進 log 聚合（Loki 等）。
