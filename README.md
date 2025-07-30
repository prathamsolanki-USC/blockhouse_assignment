# script2.ipynb

A self‑contained Google‑Colab notebook that cleans raw MBP‑10 order‑book data, builds a minute‑bar impact table, and fits simple slippage models for three test tickers.

---

## 1. What the notebook does

| Section | Purpose |
|---------|---------|
| **Mount Google Drive** | Gives the notebook access to the project directory you already have in Drive. |
| **Configuration** | Set `base_path`, then derive:<br>• `RAW_DIR = base_path + "Data"` – raw CSV folders, one per ticker<br>• `CLEANED_DIR = base_path + "cleaned"` – auto‑created parquet output<br>• `SLIPPAGE_DIR = base_path + "slippage"` – simulated order‑cost tables |
| **Data cleaning** | *For every CSV in `Data/<TICKER>/`*<br>1. Keep only `ts_event`, `seq_num`, `side`, `depth`, `price`, `size`.<br>2. Convert `price` from nano‑dollars to dollars (`price_$`).<br>3. Convert `size` to integer `shares`.<br>4. Drop impossible rows and enforce `seq_num` monotonicity.<br>5. Save to `cleaned/<TICKER>/<DATE>.parquet`. |
| **Minute‑bar snapshot builder** | Resamples each parquet file to one snapshot per minute, keeping the freshest 10‑level book.  Adds helper columns like spread, mid‑price, book imbalance, and a “minute_of_day” index. |
| **Instant market‑order simulator** | For each snapshot and each order size in `[50, 100, …, 300]`, walks the ask/bid ladder to compute VWAP and absolute slippage.  Results are written to `slippage/<TICKER>_<DATE>_impact.parquet`. |
| **Model fitting & diagnostics** | Pools the slippage table across days; fits simple polynomial regressions (degrees 1‑4) of slippage on order size, spread, and volatility; prints R² and plots fitted curves. |
| **Visualisations** | Quick matplotlib plots: spreads through the day, slippage vs size, model fits.  These are purely exploratory and can be commented out if running head‑less. |

---

## 2. Folder expectations

```
<base_path>/
├── Data/                ← three sub‑folders already supplied
│   ├── FROG/  *.csv
│   ├── CRWV/  *.csv
│   └── SOUN/  *.csv
├── script2.ipynb        ← this notebook
└── (created at runtime)
    ├── cleaned/
    │   └── <TICKER>/ *.parquet
    └── slippage/
        └── *.parquet
```

No other scripts are required.

---

## 3. How to run

1. Open **script2.ipynb** in Google Colab.  
2. Set `base_path` to the parent folder containing *Data/*.  
   ```python
   base_path = "/content/drive/MyDrive/blockhouse/"
   ```  
3. `Runtime ▸ Run all` – cleaning, snapshots, simulations, and model fits take ~5–10 minutes per ticker on Colab CPU.

---

## 4. Dependencies

The notebook installs nothing fancy—just:

* `pandas`
* `numpy`
* `pyarrow` (for parquet)
* `matplotlib`
* `scikit‑learn`
* `tqdm`

Google Colab already has these; if you run locally just `pip install -r requirements.txt` with the same names.

---

## 5. Output artefacts

| File / folder | What it is |
|---------------|------------|
| **cleaned/** | Per‑minute, cleaned order book snapshots (parquet) |
| **slippage/** | Impact tables: `(timestamp_min, order_size, vwap, abs_slip, spread, minute_of_day, …)` |
| **plots** (inline) | Quick‑look charts to sanity‑check fits |

These outputs feed directly into the later optimisation step (Almgren‑Chriss style scheduler).

---

## 6. Reproducibility tips

* Cleaning is deterministic—delete `cleaned/` and re‑run to reproduce.  
* If you tweak order‑size ladders or add new tickers, only the simulation cell needs rerunning.  
* Commit the notebook and the two output folders to your fork so reviewers can rerun *Start‑to‑Finish* with a single click.

---

*Last updated:* 2025‑07‑30
