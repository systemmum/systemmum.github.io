# Kodak DCS RAW (`.TIF` / `.DCR`) to PNG Converter

> **Language / 語言選擇：** [English](#english) | [廣東話 (Cantonese)](#cantonese-廣東話版本)

---

## English

A zero-dependency, 100% client-side web application for decoding, developing, and converting legacy **Eastman Kodak DCS (Digital Camera System)** RAW TIFF files (`.tif`, `.tiff`, `.dcr`) into standard `.png` images directly in the browser.

### Why This Tool Exists

In the 1990s and early 2000s, Kodak stored raw CCD sensor captures inside standard **TIFF 6.0** containers (`.tif` / `.tiff`, later renamed to `.dcr`). Because of this wrapper structure, modern operating systems and standard image viewers often fail to open them properly—either displaying only a tiny embedded thumbnail or failing to decode Kodak's proprietary sensor compression.

This application implements a full binary TIFF/SubIFD parser and RAW image processing pipeline in vanilla JavaScript, allowing photographers and archivists to recover and convert vintage Kodak DCS files without uploading sensitive or large RAW files to any external server.

### How the Conversion Process Works

When a file is loaded or dropped into the browser, it passes through a five-stage client-side pipeline:

```text
┌─────────────────────┐
│ 1. Binary Ingestion │  Reads raw file bytes via FileReader / ArrayBuffer (100% local)
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 2. TIFF 6.0 & IFD   │  Detects Endianness (II/MM), parses IFD0, and follows Tag 330
│    Tree Traversal   │  (SubIFDs) & Tag 65026 (KodakIFD) to locate full-res RAW data
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 3. Sensor Stream    │  Unpacks 8/10/12/16-bit uncompressed data or decodes Kodak
│    Decompression    │  lossless DPCM bitstreams (Comp 65000 / 32867 / 34713) & LUTs
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 4. Bayer Demosaic & │  Subtracts black pedestal, applies R/B white balance gains,
│    Color Pipeline   │  interpolates RGB (Malvar-He-Cutler / Bilinear), & applies CCM
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 5. Tone Mapping &   │  Applies gamma curve + contrast S-curve, renders to HTML5
│    PNG Export       │  Canvas, and exports lossless single or batch .PNG files
└─────────────────────┘
```

#### Stage-by-Stage Breakdown

1. **TIFF 6.0 & `SubIFD` Binary Parsing (`TiffDcsParser`)**
   * Detects Little-Endian (`II`) and Big-Endian (`MM`) byte order headers.
   * Traverses the primary `IFD0` (which typically holds a small low-resolution RGB preview) and automatically follows **Tag `330` (`0x014A` / `SubIFDs`)** to locate the high-resolution single-channel CCD sensor array.
   * Scans header blocks for embedded Kodak ASCII shooting parameters (camera model, serial numbers, shutter/aperture notes) and exposes all tags in an interactive **IFD & Metadata Inspector**.

2. **RAW Sensor Stream Decoding (`KodakDcsDecoder`)**
   * Reassembles strip-based (`StripOffsets` / `StripByteCounts`) or tile-based (`TileOffsets` / `TileByteCounts`) sensor payloads.
   * **Uncompressed RAW (`Compression = 1`):** Unpacks 8-bit, tightly packed 10-bit (4 pixels per 5 bytes), packed/word-aligned 12-bit, and 16-bit sensor streams.
   * **Kodak Lossless DPCM (`Compression = 65000`, `32867`, `34713`):** Decodes Kodak's variable-bitlength delta-encoded horizontal pixel pairs with per-scanline predictor resets.
   * **Linearization Curve:** Automatically applies Kodak's embedded linearization lookup table (**Tag `65100`**) when present.

3. **Bayer CFA Demosaicing (`RawDeveloper`)**
   * Converts the single-channel mosaic sensor grid into full-color RGB using either:
     * **Malvar-He-Cutler (5×5):** High-quality gradient-corrected linear interpolation for sharp edges and minimal color moiré.
     * **Bilinear Interpolation:** Fast 3×3 neighborhood averaging.
   * Supports manual selection of **`GRBG`**, **`RGGB`**, **`GBRG`**, **`BGGR`**, or **`Monochrome`** (for B&W sensors such as the DCS 420m / 460m).

4. **Color Matrix & Tone Curve**
   * Applies black-level pedestal subtraction, white-balance channel multipliers, exposure compensation (`EV`), and a `3×3` CCD Color Correction Matrix (**Kodak Ektachrome Vivid Boost**, **Kodak DCS Neutral CCD**, **Kodachrome Classic**, or **Linear Sensor RGB**).
   * Encodes output values with configurable gamma and contrast curves before writing to an `<canvas>` `ImageData` buffer.

5. **Lossless PNG Export**
   * Exports either the active image (`Download PNG`) or all queued files (`Download All`) using native browser `canvas.toBlob('image/png')`.

### Default Calibration Presets

The darkroom controls are pre-calibrated for classic Kodak DCS CCD output:

| Parameter | Default Value | Notes |
| :--- | :--- | :--- |
| **Bayer CFA Pattern** | `GRBG` | Standard sensor readout for Kodak DCS 420 / 460 / 520 / 620 |
| **Demosaic Algorithm** | `Malvar-He-Cutler` | 5×5 gradient-corrected interpolation |
| **White Balance (Red Gain)** | `1.30x` | Manual channel multiplier |
| **White Balance (Blue Gain)** | `1.30x` | Manual channel multiplier |
| **Black Level Pedestal** | `21` | Sensor dark-current floor offset |
| **Tone Contrast** | `1.30` | Midtone S-curve contrast |
| **Output Gamma** | `2.20` | Standard sRGB gamma encoding |
| **Kodak CCD Color Profile** | `Kodak Ektachrome Vivid Boost` | Enhanced slide-film color matrix |

### Supported Camera Models

Designed for legacy Kodak Professional DCS digital backs and SLRs, including:
* **Kodak DCS 200 / 420 / 460** (Nikon N8008s / N90s bodies)
* **Kodak EOS DCS 1 / 3 / 5** (Canon EOS-1N bodies)
* **Kodak DCS 520 / 560** (Canon EOS-1N D2000 / D6000)
* **Kodak DCS 620 / 620x / 660 / 760** (Nikon F5 bodies)
* **Monochrome Variants** (`DCS 420m`, `DCS 460m`, `DCS 760m`) via the `Monochrome` CFA mode

### Usage & Deployment

Because the entire application is self-contained in a single HTML file (`kodak_dcs_converter.html`), no build step, Node.js runtime, or backend server is required:

1. **Run Locally:** Open `kodak_dcs_converter.html` directly in any modern browser (Chrome, Firefox, Edge, Safari).
2. **Host on GitHub Pages:**
   * Rename `kodak_dcs_converter.html` to `index.html` in your repository root.
   * Enable **GitHub Pages** under **Settings → Pages** and select your main branch.
3. **Test Without Sample Files:** Click **"Test with Synthetic DCS RAW Sample"** on the dropzone to generate an in-memory multi-IFD Kodak DCS 460C `.TIF` test file and inspect the full conversion pipeline.

### Privacy & Security

* **100% Client-Side Execution:** All binary parsing, decompression, demosaicing, and PNG encoding happen locally in browser memory.
* **No Network Transfers:** Your `.TIF` / `.DCR` files never leave your device.

---

## Cantonese (廣東話版本)

呢個係一個完全唔需要任何外部依賴套件（Zero-dependency）、**100% 喺瀏覽器本地端（Client-Side）運行**嘅網頁工具，專門用嚟解碼、沖曬同埋將舊款 **Eastman Kodak DCS（Digital Camera System）** 數碼相機影出嚟嘅 RAW TIFF 檔案（`.tif`、`.tiff`、`.dcr`）直接轉做標準 `.png` 圖片。

### 點解需要呢個工具？

喺 1990 年代至 2000 年代初，Kodak 將 CCD 感光元件嘅原始 RAW 數據封裝喺標準 **TIFF 6.0** 格式入面（副檔名係 `.tif` / `.tiff`，後期改叫 `.dcr`）。因為呢種特別嘅封裝結構，現代作業系統同一般睇圖軟件好多時都開唔到呢啲相——要唔係淨係顯示到一張好細嘅內嵌縮圖，就係完全解碼唔到 Kodak 嘅獨家壓縮格式。

呢個網頁用純 JavaScript 寫咗一套完整嘅 TIFF / SubIFD 二進位解析器同 RAW 影像處理管線（Pipeline），等攝影師同檔案保育人員唔使將大體積或者私密嘅 RAW 檔上傳去任何外部伺服器，都可以直接喺瀏覽器救返同轉換呢批經典 Kodak DCS 相片。

### 轉換處理流程係點運作？

當你揀選或者拖放（Drag & Drop）檔案入網頁之後，系統會經過以下五個本地處理階段：

```text
┌─────────────────────┐
│ 1. 二進位檔案讀取   │  透過 FileReader / ArrayBuffer 直接讀取檔案（100% 本地處理）
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 2. TIFF 6.0 同 IFD  │  偵測位元組序（II/MM），解析 IFD0，並追蹤 Tag 330 (SubIFDs)
│    目錄樹狀結構解析 │  同 Tag 65026 (KodakIFD) 搵出真正嘅全解像度 RAW 感光數據
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 3. 感光元件數據串流 │  解包 8/10/12/16-bit 無壓縮數據，或者解碼 Kodak 無損 DPCM
│    解壓縮           │  位元串流（壓縮編碼 65000 / 32867 / 34713）同線性化對照表
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 4. 拜耳陣列去馬賽克 │  扣除黑電平基底值、套用紅/藍白平衡增益、進行 RGB 色彩插值
│    同色彩管線處理   │  （Malvar-He-Cutler / 雙線性插值）並套用 CCD 色彩矩陣
└──────────┬──────────┘
           ▼
┌─────────────────────┐
│ 5. 色調映射同       │  套用 Gamma 曲線同對比度 S 曲線，渲染到 HTML5 Canvas，
│    PNG 無損匯出     │  再匯出單張或者批次無損 .PNG 圖片
└─────────────────────┘
```

#### 五個階段詳細說明

1. **TIFF 6.0 同 `SubIFD` 二進位解析（`TiffDcsParser`）**
   * 自動識別 Little-Endian（`II`）同 Big-Endian（`MM`）位元組序標頭。
   * 讀取主目錄 `IFD0`（通常入面只係擺咗一張低解像度 RGB 預覽縮圖），再自動追蹤 **Tag `330`（`0x014A` / `SubIFDs`）** 搵出真正高解像度嘅單通道 CCD 感光元件原始數據。
   * 掃描檔案標頭入面內嵌嘅 Kodak ASCII 拍攝參數（相機型號、機身編號、快門/光圈紀錄），並喺互動式 **IFD & Metadata 檢視器**入面列出所有標籤數據。

2. **RAW 感光元件數據解碼（`KodakDcsDecoder`）**
   * 重新組合條帶式（`StripOffsets` / `StripByteCounts`）或者分塊式（`TileOffsets` / `TileByteCounts`）嘅感光元件數據。
   * **無壓縮 RAW（`Compression = 1`）：** 支援解包 8-bit、緊密封裝嘅 10-bit（每 5 bytes 存 4 個像素）、封裝/字元對齊 12-bit 同 16-bit 原始數據流。
   * **Kodak 無損 DPCM 壓縮（`Compression = 65000`、`32867`、`34713`）：** 解碼 Kodak 可變位元長度（Variable-bitlength）水平像素差分編碼，並喺每一條掃描線開頭重置預測器。
   * **線性化曲線（Linearization Curve）：** 如果檔案內嵌咗 Kodak 線性化對照表（**Tag `65100`**），系統會自動套用還原線性亮度。

3. **拜耳陣列（Bayer CFA）去馬賽克（`RawDeveloper`）**
   * 將單通道黑白馬賽克感光網格還原成全彩 RGB 影像，提供兩種演算法選擇：
     * **Malvar-He-Cutler (5×5)：** 高畫質梯度修正線性插值，邊緣更銳利，同時減少偽色同摩爾紋（Moiré）。
     * **Bilinear Interpolation（雙線性插值）：** 快速 3×3 鄰近像素平均插值。
   * 支援手動切換 **`GRBG`**、**`RGGB`**、**`GBRG`**、**`BGGR`** 拜耳排列，或者 **`Monochrome`**（專門畀 DCS 420m / 460m 等純黑白感光元件使用）。

4. **色彩矩陣同色調曲線**
   * 執行黑電平基底（Black Level Pedestal）扣除、紅/藍白平衡通道增益（White Balance Gains）、曝光補償（`EV`），並套用 `3×3` CCD 色彩校正矩陣（支援 **Kodak Ektachrome Vivid Boost**、**Kodak DCS Neutral CCD**、**Kodachrome Classic** 同 **Linear Sensor RGB**）。
   * 寫入 `<canvas>` `ImageData` 緩衝區之前，會先經過可自訂嘅 Gamma 同對比度曲線編碼。

5. **無損 PNG 匯出**
   * 透過瀏覽器原生 `canvas.toBlob('image/png')` 匯出當前預覽圖片（`Download PNG`），或者一鍵批次匯出排隊清單入面所有相片（`Download All`）。

### 預設沖曬校準參數

右側嘅數碼暗房控制面板已經預先針對經典 Kodak DCS CCD 色彩特性校準好預設值：

| 參數項目 | 預設數值 | 說明 |
| :--- | :--- | :--- |
| **拜耳陣列排列（Bayer CFA Pattern）** | `GRBG` | Kodak DCS 420 / 460 / 520 / 620 嘅標準感光元件排列 |
| **去馬賽克演算法（Demosaic Algorithm）** | `Malvar-He-Cutler` | 5×5 梯度修正高畫質插值 |
| **白平衡（紅色增益 Red Gain）** | `1.30x` | 手動紅色通道倍數 |
| **白平衡（藍色增益 Blue Gain）** | `1.30x` | 手動藍色通道倍數 |
| **黑電平基底（Black Level Pedestal）** | `21` | 感光元件暗電流底噪偏移值 |
| **色調對比度（Tone Contrast）** | `1.30` | 中間調 S 曲線對比度 |
| **輸出 Gamma（Output Gamma）** | `2.20` | 標準 sRGB Gamma 編碼 |
| **Kodak CCD 色彩設定檔** | `Kodak Ektachrome Vivid Boost` | 增強型正片菲林色彩矩陣 |

### 支援嘅相機型號

專為舊款 Kodak Professional DCS 數碼機背同單反相機而設，包括：
* **Kodak DCS 200 / 420 / 460**（Nikon N8008s / N90s 機身）
* **Kodak EOS DCS 1 / 3 / 5**（Canon EOS-1N 機身）
* **Kodak DCS 520 / 560**（Canon EOS-1N D2000 / D6000）
* **Kodak DCS 620 / 620x / 660 / 760**（Nikon F5 機身）
* **純黑白版本（`DCS 420m`、`DCS 460m`、`DCS 760m`）**（可揀選 `Monochrome` CFA 模式）

### 使用方法同 GitHub Pages 部署

成個應用程式已經整合喺單一 HTML 檔案（`kodak_dcs_converter.html`）入面，完全唔需要編譯（Build）、Node.js 環境或者後端伺服器：

1. **本地直接開啟：** 用任何現代瀏覽器（Chrome、Firefox、Edge、Safari）直接雙擊打開 `kodak_dcs_converter.html` 就用得。
2. **部署上 GitHub Pages：**
   * 將 Repository 根目錄入面嘅 `kodak_dcs_converter.html` 改名為 `index.html`。
   * 喺 GitHub 專案入面去 **Settings → Pages**，揀選你嘅主分支（Main branch）啟用 GitHub Pages 即可。
3. **冇樣本檔案都可以測試：** 喺拖放區撳 **「Test with Synthetic DCS RAW Sample」**，系統會即時喺記憶體生成一個包含多層 IFD 結構嘅 Kodak DCS 460C `.TIF` 測試檔，畀你試玩成個轉換流程。

### 私隱同安全性

* **100% 瀏覽器本地執行：** 所有二進位檔案解析、解壓縮、去馬賽克同 PNG 編碼都完全喺你部機嘅瀏覽器記憶體入面進行。
* **零網絡傳輸：** 你嘅 `.TIF` / `.DCR` 原始相片絕對唔會上傳去任何伺服器，100% 保障檔案私隱。
