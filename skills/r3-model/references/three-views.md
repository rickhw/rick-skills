# R3 Model — 三層視圖範本與範例

貫穿三層的主軸是 **Role & Responsibility（R&R）**：先講清楚「誰負責什麼、誰能存取誰」，
技術選型最後才談。源於 Conway's Law（架構反映組織）與 OOP 的封裝／介面／實作。

## R&R 釐清要點（每一層都先問）

- 這個元件的**職責**是什麼？（一句話講完，講不完代表職責過大，要拆）
- 它對外暴露什麼**介面**？隱藏什麼**實作**？
- 它的可存取性是 **public / protected / private**？誰可以呼叫它？
- 兩個技術相似的元件（如 API Gateway vs. Reverse Proxy）若同時出現，它們的 R&R
  差異是什麼？講不出差異就不該並存。

---

## 第 1 層 — High Level View（範本）

面向非技術利害關係人。回答邊界、相依、可存取性。

```
系統名稱：<name>
使用者（誰會用）：
  - <user/actor-1> — <目的>
  - <user/actor-2> — <目的>

相依系統：
  內部：<system-A>、<system-B>
  外部：<3rd-party-X>、<3rd-party-Y>

ACL（可存取性，以 public/protected/private 表達）：
  <caller> --(public)-->    <this system>
  <this system> --(private)--> <internal DB>
  <external> --(protected)--> <gateway>
```

圖：Context 圖（系統置中，四周畫使用者與內外部相依，箭頭標可存取性）。

---

## 第 2 層 — Logical View（範本）

展開角色與職責，搭配資料流與 User Story。

```
角色（Role）清單：
  - Web      — <職責一句話>
  - Console  — <職責一句話>
  - API      — <職責一句話>
  - Database — <職責一句話>
  - Cache    — <職責一句話>
  - Queue    — <職責一句話>

資料流（搭配 User Story / Scenario）：
  US-1「使用者下單」：Web → API → DB ；API → Queue（非同步）→ Worker
  US-2「查詢訂單」：  Web → API → Cache →(miss) DB

密度與溫度（標關鍵路徑）：
  Web → API        : 吞吐量 高 / 頻率 高   （熱路徑，需擴展）
  API → DB（寫）   : 吞吐量 中 / 頻率 中
  API → Queue      : 吞吐量 低 / 頻率 突發
```

重點：**對齊目標、確認理解一致**，不是談實作。角色越多代表系統越複雜。

圖：元件圖 / 資料流圖（角色為節點，箭頭標資料流與冷熱度）。

---

## 第 3 層 — Physical View（範本）

往下落到可部署的形態。

```
角色 → 部署形態 / 副本 / 技術選型：
  Web      → Deployment, N=3 replicas, Nginx + React SPA
  API      → Deployment, N=4 replicas, Java/SpringBoot, HPA on CPU
  Worker   → Deployment, N=2 replicas, Python/FastAPI consumer
  Database → StatefulSet, primary+2 replica, PostgreSQL
  Cache    → Deployment, N=2, Redis
  Queue    → Managed, AWS SQS（或 RabbitMQ / NATS）

可觀測性（O11y）脈絡：
  metrics : 每個 Deployment 的 RPS、p99 latency、error rate、queue depth
  logging : 結構化日誌，含 trace id
  tracing : Web→API→DB 全鏈路 trace
```

圖：部署圖（K8s namespace / pod / container，標副本數與外部受管服務）。

---

## 不另設第四層

需要描述細部行為時，用**循序圖**（呼叫順序、同步／非同步）與**狀態機**（資源生命週期）
補充，而非再畫一層程式碼級架構圖。
（資源狀態機的設計可搭配本 repo 的 `restful-api-design` 技能。）

---

## 完整範例（最小電商）

- **High Level**：使用者（消費者、客服 Console）；外部相依金流（public 對外）、
  簡訊供應商（protected）；內部 DB（private）。
- **Logical**：角色 = Web、API、Worker、DB、Cache、Queue。熱路徑為 Web→API→Cache/DB；
  下單走 API→Queue→Worker 非同步。
- **Physical**：API 4 副本（依 CPU 自動擴展）、Worker 2 副本、PostgreSQL 一主二從、
  Redis 2 副本、佇列用受管 SQS；全鏈路 tracing 從 Web 到 DB。

三層講的是**同一個系統**，只是抽象層次不同——上層給人看懂邊界與目標，下層給人部署與維運。
