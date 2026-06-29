---
name: iterative-delivery
description: 以「版本線迭代」交付產品服務的工程規範與流程。當需要規劃一個新版本、用關卡式設計文件（需求→URD→系統設計→任務）開工、決定分支與 git tag 命名、執行「tag as release」並從 image registry 部署 production、用自動測試把關每次 release、為每個 bug 補回歸測試、套用「寫入 async／讀取 cache」的 API 原則、寫 Release Checklist 或出事 Runbook，或為新產品建立可追溯／可重現的交付流程時使用。觸發語句例如：「開一個新版本 / 這版要怎麼開工」、「幫我 tag as release 並部署」、「這個產品的開發流程規範」、「release checklist」、「production 出事怎麼查」、「新服務的工程慣例」。
---

# 版本線迭代交付（Iterative Delivery）

## 概述

本技能萃取自 Uriel（一個正式營運的雲端服務）的開發規範，是一套**端到端的工程交付方法論**：
以「**版本線迭代**」為節奏，用**關卡式設計文件**收斂需求、用**自動測試**把關每次釋出、
用**從 registry 取 artifact 的可重現部署**上 production。核心信念：

> **每一次改動都要可討論、可追溯、可重現、可回滾。**
> 討論先於實作；產出物（image）一律來自 registry；綠燈測試才算完成。

這是流程方法論，不是腳手架產生器。它規範「**怎麼從一個想法走到 production**」，
與 `software-versioning`（版本號／artifact 策略）、`restful-api-design`（API 設計）、
`r3-model`（架構描述）互補——本技能是把它們串起來的**交付骨幹**。

## 何時使用本技能

- 規劃一個新版本，或決定「這版要做什麼、怎麼開工」。
- 為新產品建立開發 → 測試 → 釋出 → 部署的流程與文件結構。
- 決定分支 / git tag 命名，或執行「tag as release」與 production 部署。
- 寫 Release Checklist、出事 Runbook，或修 bug 後補回歸測試。
- 設計對外（End User / Customer）的寫入 / 讀取 API 行為。

## 七條核心原則

1. **討論先於實作**：需求有疑問先問、Persona 沒指定一定先問；review 通過、明確說「可以開工」之前不寫產品程式碼。
2. **關卡式文件**：每個版本線一個目錄，依序產出 `00-Req` →（AI 產）`01-URD` / `02-SystemDesign` / `03-Tasks`，Rick review 過才開分支。
3. **測試是必要的**：每個版本完成都要讓整套測試綠燈才算完；**每個 bug 都要補一個回歸測試案例**。
4. **production 一律從 registry**：正式部署只 pull registry 的 image，不在 prod 臨時 build；artifact 可追溯、可重現。
5. **保留決策痕跡**：踩過的坑、權衡、決策寫進對應文件 / 註解 / commit，不要靜默修。
6. **不要過度抽象**：先實作可運作的版本，重構等需求或風險證明需要再做。
7. **寫入 async、讀取 cache**：End User / Customer 的 `POST/PUT/PATCH` 一律走 queue（202 + optimistic UI），讀取一律有 cache。

## 流程一眼看懂

```text
Req(00) ──► 設計文件(01 URD / 02 SystemDesign / 03 Tasks) ──► review 關卡
                                                                 │ 通過 + 「可以開工」
                                                                 ▼
                       git checkout -b feat/vX.Y-slug ──► 實作 ──► 測試全綠 ──► 更新文件
                                                                 │
                              「tag as release」：commit(VERSION 升乾淨) ──► merge main --no-ff
                                  ──► tag vX.Y.Z ──► build+push image 到 registry ──► prod 部署(registry)
                                  ──► /readyz / sanity 驗證 ──► 記錄部署驗證
```

## 參考檔（需要時再讀）

- **`references/iteration-workflow.md`** — 版本線目錄結構、四份設計文件的內容與關卡、文件依「對象」分層（內部 devguide / 迭代 / 對外 site）。搭配 `templates/` 的 5 份範本。
- **`references/branch-and-release.md`** — 分支與 tag 角色與命名、「tag as release」的固定步驟、registry 部署、`/readyz` 把關、rollback。
- **`references/async-api-runtime.md`** — 寫入 async（202 + event_id + optimistic + 冪等）、讀取 cache、OpenAPI、keyset 分頁、錯誤碼慣例、正常路徑也要寫 log。
- **`references/operational-pitfalls.md`** — 從真實事故累積的硬傷清單（位元組 vs 字元、反向代理後的 real IP、compose project 隔離、nginx upstream 快取、子路徑靜態站、第三方 cookie…）。**動到對應領域前先讀這份。**
- **`templates/`** — `Requirement.md`（需求總表，放 `docs/` 根）+ `00-Req.md` / `01-URD.md` / `02-SystemDesign.md` / `03-Tasks.md` / `ReleaseNotes.md`（複製到新版本目錄）。

## 套用到新產品的最小起手式

1. 建 `docs/` 三層：`00-devguide/`（內部：環境/架構/部署/測試）、`10-iteration/`（每版目錄 + `CurrentStatus.md` + `CHANGELOG.md` + `Roadmap.md`）、`20-site/`（對外：功能/條款/API-spec）。
2. 建 `docs/Requirement.md` 需求總表：所有 FR / NFR 配發**唯一序號** `FR#NN` / `NFR#NN`，記狀態（Todo/Doing/Done）與版本，序號串接所有文件與 commit（見 `references/iteration-workflow.md`）。
3. 第一個版本線 `vX.Y.x/` 從 `00-Req.md` 起，照關卡走。
4. 建一支「一個指令跑完整套測試」的 runner（如 `scripts/test.sh all`），當作 release 的綠燈閘門。
5. 決定 registry（image repo）與部署腳本；確保 `/readyz` 會吐出 `version`/`commit`/`built_at`，作為部署成功與 rollback 的判斷入口。
6. 預設語言、commit 風格、「不主動 commit / push / 計費操作」等協作偏好寫進 `CLAUDE.md` / `AGENTS.md`。
