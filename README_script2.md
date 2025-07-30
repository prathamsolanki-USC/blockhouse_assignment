# script2.ipynb

A one‑stop Colab notebook that **cleans MBP‑10 order‑book data, creates minute‑bar snapshots, simulates market‑order slippage, and fits simple impact models** for three example tickers.

> **Important:** *The raw CSV data is **not** committed to this repository.*  
> You must supply your own MBP‑10 files locally or in Google Drive.

---

## 1. What the notebook does

| Section | Purpose |
|---------|---------|
| **Setup & paths** | You set a single variable `base_path` that points to your local or Drive folder which **contains a `/Data` directory**: `base_path / "Data" / <TICKER> / <DAY>.csv`. |
| **Cleaning step** | Reads each CSV, keeps only the columns we need (`ts_event`, `seq_num`, `side`, `depth`, `price`, `size`), converts units, drops corrupt rows, checks `seq_num` order, then writes parquet files to `base_path/clean/<TICKER>/...`. |
| **Snapshot builder** | Resamples to one book snapshot per minute (best 10 levels) and records spread, mid‑price, depth, imbalance, realised 1‑min volatility. |
| **Impact simulation** | For each snapshot and size ladder `[50, 100, …]` shares, walks the book to compute VWAP and absolute slippage; saves results to `base_path/slippage/`. |
| **Model fitting** | Pools the impact tables, fits linear vs convex power‑law slippage models, prints diagnostics, plots fits. |

---

## 2. Folder layout **expected at runtime**

```
<base_path>/              ← you point to this, not included in repo
├── Data/
│   ├── FROG/ 2025‑02‑03.csv …
│   ├── CRWV/ *.csv
│   └── SOUN/ *.csv
└── (generated)
    ├── clean/
    └── slippage/
```

*Only the notebook itself (`script2.ipynb`) and this README are version‑controlled.*  
Large raw data stays local.

---

## 3. Quick‑start (Colab)

```python
from google.colab import drive
drive.mount("/content/drive")

base_path = Path("/content/drive/MyDrive/blockhouse")  # 🔁 edit me
```

Then **Runtime ▸ Run all**.  On a Colab CPU instance the full pipeline takes 5‑10 min per ticker.

---

## 4. Environment

No exotic packages:

```text
pandas numpy pyarrow matplotlib scikit‑learn tqdm
```

Colab already has them; locally just:

```bash
pip install -r requirements.txt
```

---

## 5. .gitignore snippet (already included)

```
# never commit data or derived artefacts
Data/
clean/
slippage/
```

---

## 6. Re‑running

Delete `clean/` or `slippage/` and rerun the respective notebook cells.  Cleaning is deterministic; simulation depends only on the size ladder constant in the cell header.

---

*Last updated:* 2025‑07‑30
