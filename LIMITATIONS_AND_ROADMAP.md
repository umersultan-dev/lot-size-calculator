# Limitations, Unfinished Areas and Roadmap

**Read this first if you come back to the project after months.** It is written to take 2 to 5 minutes: what is incomplete, what to be careful about, what was decided on purpose, and what to build next, in priority order.

* **Status when this was written:** V2.1 (`lot\_size\_calculator\_11.html`), November 2026.
* **Overall state:** feature-complete for its original goal. The calculation core is solid and heavily tested. The open items below are about data freshness, offline use, and polish.
* **Companion file:** [README.md](README.md) explains what the calculator is and how to use it.

\---

## 1\. At a glance

|#|Area|Status|Impact|Effort to fix|
|-|-|-|-|-|
|1|Exchange rates are **daily ECB, not real-time**|By design, limitation|Medium on volatile days|Medium (needs a keyed provider)|
|2|Rate service addresses **never tested on a real network**|Unverified|High if it fails (falls back to saved/manual)|Very low (10-minute check)|
|3|**Styling and icons need the internet**|Known gap|Page looks broken if first opened offline|Medium|
|4|Instrument specs are **broker-dependent**|Inherent|Wrong lots if the spec does not match your broker|None (user must verify)|
|5|Saved data lives in **one browser only**|Limitation|Data loss if browser data is cleared|Low (backup exists)|
|6|Add form can **overwrite** an existing symbol|Known risk|Accidental overwrite|Low|
|7|Not modelled: spread, commission, swap, slippage, margin|Out of scope|Real risk slightly differs|Medium to high|
|8|Accessibility and mobile layout **not audited**|Gap|Some users blocked|Medium|
|9|Single 2,650-line file, **no tests in the repo**|Maintenance debt|Slower, riskier changes|Medium|

\---

## 2\. Exchange rates (the most important limitation)

**What it does today.** When the instrument's quote currency differs from your account currency (for example USDJPY on a USD account), the calculator converts the tick value using the **latest daily ECB reference rate** from the free Frankfurter service. The ECB publishes **once per business day (around 16:00 CET), not on weekends or ECB holidays**. So the rate can be hours or days old and will not reflect a sudden move.

**How big is the error?** It is 1 to 1: a rate that is 1% off makes the real risk 1% off. On a normal day that is small, usually smaller than the error from rounding a lot to 0.01. On a volatile day (news, intervention) it can be several percent. **Mitigation built in:** the rate box is editable. On volatile days type your broker's or platform's own rate. It is flagged "Manual Rate" and is never overwritten automatically.

**Why there is no live rate.** A real-time or near-real-time source needs a provider with an API key. That was deliberately postponed (a keyed provider adds signup, key handling, rate limits and terms).

**Fallback chain implemented** (important to understand before changing it):

1. Fresh fetch, whole table in one request (every supported currency per 1 USD), saved with its ECB date and fetch time.
2. If the service is unreachable: the **last saved table**, with its age shown (under 24 h amber, 1 to 7 days orange, over 7 days red "VERY OLD").
3. If nothing has ever been saved: **no rate**. The box stays empty, all results show a dash, and a red banner asks the user to enter a rate. This was a deliberate safety decision: the calculator must never show a believable but invented number to someone who may trade on it.

Older versions used a built-in rough rate table. It was removed on purpose. Do not bring it back.

**Roadmap: optional "live rate boost".** Add an optional keyed provider *in front of* the ECB chain (live provider, then daily ECB, then saved table, then manual). Notes from the research done in November 2026 (vendor terms change, so re-verify before building):

* Keyless free sources (ExchangeRate-API open endpoint and similar) are also daily, so they only help as extra redundancy.
* Open Exchange Rates free plan: hourly updates, API key, roughly 1,000 requests per month (tight if refreshing often).
* Twelve Data free plan: API key, about 8 requests per minute and 800 per day, with an exchange-rate endpoint described as real-time. **Not verified:** how delayed the free tier really is, whether the browser may call it directly (CORS), and the terms of use.
* Smaller providers advertising free 60-second rates are marketing claims with inconsistent limits; do not trust without testing.
* **Do not scrape** Google or Yahoo: no official API, blocked by browsers, breaks without warning.
* **Key handling:** let the user paste the key into a settings box stored in the browser. Never write it into the HTML file or the GitHub repository. Anyone with access to that browser can read it.
* Fetch only when a cross-currency pair is active, cache for a few minutes, and keep the ECB chain as the automatic backup, because free plans change.
* A small throwaway diagnostic page that tests a provider from the user's own browser (CORS, delay, limits) is the safest first step.

\---

## 3\. Never verified on a real network (do this first)

All development testing ran in a sandbox **with no internet**, using a simulated rate server. The behaviour of the real service was never observed. Before trusting the rate features, open the calculator online and check:

1. Pick **USDJPY** on a USD account. The badge should read **ECB DAILY RATE** with a real date and a rate near 0.0064 to 0.0066.
2. The code first requests `https://api.frankfurter.dev/v1/latest?base=USD\&symbols=EUR,GBP,JPY,CAD,AUD` and then `https://api.frankfurter.app/latest?from=USD\&to=EUR,GBP,JPY,CAD,AUD`. The **"several currencies in one request" format is assumed from the service's documentation, not tested.** If the service rejects it, the code falls back to a single-pair request (also untested live). If every rate request fails, the calculator drops to saved or manual rates and the badge shows it.
3. Confirm the header chip turns **Offline** when Wi-Fi is switched off and returns to **Online** afterwards.

If any of this fails, the fix is almost certainly a URL or parameter in section 6 of the code (`fetchRateTable`, `fetchRateFromService`, `pingRateService`).

Also unconfirmed: the **visual styling** was never checked by the automated tests, because the styling library could not load in the sandbox. Open it on a desktop and on a phone and look at every card, dialog and the Asset Manager.

\---

## 4\. Offline use and external dependencies

The calculation logic is all inside the file and works offline. But three things load from the internet:

|Dependency|Used for|
|-|-|
|Tailwind CSS (Play CDN script)|All styling|
|Font Awesome 6.4 (cdnjs)|Icons|
|Google Fonts (Inter)|Typeface|

**Effect:** if the file is opened (or reloaded) **with no connection**, the page appears unstyled and icons are missing. If the connection drops *after* the page loaded, nothing changes. A slow or blocked CDN causes the same problem.

Also note that the Tailwind Play CDN is documented as not intended for production, and the CDN scripts have no integrity (SRI) pinning.

**Roadmap options (pick one):**

* **A. Inline everything (works on `file://` too).** Compile only the Tailwind classes actually used (roughly 250 to 300) into a `<style>` block, replace the roughly two dozen Font Awesome icons with inline SVG, and fall back to the system font. Best done with the **real Tailwind compiler through npm** (identical result). Hand-writing the CSS is possible but must be compared visually. Any new class added later must be regenerated. Font Awesome Free SVGs need an attribution comment (CC BY 4.0).
* **B. Service worker (hosted on GitHub Pages only).** A service worker can cache the CDN files after the first visit. Needs HTTPS hosting; does not work for a file opened from disk.
* **C. Self-host the files** next to the HTML. Simple, but breaks the "one file" idea.

\---

## 5\. What the calculator does **not** model

* **Spread, commission, swap/overnight fees and slippage.** The stop loss is treated as an exact exit price. Real losses are usually slightly larger.
* **Margin and leverage limits.** The "leverage" readout is simply notional exposure divided by balance. It does not check whether your broker would allow the trade.
* **Lot step, minimum and maximum lot rules.** Only simple rounded-down suggestions are shown. They do not know your broker's actual limits.
* **Inverse or exotic contracts** (for example coin-margined futures) and instruments whose profit is not simply `tick size x contract size` in the quote currency.
* **Conversion at your broker's rate.** It uses the ECB cross rate, which can differ from your broker's.
* **Only six currencies** are supported for quote and account currency: USD, EUR, GBP, JPY, CAD, AUD (the code list is `VALID\_QUOTES`). Adding more means extending that list and confirming the rate service supports them.
* **Single target.** One take profit only, with no partial exits or scaling.

\---

## 6\. Instrument specifications and sample prices

* The 15 built-in specifications were audited (each checked against standard contract definitions and against the risk formula). **They are standard values, not your broker's.** Index and crypto contracts in particular vary. If a symbol on your platform has a different contract size or tick size, edit it in the Asset Manager.
* The sample **entry and stop loss prices** shown when selecting an instrument (for example BTC 95,000) are illustrative and become stale. Updating them was **considered and deliberately declined**: the calculator needs the user's own prices, and any "refresh" would be a guess. Do not spend time here unless a live price feed is added.

\---

## 7\. Data storage and portability

* Data is saved in the browser's `localStorage`. It belongs to **one browser, on one device**.
* A file opened from disk (`file://`) and the same file served from GitHub Pages (`https://`) are **different storage areas**. Whether differently named or located copies of the file on disk share storage depends on the browser and was **never confirmed**.
* Clearing browser data erases everything. **Mitigation built in:** Export Backup and Import Backup (the backup contains instruments and settings, not rates).
* No cloud sync, accounts or multi-device support.

**Roadmap ideas:** a periodic "you have not exported a backup in a while" reminder; optional cloud sync only if a backend is ever introduced.

\---

## 8\. Behaviours that may surprise you (known and accepted for now)

* **Adding a symbol that already exists replaces it.** In the Add form, typing an existing symbol such as `XAUUSD` and saving overwrites that instrument. The message says "updated", but there is no warning first. *Recommended fix: ask for confirmation, or block it and point to the Edit button.*
* **Saving an instrument switches the calculator to it.** This is existing behaviour. Your typed prices are kept only if that instrument was already selected.
* **Factory reset only resets instrument specifications.** Balance, risk settings and trade inputs are untouched.
* **Category tabs are fixed to seven names.** A custom category such as "Agriculture" appears only under ALL and Custom, not in its own tab.
* **Markup characters** (`< > " ' backtick backslash`) are stripped from asset names, categories and symbols when saved data or a backup is loaded, so those characters will not survive a reload.
* **At least one instrument must remain:** you cannot delete the last one.
* **The reward at the target assumes the exact unrounded lot**, so it is slightly optimistic if your broker forces you to round down.

\---

## 9\. User interface, accessibility and testing gaps

* **Accessibility was not audited.** Confirmation dialogs do not trap keyboard focus, list rows in multi-select mode are mouse/touch only, and nothing was tested with a screen reader.
* **Mobile:** the layout was designed responsively and structure-tested at phone width, but real-device visual review is still owed (see section 3).
* **Browser coverage:** tested in Chromium only. Firefox and Safari are unverified, especially backup download/upload and `localStorage` on `file://`.
* **Tests:** during development about 300 automated browser checks were run (a real headless Chromium driven by Playwright, with a simulated rate server and fake clock). **Those test scripts are not part of this repository.** If you continue development, recreate a regression suite first. The areas to cover are: lot-size math per instrument, R:R engine, saving and loading (including corrupt/tampered data), rate fallback tiers and the "no rate" blocked state, connectivity chip, Asset Manager (search, tabs, add/edit/delete, multi-delete), backup round trip with malicious files, and factory reset.

\---

## 10\. Code and maintenance

* The whole project is **one HTML file of about 2,650 lines (130 KB)**. That was a deliberate choice (no build step, no IDE needed) and it is still workable, but it is near the point where splitting makes sense.
* **Suggested split** when you next make a big change: `index.html` (markup), `styles.css`, and `app.js` (or several modules: data, storage, fx, engine, asset-manager). The code is already organised in numbered sections that map onto this.
* Several UI parts are built by string templates with `innerHTML`. User-provided text is escaped (`escapeHtml`) and loaded data is sanitised (`sanitizeAssets`). Keep doing both when adding features.
* There is no linter, formatter or CI.

\---

## 11\. Security and privacy notes

* There is **no backend and no account**. None of your balance, risk, prices or instruments is ever sent anywhere. The page only requests its styling/icon/font files from the CDNs and currency rates from the Frankfurter service (which receives only currency codes, plus your IP address as any web request does).
* Imported backup files are validated (type, version, size limit, field checks) and special keys such as `\_\_proto\_\_` are ignored.
* Third-party scripts load without integrity hashes (see section 4).
* If a live-rate provider with a key is added, the key sits in browser storage and is readable by anyone using that browser. Never commit it.

\---

## 12\. Prioritised roadmap

**Do soon (small, high value)**

1. **Verify the real rate service from a normal browser** (section 3). About 10 minutes.
2. **Look at the real visuals** on desktop and phone and fix anything off.
3. **Confirm before overwriting an existing symbol** (section 8).
4. **Choose a license** and add a screenshot to the README.

**Next (medium)**

5. **Offline styling** (section 4, option A or B).
6. **Add the regression test suite to the repository** (section 9).
7. **Accessibility pass:** focus trapping in dialogs, keyboard selection in the list, labels, contrast check.

**Later (optional, bigger)**

8. **Live-rate boost** with a keyed provider (section 2). Test it with a throwaway page first.
9. **More currencies**, plus an editable list of quote currencies.
10. **Cost-aware sizing:** optional spread and commission fields, a broker lot-step setting, margin check.
11. **Multiple take-profit levels** and partial exits.
12. **Split into several files**, and add a build or lint step.
13. **Backup reminder**, and a light theme if wanted.

\---

## 13\. Decisions already made (do not re-open without a reason)

|Decision|Reason|
|-|-|
|Keep the formula and the overall UI structure unchanged|The core was correct; changes were limited to bugs, features and honesty|
|Keep the ECB daily source for now, with honest labels|Free, no key; real-time was judged not urgent|
|Never show an invented exchange rate; leave results blank until the user enters one|A believable wrong number is more dangerous than a blank|
|Do not refresh sample entry/stop prices|The user always enters their own|
|Reset restores instrument specs only|It was the explicit scope ("Reset Specs")|
|The live-rate boost and offline styling come last|Not urgent|
|Single file, no build step|Simple to run, share and host|

\---

## 14\. Before publishing checklist

* \[ ] Rename `lot\_size\_calculator\_11.html` to `index.html` (for GitHub Pages).
* \[ ] Add a `LICENSE` file.
* \[ ] Open the live page once online and run the checks in section 3.
* \[ ] Look at the page on a phone.
* \[ ] Add a screenshot to the README.
* \[ ] Keep the README and this file in the repository root.

\---

## Appendix: how this was built

The project began as a single-file calculator generated with another AI assistant, then was audited and extended step by step with Claude (Anthropic). Each step produced a numbered file, was tested in a real headless Chromium browser (with a simulated rate server and fake clock, since the sandbox had no internet), and was only followed by the next step after the owner confirmed it. The development history is summarised in the version table in the README.

