---
name: software-versioning
description: 規劃軟體的版本管理與產出物（artifact）管理策略，貫穿整個 SDLC。當需要決定版本號規則（SemVer X.Y.Z）、設計 build/release 命名慣例、區分 dev 與 release 產出物、把 config 與 artifact 分離、選擇 Version-First（產品）或 Version-Late（專案）策略、規劃版本如何流經 dev/test/staging/prod、建立可追溯與可重現的交付，或釐清「版本管理 vs 版本控制系統／分支策略」時使用。觸發語句例如：「這個專案的版本號要怎麼訂」、「設計 artifact 的命名與發布流程」、「dev 跟 release build 怎麼區分」、「版本要 SemVer 還是用時間戳」。
---

# 軟體版本管理與產出物管理（Versioning & Artifact）

## 概述

**版本號是溝通的介面。** 版本管理的目的，是讓 PM、開發、QA、維運在談「軟體現在處於什麼
狀態」時有共同基準，並提供變更範圍的索引與問題追蹤的參照。忽視這個介面，代表團隊成熟度不足。

本技能涵蓋三件常被混為一談、其實要分開處理的事：

1. **版本管理（SDLC 視角）** — 版本如何定義、產出物如何抵達使用者。
2. **版本控制系統 / 分支策略（工具視角）** — Git/SVN/P4 與分支操作；解決的是**團隊協作**。
3. **產出物管理（Artifact）** — 解決的是**軟體交付標準化**（一致、可追溯、可重現）。

> 分支策略與產出物管理解決的問題不同，不必互相牽就。

## 何時使用本技能

- 決定版本號規則、設計 build／release 命名慣例
- 區分 dev 與 release 產出物、把 config 從 artifact 抽離
- 選擇 Version-First（產品）或 Version-Late（專案）策略
- 規劃版本如何流經 dev / test / staging / prod，並維持可追溯性
- 釐清「版本管理 vs 版本控制系統 vs 分支策略」的分工

## 核心原則

1. **版本號是溝通介面** — 沒人拿來溝通的版本號沒有意義。
2. **版本代表「階段性的功能集合」，不是時間或進度** — 1.6.0 含 10 個功能，測試沒過就
   持續以 1.6.0 測，版本號不變。
3. **版本號只在「發布程序完成後」遞增**，開發過程中不動。
4. **Single Codebase, Multiple Deployment** — 同一份產出物部署到各環境，靠**外部 config**
   區別；artifact 與 config 是 **1 對多**，絕不把 config 編進 artifact。
5. **dev 與 release 是兩種產出物**（見下）。
6. **工具是輔助流程，標準程序為主、自動化為次。**
7. **第一天就交付 Hello World** — 從專案起點就由 CI server 產生可交付的 artifact，
   而不是靠 IDE 匯出。

## 版本號（SemVer：X.Y.Z）

- **X（Major）**：重大功能、不向後相容、重大架構改變，或行銷里程碑。
- **Y（Minor）**：新增功能但維持向後相容。
- **Z（Patch）**：已發布產品的重大 bug 修補，不加新功能；同 Y 下的 Z 應彼此相容。
- **Q（選用第四碼）**：與軟體工程無關的修補（語系、文件等）。

補充建置資訊（組成完整版本識別）：Build ID（`YYYYMMDD-HHmm`）、Revision（SVN 序號或
Git hash）、Tag（debug/release/專案代號）、Customer ID。
不一定要 SemVer——Chrome 的單號版本一樣可行。細節與命名慣例見
`references/semver-and-numbering.md`。

## dev vs release 產出物

- **dev**：功能尚未驗證，僅團隊內部使用；可含 debug、後門、旗標、選擇性編譯；
  可有多個（時間序）。例：`RickLds-v1.0.0-dev-b20220406-1200.zip`。
- **rel**：功能已驗證、過認證、可交付客戶；關閉 debug、最佳化／簽章；**每個發布週期只有
  一個**。例：`RickLds-v1.0.0-rel.zip`（附 md5/sha256 校驗）。

詳見 `references/artifact-management.md`。

## 兩種版本策略：先給號 vs 後給號

- **Version-First（適合產品）**：階段開始就宣告版本（SemVer），由 PM 宣布版本觸發首個
  build。適用 OSS、工具、函式庫、SDK、API；需搭配 LTS 管理。
- **Version-Late（適合專案）**：在發布／部署時才決定版本，適合快速迭代、無外部相依者；
  以 commit hook、GitOps、時間戳或 SemVer 計算自動產生。

選錯策略（尤其專案放著不給號）會導致：沒人知道現在跑哪版、無法追溯、各環境可能跑不同
程式碼而不自知、QA 測試基準漂移。判斷準則見 `references/versioning-in-sdlc.md`。

## 你應該產出的內容

套用本技能時，至少交付：
1. **版本策略**：Version-First 或 Version-Late，及理由。
2. **版本號規則**：採用的格式（SemVer X.Y.Z[.Q] 或其他）與各碼語意。
3. **產出物規格**：dev/rel 區分、檔名命名慣例、校驗與 config 分離方式。
4. **版本在 SDLC 的流動**：何時給號、何時凍結、如何在各環境保持同一 artifact 與可追溯性。

## 參考文件

- `references/semver-and-numbering.md` — X.Y.Z(.Q) 各碼語意、build metadata、檔名命名
  慣例、firmware／第三方供應商交付規範。
- `references/artifact-management.md` — dev vs rel 兩層分類、artifact≠config、
  分支策略 vs 產出物管理、第一天交付 Hello World。
- `references/versioning-in-sdlc.md` — Version-First vs Version-Late、版本作為溝通介面、
  Single Codebase Multiple Deployment、忽視版本的後果與根因。
