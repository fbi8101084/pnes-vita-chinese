# Build Instructions / 編譯說明

This document describes how to build **pNES 7.1 for PS Vita with Chinese UTF-8 ROM filename support** on macOS.

本文說明如何在 macOS 上編譯 **pNES 7.1 PS Vita 中文檔名支援版本**。

本版本沒有修改模擬器核心。pNES 7.1 本身已具備 UTF-8 字串解碼能力，主要修改為將官方預設字型替換成支援中文字形的 **Noto Sans SC**。

---

## Upstream / 上游專案

pEMU / pNES:

https://github.com/Cpasjuste/pemu

Base version / 基礎版本：

```text
v7.1
```

Font / 字型：

```text
NotoSansSC-VF.ttf
```

Noto CJK:

https://github.com/notofonts/noto-cjk

---

# 1. Install macOS Dependencies

安裝 macOS 編譯工具：

```bash
brew install cmake git wget pkg-config
```

確認：

```bash
cmake --version
git --version
pkg-config --version
```

---

# 2. Install VitaSDK

建立工作目錄：

```bash
cd ~/work
```

下載 VitaSDK package manager：

```bash
git clone https://github.com/vitasdk/vdpm.git
cd vdpm
```

安裝 VitaSDK：

```bash
./bootstrap-vitasdk.sh
```

設定環境變數：

```bash
export VITASDK=/usr/local/vitasdk
export PATH=$VITASDK/bin:$PATH
```

確認 Vita cross compiler：

```bash
which arm-vita-eabi-gcc
```

正常應該指向：

```text
/usr/local/vitasdk/bin/arm-vita-eabi-gcc
```

---

# 3. Install VitaSDK Dependencies

安裝 pNES Vita build 所需套件：

```bash
vdpm libvita2d freetype bzip2 libpng libconfig zlib sdl2 curl openssl tinyxml2 libarchive minizip zstd xz Box2D
```

---

# 4. Clone pEMU

回到工作目錄：

```bash
cd ~/work
```

下載 pEMU：

```bash
git clone --recursive https://github.com/Cpasjuste/pemu.git
cd pemu
```

切換到 pNES 7.1：

```bash
git checkout v7.1
git submodule update --init --recursive
```

確認版本：

```bash
git describe --tags --always
```

應顯示：

```text
v7.1
```

---

# 5. Replace the Default Font

pNES 使用的預設字型位於：

```text
data/common/romfs/skins/default/default.ttf
```

先備份官方字型：

```bash
cd ~/work/pemu/data/common/romfs/skins/default

cp default.ttf default-original.ttf
```

將：

```text
NotoSansSC-VF.ttf
```

複製進此目錄，並重新命名為：

```text
default.ttf
```

例如：

```bash
cp ~/Downloads/NotoSansSC-VF.ttf \
   ~/work/pemu/data/common/romfs/skins/default/default.ttf
```

Noto Sans SC 使用 SIL Open Font License 1.1。

---

# 6. Configure the Vita Build

建立獨立 build directory：

```bash
cd ~/work/pemu

mkdir -p cmake-build
cd cmake-build
```

設定環境：

```bash
export VITASDK=/usr/local/vitasdk
export PATH=$VITASDK/bin:$PATH
export PKG_CONFIG_PATH=/usr/local/vitasdk/arm-vita-eabi/lib/pkgconfig:$PKG_CONFIG_PATH
```

執行 CMake：

```bash
cmake -G "Unix Makefiles" \
    -DCMAKE_BUILD_TYPE=Release \
    -DPLATFORM_VITA=ON \
    -DOPTION_MPV_PLAYER=OFF \
    ..
```

成功時最後應看到：

```text
-- Configuring done
-- Generating done
```

---

# 7. Build pNES VPK

執行：

```bash
make pnes.vpk
```

成功時應看到：

```text
[100%] Built target pnes.vpk
```

產生的 VPK 位於：

```text
cmake-build/src/cores/pnes/pnes.vpk
```

例如：

```bash
ls -lh src/cores/pnes/pnes.vpk
```

---

# 8. Incremental Builds

第一次完整編譯需要較長時間。

之後如果只修改字型或少量程式碼，不需要刪除 `cmake-build`。

直接執行：

```bash
cd ~/work/pemu/cmake-build

make pnes.vpk
```

Make 會只重新編譯有變動的部分。

---

# Troubleshooting / 疑難排解

## `pkg-config` not installed

如果 CMake 出現：

```text
I require pkg-config but it's not installed.
```

安裝：

```bash
brew install pkg-config
```

確認：

```bash
pkg-config --version
```

---

## `libconfig` not found

部分 VitaSDK 環境可能會出現：

```text
None of the required 'libconfig' found
```

即使：

```bash
vdpm libconfig
```

顯示：

```text
libconfig is up to date
```

先確認 library 與 header 已經存在：

```bash
find /usr/local/vitasdk/arm-vita-eabi \
    \( -name 'libconfig.a' -o -name 'libconfig.h' -o -name 'libconfig++.h' \)
```

如果 `libconfig.a` 與 `libconfig.h` 都存在，但缺少：

```text
libconfig.pc
```

建立 pkg-config directory：

```bash
mkdir -p /usr/local/vitasdk/arm-vita-eabi/lib/pkgconfig
```

建立：

```text
/usr/local/vitasdk/arm-vita-eabi/lib/pkgconfig/libconfig.pc
```

內容：

```ini
prefix=/usr/local/vitasdk/arm-vita-eabi
exec_prefix=${prefix}
libdir=${prefix}/lib
includedir=${prefix}/include

Name: libconfig
Description: C/C++ configuration file library
Version: 1.8.2
Libs: -L${libdir} -lconfig
Cflags: -I${includedir}
```

設定：

```bash
export PKG_CONFIG_PATH=/usr/local/vitasdk/arm-vita-eabi/lib/pkgconfig:$PKG_CONFIG_PATH
```

再重新執行 CMake。

---

## `cannot find -lbox2d`

如果最後 Linking 階段出現：

```text
ld: cannot find -lbox2d
```

安裝 VitaSDK Box2D：

```bash
vdpm Box2D
```

確認：

```bash
find /usr/local/vitasdk/arm-vita-eabi/lib -iname '*box2d*'
```

然後直接重新：

```bash
make pnes.vpk
```

不需要刪除 `cmake-build`，已經完成的 object files 可以繼續使用。

---

# Why Chinese Filenames Work / 中文檔名原理

pNES 7.1 的文字渲染系統本身已經包含 UTF-8 decoding。

其文字處理流程大致為：

```text
UTF-8 ROM filename
        ↓
UTF-8 decoder
        ↓
Unicode codepoint
        ↓
FreeType
        ↓
Font glyph
        ↓
PS Vita display
```

因此中文 ROM 檔名無法顯示的主要原因不是 pNES 無法讀取 UTF-8，而是官方預設：

```text
default.ttf
```

缺少中文字形。

將其替換為支援中文的 **Noto Sans SC** 後，即可利用 pNES 原本的 UTF-8 與 FreeType 支援顯示中文檔名。

No emulator core modifications are required.

---

# Tested / 實機測試

Tested on a real PS Vita.

已於 PS Vita 實機測試 UTF-8 中文 `.nes` 檔名，例如：

```text
雙截龍3中文版.nes
雙截龍3.nes
雙截龍4.nes
台灣16張麻將(中文).nes
古巴戰士.nes
```

中文檔名可以正常顯示，ROM 可以正常啟動。

---

# License / 授權

pEMU / pNES remains subject to the license of the upstream project:

https://github.com/Cpasjuste/pemu

Noto Sans SC is distributed under the **SIL Open Font License 1.1**.

See:

```text
FONT-LICENSE.txt
```

for the full font license.

This project does **not** include any ROM files.

---

# Disclaimer

This is an unofficial community build.

This project is not affiliated with or endorsed by the original pEMU / pNES developers.

本專案為非官方社群版本，與 pEMU / pNES 原作者無官方隸屬或背書關係。
