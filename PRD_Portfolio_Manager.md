# PRD: Personal Portfolio Manager

**Version:** 1.0 | **Date:** 2026-06-26 | **Author:** Simran Modi

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

## 3. Asset Classes & Attributes

### 3.1 Stocks (Equity)
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

### 3.2 Futures & Options (F&O)
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

### 3.3 Mutual Funds
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

### 3.4 ETFs
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

## 4. Core Functionality

### 4.1 Buy (Add to Portfolio)
- User enters purchase details per asset class (fields above)
- System validates the instrument exists (see §5 Validations)
- Entry is added to the holdings ledger
- Portfolio totals are recalculated

### 4.2 Sell (Reduce from Portfolio)
- User selects existing holding and enters sale details:
  - Sell Date
  - Quantity / Units / Lots sold
  - Sell Price / NAV at redemption
  - Brokerage & Charges on sell side
  - Sell Order ID (optional)
- System validates sufficient quantity exists (no short selling)
- For partial sells, remaining quantity stays in portfolio
- Realized P&L is calculated and logged in a **Transactions Ledger**

### 4.3 Transactions Ledger
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

## 5. Validations

| # | Rule | Behavior |
|---|---|---|
| V1 | **Listed instruments only** | Buy is rejected if the stock/F&O underlying/MF scheme/ETF is not listed on NSE/BSE or AMFI. Validate against a master list (fetched or maintained). |
| V2 | **No short selling** | Sell quantity cannot exceed current holding quantity for that specific instrument. For F&O, sell lots ≤ held lots for the same underlying + expiry + strike. |
| V3 | **Valid lot size (F&O)** | Quantity must be a multiple of the exchange-defined lot size. |
| V4 | **Expiry date validation (F&O)** | Cannot buy an expired contract. Expired contracts auto-settle on expiry. |
| V5 | **Positive values** | Price, quantity, and amount must be > 0. |
| V6 | **Date sanity** | Sell date ≥ Buy date. Transaction date ≤ today (no future dating). |
| V7 | **Duplicate check** | Warn if an identical transaction (same instrument, date, qty, price) already exists. |

---

## 6. Portfolio Dashboard & Calculations

### 6.1 Current Holdings View
For each holding, display:
- **Current Market Price (CMP)** — latest price / NAV
- **Current Value** — Quantity × CMP
- **Invested Value** — Quantity × Weighted Avg Buy Price
- **Unrealized P&L (₹)** — Current Value − Invested Value
- **Unrealized P&L (%)** — (Unrealized P&L / Invested Value) × 100
- **Weight in Portfolio (%)** — Current Value / Total Portfolio Value × 100
- **Day Change (₹ and %)** — based on previous close

### 6.2 Portfolio-Level Metrics (Weighted Average of All Components)
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

### 6.3 Historical Portfolio Value (End-of-Day)
- Store or compute **portfolio value at each market close date**
- Used for portfolio value chart over time
- Approach: for each date, sum (holding quantity on that date × closing price on that date)

### 6.4 Real-Time Portfolio Value (Market Hours)
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

## 7. Reports & Analytics

| Report | Description |
|---|---|
| **Holdings Summary** | All current holdings grouped by asset class |
| **Transaction History** | Filterable log of all buys and sells |
| **Realized P&L Report** | Closed trades with buy/sell details and profit/loss |
| **Tax Report (Capital Gains)** | STCG vs LTCG classification based on holding period (equity: 1 yr, debt MF: 3 yr). For F&O: all gains are business income |
| **Dividend / Distribution Log** | Track dividends received (stocks/MF) — manual entry for v1 |
| **Monthly/Yearly Summary** | Portfolio value, P&L, inflows/outflows by period |

---

## 8. Tax & Regulatory Considerations (India-Specific)

| Item | Rule |
|---|---|
| **STCG (Equity/ETF)** | Holding < 12 months → taxed at 15% |
| **LTCG (Equity/ETF)** | Holding ≥ 12 months → taxed at 10% above ₹1L exemption |
| **STCG (Debt MF)** | Holding < 36 months → taxed at slab rate |
| **LTCG (Debt MF)** | Holding ≥ 36 months → taxed at 20% with indexation |
| **F&O** | Treated as business income, taxed at slab rate. Audit required if turnover > ₹10 Cr (or ₹2 Cr with conditions) |
| **Grandfathering** | For pre-31-Jan-2018 equity holdings, LTCG cost basis = max(actual cost, FMV on 31-Jan-2018) |

---

## 9. Data Model (Simplified)

```
instruments          (master list: symbol, name, exchange, asset_class, sector, lot_size, is_active)
transactions         (id, date, instrument_id, action, qty, price, charges, notes)
holdings             (derived: instrument_id, total_qty, avg_buy_price — recomputed from transactions)
daily_prices         (instrument_id, date, open, high, low, close, volume)
portfolio_snapshots  (date, total_value, invested_value, pnl)
dividends            (id, date, instrument_id, amount, type)
```

---

## 10. Phase Plan

| Phase | Scope | Platform |
|---|---|---|
| **Phase 1** | Google Sheet with all buy/sell entry, validations via Apps Script, basic portfolio view, EOD value tracking | Google Sheets |
| **Phase 2** | Web app — migrate data model, full CRUD, transaction ledger, dashboard with charts | Web Application |
| **Phase 3** | Real-time prices, XIRR/CAGR calculations, tax reports, alerts | Web Application |
| **Phase 4** | Broker CSV import (Zerodha, Groww contract notes), auto-reconciliation | Web Application |

---

## 11. Non-Functional Requirements

- **Single user** — no multi-tenancy needed for v1
- **Data privacy** — all data stays local or in user's own Google account (Phase 1) / self-hosted DB (Phase 2+)
- **Backup** — Google Sheet auto-saves; web app needs export-to-CSV/Excel
- **Performance** — dashboard loads in < 3 seconds; real-time refresh ≤ 60s
- **Mobile responsive** — web app should work on mobile browsers

---

## 12. Open Questions / Decisions Needed

1. **Broker integration priority** — which broker's CSV/contract note format to support first?
2. **Currency** — INR only, or support USD holdings (US stocks via Vested/INDmoney)?
3. **SIP tracking** — auto-generate recurring SIP entries or manual each time?
4. **Alerts** — price alerts, P&L threshold alerts, SIP due date reminders?
5. **F&O strategy grouping** — group legs of a spread/straddle as a single strategy?
6. **Historical data backfill** — import existing portfolio from broker statements, or start fresh?

---

*Next step: Review this PRD, discuss open questions, then finalize technology stack for Phase 1 (Google Sheet) and Phase 2 (Web App).*
