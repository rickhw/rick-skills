# 分支、Tag 與釋出 / 部署

## 分支命名

- Feature：`feat/v<X.Y>-<slug>`（例 `feat/v0.13-official-site`）
- Bug fix：`fix/v<X.Y.Z>-<slug>`（例 `fix/v0.13.4-comment-length`）
- 永遠**從 `main` 開新分支**；merge 回 `main` 用 **`--no-ff`** 保留分支歷史。
- 已 merge 的分支等負責人確認後再刪（`git branch -d`）。

## Tag 角色

- git tag `vX.Y.Z` = 正式釋出，對齊 merge commit。
- 開發中 `main` 的 `VERSION` 為 `vX.Y.Z-dev`；release 時升成乾淨 `vX.Y.Z`，兩次 release 之間停在剛釋出的乾淨號。
- **只有在「tag as release」流程才打 tag**；負責人沒說、也沒指定 tag 名時不要自行打 tag。

## 「tag as release」固定步驟

負責人說「tag as release」時，固定語意（缺一不可）：

1. Commit release 分支所有內容（含 `VERSION` 升成乾淨 `vX.Y.Z`）。commit message 三段式：**主要目標 / 主要區塊 / 不在範圍**，HEREDOC 寫。
2. merge back `main`（`--no-ff`）。
3. 在 `main`（merge commit）打 **annotated** tag `vX.Y.Z`。
4. build image 並 **push 到 registry**（push `main` + tag 到 origin 也是 release 的一部分）。
5. **部署 production via registry**：prod 先 `git pull` 到該 release commit（取設定檔 / compose / nginx），再從 registry pull image 部署。

> 為什麼固定 commit→merge→tag→push image→deploy：git tag 與 registry image 都對齊同一個 merge commit；
> rollback = revert merge commit + 重新部署舊 tag 的 image。**非** release 情境不要主動 push。

## 部署鐵則

- Production **只用 registry 部署**；local-build 僅限 dev / 緊急。
- 部署前：prod 已 `git pull --ff-only` 到 release commit、能登入 registry、已備份 DB、先 `--dry-run`。
- image 的 `version` / `commit` / `built_at` metadata 在 build/push 時注入。
- **`/readyz` 是 release / rollback 的判斷入口**：部署後 `version` 不得是 `dev`、`commit` / `built_at` 不得是 `unknown`；否則視為部署失敗。跑一支 production sanity 腳本（公開端點 200、guard 端點 401/403、版本正確）。
- 部署後在 `CurrentStatus.md` 記一段「部署驗證（日期 / host / 結果）」。

## 有資料庫時

- migration 編號、up/down 對稱、**純新增不破壞舊資料**；既有資料用 grandfather 策略遷移。
- 部署流程自動套用 migration；prod 部署前確認 migration 版本、看過 `*.sql` 內容。
- **建立新 image role（新服務）時**：registry repo 要先建、CI/部署身分的 IAM/權限要涵蓋（push 與 pull 是不同身分），否則 release 會卡在「repo 不存在 / 無權限」。

## Rollback

1. `git revert` merge commit（或 checkout 舊 release commit）。
2. 從 registry 重新部署**舊 tag** 的 image。
3. `/readyz` + sanity 確認回到舊版。
