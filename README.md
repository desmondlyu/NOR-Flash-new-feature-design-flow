# NOR Flash New Feature Design Flow

這個 repo 提供兩個相互銜接的 Skill，協助設計 NOR Flash 新功能的 DV 流程，並把測試計畫轉成 Writer／Programmer 的 C code。

## 版本

本次版本：`20261110_01`

| Skill | 用途 | 版本路徑 |
|---|---|---|
| `nor_flash_dv_design` | 以狀態圖分析功能、規則、反例與測項 | `skills/nor_flash_dv_design/SKILL.md` |
| `writer_program_flow_design` | 把測試計畫轉成 Writer 流程、C code 與測試 log | `skills/writer_program_flow_design/SKILL.md` |

## 建議使用順序

1. 先使用 `nor_flash_dv_design`。
2. 依 Datasheet 建立狀態圖、`R*`／`F*`／`U*` 規則與測項表。
3. 確認狀態圖與規則清單。
4. 再使用 `writer_program_flow_design`。
5. 依測項表建立 Writer 流程、判定引擎、log 與復原流程。
6. 編譯並在 DUT 上實測。
7. 將實測 log 回填到測試計畫，再同步計畫與程式。

## 目錄內容

```text
skills/
├── nor_flash_dv_design/
│   └── SKILL.md
└── writer_program_flow_design/
    ├── SKILL.md
    └── references/
        ├── firmware_checklist.md
        └── log_format.md
```

## 使用限制

- Datasheet 是功能規格的主要來源。
- 狀態圖是模型，不等同於晶片實作。
- 未經 DUT 實測的反例只代表模型推演結果。
- `writer_program_flow_design` 不會自行發明底層 SPI 函式。
- C code、測試計畫與實測 log 的步驟編號、指令、預期值與判定代碼必須保持一致。

## 版本資訊

版本格式為 `YYYYMMDD_NN`。版本寫在兩份 `SKILL.md` 的 front matter。

## Writer log 輸出限制

`writer_program_flow_design` 不會固定產生 `HOW TO READ THIS LOG` 區塊。Writer 只輸出啟動測試與判定所需的資訊，避免舊編譯器因 C 原始檔過大而失敗。

