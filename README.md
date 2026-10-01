# AI Receiving Manager — Autonomous Dock Inspection POC

**Cube Buildathon · 01 · Receiving Manager · Solution Implementation**

A clean, professional, and functional frontend for an **AI-powered Receiving Manager** designed for warehouse receiving docks and 3PL inbound operations. The system autonomously inspects incoming carton images, performs optical barcode extraction, reconciles products against the **Product Master Data Catalog**, localizes physical package damage via **MobileNetV3-Large**, and issues instantaneous **PASS** or **FAIL** receiving signals with full decision traceability.

---

## 1. Clean 2-Tab Architecture

The application is structured into two dedicated workflows:

```
+---------------------------------------------------------------------------------+
|                                RECEIVING AI                                     |
|              [ ⦿ Scan & Inspect ]    [ 📄 Product Master Data (3) ]             |
+---------------------------------------------------------------------------------+
```

### Tab 1: Scan & Inspect (`#inspectView`)
* **Zero Manual Form Inputs**: No typing of barcodes, PO numbers, or SKU names.
* **Carton Image Drag & Drop**: Drop any carton photo (JPG, PNG, WebP, SVG).
* **Defect Localization Hitboxes**: Overlays precise, pulsating bounding boxes on the carton image indicating damage zones (crushed corrugation, punctures) with model confidence scores.
* **Barcode Detection & Error Handling**:
  * **When Barcode Found**: Extracted barcode is immediately matched against the active Product Master Database.
  * **When No Barcode Found**: Explicitly displays **"NO BARCODE DETECTED"** error banner, skips fake data, and fails receiving compliance.
* **Instant Signal Verdicts**:
  * 🟢 **GREEN SIGNAL (PASS)**: Intact packaging + Barcode matches active manifest catalog.
  * 🔴 **RED SIGNAL (FAIL)**: Packaging compromised (crushed/punctured) OR unreadable/unregistered barcode.
* **Tight Image Viewport**: Sized to the exact dimensions of the carton image with zero empty black letterbox space.

### Tab 2: Product Master Data (`#productsView`)
* **Ground-Truth Manifest Database**: Serves as the catalog against which all scanned cartons are verified.
* **CSV & JSON Manifest Import**: Drag and drop manifest files (including `data/receiving_sample.csv`) to update expected SKUs, barcodes, quantities, and suppliers.
* **Interactive Management**: Add custom product lines, delete outdated SKUs, search/filter active records, or reset to default demo data.

---

## 2. Quick Start / How to Run Locally

Built with standard **HTML5**, **CSS3**, and **vanilla JavaScript** without any external dependencies or build tools.

### Option A: Local Python Server (Recommended)
```bash
# In terminal at project directory:
python -m http.server 3000
```
Open `http://localhost:3000` in your web browser.

### Option B: Direct Browser Launch
Open `index.html` directly in Chrome, Edge, Safari, or Firefox:
```text
file:///d:/RCV/index.html
```

---

## 3. Project File Structure

```text
d:\RCV\
├── index.html          # Semantic 2-tab HTML5 layout
├── style.css           # Modern SaaS warehouse design system with green/red signals & hitboxes
├── script.js           # Autonomous scanner logic, barcode OCR matching, manifest state
├── README.md           # Documentation & user guide
├── assets\
│   ├── carton-intact.svg    # Pristine carton (Barcode: 789102938475 -> SKU-ELECTRONICS-402)
│   ├── carton-damaged.svg   # Crushed carton with structural failure & no barcode
│   └── carton-label.svg     # High-resolution shipping barcode label
└── data\
    └── receiving_sample.csv # Reference dataset for receiving unit trials
```

---

## 4. Quick Demo Test Cases

In Tab 1 (Scan & Inspect), use the **⚡ Quick Samples** buttons:

1. **Intact Carton (`carton-intact.svg`)**:
   * MobileNetV3 detects `INTACT / NO DEFECTS` (97.4% confidence).
   * Barcode scanner reads `789102938475`.
   * Matched with `SKU-ELECTRONICS-402` (Apex Global Logistics, 100 Units).
   * **Signal**: 🟢 **PASS (Shipment Approved)**.

2. **Damaged Carton (`carton-damaged.svg`)**:
   * MobileNetV3 localizes `CRUSH & PUNCTURE DAMAGE` (98.2% confidence).
   * Barcode status: `NO BARCODE DETECTED`.
   * Displays pulsating red hitbox over damaged region.
   * **Signal**: 🔴 **FAIL (Shipment Rejected)**.

3. **Shipping Barcode Label (`carton-label.svg`)**:
   * High-resolution linear barcode scan.
   * Matches active manifest catalog.
   * **Signal**: 🟢 **PASS**.

---

## 5. Connecting a Real Backend / AI Model

To connect a live FastAPI / Flask backend running MobileNetV3 and barcode reading (e.g. PyZbar / OpenCV):
1. In `script.js`, replace the `runVisionAndBarcodeClassification` handler with a `fetch` call sending the `FormData` containing the carton image to your endpoint.
2. Return JSON structure:
   ```json
   {
     "barcode": "789102938475",
     "condition": "INTACT",
     "confidence": 0.974,
     "damageScore": 0.026,
     "hitboxes": [
       { "x": 20, "y": 20, "w": 60, "h": 65, "type": "intact", "label": "INTACT (97.4%)" }
     ]
   }
   ```

---

## 6. Buildathon Context & Position in Chain

```text
 Supplier delivery      Inbound to Amazon     Outbound to buyer     Customer return        Money back
 ┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
 │ 01 Receiving │ ───▶ │ 02 Prep      │ ───▶ │ 03 Pack      │ ───▶ │ 04 Returns   │      │ 05 Recovery  │
 │ condition on │      │ compliance   │      │ contents at  │      │ condition &  │      │ reads all    │
 │ arrival      │      │ proof        │      │ seal         │      │ disposition  │      │ four → claim │
 └──────┬───────┘      └──────┬───────┘      └──────┬───────┘      └──────┬───────┘      └──────▲───────┘
        └─────────────────────┴─────────────────────┴─────────────────────┴─────────────────────┘
```
