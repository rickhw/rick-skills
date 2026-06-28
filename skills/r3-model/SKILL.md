---
name: r3-model
description: 以 R3 Model（三層架構視圖：High Level View／Logical View／Physical View，核心為 Role & Responsibility）描述與設計系統架構。當需要畫系統架構圖、撰寫架構文件、向不同對象（高層／技術／維運）說明系統、定義各服務角色與職責、規劃部署拓樸與副本數，或為可觀測性（O11y）建立脈絡時使用。觸發語句例如：「幫我畫這個系統的架構圖」、「整理一份架構文件」、「這個系統有哪些角色與職責」、「規劃部署架構」。R3 Model 與 C4 Model 概念相近。
---

# R3 Model（三層架構視圖）

## 概述

R3 Model 是一套**三層架構視圖**的方法論，用三種抽象層次描述同一個系統，讓不同對象
（高層、技術團隊、維運）都能在對應層次上對齊理解。其哲學核心是 **Role & Responsibility
（R&R，角色與職責）**，建立在物件導向的「封裝、介面、實作」與 **Conway's Law**
（系統架構應反映組織結構）之上。

> 它與後來流行的 C4 Model 概念相近（R3 約成形於 2017，早於作者後來才得知的 C4）。
> 兩者都是「同一系統、多層抽象」，但 R3 以 **R&R** 為主軸貫穿三層。

核心心法：**先談角色與職責，再談技術選型。** API Gateway、Load Balancer、Reverse
Proxy 技術上相似，差別其實在於它們在系統中的 R&R 不同。

## 何時使用本技能

- 畫系統架構圖、撰寫／審查架構文件
- 對不同對象（非技術高層、技術團隊、維運）說明同一系統
- 釐清系統中各服務的角色與職責（R&R）
- 規劃部署拓樸、容器化與副本數
- 為可觀測性（metrics／logging／tracing）建立脈絡

## 三層視圖（依序往下展開）

### 第 1 層 — High Level View（高階視圖）

對應 C4 的 *Context*。面向**非技術利害關係人**，呈現：
- 使用者定義（誰會用）
- 內部與外部系統相依
- **ACL（Access Control List）**：系統之間的關係與可存取性

以 OOP 的 **public / protected / private** 概念表達相依與可存取性。
這層回答：「這個系統與外界的邊界、依賴、誰能存取誰」。

### 第 2 層 — Logical View（邏輯視圖／服務定義）

展開系統職責，定義各個**角色（Role）**，例如：Web、Console、Database、Cache、Queue。
重點：
- **資料流（Data Flow）**：搭配 User Story 與 Scenario 描述系統行為。
- **密度與溫度**：用**吞吐量（throughput）**與**頻率（frequency）**表達各路徑的關鍵程度／
  熱度。
- **角色複雜度**：系統的複雜度與角色數量正相關（像樂團成員或 Scrum Team 的組成）。
- **雙向溝通**：搭配 User Story 後，技術與非技術對象都能看懂。

> 本層重點**不是實作，而是對齊目標、確認各方理解一致**。

### 第 3 層 — Physical View（實體視圖／Go Live）

把邏輯視圖往下延伸到**可部署**的層次：
- **行程層級實例化**：對應 Kubernetes 的 pod／container。
- **副本多重性**：每個邏輯角色在執行期以 **N 副本（N replicas）** 運行。
- **實作細節 / 技術選型**：
  - API server：Node.js/Express、Python/FastAPI、Java/SpringBoot
  - DB：MySQL／PostgreSQL
  - Queue：RabbitMQ／SQS／NATS
- **維運就緒**：為可觀測性（O11y）的 metrics、logging、tracing 提供必要脈絡。

## 不另設「第四層程式碼層」

R3 刻意**不**再往下加一層程式碼級視圖。需要描述細部行為時，改用
**循序圖（sequence diagram）** 與 **狀態機（state machine）**。

## 你應該產出的內容

套用本技能時，至少交付：
1. **High Level View** — 使用者、內外部相依、ACL（以 public/protected/private 表達）。
2. **Logical View** — 角色清單 + 各角色職責，搭配資料流與 User Story；標註關鍵路徑的
   吞吐量／頻率。
3. **Physical View** — 各角色的部署形態（pod/container）、副本數、技術選型、O11y 脈絡。

每一層都建議附一張圖（高階用 Context 圖；邏輯用元件／資料流圖；實體用部署圖）。
細部行為改用循序圖／狀態機補充。詳見 `references/three-views.md`。

## 參考文件

- `references/three-views.md` — 三層視圖逐層的填寫範本、R&R 釐清要點，以及一個完整範例。
