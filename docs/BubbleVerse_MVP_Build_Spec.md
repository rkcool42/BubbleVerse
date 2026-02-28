# BubbleVerse — Bubble.io Build Specification (v0.1 MVP)

This document is an implementation-ready blueprint for a **mobile-first Bubble.io app** named **BubbleVerse**, inspired by CryptoBubbles-style stock bubbles.

---

## 1) Product Scope

**Goal:** Build a clean, performant, Android-wrapper-ready MVP using static/demo Indian stock data.

### Included in v0.1
- Bubble-based stock dashboard
- Tap-to-open stock detail panel
- Sortable/searchable markets list
- Portfolio holding entry + gain/loss calculator
- Profile placeholder section

### Explicitly Excluded
- Payments
- AI predictions
- Social feed
- Notifications
- News integration
- Advanced analytics
- Live APIs / real-time sockets

---

## 2) Global Page Setup (Bubble Editor)

- **Page name:** `index`
- **Page width:** `390`
- **Responsive engine:** ON
- **No horizontal scrolling:** enabled by keeping all major groups fixed to full width and no overflow x
- **Vertical scrolling:** allowed
- **Architecture:** single-page UI with visibility-based tab switching

### Design Tokens
- **App background:** `#0F172A` (deep navy)
- **Positive gradient:** Emerald ➜ Teal (e.g., `#10B981` ➜ `#14B8A6`)
- **Negative gradient:** Crimson ➜ Pink (e.g., `#DC2626` ➜ `#EC4899`)
- **Text:** White / high-contrast
- **Radius:** 12–20px
- **Shadow:** soft blur, low opacity
- **Grid spacing:** 8px increments

---

## 3) Database Schema

## Data Type: `Stock`
Fields:
- `Name` (text)
- `Ticker` (text)
- `Price` (number)
- `PercentChange` (number)
- `MarketCap` (number)
- `Sector` (text)

## Data Type: `PortfolioHolding`
Fields:
- `Stock` (Stock)
- `Quantity` (number)
- `BuyPrice` (number)

**Auth:** Not required for MVP.

---

## 4) Seed Data (Demo Indian Stocks)

Import `data/demo_indian_stocks.csv` into `Stock`.

Recommended stock count: 10.

---

## 5) Main Layout Structure

Create these top-level groups (full width, page-level):
- `Group_Dashboard` (visible on page load)
- `Group_Markets` (hidden on load)
- `Group_Portfolio` (hidden on load)
- `Group_Profile` (hidden on load)

Create fixed bottom nav:
- `Group_BottomNav` (fixed to bottom, full width)
  - `Btn_Dashboard`
  - `Btn_Markets`
  - `Btn_Portfolio`
  - `Btn_Profile`

### BottomNav Workflow Pattern
For each nav button click:
1. Hide all main groups
2. Show selected group

Implementation tip: chain 5 actions in one workflow for deterministic state.

---

## 6) Dashboard Tab — Bubble Canvas

## Container
- `Group_BubbleCanvas`
  - full width
  - min height: viewport minus bottom nav
  - background: deep navy

## Repeating Group
- `RepeatingGroup_Stocks`
  - Type: `Stock`
  - Data source: `Search for Stocks`
  - Layout style: fixed columns
  - Columns: 2 (small phones), optional 3 for wider devices
  - No overlap
  - Consistent inner/outer padding

## Cell Composition
Inside each cell create `Group_Bubble`:
- circular shape (`border radius: 999px`)
- centered content
- width/height equal
- subtle shadow + slight glow
- transition feel: ~150ms (via style/state changes)

### Bubble Sizing (Stable conditional ranges)
Set base width to 110 and add conditions:
- If `Current cell's Stock's MarketCap >= 2500000000000` ➜ width/height 140
- If `MarketCap >= 1000000000000 and < 2500000000000` ➜ width/height 120
- Else ➜ width/height 100

(Adjust numbers to your demo units, but keep exactly 3 bands for performance.)

### Bubble Color Conditionals
- If `PercentChange > 0`: emerald→teal gradient
- If `PercentChange < 0`: crimson→pink gradient
- Else: neutral slate gradient

## Bubble Text
Inside `Group_Bubble`:
- `Text_Ticker` (bold, centered)
- `Text_PercentChange` (larger, bold, centered)
- White text, center align both axes

Format percent text as:
- `+X.XX%` when positive
- `X.XX%` otherwise

## Bubble Interaction
Workflow on `Group_Bubble` click:
1. (Optional micro-interaction) set custom state `isPressed=yes` for 150ms style bump
2. Show `Popup_StockDetail`
3. Display data = `Current cell's Stock`

---

## 7) Popup — Stock Detail Panel

Create `Popup_StockDetail`:
- Position: bottom aligned
- Width: full
- Height: ~60% viewport
- Top corners rounded
- Light glass effect (semi-transparent light background + blur-like styling)
- Soft shadow

Contents:
- `Text_Name`
- `Text_Price` (₹ formatted)
- `Text_PercentChange`
- `Text_MarketCap`
- `Text_Sector`
- `Btn_AddPortfolio`

### Add-to-Portfolio Workflow
When `Btn_AddPortfolio` clicked:
1. Create a new `PortfolioHolding`
   - `Stock = Parent group's Stock` (or popup thing)
   - `Quantity = 1` (default for quick-add)
   - `BuyPrice = Popup Stock's Price`
2. Show toast/alert: `Added to portfolio`
3. Optionally close popup

---

## 8) Markets Tab

`Group_Markets` contains:
- `Input_SearchTicker`
- `Dropdown_SortBy`
- `RepeatingGroup_MarketList` (Type: Stock, vertical list)

Row fields:
- Ticker
- Price
- PercentChange (conditional green/red)
- MarketCap

### Data Source Logic
Base query: `Search for Stocks`

Filter:
- Ticker contains `Input_SearchTicker's value:trimmed:lowercase`

Sort options in `Dropdown_SortBy`:
- `MarketCap (High-Low)`
- `Top Gainers`
- `Top Losers`
- `Ticker (A-Z)`

Use `:sorted by` conditionals by dropdown value.

---

## 9) Portfolio Tab

`Group_Portfolio` sections:

## A) Manual Holding Input
- `Dropdown_SelectStock` (choices = Stocks)
- `Input_Quantity` (number)
- `Input_BuyPrice` (number)
- `Btn_SaveHolding`

Workflow (`Btn_SaveHolding`):
- Create `PortfolioHolding` with selected stock, quantity, buy price.
- Reset inputs.

## B) Portfolio Summary List
- `RepeatingGroup_PortfolioHoldings` (Type: PortfolioHolding)

Each row displays:
- `Stock Ticker`
- `Invested = Quantity × BuyPrice`
- `Current Value = Quantity × Stock.Price`
- `Gain/Loss = (Stock.Price − BuyPrice) × Quantity`

Color rules for Gain/Loss text:
- Positive ➜ Green
- Negative ➜ Red
- Zero ➜ Neutral

---

## 10) Profile Tab

`Group_Profile` includes:
- App name (`BubbleVerse`)
- Version text (`v0.1 MVP`)
- Optional `Toggle_DarkLight` (non-critical placeholder)
- Pro features placeholder text/card

No auth, billing, or gated logic.

---

## 11) Performance Guardrails

- No physics simulation
- No heavy chart libraries
- No live API polling
- No realtime workflows
- Keep conditionals minimal (3 bubble-size bands, straightforward color conditions)
- Use static seeded data only

---

## 12) Android Wrapper Readiness Checklist

- 390px mobile-first layout verified
- Bottom navigation fixed and touch-friendly
- One-page architecture (group visibility switching)
- No browser-only dependencies
- No external scripts required for core UX
- All major actions complete within <=2 taps

---

## 13) Acceptance Checklist

- [x] Bubble-based stock dashboard
- [x] Tap-to-open stock detail popup panel
- [x] Sortable/searchable markets list
- [x] Functional portfolio calculator (invested/current/gain-loss)
- [x] Clean mobile UI inspired by CryptoBubbles
- [x] Android wrapper ready

**Version:** 0.1 MVP  
**Priority:** Visual clarity + smooth UX + performance stability
