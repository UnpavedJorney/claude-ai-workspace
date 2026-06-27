# PRD: Personal Portfolio Manager

**Version:** 1.2 | **Date:** 2026-06-26 | **Author:** Simran Modi

---

## 1. Overview

A personal portfolio management tool — starting as a Google Sheet, then evolving into a web application — to track, manage, and analyze holdings across **Stocks, Futures & Options (F&O), Mutual Funds (MF), and ETFs** on Indian exchanges.

---

## 2. User Persona

- Individual retail investor managing a personal portfolio
- Trades across multiple asset classes (equity, derivatives, MF, ETF)
- Needs consolidated view of holdings, P&L, and portfolio health
- Indian market context (NSE/BSE)

---

## 3. One-Time Setup (Initial Configuration)

A setup screen that runs on first launch (and is editable later from Settings). All trading, brokerage, and currency defaults flow from this configuration.

### 3.1 Market / Exchange Selection

| Field | Description |
|---|---|
| Market — Exchange | Dropdown with selectable market-exchange pairs |
| Default (v1) | **India : NSE/BSE** — single selectable option |
| Future additions | USA : NYSE/NASDAQ, UK : LSE, etc. |

- v1: Only one option available — **India : NSE/BSE** (pre-selected, read-only feel but shown as dropdown for future extensibility)
- Later: multi-select to enable trading across multiple markets simultaneously
- Exchange selection drives: instrument master list, lot sizes, trading hours, holiday calendar

### 3.2 Currency Selection

| Field | Description |
|---|---|
| Base Currency | Dropdown — currency tied to the selected market |
| Default (v1) | **INR (₹)** — single selectable option |
| Future additions | USD ($), GBP (£), EUR (€), etc. |

- v1: Only **INR** available, auto-linked to India market
- Later: when multiple markets are enabled, each market maps to its default currency (USA → USD, etc.)
- All portfolio values, P&L, and reports display in the base currency
- Future: currency conversion for cross-market portfolio consolidation

### 3.3 Brokerage Setup

A table-based configuration where the user enters brokerage charges for each product type. These values are auto-applied to every transaction.

#### Brokerage Configuration Table

| Product | Charge Type | Buy Brokerage | Sell Brokerage | Notes |
|---|---|---|---|---|
| **Stocks (Delivery)** | Fixed (₹) _or_ Variable (%) | e.g., ₹20 or 0.03% | e.g., ₹20 or 0.03% | Both sides charged |
| **Stocks (Intraday)** | Fixed (₹) _or_ Variable (%) | e.g., ₹20 or 0.03% | ₹0 / 0% | One-sided brokerage — broker charges only on one leg |
| **Options** | Fixed (₹) _or_ Variable (%) | e.g., ₹20 flat | e.g., ₹20 flat | Per-lot or per-order as per broker |
| **Futures** | Fixed (₹) _or_ Variable (%) | e.g., ₹20 or 0.01% | e.g., ₹20 or 0.01% | Both sides charged |
| **Mutual Funds** | Fixed (₹) _or_ Variable (%) | ₹0 (typically zero) | ₹0 (exit load handled separately) | Most direct MF platforms charge zero |
| **ETFs** | Fixed (₹) _or_ Variable (%) | e.g., ₹20 or 0.03% | e.g., ₹20 or 0.03% | Treated like equity delivery |

#### Field Details

| Column | Description |
|---|---|
| **Product** | Pre-filled rows: Stocks (Delivery), Stocks (Intraday), Options, Futures, Mutual Funds, ETFs |
| **Charge Type** | Toggle/dropdown: **Fixed (₹)** = flat fee per order, **Variable (%)** = percentage of transaction value |
| **Buy Brokerage** | Brokerage charged on buy/entry side |
| **Sell Brokerage** | Brokerage charged on sell/exit side |

#### Brokerage Rules & Behavior
- **Stocks (Intraday):** Broker typically charges only one-sided brokerage. User enters the charged side (usually buy) and sets the other to ₹0 / 0%.
- **Variable brokerage cap:** If charge type is Variable, optionally allow a max cap (e.g., 0.03% or max ₹20 — whichever is lower). This matches brokers like Zerodha.
- **Per-order vs per-lot:** For F&O, clarify if the flat fee is per order or per lot (v1: assume per order, add per-lot toggle later).
- **MF exit load:** Not brokerage — handled separately in the MF sell flow (exit load % and lock-in period from scheme metadata).
- **Auto-apply:** When a buy/sell transaction is entered, brokerage is pre-filled from this setup. User can override per transaction.
- **Editable anytime:** User can update brokerage rates from Settings; changes apply to future transactions only (past transactions retain their recorded charges).

#### Statutory Charges (Auto-Calculated, Non-Editable)
These are government/exchange mandated and computed automatically on every transaction:

| Charge | Rate | Applied To |
|---|---|---|
| **STT (Securities Transaction Tax)** | 0.1% delivery, 0.025% intraday sell, 0.0125% options sell, 0.01% futures sell | As per product |
| **Exchange Transaction Charges** | ~0.00345% (NSE) | All segments |
| **GST** | 18% on (brokerage + exchange charges) | All |
| **SEBI Turnover Fee** | ₹10 per crore | All |
| **Stamp Duty** | 0.015% (delivery buy), 0.003% (intraday/F&O buy) | Buy side only |

- These rates are pre-configured and updated when regulations change
- Displayed as a breakdown in each transaction's charges section

### 3.4 Setup Screen Summary

| Step | User Action | v1 Behavior |
|---|---|---|
| 1. Market | Select market-exchange | India : NSE/BSE (only option) |
| 2. Currency | Select base currency | INR (only option) |
| 3. Brokerage | Fill 6-row brokerage table | Pre-filled with Zerodha defaults, editable |
| 4. Confirm | Save setup | Configuration saved, proceed to portfolio |

- **Pre-filled defaults:** v1 ships with Zerodha's brokerage rates as defaults (₹20 flat or 0.03% whichever is lower for equity; ₹20 flat for F&O)
- **Skip option:** User can skip setup → defaults apply, editable later from Settings
- **Reset:** Option to reset brokerage to defaults

---

## 4. Asset Classes & Attributes

### 4.1 Stocks (Equity)
| Field | Description |
|---|---|
| Symbol / Ticker | NSE/BSE symbol (e.g., RELIANCE, TCS) |
| Exchange | NSE / BSE |
| Buy Date | Date of purchase |
| Quantity | Number of shares |
| Buy Price | Price per share at purchase |
| Brokerage & Charges | STT, brokerage, GST, stamp duty, exchange charges |
| Buy Order ID | Broker order reference (optional) |
| Broker Name | Zerodha, Groww, Angel One, etc. |
| Notes | Free-text (e.g., "earnings play", "long-term hold") |

### 4.2 Futures & Options (F&O)
| Field | Description |
|---|---|
| Underlying Symbol | e.g., NIFTY, BANKNIFTY, RELIANCE |
| Instrument Type | Future / Call Option / Put Option |
| Expiry Date | Contract expiry |
| Strike Price | For options only |
| Lot Size | Exchange-defined lot size |
| Number of Lots | Lots purchased |
| Total Quantity | Lot Size × Number of Lots (auto-calculated) |
| Buy/Sell Date | Entry date |
| Entry Price | Premium or futures price per unit |
| Brokerage & Charges | STT, brokerage, GST, stamp duty |
| Margin Blocked | Initial + exposure margin |
| Buy Order ID | Broker order reference (optional) |
| Broker Name | Broker used |
| Notes | Strategy notes (e.g., "hedged with PUT 23000") |

### 4.3 Mutual Funds
| Field | Description |
|---|---|
| Fund Name | Full scheme name |
| AMC | Asset Management Company |
| Fund Category | Equity / Debt / Hybrid / ELSS / Index / Sectoral |
| Folio Number | AMC folio reference |
| Purchase Date | Date of investment |
| Purchase Type | Lump Sum / SIP |
| Amount Invested (₹) | Rupees invested |
| NAV at Purchase | Net Asset Value on purchase date |
| Units Allotted | Amount / NAV (auto-calculated) |
| Platform | Groww, Kuvera, Coin, MFU, direct AMC |
| Notes | Free-text |

### 4.4 ETFs
| Field | Description |
|---|---|
| Symbol / Ticker | NSE symbol (e.g., NIFTYBEES, GOLDBEES) |
| ETF Category | Equity / Gold / Debt / International |
| Buy Date | Date of purchase |
| Quantity | Number of units |
| Buy Price | Price per unit |
| Brokerage & Charges | Same as equity |
| Broker Name | Broker used |
| Notes | Free-text |

---

## 5. Core Functionality

### 5.1 Buy (Add to Portfolio)
- User enters purchase details per asset class (fields above)
- System validates the instrument exists (see §5 Validations)
- Entry is added to the holdings ledger
- Portfolio totals are recalculated

### 5.2 Sell (Reduce from Portfolio)
- User selects existing holding and enters sale details:
  - Sell Date
  - Quantity / Units / Lots sold
  - Sell Price / NAV at redemption
  - Brokerage & Charges on sell side
  - Sell Order ID (optional)
- System validates sufficient quantity exists (no short selling)
- For partial sells, remaining quantity stays in portfolio
- Realized P&L is calculated and logged in a **Transactions Ledger**

### 5.3 Transactions Ledger
Every buy and sell is recorded as an immutable transaction log:
| Field | Description |
|---|---|
| Transaction ID | Auto-generated unique ID |
| Date | Transaction date |
| Asset Class | Stock / F&O / MF / ETF |
| Symbol / Fund Name | Identifier |
| Action | BUY / SELL |
| Quantity / Units / Lots | Amount transacted |
| Price / NAV | Per-unit price |
| Total Value (₹) | Quantity × Price |
| Charges (₹) | All-in charges |
| Net Value (₹) | Total ± Charges |
| Running Balance | Remaining quantity after this transaction |
| Notes | Free-text |

---

## 6. Validations

| # | Rule | Behavior |
|---|---|---|
| V1 | **Listed instruments only** | Buy is rejected if the stock/F&O underlying/MF scheme/ETF is not listed on NSE/BSE or AMFI. Validate against a master list (fetched or maintained). |
| V2 | **No short selling** | Sell quantity cannot exceed current holding quantity for that specific instrument. For F&O, sell lots ≤ held lots for the same underlying + expiry + strike. |
| V3 | **Valid lot size (F&O)** | Quantity must be a multiple of the exchange-defined lot size. |
| V4 | **Expiry date validation (F&O)** | Cannot buy an expired contract. Expired contracts auto-settle on expiry. |
| V5 | **Positive values** | Price, quantity, and amount must be > 0. |
| V6 | **Date sanity** | Sell date ≥ Buy date. Transaction date ≤ today (no future dating). |
| V7 | **Duplicate check** | Warn if an identical transaction (same instrument, date, qty, price) already exists. |
| V8 | **Auto-settle expiring lots** | System must auto-close all F&O positions expiring today at closing price after market close (AP1). No manual intervention required. |
| V9 | **Daily EOD snapshot** | System must generate a portfolio snapshot every trading day after market close (AP2). Duplicate snapshots for the same date are prevented. |

---

## 7. Portfolio Dashboard & Calculations

### 7.1 Current Holdings View
For each holding, display:
- **Current Market Price (CMP)** — latest price / NAV
- **Current Value** — Quantity × CMP
- **Invested Value** — Quantity × Weighted Avg Buy Price
- **Unrealized P&L (₹)** — Current Value − Invested Value
- **Unrealized P&L (%)** — (Unrealized P&L / Invested Value) × 100
- **Weight in Portfolio (%)** — Current Value / Total Portfolio Value × 100
- **Day Change (₹ and %)** — based on previous close

### 7.2 Portfolio-Level Metrics (Weighted Average of All Components)
| Metric | Formula |
|---|---|
| **Total Portfolio Value** | Sum of current value of all holdings |
| **Total Invested** | Sum of net invested across all holdings |
| **Total Unrealized P&L** | Total Value − Total Invested |
| **Total Realized P&L** | Sum of all closed trade P&L from transactions ledger |
| **Overall P&L** | Realized + Unrealized |
| **Portfolio Daily Change** | Weighted sum of each holding's daily % change |
| **XIRR / Annualized Return** | Time-weighted return using cash flow dates (buy = outflow, sell = inflow, current value = terminal inflow) |
| **CAGR** | (Current Value / Invested)^(1/years) − 1 |
| **Asset Class Allocation** | % split across Stocks, F&O, MF, ETF |
| **Sector Allocation** | % split by sector (requires sector mapping for stocks/ETFs) |

### 7.3 Historical Portfolio Value (End-of-Day)
- Store or compute **portfolio value at each market close date**
- Used for portfolio value chart over time
- Approach: for each date, sum (holding quantity on that date × closing price on that date)

### 7.4 Real-Time Portfolio Value (Market Hours)
- During market hours (9:15 AM – 3:30 PM IST, Mon–Fri, non-holiday):
  - Fetch live prices for stocks and ETFs
  - MF NAVs are EOD only (show previous NAV with label "NAV as of [date]")
  - F&O live prices for open contracts
- **Data source:** Free APIs with acceptable delay (15-min delayed is fine for v1)
  - Yahoo Finance API (unofficial, free, 15-min delay)
  - Google Finance (scraping, fragile)
  - NSE India API (unofficial endpoints)
  - AMFI NAV feed (official, free, EOD)
- Refresh interval: every 60 seconds during market hours (configurable)

---

## 8. EOD Batch Operations (Post-Market Close)

An automated batch process that runs after market close (after 3:30 PM IST) each trading day. This handles expiry settlement and daily portfolio snapshotting.

### 8.1 Auto-Process List

The following operations run automatically in sequence after market close:

| # | Auto-Process | Description | Trigger |
|---|---|---|---|
| **AP1** | **Auto-Settle Expiring F&O Lots** | Close all F&O contracts expiring today by generating sell transactions at closing price | Daily, on expiry dates |
| **AP2** | **Daily EOD Portfolio Snapshot** | Calculate and store end-of-day portfolio value for all holdings | Daily, every trading day |

### 8.2 AP1 — Auto-Settle Expiring F&O Lots

**Purpose:** F&O contracts that expire today must be closed out. The system auto-generates sell transactions so the portfolio reflects settled positions.

**Process:**
1. Identify all open F&O holdings where **Expiry Date = Today**
2. Fetch the **closing price** of each expiring contract from market data
3. For each expiring lot, auto-create a sell transaction:
   - Sell Date = Today
   - Sell Price = Closing price of the contract
   - Quantity = Full remaining lots held
   - Action = SELL (auto-settled)
   - Charges = Brokerage as per setup + statutory charges
   - Notes = "Auto-settled on expiry"
   - Source = "SYSTEM_AUTO" (to distinguish from manual trades)
4. Realized P&L is calculated and logged in the Transactions Ledger
5. Holding is removed from active portfolio (quantity → 0)

**For Options specifically:**
- **ITM (In-The-Money) options:** Auto-settled at intrinsic value (closing price of underlying − strike price for calls, strike − closing for puts)
- **OTM (Out-of-The-Money) options:** Expire worthless — sell transaction at ₹0 (full premium loss realized)
- **STT on exercised options:** Higher STT rate (0.125%) applied on ITM options exercised at expiry

**Validations for AP1:**
- Only processes contracts where expiry date matches today's date
- Skips if no expiring lots exist (no-op)
- Logs a summary of all auto-settled contracts for user review

### 8.3 AP2 — Daily EOD Portfolio Snapshot

**Purpose:** Capture a daily record of total portfolio value at market close for historical tracking, charts, and performance analysis.

**Process:**
1. Fetch **closing prices** for all held instruments:
   - Stocks & ETFs: NSE/BSE closing price
   - F&O: Settlement price from exchange
   - Mutual Funds: NAV published by AMC (available by ~11 PM IST; if not yet available, use previous NAV and update when published)
2. For each holding, calculate:
   - Current Value = Quantity × Closing Price / NAV
   - Day Change = Current Value − Previous Day Value
3. Store a snapshot in the **EOD Portfolio Table**:

#### EOD Portfolio Table Schema

| Field | Description |
|---|---|
| Date | Trading date |
| Total Portfolio Value (₹) | Sum of all holdings at closing price |
| Total Invested (₹) | Sum of all cost basis |
| Unrealized P&L (₹) | Total Value − Total Invested |
| Unrealized P&L (%) | % change from invested |
| Day Change (₹) | Today's value − Yesterday's value |
| Day Change (%) | % change from previous day |
| Realized P&L Today (₹) | P&L from any sells (including auto-settled expiries) today |
| Asset Breakdown — Stocks (₹) | Stock holdings value |
| Asset Breakdown — F&O (₹) | F&O holdings value |
| Asset Breakdown — MF (₹) | Mutual fund holdings value |
| Asset Breakdown — ETF (₹) | ETF holdings value |
| Holdings Count | Number of distinct instruments held |
| Auto-Settled Today | Count of F&O lots auto-settled (0 if none) |
| Snapshot Status | COMPLETE / PARTIAL (if MF NAV pending) / FAILED |

**Validations for AP2:**
- Runs only on trading days (skip weekends and NSE holidays)
- If closing price is unavailable for any instrument, mark snapshot as PARTIAL and retry later
- Prevents duplicate snapshots for the same date
- If AP1 ran (expiry settlements), AP2 includes those settled positions in today's realized P&L

### 8.4 Batch Execution Rules

| Rule | Detail |
|---|---|
| **Execution order** | AP1 (settle expiries) runs first → AP2 (snapshot) runs after |
| **Trigger time** | Configurable; default 4:00 PM IST (30 min after market close to allow price feeds to settle) |
| **Manual trigger** | User can manually trigger the batch from Settings (e.g., if it failed or was missed) |
| **Re-run safety** | Idempotent — re-running on the same day does not create duplicate transactions or snapshots |
| **Failure handling** | If AP1 fails, AP2 still runs (they are independent after AP1 completes). Failures are logged. |
| **Non-trading days** | Batch does not run on weekends or market holidays. Holiday calendar sourced from NSE. |
| **Audit log** | Each batch run logs: timestamp, processes executed, items affected, status, errors if any |

---

## 9. Reports & Analytics

| Report | Description |
|---|---|
| **Holdings Summary** | All current holdings grouped by asset class |
| **Transaction History** | Filterable log of all buys and sells |
| **Realized P&L Report** | Closed trades with buy/sell details and profit/loss |
| **Tax Report (Capital Gains)** | STCG vs LTCG classification based on holding period (equity: 1 yr, debt MF: 3 yr). For F&O: all gains are business income |
| **Dividend / Distribution Log** | Track dividends received (stocks/MF) — manual entry for v1 |
| **Monthly/Yearly Summary** | Portfolio value, P&L, inflows/outflows by period |

---

## 10. Tax & Regulatory Considerations (India-Specific)

| Item | Rule |
|---|---|
| **STCG (Equity/ETF)** | Holding < 12 months → taxed at 15% |
| **LTCG (Equity/ETF)** | Holding ≥ 12 months → taxed at 10% above ₹1L exemption |
| **STCG (Debt MF)** | Holding < 36 months → taxed at slab rate |
| **LTCG (Debt MF)** | Holding ≥ 36 months → taxed at 20% with indexation |
| **F&O** | Treated as business income, taxed at slab rate. Audit required if turnover > ₹10 Cr (or ₹2 Cr with conditions) |
| **Grandfathering** | For pre-31-Jan-2018 equity holdings, LTCG cost basis = max(actual cost, FMV on 31-Jan-2018) |

---

## 11. Data Model (Simplified)

```
app_config           (market, exchange, currency, setup_complete)
brokerage_config     (product, charge_type, buy_brokerage, sell_brokerage)
instruments          (master list: symbol, name, exchange, asset_class, sector, lot_size, is_active)
transactions         (id, date, instrument_id, action, qty, price, charges, notes, source[MANUAL/SYSTEM_AUTO])
holdings             (derived: instrument_id, total_qty, avg_buy_price — recomputed from transactions)
daily_prices         (instrument_id, date, open, high, low, close, volume)
portfolio_eod        (date, total_value, invested_value, unrealized_pnl, day_change, realized_pnl_today, stocks_value, fo_value, mf_value, etf_value, holdings_count, auto_settled_count, status)
batch_run_log        (id, date, trigger_time, ap1_status, ap1_items, ap2_status, errors)
dividends            (id, date, instrument_id, amount, type)
```

---

## 12. Phase Plan

| Phase | Scope | Platform |
|---|---|---|
| **Phase 1** | Google Sheet with all buy/sell entry, validations via Apps Script, basic portfolio view, EOD value tracking | Google Sheets |
| **Phase 2** | Web app — migrate data model, full CRUD, transaction ledger, dashboard with charts | Web Application |
| **Phase 3** | Real-time prices, XIRR/CAGR calculations, tax reports, alerts | Web Application |
| **Phase 4** | Broker CSV import (Zerodha, Groww contract notes), auto-reconciliation | Web Application |

---

## 13. Non-Functional Requirements

- **Single user** — no multi-tenancy needed for v1
- **Data privacy** — all data stays local or in user's own Google account (Phase 1) / self-hosted DB (Phase 2+)
- **Backup** — Google Sheet auto-saves; web app needs export-to-CSV/Excel
- **Performance** — dashboard loads in < 3 seconds; real-time refresh ≤ 60s
- **Mobile responsive** — web app should work on mobile browsers

---

## 14. Open Questions / Decisions Needed

1. **Broker integration priority** — which broker's CSV/contract note format to support first?
2. **Currency** — INR only, or support USD holdings (US stocks via Vested/INDmoney)?
3. **SIP tracking** — auto-generate recurring SIP entries or manual each time?
4. **Alerts** — price alerts, P&L threshold alerts, SIP due date reminders?
5. **F&O strategy grouping** — group legs of a spread/straddle as a single strategy?
6. **Historical data backfill** — import existing portfolio from broker statements, or start fresh?

---

*Next step: Review this PRD, discuss open questions, then finalize technology stack for Phase 1 (Google Sheet) and Phase 2 (Web App).*
