# 版本線迭代流程與文件分層

## 版本線目錄

每個版本線（minor，如 `v0.13.x`）一個目錄，內含編號文件：

| 檔案 | 產出者 | 用途 |
|---|---|---|
| `00-Req.md` | 產品負責人 | 該版本的原始需求（FR / NFR）。**起點**。 |
| `01-URD.md` | AI Agent | User Requirement：Persona、User Story、驗收條件、決策點。 |
| `02-SystemDesign.md` | AI Agent | 系統設計與實作方向、資料模型、API 摘要、測試計畫。 |
| `03-Tasks.md` | AI Agent | 工作拆解、驗收、未納入項目。 |
| `ReleaseNotes.md` | AI Agent | 該版本線釋出摘要與 tag 對照（完成後寫）。 |

> 視需要再加 `Setup.md` / `UAT.md` / `bench/` / `QUESTIONS.md`（把待決問題留給負責人）。
> 輕量版本（純設定 / 文件 / 小修）可由負責人指示「不需 URD / SystemDesign」，直接從 Req 或
> 一份 `QUESTIONS.md` 起手——但**仍要走 review 關卡與測試把關**。

## 關卡式流程（嚴格遵守順序）

```text
1. 負責人寫 00-Req.md
2. AI 讀 Req → 有疑問先問（Persona 未指定一定先問）→ 產 01/02/03 設計文件
3. 負責人 review 設計文件：
     有問題 → 退回修改 → 再 review
     沒問題 + 明確說「可以開工了」 → 進入 4
4. AI 第一個動作：git checkout -b <branch>（見 branch-and-release.md）
5. 依 03-Tasks 實作 → 整套測試綠燈 → 更新 ReleaseNotes / CHANGELOG / CurrentStatus
```

**review 通過、branch 開好之前，不寫任何產品程式碼。** 這是硬規則。

`02-SystemDesign.md` 經負責人確認開工後，回頭更新內部架構文件（`00-devguide/Arch.md`）。
SystemDesign 只寫該版本 API 摘要；詳細 API 規格放對外的 `API-spec.md`。

## 文件依「對象」分三層

| 目錄 | 對象 | 內容 |
|---|---|---|
| `docs/00-devguide/` | 內部 / 寫 backend 的 Agent | 開發環境、架構（`Arch.md`）、部署（含 Release Checklist + Runbook）、環境對照、測試、demo |
| `docs/10-iteration/` | 內部 / 每次迭代 | 各版本目錄、`CurrentStatus.md`（版本入口 + 接手指南 + 各版本線狀態）、`CHANGELOG.md`、`Backlog.md` |
| `docs/20-site/` | 對外 / End User & Customer | Persona、功能說明、使用者指南、對外 ReleaseNotes、`API-spec.md`、隱私/條款 |

- `CurrentStatus.md` 是「接手 session 先讀這份」的入口：目前開發版本、各版本線狀態表、最近一次 production 部署驗證紀錄。
- `CHANGELOG.md` 依 git tag 整理（內部，可細碎）；對外 ReleaseNotes 用 **Feature / Advantage / Benefit** 角度寫。
- **單一來源**：對外法務 / 文件若同時被 app 與官網用，挑一份當 source of truth，build 時再複製 / 生成，避免兩份漂移。

## 需求追蹤：`docs/Requirement.md` 總表與 FR / NFR 唯一序號

全專案的 FR / NFR 有一份**單一索引** `docs/Requirement.md`，是需求的 source of truth：

- **唯一序號**：每筆需求配發 `FR#NN` / `NFR#NN`（兩位數補零），**永久唯一、不重用**——
  即使該需求被砍或延期，號碼也不回收。新需求取目前最大號 +1。
- **序號串接所有文件**：`00-Req` / `01-URD` / `02-SystemDesign` / `Arch.md` / `API-spec.md` /
  `03-Tasks` / commit message 引用需求時一律用 ID（如「實作 FR#03」），讓需求 → 設計 → 任務 →
  程式碼可雙向追溯。
- **每筆至少記**：ID、名稱、Persona（FR）、**狀態**（`Todo` / `Doing` / `Done`）、
  **版本**（決定在哪個版本線開發；未排程標 `Backlog`）、細節位置。
- **細節落地**：需求完整內容寫在對應版本目錄的 `00-Req.md`（或 `Backlog.md`），總表只放索引列；
  狀態 / 版本異動時同步更新總表。

> 用範本 `templates/Requirement.md` 起手。總表與 `CurrentStatus.md` 互補：前者是「需求清單」，
> 後者是「目前在做哪個版本」。

**`Requirement.md` vs `Backlog.md`（兩者分工）**：

- **`Backlog.md`**：記**片段想法**——還沒確認、還不能進 design/dev 的點子。可只有一兩句、可粗略標
  「大概哪個版本」。**不配發 ID**。
- **`Requirement.md`**：記**已確認、準備進 design/dev** 的需求，每筆有唯一 `FR#NN` / `NFR#NN`。
- **升級流程**：Backlog 的想法成熟 → 升級到 `Requirement.md` 配發 ID → 排進版本時把細節寫進該版本
  `00-Req.md`。降級（暫不做）則把版本標回 `Backlog`，ID 保留不回收。

## 接手 session 的標準開頭

1. 讀 `CurrentStatus.md` 確認目前開發版本與進度；讀 `Requirement.md` 掌握需求全貌與狀態。
2. 讀對應版本目錄的 `00-Req` / `01-URD` / `02-SystemDesign` / `03-Tasks`。
3. 跑整套測試確認環境 OK（動程式碼前先跑一次）。
4. 跟負責人確認要做哪一塊。
