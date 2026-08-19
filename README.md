# pNES Vita Chinese

PS Vita 上的 pNES 7.1 中文檔名支援版本。  
Chinese UTF-8 ROM filename support for pNES 7.1 on PS Vita.

## 中文說明

pNES 7.1 本身已具備 UTF-8 字串解碼能力，但官方預設的 `default.ttf`
缺少中文字形，因此中文 ROM 檔名在 PS Vita 上無法正常顯示。

本版本將預設字型替換為支援中文的 **Noto Sans SC**，
讓 pNES 可以正常顯示 UTF-8 中文 ROM 檔名。

例如：

- `雙截龍3中文版.nes`
- `台灣16張麻將(中文).nes`
- `古巴戰士.nes`

已於 PS Vita 實機測試。

## English

pNES 7.1 already contains UTF-8 text decoding support, but its bundled
default font does not contain Chinese glyphs.

This build replaces the default font with **Noto Sans SC**, allowing
UTF-8 Chinese ROM filenames to display correctly on PS Vita.

No emulator core modifications are required.

## Installation

1. Download `pnes-7.1-chinese.vpk` from Releases.
2. Transfer the VPK to your PS Vita.
3. Install it using VitaShell.
4. Launch pNES normally.

## Base Version

Based on pNES / pEMU 7.1 by Cpasjuste:

https://github.com/Cpasjuste/pemu

## Font

Noto Sans SC  
Copyright Google LLC

Licensed under the SIL Open Font License 1.1:

https://openfontlicense.org/

## Disclaimer

This is an unofficial community build and is not affiliated with or
endorsed by the original pEMU/pNES developers.

ROM files are not included.
