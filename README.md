# 速與競：FPGA 音樂遊戲

以 Verilog 實作、使用 Basys 3 的五軌音樂遊戲。玩家在方塊到達基準線時按下對應按鍵，已成功擊中的音符會保留並在後續輪次播放；漏掉的方塊會再次出現。提供快、正常、慢三種速度，搭配 VGA 顯示、鍵盤輸入、音訊輸出與七段顯示器。

> **目前狀態：原始提交檔案的整理版，尚未能從現有原始碼完整重建。**
> 已確認缺少 `KeyboardDecoder` 的定義，且音訊模組有重複定義。此套件未執行 Vivado 編譯、模擬或硬體驗證。

## 檔案結構

```text
.
├── README.md
├── .gitignore
├── src/
│   └── Group29_code.v
├── constraints/
│   └── Group29_code.xdc
├── docs/
│   ├── report.pdf
│   ├── demo.pptx
│   └── RECOVERY.md
└── prebuilt/
    └── Group29.bit
```

- `src/Group29_code.v`：原始 Verilog 檔，保留原始位元組內容。包含遊戲、VGA、分頻、七段顯示和音訊等模組。
- `constraints/Group29_code.xdc`：接腳、100 MHz 輸入時脈及其他 FPGA 約束。
- `docs/report.pdf`、`docs/demo.pptx`：原始專題報告與簡報，僅調整檔名。
- `prebuilt/Group29.bit`：既有 FPGA bitstream，屬於編譯產物，不能取代缺少的原始碼。未確認能由本套件內容產生同一份 bitstream。
- `docs/RECOVERY.md`：已確認的缺件、原始碼位置和重建步驟。

## 已確認的阻礙

| 項目 | 現況 | 處理方式 |
| --- | --- | --- |
| `KeyboardDecoder` | 第 68 行建立 `key_de` 實例，但沒有模組定義 | 找回原始鍵盤模組，並補上它使用的子模組 |
| `speaker_control` | 第 927、1116 行各有一份定義；忽略註解與空白後內容相同 | 保留一份定義 |
| `note_gen` | 第 1019、1208 行各有一份定義；忽略註解與空白後內容相同 | 保留一份定義 |
| 重建與驗證 | 沒有執行 Vivado、模擬或板上測試 | 補齊原始碼後重新編譯，檢查設計規則與時序，並測試硬體 |

沒有 `.xpr` 不代表一定缺少程式；Vivado 可以用來源檔與約束重新建立專案，或以 Tcl 流程重建。原有 `.xpr`、建置腳本、工具設定若找得到，仍有助於復原原本環境。

## 硬體與工具線索

- 目標板：Basys 3；XDC 註解標示 rev B。
- 頂層模組：`demo_1`。
- 輸入時脈：100 MHz；XDC 設定週期為 10 ns。
- VGA：程式設定為 640 × 480 的有效顯示區域。
- 鍵盤：透過 Basys 3 的 PS/2 介面訊號。
- 音訊：四條音訊訊號對應到 Pmod JB；原始材料沒有充分確認外接音訊模組型號、設定與完整接線，重現時需補充。
- 既有 `.bit` 標頭記錄設計名稱 `demo_1`、器件 `7a35tcpg236`、工具版本 `2019.2`、產生日期 `2023/01/16`。這些是標頭資訊，不代表本套件已經以該版本驗證。

## 原始操作線索

依程式掃描碼判讀，五軌由左至右為 `D`、`F`、`G`、`H`、`J`；仍需補回鍵盤模組並在硬體上確認。

| 控制 | 用途 |
| --- | --- |
| SW0 / `zero` | 重設遊戲捲動計數；保持為 1 時會持續重設。現有程式沒有透過此訊號清除已擊中音符紀錄 |
| SW1 / `fast` | SW1=1 且 SW2=0 時加快 |
| SW2 / `slow` | SW1=0 且 SW2=1 時放慢 |
| SW1、SW2 同值 | 正常速度 |
| `rst` / U17 | 重設 VGA、鍵盤、音訊、七段顯示相關模組；與 `zero` 的範圍不同 |

## 建立 GitHub repo

1. 在 GitHub 建立空白 repository，例如 `group29-fpga-rhythm-game`。
2. 將此資料夾裡的內容上傳到 repo；`README.md` 應位於 repo 最外層。
3. 先保留「尚未能完整重建」的狀態說明。
4. 找回鍵盤模組、移除重複定義並驗證後，再更新狀態與重建步驟。

本資料夾是可上傳的檔案套件，尚未連結遠端 GitHub，也沒有建立提交紀錄。

## 參考資料

專題內容以 `docs/report.pdf`、`docs/demo.pptx` 與附上的原始碼為依據。

- [Digilent Basys 3 Reference Manual](https://digilent.com/reference/_media/reference/programmable-logic/basys-3/basys3_rm.pdf)
- [AMD Vivado：使用來源檔與 Tcl 進行重建](https://docs.amd.com/r/2022.2-English/ug895-vivado-system-level-design-entry/Working-with-Sources-in-Non-Project-Mode)
