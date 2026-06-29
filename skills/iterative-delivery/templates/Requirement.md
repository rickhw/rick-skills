# <產品> 需求總表（單一來源）

> 全專案 FR / NFR 的**唯一索引**。序號（`FR#NN` / `NFR#NN`）一旦配發即**永久唯一、不重用**，
> 串接所有文件（各版本 `00-Req.md`、`01-URD`、`02-SystemDesign`、`Arch.md`、`API-spec.md`、commit）。
>
> - **狀態**：`Todo`（未開工）/ `Doing`（開發中）/ `Done`（已釋出）。
> - **版本**：已決定在哪個版本線開發；未排程者標 `Backlog`。
> - **細節**：每個 FR / NFR 完整內容放在 `docs/10-iteration/v<X.Y>.x/00-Req.md`（或 `Backlog.md`）。

## Functional Requirements

| ID | 名稱 | Persona | 版本 | 狀態 | 細節位置 |
|---|---|---|---|---|---|
| FR#01 | <…> | <Persona> | v0.1.x | Todo | <連結> |

## Non-Functional Requirements

| ID | 名稱 | 版本 | 狀態 | 細節位置 |
|---|---|---|---|---|
| NFR#01 | <…> | v0.1.x | Todo | <連結> |

## 維護規則

1. 新需求一律先在此表配發新序號（目前最大號 +1，不重用舊號）。
2. 細節寫進對應版本目錄的 `00-Req.md`，並在那裡標上同一個 ID。
3. SystemDesign / Arch / API-spec / Tasks / commit 引用需求時用 ID。
4. 狀態 / 版本異動時同步更新本表（單一來源）。
