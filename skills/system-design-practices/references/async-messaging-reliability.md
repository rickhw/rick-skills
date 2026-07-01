# Async / 訊息佇列的可靠性

## at-least-once 是預設現實

大多數 broker（RabbitMQ、SQS、Kafka…）保證 **at-least-once**：訊息**至少**送達一次，因此
**會重複**（消費者 crash 在 ack 之前、重投、redelivery）。任何 async 消費者都必須假設「同一則
訊息會被處理多次」。

## 冪等（idempotency）——必做

每種寫入都要能重複套用而結果不變：

- **建立**：主表 / audit 用 `event_id` 唯一鍵 + `ON CONFLICT DO NOTHING`；副作用（計數、通知）只在「真的新建」時做（回傳 created bool 判斷）。
- **編輯**：用 `last_event_id` 欄位或 audit 去重，重投不重複套用（否則 edit_count 被加兩次）。
- **刪除 / 狀態轉移**：靠「只從特定狀態轉一次」天生冪等；副作用（如計數 -1）只在真的轉移時做。

## 有界重試 + DLQ——必做

- 每則訊息的重試次數要**有上限**，超過就送 **Dead Letter Queue（DLQ）**，不要無限 requeue。
- **陷阱（真實踩過）**：用「redelivery 計數」判斷上限時要小心——**plain `Nack(requeue=true)` 不會累加 `x-death`**（`x-death` 只在經 DLX 死信時才寫）。若用這種計數，值永遠是 0/1，上限判斷永遠成立 → **poison message 無限迴圈**、CPU 長期空轉、log 灌爆。
- **可靠做法**：用 broker 提供的 delivery-count（Kafka offset / SQS ApproximateReceiveCount / RabbitMQ quorum queue 的 delivery-count），或最保守——**至多重試一次**（用 `Redelivered` 旗標）後一律進 DLQ。

## 永久錯誤 vs 暫時錯誤——必分類

- **暫時**（連線、逾時、DB 短暫不可用、429/5xx）→ 重試（有界）。
- **永久**（FK / unique / check violation、驗證失敗、4xx、資料已不存在）→ **不要重試**，直接
  丟到 DLQ 或 Ack drop（重試永遠不會成功，只會空轉）。
- 分類方式：檢查錯誤型別 / SQLSTATE（如 PostgreSQL FK=23503、unique=23505、check=23514）。

## 消費者 Ack/Nack 的正確用法

- 成功 → `Ack`。
- 暫時失敗且未達重試上限 → `Nack(requeue=true)`（或延遲重試佇列）。
- 永久失敗 / 達上限 → `Nack(requeue=false)`（→ DLQ）或 Ack drop。
- **止血知識**：`purge` 對**處理中（unacked）**的訊息無效——一直在重試迴圈的 poison 訊息永遠
  unacked。要清它得先**停消費者（訊息退回 ready）→ purge → 啟消費者**。

## 其他

- **排序**：at-least-once + 重試會打亂順序；不要假設嚴格順序，設計成可交換 / 用版本或時間戳收斂。
- **背壓 / 佇列深度**：生產速度 > 消費速度會讓 queue 無限長；要監控 queue lag、必要時限流或擴消費者。
- **DLQ 要有人看**：DLQ 深度要有告警；死信不是丟了就算，要能回放或人工處理。
