# Skyeye Device Updater（GitHub Releases）

本 repo 的目標：讓 Raspberry Pi 設備在開機時自動檢查更新並下載：
- 模型權重：`model.hef`（約 50–60 MB）
- 主程式：`run`（約 500–700 MB）

更新來源使用 GitHub Releases，並且一定會做 SHA256 驗證（VERIFY_SHA256）。

---

## Repo 文件架構（建議）

> 注意：本 repo 的「程式碼」可以很少；更新主要透過 Releases 的 assets。

建議 repo 根目錄至少包含：

- `github_updater_new.py`：設備端更新器（可打包成 exe）
- `github_update.json`：更新器設定檔（可在 repo 轉讓/改名時只改設定檔，不改程式）
- `README.md`：本文件

設備端（同資料夾）會額外生成：

- `.skyeye_update_state.json`：狀態檔，記錄目前已安裝的版本 tag
  - `installed_model_tag`
  - `installed_run_tag`

---

## 更新判斷規則

### 核心概念
模型（model）與主程式（run）**各自獨立更新**，互不綁定。

更新器每次啟動時會：

1. **model 更新**
   - 從最新 Release 開始往回找
   - 找到「最新一個同時包含」：
     - `model.hef`
     - `model.hef.sha256`
   - 如果找到的 tag 與本機 `installed_model_tag` 不同，則更新 model

2. **run 更新**
   - 從最新 Release 開始往回找
   - 找到「最新一個同時包含」：
     - `run`
     - `run.sha256`
   - 如果找到的 tag 與本機 `installed_run_tag` 不同，則更新 run

### 重要特性
- 最新 Release 沒有 run，不影響 run 更新：
  - run 會往回找到「最近一次有 run 的 Release」
- 最新 Release 沒有 model，不影響 model 更新：
  - model 會往回找到「最近一次有 model 的 Release」
- 兩者不需要在同一個 Release：
  - 例如 model 更新到 `v4`，run 更新到 `v2` 是正常情況

---

## Release 內容規範（非常重要）

### 固定檔名（請勿隨意改）
- 模型：
  - `model.hef`
  - `model.hef.sha256`
- 主程式：
  - `run`
  - `run.sha256`

> 更新器是靠檔名定位資產檔；檔名錯了會被視為「不存在」，導致設備端不會更新到你想要的版本。

### SHA256 檔案格式
`*.sha256` 支援兩種格式：
1) 只有 hash：
- `<64 位十六進位>`

2) sha256sum 常見格式：
- `<64 位十六進位>  <filename>`

更新器會取第一段 hash 進行比對。

### 只更新其中一個（允許）
- 只更新模型：Release 只上傳 `model.hef` + `model.hef.sha256`
- 只更新 run：Release 只上傳 `run` + `run.sha256`
- 兩者都更新：四個檔案都上傳

---

## 常見情況與預期行為

### 情況：v2 有 run，v3/v4 只有 model
- 設備 model=v3、run=v1：
  - model 會更新到 v4（最新含 model 的 Release）
  - run 會更新到 v2（最新含 run 的 Release）

- 設備 model=v3、run=v2：
  - model 會更新到 v4
  - run 不更新（因為 run 最新仍是 v2）

### 情況：某個 Release 漏上 sha256
- 例如有 `run` 但沒有 `run.sha256`
  - 該 Release 會被視為不可用
  - 更新器會跳過並繼續往回找上一個「檔案 + sha256」都齊全的 Release

---

## 設備端設定檔：github_update.json

範例：

```json
{
  "owner": "thiamlo",
  "repo": "skyeye",

  "model_asset_name": "model.hef",
  "model_sha256_asset_name": "model.hef.sha256",

  "run_asset_name": "run",
  "run_sha256_asset_name": "run.sha256",

  "verify_sha256": true,

  "releases_per_page": 30,
  "max_release_pages": 10,

  "timeouts": {
    "api": 20,
    "download_model": 300,
    "download_model_sha256": 60,
    "download_run": 2400,
    "download_run_sha256": 60
  },

  "skip_prerelease": true,
  "skip_draft": true
}
