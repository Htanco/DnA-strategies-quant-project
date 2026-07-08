# D+A Strategies — Quant Analyst Task Pack

Quantitative research and trading-strategy exercises completed as part of the D+A Strategies Analyst case pack. Each task is a self-contained piece of macro/markets research: a question is posed, data is assembled, a simple transparent method is applied, and the evidence is interpreted against what it implies for positioning.

## Working principles (shared across all tasks)

From the original D+A Strategies brief, every analysis in this repo follows the same four-step discipline:

1. **Pose a precise question** and articulate the economic mechanism expected to produce the result.
2. **Assemble a small set of relevant series**, brought to a common frequency and direction, with sources and transformations documented.
3. **Apply a simple, transparent procedure** that is adequate for the question and robust to minor changes in specification.
4. **Interpret the evidence in context** — state what would count against the conclusion and how the findings might inform positioning or risk controls.

## Repository structure

```
DnA-strategies-quant-project/
├── README.md
├── requirements.txt
├── .env.example              # template for the FRED API key — copy to .env locally
├── .gitignore
├── task-1-macro-surprises/
│   └── macro_surprises_analysis.ipynb
├── task-2-sp500-weekly-returns/
│   └── sp500_weekly_returns.ipynb
└── task-3-copper-china-pulse/
    ├── copper_china_pulse_analysis.ipynb
    ├── R2-Copper-and-China-writeup.docx
    ├── copper_vs_pulse.png
    ├── pulse_components.png
    ├── data_raw/              # copper.csv, exports.csv, pmi.csv, bci.csv
    └── data_clean/            # df_clean.csv (merged + engineered dataset)
```

## Setup

1. **Clone the repo** (see push/publish instructions at the bottom of this file) and `cd` into it.
2. **Create a virtual environment and install dependencies:**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```
3. **Set up your FRED API key** (needed for Task 1 and Task 3, which both pull data from the [FRED API](https://fred.stlouisfed.org/docs/api/api_key.html)):
   ```bash
   cp .env.example .env
   # then edit .env and paste in your own key:
   # FRED_API_KEY=your_fred_api_key_here
   ```
   Both notebooks load this key via `python-dotenv` (`load_dotenv()` + `os.environ["FRED_API_KEY"]`) instead of hardcoding it, and `.env` is excluded from git via `.gitignore`. See the **Security note** below — this repo previously had a real key hardcoded in plaintext.
4. **Launch Jupyter** and run the notebook for whichever task you want to inspect:
   ```bash
   jupyter notebook
   ```

---

## Task 1 — Macro Surprises and the Transmission to Markets

*Folder: [`task-1-macro-surprises/`](task-1-macro-surprises)*

**Question.** Scheduled data releases (specifically US CPI surprises) shift beliefs about the macro outlook and policy stance. How does a CPI surprise transmit, same-day, into rates, equities, and FX?

**Method.** Built an event-day table of monthly US CPI releases (2021–2024) with the surprise (actual MoM % − survey median %) alongside same-day/close-to-close changes in the 2-year Treasury yield, 10-year Treasury yield, S&P 500, and the trade-weighted dollar (DXY). Grouped reactions by surprise direction (positive/negative/zero) and compared average/median moves against the sign predictions implied by the policy-expectations, discount-rate, and interest-rate-differential channels.

**Data sources.** FRED API — `CPIAUCSL` (CPI), `DGS2` (2Y yield), `DGS10` (10Y yield), `SP500`, `DTWEXBGS` (trade-weighted dollar index). CPI surprise (actual vs. survey median) values are sourced from BLS actuals / Bloomberg consensus and entered directly in the notebook.

**Key findings.**
- The 2Y yield and S&P 500 moved in the theoretically predicted direction (policy-expectations and discount-rate channels held up).
- The 10Y yield showed a near-zero/slightly negative reaction to hot prints — consistent with the market pricing the Fed as close to a terminal rate, which compresses long-end sensitivity to incremental surprises.
- DXY moved opposite to the textbook prediction, most likely because the sample (2022–2023) caught the dollar already near multi-decade highs, and because the negative-surprise group has very few observations (n=6), making that grouping noisy.
- Full sign-prediction table and discussion are in the notebook's markdown interpretation cell.

**Files.** `macro_surprises_analysis.ipynb`. (The original task brief is not included — it's confidential under an NDA with D+A Strategies.)

---

## Task 2 — S&P 500 Weekly Returns Visualization

*Folder: [`task-2-sp500-weekly-returns/`](task-2-sp500-weekly-returns)*

**Question.** A foundational data-handling exercise: pull ~20 years of daily S&P 500 price data, resample it to weekly returns, and visualize the resulting time series.

**Method.** Used `yfinance` to download daily `^GSPC` prices from `today − 20 years` to present, saved to CSV, resampled the close price to weekly (Friday-anchored) frequency, computed percentage returns, and plotted the resulting weekly return series with `matplotlib`.

**Data source.** Yahoo Finance via `yfinance` (`^GSPC` ticker).

**Files.** `sp500_weekly_returns.ipynb`. (The original task brief is not included — it's confidential under an NDA with D+A Strategies.)

> **Note:** the task brief asks for the downloaded daily-price CSV (`SP500_daily.csv`) to be included alongside the notebook, but that file wasn't present in the original submission — only the notebook that generates it. Re-running the notebook will regenerate it locally (`yfinance` pulls live data, so exact historical values will refresh to the current date).

---

## Task 3 — R2: Copper and China's Industrial Pulse

*Folder: [`task-3-copper-china-pulse/`](task-3-copper-china-pulse)*

**Question.** Copper is often treated as a barometer of the industrial cycle. Do movements in copper prices tend to *precede* shifts in China's industrial momentum, and is that lead reliable enough to inform sector/factor positioning?

**Method.** Constructed a monthly **Industrial Pulse** as an equal-weighted average of three standardized (z-scored) series: China NBS Manufacturing PMI, China export growth (MoM, seasonally adjusted), and the OECD China Manufacturing Business Confidence Index. Expressed copper as a 3-month rolling return and plotted it against the Pulse to visualize turning points. Defined "copper rise episodes" as the 3-month return crossing above +5% (with a +3% threshold used as a robustness check) and measured the share of episodes followed by a Pulse rise within 1, 2, and 3 months.

**Data sources.**
- Copper price (`PCOPPUSDM`) and China export growth (`XTEXVA01CNM657S`) — pulled live from the FRED API.
- China NBS Manufacturing PMI — entered directly in the notebook from public NBS releases (Jan 2015–Dec 2024).
- OECD China Manufacturing Business Confidence Index (`bci.csv`) — pre-sourced and included as a static file in `data_raw/`.
- Sample window: January 2015 – December 2024 (120 monthly observations).

**Key findings.**
- Hit rate (share of copper-rise episodes followed by a Pulse rise) rises with horizon: **36%** within 1 month, **55%** within 2 months, **64%** within 3 months (11 episodes at the +5% threshold). Robust in direction at a +3% threshold (50% / 67% / 75%, 12 episodes).
- The relationship is a moderate 2–3 month lead at best, not a tight or mechanical one — it fails in roughly 1 in 3 episodes.
- Breakdowns are concentrated in identifiable regimes: 2015–16 (dollar strength depressed copper independent of China demand) and the COVID crash of early 2020 (simultaneous exogenous collapse in both series, no leading indicator held).
- Positioning implication: where the signal holds (e.g., 2020–21 restocking), early copper strength favors cyclical exposures (materials, industrials, energy) and value/small-cap tilts within them — but confidence should be conditioned on dollar direction and the absence of exogenous shocks.

**Files.** `copper_china_pulse_analysis.ipynb`, `R2-Copper-and-China-writeup.docx` (formatted write-up), `copper_vs_pulse.png` and `pulse_components.png` (saved chart outputs), `data_raw/` (raw pulled/sourced series), `data_clean/df_clean.csv` (merged, z-scored, Pulse-engineered dataset). (The original task brief is not included — it's confidential under an NDA with D+A Strategies.)

---

## Security note

The original notebooks for Task 1 and Task 3 had a real FRED API key **hardcoded in plaintext** (`Fred(api_key="...")` / `FRED_KEY = "..."`). Both have been updated to load the key from an environment variable via `python-dotenv` instead, and `.env` is listed in `.gitignore` so no secret is ever committed. `.env.example` documents the variable name without a real value.

**Before publishing this repo, rotate/regenerate that FRED API key** at https://fred.stlouisfed.org/docs/api/api_key.html — since it previously sat in plaintext in local files, treat it as compromised even though it was never pushed to GitHub, and swap in the new key locally in your own `.env`.

---

## Publishing this repo to GitHub

These steps are for you to run yourself from a terminal — nothing here has been pushed anywhere.

1. **Create the GitHub repo** (matches the setup already shown in your "Create a new repository" screen):
   - Owner: `Htanco`, name: `DnA-strategies-quant-project`, visibility: **Public**.
   - Leave **README**, **.gitignore**, and **license** all off/none — this repo already has its own `README.md` and `.gitignore`, and initializing any of those on GitHub would create a conflicting first commit that you'd have to merge with your local history. Click **Create repository** once it looks like your screenshot.

2. **From your terminal**, `cd` into the folder that was prepared for you:
   ```bash
   cd "/Users/heggo/Downloads/DnA-strategies-quant-project"
   ```

3. **Initialize git, review what will be committed, and confirm no secrets are staged:**
   ```bash
   git init
   git add .
   git status
   ```
   Check the `git status` output — you should **not** see `.env` listed (only `.env.example`). If you do see `.env`, stop and fix your `.gitignore` before committing.

4. **Commit and push:**
   ```bash
   git commit -m "Initial commit: D+A Strategies analyst tasks 1-3"
   git branch -M main
   git remote add origin https://github.com/Htanco/DnA-strategies-quant-project.git
   git push -u origin main
   ```
   (If you use SSH instead of HTTPS for GitHub auth, use `git@github.com:Htanco/DnA-strategies-quant-project.git` instead.)

5. **Verify on GitHub** that `.env` is not present in the pushed repo, and that the description field carries over correctly.
