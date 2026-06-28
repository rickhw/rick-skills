# 版本號規則與命名慣例

## X.Y.Z（SemVer）各碼語意

| 碼 | 名稱 | 何時遞增 | 例 |
|---|---|---|---|
| **X** | Major | 重大功能、**不向後相容**、重大架構改變，或行銷里程碑 | iOS 6→7 改平面化 UI；Android 3.x→4.x |
| **Y** | Minor | 新增功能但**維持向後相容** | iOS 6.0→6.1 |
| **Z** | Patch / 維護碼 | 已發布產品的重大 bug 修補，**不加新功能** | 1.8.1↔1.8.2 設定應相容、不需 merge |
| **Q** | （選用第四碼） | 與軟體工程**無關**的修補：語系、文件等 | 1.6.0.3 |

關鍵原則：
- 版本號代表**階段性的功能集合**，不是時間／進度。1.6.0 含 10 個功能，測試沒過就持續以
  1.6.0 測，**版本號不變**。
- 版本號**只在發布程序完成後遞增**。
- 同一個 Y 之下的不同 Z 應彼此相容（維護碼變動代表沒有相容性問題）。

不一定要 SemVer：Chrome 的單號（遞增大版號）一樣可行。重點是**規則一致、可溝通**。

## 補充建置資訊（Build Metadata）

完整版本識別 = 版本號 + 以下資訊：

- **Build ID**：時間戳，格式 `YYYYMMDD-HHmm`
- **Revision**：來源控制序號（SVN revision 或 Git hash）
- **Tag**：debug／release／專案代號（如 Android 4.4 的 "KitKat"）
- **Customer ID**：專案客製 build 用

範例 release 標示：
```
version: 1.6.0 | revision: 32870 | build: 20130830-1200_Luffy_Release
```

## 發布與維護流程

- 發布後分支進入 **code freeze（維護期）**。
- 發布後遇重大問題 → 出 patch（1.6.0 → 1.6.1）。
- 同時下一版（1.7.0）並行開發。

## 業界版本字串範例

- **Go module pseudo-version**：`vX.0.0-yyyymmddhhmmss-abcdefabcdef`
  （基底版本 + UTC 時間戳 + 12 碼 commit hash）。
- **Windows**：多個 dev 版 vs 單一 release；RC/Beta 反映流程定義。
- **Dapr**：GitHub 上區分 release/RC（公開）與 dev（自行 build）。

## 檔名命名慣例

通則：一致、可解析、自帶可追溯資訊。範例格式：
```
{APPNAME}_{VERSION}_{BUILD_ID}_r{REVISION}.zip
ORZ2000_1.5.0_b20130808_r2345.rar
```
- dev：帶時間戳，可多個 — `RickLds-v1.0.0-dev-b20220406-1200.zip`
- rel：每週期一個、附校驗 — `RickLds-v1.0.0-rel.zip`（md5/sha256/sha512）

## Firmware／第三方供應商交付規範（適用外包情境）

1. Firmware 版本由內部定義，供應商不可自行調整，但須含時間戳／revision。
2. 區分 development build 與 official release。
3. 供應商提供取得版本資訊的 API。
4. dev → release 的轉換須雙方同意。
5. 每個版本維護：需求文件、設計規格、changelog、測試報告、bug list、release notes。
6. 供應商提供測試報告與 firmware image 給 PM。
7. 建立供應商交付物（image、toolchain、文件）的儲存與散布協定。
8. 標準化交付：壓縮格式（ZIP/RAR）、檔名慣例、壓縮檔內含 image 與 changelog。
