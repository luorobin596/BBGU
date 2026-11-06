# 醫事人員報備支援申報 Plugin

本專案目前為 Skills-only plugin 的開發骨架，提供提交申請、查詢進度、撤銷申請三個入口。尚未驗證網站流程、實作本機儲存腳本或安裝 plugin。

## 專案結構

```text
Support_registration/
├── plugin.json
├── skills/
│   ├── submit-application/SKILL.md
│   ├── check-application-status/SKILL.md
│   └── withdraw-application/SKILL.md
├── references/                 # 需要時才新增參考文件
├── scripts/                    # 預留本機紀錄讀寫腳本
├── evals/cases.md
├── docs/plugin-skills-tutorial.md
└── README.md
```

## 工具與執行前提

- 必備：ChatGPT Chrome 擴充套件，且目前對話可使用 Chrome 操作工具。
- 選配：已連接的 Google Calendar 工具；未提供時直接使用使用者輸入的申報資料。
- 本機檔案讀寫能力：用於跨對話保存與提取操作紀錄，須符合宿主的存取權限。

這些工具由執行宿主提供。本套件不自建 MCP Server，也不處理 Google OAuth。Manifest 尚未宣告工具相依，安裝本套件不代表上述工具會自動安裝或獲得授權。

## 操作紀錄契約

三個 Skill 共用 `operations.jsonl`，每行一個 JSON 物件，每次操作追加一筆。

| 系統 | 儲存位置 |
| --- | --- |
| Windows | `%LOCALAPPDATA%\SupportRegistration\operations.jsonl` |
| macOS | `~/Library/Application Support/SupportRegistration/operations.jsonl` |

由儲存腳本解析環境變數或使用者家目錄，不寫死帳號名稱。執行平台不支援或無法取得路徑時明確回報，不自行改存到專案內。

| 欄位 | 型別與意義 |
| --- | --- |
| `operated_at` | 含時區偏移的 ISO 8601 操作時間 |
| `action` | `submit`、`query` 或 `withdraw` |
| `applicant` | 申請者姓名；操作前必須確認 |
| `application_id` | 網站提供的申請編號字串；尚未取得時為 `null` |
| `status` | 網站顯示的申請狀態；無法取得時為 `null` |
| `result` | 本次操作結果：`success`、`failure` 或 `unknown` |

`status` 與 `result` 分開：例如查詢成功時，申請仍可能是審核中。本機狀態是最後一次觀察結果，當前狀態以網站為準。依網站及申請者縮小查找範圍，不假設申請編號跨系統唯一。

後續腳本須提供追加與篩選讀取，採用併發寫入保護，並在寫入成功後才回報已保存。macOS 目錄與檔案分別使用 `700`、`600` 權限；Windows 使用 NTFS ACL 限制其他一般使用者存取。位置與權限本身不提供檔案加密。紀錄不得包含密碼、Token、憑證 PIN 或登入 Cookie。

## 待完成的實作

1. 以實際網站確認登入方式、欄位、狀態及完成判準，需要時在 `references/` 新增文件。
2. 在 `scripts/` 實作跨平台 JSONL 讀寫與權限處理，讓三個 Skill 共用。
3. 使用 `evals/cases.md` 驗證流程，再進行本機安裝測試。

目前未建立本機正式紀錄檔，也未執行任何申報操作。
