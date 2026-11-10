# NOR Flash New Feature Design Flow

讓工程師提供 Datasheet 與目標功能，由 AI 自行閱讀規格、設計測試構想，產出可直接開啟的 HTML 驗證計畫書。

## 工程師審查用驗證計畫

使用 [nor_flash_dv_design](skills/nor_flash_dv_design/SKILL.md)，目前版本 `20261110_02`。

例如：「請依這份 Datasheet，為〈功能名稱〉設計驗證計畫，輸出 HTML。」

不必提供章節與頁碼。AI 自行查閱功能、指令、暫存器與時序，將規格缺漏列為待確認。只有缺少文件、功能範圍不明或重要規格衝突時，才詢問必要問題。

輸出依「概念 → 流程 → 判讀」排列：功能規則、測試區域、流程總覽、各 Flow 的完整測項、邊界與未定義用法、復原、判讀方式及待確認問題。單檔 HTML 可離線閱讀與列印；流程使用 HTML／SVG，不依賴 Mermaid。

版面與閱讀順序參考 NOR_Flash_DV_Guide_V6.html，範本已移除案例的產品數值與實測結論。每次計畫依本次 Datasheet 重新設計。驗證計畫不代表已通過實測，也不綁定 Writer 或特定設備。

方法來源：[pstack /statechart](https://github.com/cursor/plugins/tree/main/pstack/skills/statechart)。

保留產品狀態審查、具體異常路徑與未涵蓋範圍說明，讓工程師核對 AI 的理解與測試判別能力。這是文件推演與審查計畫，不代表已執行形式驗證或晶片實測。

### 相關檔案

- [Skill](skills/nor_flash_dv_design/SKILL.md)
- [測試設計檢查](skills/nor_flash_dv_design/references/test-design-review.md)
- [HTML 輸出規範](skills/nor_flash_dv_design/references/html-output.md)
- [HTML 範本](skills/nor_flash_dv_design/assets/verification-plan-template.html)：下載後開啟可看版型；佔位符供 AI 產出計畫時替換。

## 獨立的 Writer 工具

[writer_program_flow_design](skills/writer_program_flow_design/SKILL.md) 保留給明確需要 Writer／Programmer C code 的使用者，版本 `20261110_01`。它是獨立選用工具，不是驗證計畫的必要後續步驟。本次改版不修改其韌體規則；實作前仍需另外確認設備、底層函式與完整操作條件。

版本格式沿用 `YYYYMMDD_NN`，記錄在各 SKILL.md front matter。
