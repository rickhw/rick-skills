# v<X.Y>.x System Design — <版本主題>

> Status: Draft for review。只寫該版本摘要與實作方向；詳細 API 規格放對外 `API-spec.md`。
> 對應 `00-Req.md` / `01-URD.md`。

## 1. Design Goals
- <…>；改動最小、向下相容。

## 2. Current State（相關現況）
- <既有架構 / 資料 / 流程中與本版相關的部分>

## 3. 資料模型（migration `NNNN`）
- <新表 / 欄位；為何這樣設計；grandfather 既有資料；down 對稱>

## 4. Backend
- <package / handler / worker 變更；寫入 async（202 + event_id + 冪等）、讀取 cache>
- <存取控制 / 錯誤碼 / 分頁>

## 5. 前端
- <SPA / embed 變更；optimistic UI>

## 6. 測試（regression runner）
- <單元 + 黑箱 e2e 案例；每個 bug 一個回歸 case>；整套全綠。

## 7. 部署 / 文件
- <migration、設定 / env、nginx、Arch.md / API-spec.md / Deployment.md 更新>

## 8. 決策點（負責人確認）
- D…：<結論>
