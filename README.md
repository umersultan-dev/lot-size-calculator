# Lot Size Calculator

A clean, single-file **position-size (lot size) calculator** for traders. You enter your account balance, how much you are willing to risk, and your entry and stop loss. It tells you the exact lot size, the dollar risk, the take-profit level for your chosen risk-to-reward ratio, and the money you stand to make at that target.

It works for **Forex, metals, energy, crypto and stock indices**, remembers your settings between visits, and lets you add your own instruments.

> \*\*Current version: V2.1\*\* (file `lot\_size\_calculator\_11.html`)
> Read \*\*\[LIMITATIONS\_AND\_ROADMAP.md](LIMITATIONS\_AND\_ROADMAP.md)\*\* before relying on it: it explains what the calculator does \*not\* do and what to improve next.

\---

## 1\. Quick start

**Use it locally (no install, no server):**

1. Download `lot\_size\_calculator\_11.html`.
2. Double-click it. It opens in your browser.

**Put it on the web with GitHub Pages:**

1. Create a repository and upload the HTML file. Rename it to `index.html`.
2. Repository **Settings -> Pages -> Deploy from a branch -> `main` / root**.
3. Your calculator is live at `https://<your-name>.github.io/<repo>/`.

An internet connection is needed for the page styling (loaded from CDNs) and to fetch the daily currency rate. See the limitations file for details.

\---

## 2\. How to use it

The page has numbered cards on the left and results on the right.

1. **Account** (header and first card): choose your account currency (USD, EUR, GBP, JPY, CAD or AUD) and enter your balance.
2. **Instrument**: pick a category and a symbol (for example Forex -> EURUSD).
3. **Risk**: choose **Risk %** (for example 1%) or **Risk $** (a fixed money amount).
4. **Entry and Stop Loss**: enter your entry price, then either a **Stop Loss price** (**Prices** mode) or a **Stop Loss distance** (**Points / Pips** mode). In Points/Pips mode, choose **Buy** or **Sell**.
5. **Reward Target**: choose **R:R Ratio** (any decimal, such as 1.278) or **TP Price**. The calculator shows the take-profit price, its distance, the resulting risk:reward, and the expected reward in money.
6. **Currency conversion** (appears only when needed, see below): check the rate. You can type your own broker's rate at any time.
7. **Read the results** on the right: recommended lot size, dollar risk, notional exposure, point values, rounded-down lot suggestions for common broker steps, and the target figures.

### Worked examples

**EURUSD** (USD account, balance $10,000, risk 1% = $100, entry 1.0850, stop loss 1.0800 = 50 pips)

* One lot is 100,000 EUR, so one pip is worth $10 per lot.
* Lot size = 100 / (50 x 10) = **0.20 lots**
* With R:R 1:2, the take profit is **1.0950** and the expected reward is **$200**.

**USDJPY** (same account, entry 155.00, stop loss 154.20 = 80 points)

* One point (0.01) on one lot is 1,000 JPY. At about 155 JPY per USD that is roughly $6.45.
* Risk per lot = 80 x $6.45161 = about $516.13, so lot size = 100 / 516.13 = **0.19375 lots**.
* The rate is needed here because the instrument is quoted in JPY while the account is in USD.

\---

## 3\. The math

```
Lot Size = Risk Amount / ( Stop Loss Distance (points) x Tick Value per 1.0 Lot )

Tick Value per 1.0 Lot = Tick Size x Contract Size   (in the instrument's quote currency)
                         x conversion rate           (only when quote currency != account currency)
```

* The lot size is shown **unrounded**. A separate panel suggests values rounded **down** to common broker steps, so you never exceed your intended risk.
* The expected reward at the target assumes you open the exact unrounded lot.
* Direction (Buy/Sell) is inferred automatically in Prices mode: stop loss below entry means Buy, above means Sell.

\---

## 4\. Built-in instruments (15)

Specifications are per **1.0 lot**. Always confirm contract sizes with your broker, because index and crypto contracts differ between brokers.

|Symbol|Category|Contract size|Tick size|Quote|
|-|-|-|-|-|
|EURUSD, GBPUSD, AUDUSD|Forex|100,000|0.0001|USD|
|USDCAD|Forex|100,000|0.0001|CAD|
|USDJPY, EURJPY|Forex|100,000|0.01|JPY|
|XAUUSD (Gold)|Metals|100 oz|0.01|USD|
|XAGUSD (Silver)|Metals|5,000 oz|0.01|USD|
|WTI, BRENT|Energy|1,000 barrels|0.01|USD|
|BTCUSD, ETHUSD|Crypto|1 coin|0.01|USD|
|US30|Indices|1|1.0|USD|
|US500, US100|Indices|1|0.1|USD|

For crypto, 1.0 lot is 1 coin, so a $100 price move equals $100 profit or loss.

The sample entry and stop loss prices that appear when you pick an instrument are **placeholders only**. Always enter your own.

\---

## 5\. The Asset Manager

Open it with the **Asset Manager** button in the header.

* **Browse and search**: a collapsible list with live search (try `gold`, `eur usd`, `btc`) and category tabs: ALL, Forex, Metals, Energy, Crypto, Indices, Custom.
* **Add** your own instrument (symbol, name, category, quote currency, contract size, tick size). **Edit** any instrument with its pencil button.
* **Delete** one instrument, or press **Select** to tick several and delete them together. At least one instrument always remains.
* **Backup**: **Export Backup** saves your instruments and settings to a `.json` file. **Import Backup** restores them after a confirmation step.
* **Reset to Factory Defaults** (Danger Zone) restores the 15 built-in instruments. It does not touch your balance or risk settings.
* The window stays open while you work. Close it with the **X**, the **Escape** key, or by clicking outside it. If you have typed unsaved details, it asks before discarding them.

\---

## 6\. Exchange rates and the status chip

Cross-currency conversion uses the **latest daily ECB reference rate**, fetched from the free Frankfurter service. It is **not a real-time market price**. On volatile days, type your broker's own rate into the box.

* The header chip shows **Online / Offline / Checking** and, when a rate is in use, how old it is.
* Every successful fetch saves a full rate table, so any currency pair keeps working offline using the last real ECB rate, with its age shown and colour-coded (amber, orange, red).
* If no real rate has ever been saved (for example first use while offline), the rate box stays **empty**, the results stay blank, and a red banner asks you to enter the rate. The calculator never guesses a rate.

\---

## 7\. Saving your data

Everything is stored in your browser's `localStorage`:

|Key|Contents|
|-|-|
|`lotcalc\_state\_v1`|Your instruments and settings|
|`lotcalc\_fx\_v2`|Last saved ECB rate table (with date and fetch time)|
|`lotcalc\_fx\_v1`|Older single-pair rates (still read as a backup path)|
|`univ\_calc\_assets`|Pre-V1.4 assets (migrated automatically)|

Storage belongs to one browser on one device (and possibly one file location). Use **Export Backup** to keep a copy or move to another browser.

\---

## 8\. Built with

|Layer|Technology|
|-|-|
|Structure|HTML5, one file|
|Styling|CSS3 through **Tailwind CSS** (Play CDN script) with a custom navy theme|
|Icons|**Font Awesome 6.4** (CDN)|
|Font|**Inter** (Google Fonts)|
|Logic|**Vanilla JavaScript** (no framework, no build step)|
|Rate data|**Frankfurter API** (ECB reference rates): `api.frankfurter.dev`, with `api.frankfurter.app` as backup|

Browser features used: `fetch`, `AbortController`, `localStorage`, `Blob` / `FileReader` (backup), ES2017 async/await. It was developed and tested in Chromium. Other modern browsers should work but are not verified.

### Where things are in the code

The single file is organised into numbered sections (search for the comment headers):

1. Preset instrument database
2. App state
3. DOM references
4. Persistent storage + initialisation
5. Dropdowns and readouts
6. Live currency conversion (ECB table, saved rates, fallbacks) and **6B** connectivity indicator
7. Core calculation engine and **7B** reward-target (R:R) helpers
8. Rendering results
9. Event listeners
10. Asset Manager (search, tabs, edit, delete, multi-select, backup, factory reset) and **10B** mode appliers
11. Helper utilities

The key formula lives in section 7 (`calculatePosition`).

\---

## 9\. Version history

Each numbered file bumps the version: file number N becomes V(1 + N/10).(N mod 10). Only V1.8 and later show the label in the header (earlier builds still said V1.0).

|File|Version|What changed|
|-|-|-|
|`\_1`|V1.0|Original single-file calculator with the preset database, custom asset modal, live FX conversion|
|`\_2`|V1.2|Fixed duplicate results column and unsafe offline rate fallback|
|`\_3`|V1.3|Risk:reward engine (ratio or TP price, any decimal)|
|`\_4`|V1.4|Saved settings and instruments, safe loading|
|`\_5`|V1.5|Asset Manager search and category tabs|
|`\_6`|V1.6|Safe factory reset with confirmation|
|`\_7`|V1.7|Honest ECB labelling, saved last rate, connectivity chip|
|`\_8`|V1.8|Whole-table rate saving, no invented rates, forced manual entry when no rate exists|
|`\_9`|V1.9|Spec audit, one-line risk banner, collapsible instrument browser|
|`\_10`|V2.0|Styled delete confirmation, backup export/import|
|`\_11`|V2.1|Pinned close button, click-outside and Escape to close, stays open after edits, multi-select delete|

\---

## 10\. Disclaimer

This tool performs theoretical calculations. It does not guarantee live market results, it is not financial advice, and it cannot know your broker's exact contract specifications, spreads, commissions or slippage. **Always cross-check against your trading platform before risking capital.**

## 11\. License

Not yet chosen. Add a `LICENSE` file before sharing publicly (MIT is a common simple choice).

