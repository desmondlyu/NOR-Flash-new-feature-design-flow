# NOR Flash New Feature Design Flow

讓工程師提供 Datasheet 與目標功能，由 AI 自行閱讀規格、設計測試構想，產出可直接開啟的 HTML 驗證計畫書。

## 工程師審查用驗證計畫

使用 [nor_flash_dv_design](skills/nor_flash_dv_design/SKILL.md)，目前版本 `20261112_01`。

例如：「請依這份 Datasheet，為〈功能名稱〉設計驗證計畫，輸出 HTML。」

不必提供章節與頁碼。AI 自行查閱功能、指令、暫存器與時序，將規格缺漏列為待確認。只有缺少文件、功能範圍不明或重要規格衝突時，才詢問必要問題。

輸出依「概念 → 流程 → 判讀」排列：功能規則、測試區域、流程總覽、各 Flow 的已選測項、邊界與未定義用法、復原、判讀方式及待確認問題。單檔 HTML 可離線閱讀與列印；流程使用 HTML／SVG，不依賴 Mermaid。

版面與閱讀順序參考 NOR_Flash_DV_Guide_V6.html，範本已移除案例的產品數值與實測結論。每次計畫依本次 Datasheet 重新設計。驗證計畫不代表已通過實測，也不綁定 Writer 或特定設備。

方法來源：[pstack /statechart](https://github.com/cursor/plugins/tree/main/pstack/skills/statechart)。

保留產品狀態審查、具體異常路徑與未涵蓋範圍說明，讓工程師核對 AI 的理解與測試判別能力。這是文件推演與審查計畫，不代表已執行形式驗證或晶片實測。

### 測項取捨與 CP 建議

先驗證新產品功能與邏輯，再依需求評選 CP 測項。每項交代情境依據、目標錯誤及不可取代性；保留指定探索、合併等效條件，只展開已選測項。CP 候選另列攔截證據，不以成本刪除必要驗證。

### 機台與 Writer 範圍

預設排除高頻與外部 RESET、/HOLD、/WP pin 控制。Software Reset 優先，保留必要等待、輪詢、timeout 與 Reset recovery time。真實上下電僅列機台允許時的補充建議，確認能力並納入範圍後才展開；Software Reset 結果不能替代上下電驗證。無法復原時停止相依測試。

指定 Writer／Programmer 時沿用合法低頻設定，不掃頻、不做 SPEED 裕量驗證。只有明確要求才接續 Writer 實作，並傳遞已選測項與設備限制；候選不自動變成程式流程。

例如：「請依這份 Datasheet 驗證〈新功能〉，保留我指定的邊界與未定義序列，合併等效情境。另列 CP 候選及攔截依據，不估測試時數。輸出 HTML，不產生程式。」

### 完整敘述與流程圖

測項直接寫位址、資料與條件。每個 Flow 都有專屬 SVG，圖與表格可雙向跳轉。全部文字先經雙技能實質改寫才轉 HTML，必要參考缺漏時停在草稿。

### 文字編輯參考

每次計畫均先嘗試讀取並套用 [speak-human-tw](https://github.com/Raymondhou0917/speak-human-tw) 與 [asd-ste100](https://github.com/danyuchn/asd-ste100-skill)。未安裝時從原 repo 讀取；直接取得失敗才用 Web Search 尋找原始文件。永久安裝依使用者授權與平台方式辦理。必要技能或參考文件取得失敗時，停止於文字草稿，不輸出正式 HTML。

完整使用時機、下載安裝、搜尋備援與保真編輯規則見 SKILL.md 的「用詞與文字」。繁中借用 STE 的句子結構原則，不宣稱正式標準認證。

### 相關檔案

- [Skill](skills/nor_flash_dv_design/SKILL.md)
- [測試設計檢查](skills/nor_flash_dv_design/references/test-design-review.md)
- [HTML 輸出規範](skills/nor_flash_dv_design/references/html-output.md)
- [HTML 範本](skills/nor_flash_dv_design/assets/verification-plan-template.html)：下載後開啟可看版型；佔位符供 AI 產出計畫時替換。


## 安裝到 CLI

以下安裝的是 `nor_flash_dv_design`，不包含獨立的 Writer skill。需先安裝 Git 與要使用的 CLI。本節路徑依 2026-11-10 查閱的官方文件整理；提供的是手動複製方式，不需要額外套件管理器。

### 1. 選擇安裝範圍與路徑

個人安裝可跨專案使用；專案安裝只供該專案使用，也可提交到專案版本控制供團隊共用。選一種即可，避免同名 skill 重複安裝。

| 工具 | 個人安裝：完整 skill 資料夾路徑 | 專案安裝：相對於專案根目錄 |
|---|---|---|
| GitHub Copilot CLI | `~/.copilot/skills/nor_flash_dv_design/` | `.github/skills/nor_flash_dv_design/` |
| Codex CLI | `~/.agents/skills/nor_flash_dv_design/` | `.agents/skills/nor_flash_dv_design/` |
| Antigravity CLI | `~/.gemini/config/skills/nor_flash_dv_design/` | `.agents/skills/nor_flash_dv_design/` |
| Claude Code | `~/.claude/skills/nor_flash_dv_design/` | `.claude/skills/nor_flash_dv_design/` |

`~` 代表執行 CLI 的使用者家目錄。Windows PowerShell 用 `$HOME`，例如 `C:\Users\你的帳號\.copilot\skills\nor_flash_dv_design\`；macOS／Linux 常見為 `/Users/你的帳號` 或 `/home/你的帳號`。若 CLI 在 WSL 執行，請安裝到 WSL 內的家目錄，而不是 Windows 使用者目錄。

Antigravity 上表採目前官方的共用設定路徑，可供 CLI 與 App 使用。不要將舊版 IDE 的 `~/.gemini/antigravity/skills/` 當成 CLI 的主要安裝位置。Copilot、Codex 與 Antigravity 也可共用專案的 `.agents/skills/`；同一份 skill 不需在它們各自讀取的位置重複放置。

### 2. 下載並複製完整資料夾

必須複製 repo 內的 `skills/nor_flash_dv_design/` 整個資料夾。安裝後，`SKILL.md`、`references/` 與 `assets/` 必須位於同一層；不能只下載 SKILL.md，也不能把整個 repo 放進以 skill 命名的資料夾後就當成安裝完成。

#### Windows：PowerShell

以下範例預設安裝到 Copilot CLI。先修改 `$skillRoot`，四行選一行即可。程式碼在獨立區塊執行，目標已存在時會停止，不覆蓋既有版本。

```powershell
& {
    $ErrorActionPreference = 'Stop'
    $skillRoot = Join-Path $HOME '.copilot/skills'             # GitHub Copilot CLI
    # $skillRoot = Join-Path $HOME '.agents/skills'            # Codex CLI
    # $skillRoot = Join-Path $HOME '.gemini/config/skills'     # Antigravity CLI
    # $skillRoot = Join-Path $HOME '.claude/skills'            # Claude Code

    $destination = Join-Path $skillRoot 'nor_flash_dv_design'
    if (Test-Path -LiteralPath $destination) {
        throw "目標已存在，請先依下方更新方式處理：$destination"
    }
    $downloadPath = Join-Path ([IO.Path]::GetTempPath()) ('nor-dv-' + [guid]::NewGuid())
    git clone --depth 1 https://github.com/desmondlyu/NOR-Flash-new-feature-design-flow.git $downloadPath
    if ($LASTEXITCODE -ne 0) { throw 'Git 下載失敗，尚未安裝。' }

    New-Item -ItemType Directory -Force -Path $skillRoot | Out-Null
    Copy-Item -LiteralPath (Join-Path $downloadPath 'skills/nor_flash_dv_design') -Destination $destination -Recurse
    foreach ($item in @('SKILL.md', 'references/test-design-review.md', 'references/html-output.md', 'assets/verification-plan-template.html')) {
        if (-not (Test-Path -LiteralPath (Join-Path $destination $item))) {
            throw "安裝檔案缺漏：$item"
        }
    }
    Write-Output "已複製完整 skill：$destination"
    Write-Output "下載來源暫存於：$downloadPath"
}
```

只安裝到專案時，先切換到該專案根目錄，將上方 `$skillRoot` 改為下列其中一行，其餘不變：

```powershell
$skillRoot = Join-Path (Get-Location).Path '.github/skills' # Copilot CLI
$skillRoot = Join-Path (Get-Location).Path '.agents/skills' # Codex CLI / Antigravity CLI
$skillRoot = Join-Path (Get-Location).Path '.claude/skills' # Claude Code
```

#### macOS／Linux／WSL：Bash

以下同樣預設 Copilot CLI，請只選一個 `skill_root`。括號讓錯誤只停止這段安裝，不結束原本的終端機。

```bash
(
    set -eu
    skill_root="$HOME/.copilot/skills"          # GitHub Copilot CLI
    # skill_root="$HOME/.agents/skills"        # Codex CLI
    # skill_root="$HOME/.gemini/config/skills" # Antigravity CLI
    # skill_root="$HOME/.claude/skills"        # Claude Code

    destination="$skill_root/nor_flash_dv_design"
    if [ -e "$destination" ] || [ -L "$destination" ]; then
        echo "目標已存在，請先依下方更新方式處理：$destination" >&2
        exit 1
    fi
    download_path="$(mktemp -d)"
    git clone --depth 1 https://github.com/desmondlyu/NOR-Flash-new-feature-design-flow.git "$download_path/repo"
    mkdir -p "$skill_root"
    cp -R "$download_path/repo/skills/nor_flash_dv_design" "$destination"
    for item in SKILL.md references/test-design-review.md references/html-output.md assets/verification-plan-template.html; do
        test -f "$destination/$item"
    done
    echo "已複製完整 skill：$destination"
    echo "下載來源暫存於：$download_path"
)
```

專案安裝時，先切換到專案根目錄，再將上方 `skill_root` 改為其中一行：

```bash
skill_root="$PWD/.github/skills" # Copilot CLI
skill_root="$PWD/.agents/skills" # Codex CLI / Antigravity CLI
skill_root="$PWD/.claude/skills" # Claude Code
```

不使用 Git 時，也可從本 repo 的 **Code → Download ZIP** 下載、解壓縮，再手動複製 `skills/nor_flash_dv_design/` 到上表位置。兩種方式都需保留完整子目錄。

### 3. 確認載入並開始使用

複製完成只代表檔案到位，還要在 CLI 確認能讀取此 skill。使用該工具的技能清單或重新開啟工作階段載入；工具版本、停用設定與組織政策可能影響載入。

- GitHub Copilot CLI：在互動介面輸入 `/skills`，確認技能存在且已啟用。
- Codex CLI：輸入 `/skills` 或鍵入 `$` 選取技能。
- Antigravity CLI：在輸入框鍵入 `/`，查找載入的技能；若未列出，確認目前工作目錄與安裝位置。
- Claude Code：輸入 `/skills` 查看，或從 `/` 選單選取。

請使用清單中實際顯示的名稱。本 repo 目前的名稱為 `nor_flash_dv_design`，資料夾沿用相同名稱。若工具回報 front matter 或名稱格式錯誤，保留錯誤訊息並檢查該版本的規則，不把成功複製當作已成功載入。

提供 Datasheet 的本機路徑與目標功能，例如在 CLI 對話中輸入：

> 請使用 nor_flash_dv_design，讀取「C:/work/datasheets/product.pdf」，為〈功能名稱〉設計工程師審查用 HTML 驗證計畫。輸出到目前專案，不產生測試程式。

macOS／Linux 請換成實際檔案路徑。不需自行整理章節與頁碼；AI 會查閱規格。輸出的 HTML 可直接用瀏覽器開啟。

### 4. 文字編輯 skill 與更新方式

speak-human-tw 與 asd-ste100 不包含在本 repo 內。未安裝時，本 skill 會依文字編輯規則嘗試讀取原 repo；要在離線環境使用，需事先備妥這兩份 skill 及其參考文件。永久安裝時，各自放在所選 CLI 的 skills 根目錄下，與 `nor_flash_dv_design` 並列，不放進它的子目錄。

本節採複製安裝，repo 更新不會自動同步到已安裝副本。更新時先將原本的 `nor_flash_dv_design` 移到 skills 掃描目錄之外備份，再重新執行安裝步驟；若曾自行修改內容，先比較差異。不要只覆蓋 SKILL.md 而留下舊版 references 或 HTML 範本。

### 官方路徑依據

- [GitHub Copilot CLI：Adding agent skills](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
- [Codex：Build skills／Where Codex loads local skills](https://learn.chatgpt.com/docs/build-skills)
- [Antigravity：Agent skills／CLI skill locations](https://antigravity.google/docs/skills)
- [Claude Code：Extend Claude with skills](https://code.claude.com/docs/en/skills)

## 獨立的 Writer 工具

[writer_program_flow_design](skills/writer_program_flow_design/SKILL.md) 保留給明確需要 Writer／Programmer C code 的使用者，版本 `20261110_02`。一般計畫不強制使用；指定 Writer 時先完成 HTML 計畫，再依使用者要求接續本工具實作。沿用低頻與 Software Reset 範圍，不重新加入 SPEED、上下電或 /HOLD 測試。實作前確認必要的底層函式與操作條件。

版本格式沿用 `YYYYMMDD_NN`，記錄在各 SKILL.md front matter。
