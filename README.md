# QuantData

QuantData is a collection of Jupyter notebooks for automating quantitative factor research. It uses the OpenAI API to interpret factor descriptions (even from annotated images), generate Python implementations, validate them against minute-level market data, and evaluate the resulting factors before combining them into multi-factor portfolios.

---

## Key Features

- **LLM-driven factor ideation**: Capture factor definitions from prompts or screenshots, turn them into runnable code, and iterate automatically when errors occur.  
- **Batch factor computation and testing**: Generate factor values from minute data, merge with daily aggregates, neutralize by market value, and produce ranked deciles with backtests for each factor notebook.  
- **Multi-factor ranking**: Combine individual factor ranks into a single feather dataset ready for downstream portfolio construction.  

---

## Repository Layout

- `API_factor_process.ipynb` – Loads API credentials from `.env`, instantiates the OpenAI client, and defines the four-step workflow (`step1` – `step4`) that analyzes factor images, proposes code, retries on failure, and surfaces output paths.  
- `factor_test/` – Folder of per-factor notebooks (e.g., `factor_109.ipynb`) that compute factor values, export monthly results, merge them with daily bars, neutralize, rank into deciles, and backtest both long-only and long-short implementations.  
- `factor_combine_rank.ipynb` – Merges multiple factor rank files with the daily price table to build a consolidated `rank_combine.feather` dataset for multi-factor selection.  

---

## Prerequisites

Install the Python stack used across the notebooks:

- **Workflow & helpers**: `openai`, `streamlit`, `python-dotenv`, `pathlib`, `time`, `base64`, `os`  
- **Data & analytics**: `pandas`, `numpy`, `statsmodels`, `matplotlib`  

---

## Configuration

1. Create a `.env` file in the repository root with your API token, e.g.:

   ```ini
   API_KEY=your_openai_like_service_key

The notebooks read this file and build the `OpenAI` client with the configured `base_url` and key.

2. Review and adjust the Windows-style data paths (e.g., `F:\QuantData...`) to match your local storage before running the notebooks.

---

## Data Expectations

- Minute-level inputs live under `F:\QuantData\minute_data` and are named like  
  `YYYYMM_oneminute.feather` for batch processing.

- Generated factor outputs are saved to  
  `F:\QuantData\factor_result` (per factor) and  
  `F:\QuantData\factor_result_allmonth<factor_name>` (aggregated) for later merging.

- Daily price data is expected at  
  `F:\QuantData\AShareEODPrices_allmonth.feather`  
  for alignment, neutralization, and ranking.

- Quantile ranks for each factor are written to  
  `F:\QuantData\factor_rank\rank_<id>.feather`,  
  and combined ranks land in  
  `F:\QuantData\factor_rank\rank_combine.feather`.


---

## Typical Workflow

1. **Describe a factor**:  
   Supply an explanatory image or prompt to `API_factor_process.ipynb`;  
   `step1` and `step2` analyze the factor and draft code, while `step3` executes and auto-regenerates until it runs successfully.  
   Use `step4` to surface the output file path.

2. **Materialize factor values**:  
   Execute the relevant notebook in `factor_test/` to batch-process monthly minute files via `generate_factor_files`, producing per-factor feather outputs.

3. **Integrate and evaluate**:  
   Merge factor results with daily bars, neutralize by log market value, bucket into deciles, and backtest the spread between top and bottom groups.

4. **Build multi-factor ranks**:  
   After individual factors are ranked, run `factor_combine_rank.ipynb` to join them with the master price table and export the consolidated ranking dataset.

---

## Tips

- Keep an eye on API usage: the notebooks expose commented alternatives for different OpenAI-compatible endpoints if you need to switch providers.
- Regeneration logic persists new code and error logs to disk (`factor_pycode`, `factor_code_string`), making it easier to inspect failed iterations and rerun only the necessary pieces.
- The backtest cells plot cumulative net values for each decile plus a long-short series; rerun them after adjusting factors or rebalance periods to visualize performance shifts.
