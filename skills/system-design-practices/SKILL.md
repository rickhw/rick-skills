---
name: system-design-practices
description: 設計「可靠、可降級、資源有界」的服務時的系統設計實務。當需要設計 async / 訊息佇列消費者（冪等、有界重試、DLQ、poison-message 處理）、規劃相依失效時的優雅降級（cache / queue / DB 掛掉時仍能部分服務而不級聯）、避免資源失控（無限迴圈 / log 狂寫 / 記憶體 / 磁碟 / 連線）、對齊限制口徑（位元組 vs 字元、配額、rate limit），或把可觀測性當成設計輸入時使用。觸發語句例如：「這個 queue consumer 怎麼設計才不會爆」、「DB / Redis 掛掉時要怎麼降級」、「怎麼避免 poison message / 無限重試」、「這個服務的可靠性 / 韌性設計」、「幫我做 system design review」。
---

# 系統設計實務（可靠 · 降級 · 資源有界）

## 概述

這套原則萃取自 Uriel（正式營運服務）踩過的真實事故——**設計時就該內建、事後才補會很痛**的
可靠性實務。它與 `iterative-delivery` 的 `operational-pitfalls`（事後的 war stories）互補：
**那邊是「壞掉長怎樣」，這邊是「設計時怎麼避免整類壞法」。**

核心心態：

> **Happy path 只是最不重要的那條路。** 分散式服務的設計品質，取決於「相依失效、
> 重試、飽和、資源耗盡」這些**非 happy path** 有沒有被事前設計。

## 何時使用本技能

- 設計 async / 訊息佇列的消費者（at-least-once、冪等、重試、DLQ）。
- 規劃相依（DB / cache / queue / 外部 API）失效時的行為（降級 vs 級聯）。
- Review 一個服務「會不會在壓力 / 故障下失控」（CPU 空轉、log 狂寫、記憶體 / 磁碟爆）。
- 決定限制（長度 / 配額 / rate limit）與其一致性、以及要監控哪些飽和訊號。

## 七條核心原則

1. **每個外部呼叫都可能失敗**：逾時、重試上限、降級路徑要事前設計，不能只寫 happy path。
2. **at-least-once ⇒ 冪等 + 有界重試 + DLQ**：async 訊息一定會重投；缺這三件事遲早爆（poison-message 無限迴圈是經典）。
3. **永久錯誤 ≠ 暫時錯誤**：分類清楚。永久錯誤（FK / 驗證失敗 / 400 類）直接丟（DLQ / drop），別無限重試；只有暫時錯誤（連線、逾時、429/5xx）才重試。
4. **相依失效要降級、不要級聯**：cache 掛 → 仍可讀（回源 + 降載）；queue 掛 → 寫入退化（明確拒絕 / 緩衝）；DB 掛 → 只讀 / 延遲。`fail-closed` 用在安全閘門、`fail-open` 用在可用性——**選對**。
5. **凡事有界**：重試次數、queue 深度、log 量、連線數、記憶體、磁碟、迴圈——**沒有上限的東西遲早失控**。
6. **限制單一口徑**：長度 / 配額 / rate limit 在所有檢查點用同一種計法（應用層與 DB CHECK 一致；字元就都用字元）。
7. **可觀測性是設計輸入**：關鍵飽和訊號（DLQ 深度、訊息重投率、queue lag、disk %、CPU / 記憶體飽和）在設計時就決定要有指標 + 告警，否則只能被動從趨勢圖發現。

## 參考檔（需要時再讀）

- **`references/async-messaging-reliability.md`** — at-least-once 語意、冪等策略、有界重試 + DLQ、poison-message、消費者 Ack/Nack 的正確用法與常見陷阱。
- **`references/failure-isolation-degradation.md`** — 相依（DB / cache / queue / 外部）失效模式與對應降級策略、fail-closed vs fail-open、逾時 / 隔離 / blast radius。
- **`references/resource-safety.md`** — 把一切設上界（重試 / log / 連線 / 記憶體 / 磁碟）、資料保留與清理、配額與 rate limit、限制口徑一致、可觀測性作為設計輸入。

## 用法

設計新服務 / 消費者時，把七條原則當 checklist 逐條回答「這裡怎麼處理」；review 既有設計時，
用它找「哪一條沒做」。落地細節看對應 reference；跨到交付 / 部署流程時搭配 `iterative-delivery`。
