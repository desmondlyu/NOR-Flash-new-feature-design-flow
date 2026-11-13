<div align="center">

# NOR Flash New Feature Design Flow

### *把 Datasheet 變成工程師看得懂、查得回去的驗證計畫。*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![DV Skill](https://img.shields.io/badge/skill-nor__flash__dv__design-blue.svg)](skills/nor_flash_dv_design/SKILL.md)
[![Writer Skill](https://img.shields.io/badge/skill-writer__program__flow__design-green.svg)](skills/writer_program_flow_design/SKILL.md)

<br>

<table>
<tr><td align="left">

📘 &nbsp;手上有 Datasheet，卻要花很多時間整理功能、指令、暫存器與時序？<br>
🧭 &nbsp;測試流程列了很多項目，卻不容易看出每一項要驗證什麼、怎麼判讀？<br>
🧪 &nbsp;想探索邊界或未定義用法，又不想把 AI 猜測當成規格結論？

</td></tr>
</table>

### ✨ 這個 repo 把規格閱讀、測試設計與後續 Writer 實作接成一條流程。

</div>

## 30 秒上手

這個 repo 主要提供兩個可安裝到 AI CLI 的 skill：

| Skill | 用途 | 產出 |
| :-- | :-- | :-- |
| `nor_flash_dv_design` | 依 Datasheet 設計 NOR Flash 新功能驗證 | 工程師審查用 HTML／Markdown 計畫書 |
| `writer_program_flow_design` | 把已確認的計畫轉成 Writer／Programmer 流程 | C code、printf log、測試總表與實測回填規則 |

一般使用先安裝 `nor_flash_dv_design`。只有明確要寫 Writer／Programmer C code 時，才接著使用 `writer_program_flow_design`。

### GitHub Copilot CLI

```powershell
git clone https://github.com/desmondlyu/NOR-Flash-new-feature-design-flow.git $HOME\.copilot\skills\nor_flash_dv_design
```

重新開啟 Copilot CLI，輸入 `/skills`，確認 `nor_flash_dv_design` 已載入。

接著提供 Datasheet 路徑與目標功能：

> 請使用 `nor_flash_dv_design`，讀取 `C:\work\datasheets\product.pdf`，為「功能名稱」設計工程師審查用 HTML 驗證計畫。輸出到目前專案，不產生測試程式。

### Codex CLI、Antigravity CLI、Claude Code

先依下方[安裝](#安裝)選擇對應的 skill 路徑，再重新開啟 CLI 並從技能清單確認是否載入。

## 這個專案解決什麼問題？

NOR Flash 的新功能驗證通常不只是「列出幾個 command」。一份可審查的計畫，至少要回答：

- Datasheet 定義了哪些功能規則、狀態與限制？
- 每個測項要操作什麼位址、資料、指令與前置條件？
- 正常情境、錯誤情境、邊界情境與未定義用法要如何區分？
- 讀回什麼結果才算 PASS、FAIL、RETEST 或待設計確認？
- 哪些測項適合保留為 CP 量產篩選，理由與攔截證據是什麼？
- 測試後能否復原？若不能復原，哪些相依測試必須停止？

本 repo 的核心目的，是把這些問題整理成可以由工程師逐項核對的文件，而不是產生一份看起來完整、卻無法追溯依據的測試清單。

## 它會產出什麼？

### `nor_flash_dv_design`

輸出依照「概念 → 流程 → 判讀」排列：

1. 功能規則與測試範圍
2. 產品狀態與操作流程
3. 已選測項、直接位址、資料與條件
4. 邊界、非法或 Datasheet 未定義用法
5. 復原方式與停止規則
6. PASS／FAIL／RETEST／待確認的判讀方式
7. CP 候選與不可取代的攔截證據

計畫書通常輸出為單檔 HTML，可離線開啟、列印與審查。流程圖使用 HTML／SVG，不依賴 Mermaid。

### `writer_program_flow_design`

這是獨立的下一階段工具。它接收已確認的 HTML／Markdown 計畫，協助建立：

- Auto Test Flow 骨架與步驟編號
- SPI 指令序列與狀態輪詢
- 統一的 PASS／FAIL／NOT READY／RETEST 判定
- 工程師看得懂的英文 printf log
- `TEST SUMMARY`、`FINAL RESULT` 與 `Follow-up`
- 編譯檢查、參考 log 與實測結果回填

計畫書是依據，C code 跟隨計畫書。兩者不一致時，先停下來確認，不自行猜測哪一份才是正確版本。

## 使用流程

### 1. 準備輸入

提供：

- Datasheet 或規格文件的本機路徑
- 要驗證的新功能
- 必須保留的邊界或未定義序列
- 是否需要列出 CP 候選
- 是否需要接續 Writer 實作

不必先整理章節與頁碼。AI 會自行查閱功能、指令、暫存器與時序；只有文件缺少、範圍不明或規格互相衝突時才詢問必要問題。

### 2. 先完成驗證計畫

建議使用以下提示：

> 請依這份 Datasheet 驗證「新功能」，保留我指定的邊界與未定義序列，合併等效情境。另列 CP 候選及攔截依據，不估測試時數。輸出 HTML，不產生程式。

計畫書是文件推演與工程師審查用資料，不代表已完成形式驗證或晶片實測。

### 3. 確認範圍後再接 Writer

只有在使用者明確要求 Writer／Programmer C code 時，才使用 `writer_program_flow_design`。它會沿用計畫中已選測項與設備限制，不會把所有候選自動展開成程式。

預設範圍包括：

- 合法低頻設定，不掃頻、不做 SPEED 裕量驗證
- Software Reset、必要等待、輪詢、timeout 與 Reset recovery time
- 依計畫保留正驗、反驗、邊界與未定義用法

預設排除高頻、外部 RESET、`/HOLD`、`/WP` pin 與真實上下電。若設備確實支援且使用者明確納入，才擴充這些範圍。

## 重要邊界

- **AI 提出的情境不是規格結論。** Datasheet 未定義的行為會標為待確認，不會包裝成正式 PASS。
- **驗證計畫不是實測結果。** 文件產出完成，不代表晶片已通過測試。
- **Software Reset 不能取代上下電驗證。** 若功能依賴真正的 Power OFF／ON，必須確認機台能力後另列範圍。
- **無法復原時停止相依測試。** 不會為了把流程跑完而忽略樣品狀態。
- **CP 候選不等於量產測項。** 是否納入仍由工程、產品與量產條件共同決定。
- **Writer 不會自動產生。** 需明確要求，且必須先有可審查的計畫書。

## 安裝

### 安裝前

需要：

- Git
- 你要使用的 AI CLI
- 本 repo 的完整 `skills/nor_flash_dv_design/` 資料夾

不要只複製 `SKILL.md`。`references/` 與 `assets/` 也要保留在同一個 skill 資料夾內。

### 安裝位置

| CLI | 個人安裝 | 專案安裝 |
| :-- | :-- | :-- |
| GitHub Copilot CLI | `$HOME\.copilot\skills\nor_flash_dv_design\` | `.github\skills\nor_flash_dv_design\` |
| Codex CLI | `$HOME\.agents\skills\nor_flash_dv_design\` | `.agents\skills\nor_flash_dv_design\` |
| Antigravity CLI | `$HOME\.gemini\config\skills\nor_flash_dv_design\` | `.agents\skills\nor_flash_dv_design\` |
| Claude Code | `$HOME\.claude\skills\nor_flash_dv_design\` | `.claude\skills\nor_flash_dv_design\` |

Windows PowerShell 使用 `$HOME`。若 CLI 在 WSL 執行，請安裝到 WSL 內的家目錄，不要混用 Windows 路徑。

### Windows PowerShell

以下範例安裝到 GitHub Copilot CLI。要使用其他 CLI，只需修改 `$skillRoot` 那一行。

```powershell
$ErrorActionPreference = 'Stop'
$skillRoot = Join-Path $HOME '.copilot\skills'
$destination = Join-Path $skillRoot 'nor_flash_dv_design'
$downloadPath = Join-Path ([IO.Path]::GetTempPath()) ('nor-dv-' + [guid]::NewGuid())

if (Test-Path -LiteralPath $destination) {
    throw "目標已存在，請先備份後再更新：$destination"
}

git clone --depth 1 https://github.com/desmondlyu/NOR-Flash-new-feature-design-flow.git $downloadPath
if ($LASTEXITCODE -ne 0) {
    throw 'Git 下載失敗，尚未完成安裝。'
}

New-Item -ItemType Directory -Force -Path $skillRoot | Out-Null
Copy-Item -LiteralPath (Join-Path $downloadPath 'skills\nor_flash_dv_design') `
    -Destination $destination -Recurse

foreach ($item in @(
    'SKILL.md',
    'references\test-design-review.md',
    'references\html-output.md',
    'assets\verification-plan-template.html'
)) {
    if (-not (Test-Path -LiteralPath (Join-Path $destination $item))) {
        throw "安裝檔案缺漏：$item"
    }
}

Write-Output "已安裝：$destination"
```

專案安裝時，將 `$skillRoot` 改成：

```powershell
$skillRoot = Join-Path (Get-Location).Path '.github\skills' # Copilot CLI
$skillRoot = Join-Path (Get-Location).Path '.agents\skills' # Codex / Antigravity
$skillRoot = Join-Path (Get-Location).Path '.claude\skills' # Claude Code
```

### macOS／Linux／WSL

以下範例安裝到 GitHub Copilot CLI：

```bash
set -eu
skill_root="$HOME/.copilot/skills"
destination="$skill_root/nor_flash_dv_design"
download_path="$(mktemp -d)"

if [ -e "$destination" ] || [ -L "$destination" ]; then
    echo "目標已存在，請先備份後再更新：$destination" >&2
    exit 1
fi

git clone --depth 1 https://github.com/desmondlyu/NOR-Flash-new-feature-design-flow.git "$download_path/repo"
mkdir -p "$skill_root"
cp -R "$download_path/repo/skills/nor_flash_dv_design" "$destination"

test -f "$destination/SKILL.md"
test -f "$destination/references/test-design-review.md"
test -f "$destination/references/html-output.md"
test -f "$destination/assets/verification-plan-template.html"

echo "已安裝：$destination"
```

其他 CLI 的 `skill_root`：

```bash
skill_root="$HOME/.agents/skills"        # Codex CLI
skill_root="$HOME/.gemini/config/skills" # Antigravity CLI
skill_root="$HOME/.claude/skills"        # Claude Code
```

不使用 Git 時，也可以從 **Code → Download ZIP** 下載本 repo，再手動複製完整的 `skills/nor_flash_dv_design/`。

### 確認是否載入

複製檔案只代表安裝完成，不代表 CLI 已經載入：

- GitHub Copilot CLI：輸入 `/skills`
- Codex CLI：輸入 `/skills`，或使用 `$` 選取 skill
- Antigravity CLI：輸入 `/` 查找 skill
- Claude Code：輸入 `/skills`

請以 CLI 實際列出的名稱為準。若清單沒有出現，先重新開啟工作階段，再檢查目前工作目錄與安裝位置。

### 更新已安裝版本

這個 repo 的更新不會自動同步到已安裝副本。更新前請先把舊資料夾移到 skills 掃描範圍外備份，再重新執行安裝；若有自行修改，請先比較差異。

不要只覆蓋 `SKILL.md`，否則可能留下舊版的 `references/` 或 HTML 範本。

## 文字編輯參考

`nor_flash_dv_design` 會優先嘗試使用：

- [speak-human-tw](https://github.com/Raymondhou0917/speak-human-tw)：繁體中文與台灣用語
- [asd-ste100](https://github.com/danyuchn/asd-ste100-skill)：清楚、可判讀的技術句子

兩者不包含在本 repo 內。未安裝時，skill 會依規則嘗試讀取原始 repo；若必要參考文件無法取得，應停在文字草稿，不輸出正式 HTML。

這裡借用 STE 的句子結構原則，不宣稱正式標準認證。

## 相關檔案

- [NOR Flash DV skill](skills/nor_flash_dv_design/SKILL.md)
- [測試設計檢查](skills/nor_flash_dv_design/references/test-design-review.md)
- [HTML 輸出規範](skills/nor_flash_dv_design/references/html-output.md)
- [HTML 計畫範本](skills/nor_flash_dv_design/assets/verification-plan-template.html)
- [Writer／Programmer skill](skills/writer_program_flow_design/SKILL.md)
- [測試用途詞彙表](GLOSSARY.md)

## 常見問題

### 這個 repo 會直接替我產生 C code 嗎？

不會。一般流程先產出工程師審查用計畫書。只有明確使用 `writer_program_flow_design`，才會依已確認的計畫接續設計 Writer 流程。

### 可以只給功能名稱，不給 Datasheet 章節嗎？

可以。請提供 Datasheet 本機路徑與目標功能。AI 會自行找功能、指令、暫存器與時序；規格不足時才提出必要問題。

### 未定義用法會被判定為 FAIL 嗎？

不會直接判定。規格明確禁止的行為與 Datasheet 未定義的行為會分開標示，後者通常列為待確認或探索結果。

### 這份計畫書可以取代實測嗎？

不行。它是規格推演與工程師審查文件，不代表 Writer 已執行，也不代表晶片已通過驗證。

### 為什麼不預設加入上下電、`/HOLD` 或 `/WP` 測試？

這些項目需要機台與硬體能力。預設先用 Datasheet 支援的 Software Reset 維持可執行範圍；使用者確認設備能力並明確納入後，才擴充測試。

## 參考方法

- [pstack / statechart](https://github.com/cursor/plugins/tree/main/pstack/skills/statechart)：以產品狀態與操作路徑組織驗證構想
- [speak-human-tw](https://github.com/Raymondhou0917/speak-human-tw)：繁體中文文字編輯參考
- [asd-ste100](https://github.com/danyuchn/asd-ste100-skill)：技術文字清晰度參考

## 授權

[MIT License](LICENSE)
