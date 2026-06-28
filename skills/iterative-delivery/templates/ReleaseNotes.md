# v<X.Y>.x Release Notes — <版本主題>

> 版本線主題：<一句話>。

## Tag 對照

| Tag | 說明 |
|---|---|
| `v<X.Y.Z>`（git tag `v<X.Y.Z>`，已 merge `main`） | <該 patch 的重點> |

## 新增功能 / 變更
### <FR#1 標題>
- <做了什麼>

## 資料模型
- migration `NNNN`：<…>（純新增、down 對稱、grandfather）。

## 設定 / 部署
- <新 env / nginx / registry image role / 部署注意>

## 測試
- <新增的測試（含 regression）>；整套 N passed / 0 failed。

## 部署驗證（YYYY-MM-DD，host，via registry）
- release flow 走完；`/readyz`=`v<X.Y.Z>` / `commit` 非 unknown；sanity 全綠；公開端點驗證。

## 不在本版範圍
- <…>
