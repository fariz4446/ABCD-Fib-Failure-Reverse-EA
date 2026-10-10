# ABCD Fib Failure Reverse EA + Fractal ZigZag Indicator (MT5)

A MetaTrader 5 Expert Advisor that trades the **retracement of an A-B leg** found by a non-repainting ZigZag, with **3 take-profit positions** per setup. Includes a companion chart indicator that draws the ZigZag and Fibonacci levels.

> ⚠️ **Trading involves risk.** This is not financial advice. Test on a **demo account** first. Past results do not guarantee future results.

---

## 📦 Files

| File | Type | Purpose |
|---|---|---|
| `ABCD_Fib_Failure_Reverse_EA.ex5` (v3.00) | Expert Advisor | Trades the retracement setup automatically (3 positions per setup) |
| `ABCD_Fib_Failure_Fractal_ZigZag.ex5` (v1.00) | Indicator | Draws the ZigZag, fractal arrows and Fibonacci levels on the chart |

Both work on **MetaTrader 5 only** (not MT4). The EA has its own built-in ZigZag, so it **does not need the indicator** to trade. The indicator is only for viewing.

---

## 🧠 Strategy: how the EA opens a trade

The EA works on closed candles of the chart timeframe and follows these steps.

### 1. Find the A-B leg (non-repainting ZigZag)
- A candle becomes a **pivot high** if its high is the highest among **13 candles to its left** and **3 candles to its right**. A **pivot low** is the same with lows.
- Because it waits for 3 closed candles after the pivot, the pivot **never repaints or moves**.
- Pivots closer than 5 points to the previous opposite pivot are ignored.
- The **last two confirmed pivots** form the leg: **A** = second-to-last pivot, **B** = latest pivot.

### 2. Decide the direction
- **B is a high** (the leg went up from A to B) → the EA waits for a pullback and looks for a **BUY**.
- **B is a low** (the leg went down from A to B) → the EA waits for a pullback and looks for a **SELL**.

This is the **reverse of the Fibonacci failure idea**: instead of selling the failure of a bullish leg, it buys the pullback of that leg (and the mirror for sells).

### 3. Wait for the retracement entry zone
The EA measures how deep price has pulled back from B toward A, as a fraction of the A-B range.

| Condition | Default |
|---|---|
| Enter when the pullback is **at least** | **0.705** of A-B |
| But **not deeper than** | **0.886** of A-B |

- Pullback shallower than 0.705 → keep waiting.
- Pullback deeper than 0.886 → skip (too close to A), keep waiting.
- Price breaks **beyond Point A** → the setup is **cancelled**.

### 4. Entry checks
When price is inside the 0.705 to 0.886 zone, the EA opens the trade at **market price** only if:
- the **spread** is at or below **60 points**,
- fewer than **3** positions of this EA (same symbol and magic number) are open,
- this A-B leg has not already been traded (**one setup per leg**).

### 5. Open 3 positions with one stop loss and three targets
| Item | Rule (defaults) |
|---|---|
| Positions | **3**, each with **one third** of the total lot (0.06 total = 0.02 each) |
| Stop loss | **150 points beyond Point A** |
| TP1 | Entry ± **0.8 × A-B range** |
| TP2 | Entry ± **1.2 × A-B range** |
| TP3 | Entry ± **1.6 × A-B range** |

Buys use targets above the entry, sells use targets below.

### 6. Managing the setup
- A **new confirmed pivot** creates a new A-B leg and replaces the old setup.
- If buy and sell positions ever exist at the same time, the EA keeps the **newest direction** and closes the other (option: *Close trades on opposite signal*).
- If an order fails, the EA retries up to **3 times** for that leg.

### Example (BUY)
1. A = 4000.00 (low), B = 4100.00 (high). Range = 100.
2. Entry zone: price pulls back to between 4029.50 (0.705) and 4011.40 (0.886).
3. EA buys 3 positions at market. TP1 = entry + 80, TP2 = entry + 120, TP3 = entry + 160. SL = 150 points below 4000.00.

---

## ⚙️ Main inputs

| Input | Default | Meaning |
|---|---|---|
| Total Lot Size | 0.06 | Split across 3 positions. Lots are rounded down to the broker's lot step, with the broker's minimum as a floor |
| Close trades on opposite signal | true | Keep only the newest direction |
| Magic Number | 884422 | Identifies this EA's trades |
| Max Slippage / Max Spread | 30 / 60 points | Order slippage and spread filter |
| ZigZag Depth / Confirm bars | 13 / 3 | Bars left and right of a pivot |
| ZigZag Deviation | 5 points | Minimum distance between opposite pivots |
| Only accept Fractal pivots | false | Extra pivot filter |
| Entry Fib | 0.705 | Minimum pullback to enter |
| Max Depth Fib | 0.886 | Skip if pullback is deeper |
| Require HH/HL | false | BUY needs higher low + higher high, SELL needs lower high + lower low |
| SL buffer | 150 points | Distance beyond Point A |
| TP1 / TP2 / TP3 multiplier | 0.8 / 1.2 / 1.6 | Times the A-B range |
| Show status on chart | true | Shows the current state and why there is no entry |

---

## 📈 The indicator

`ABCD_Fib_Failure_Fractal_ZigZag` draws the ZigZag line, fractal arrows, the live Fibonacci of the current A-B leg and buy/sell arrows.

> Note: the indicator's arrows and Entry/SL/TP boxes show the **original Fibonacci failure signal (0.786 level)**, not the reverse entries the EA takes. Use it to see the ZigZag and Fibonacci levels. Use the EA's on-chart status text to see the EA's own entry level.

---

## 🔧 Installation

1. Download both `.ex5` files from this page (click the file, then **Download**, or use **Code → Download ZIP**).
2. Open MetaTrader 5 and click **File → Open Data Folder**, then open the `MQL5` folder.
3. Copy the files:
   - `ABCD_Fib_Failure_Reverse_EA.ex5` → `MQL5\Experts`
   - `ABCD_Fib_Failure_Fractal_ZigZag.ex5` → `MQL5\Indicators`
4. In MT5, right-click **Navigator** and choose **Refresh**.

## ▶️ How to use

1. Open a chart of the symbol you want to trade (for example XAUUSD) and choose your timeframe.
2. Make sure the **Algo Trading** button in the toolbar is **green**.
3. Drag the **EA** from Navigator onto the chart.
4. On the **Common** tab tick **Allow Algo Trading**, review the **Inputs** tab, and click **OK**.
5. A smiley face in the chart's top-right corner means the EA is running. The chart text shows its current state.
6. (Optional) Drag the **indicator** onto the same chart to see the ZigZag and Fibonacci levels.

---

## ✅ Tips

- Test on a **demo account** for a few weeks before using real money.
- Start with a **low lot size**. Three positions open per setup, so your real exposure is the total of all three.
- Point values depend on the broker's digits. 150 points can be a different price distance on different symbols and brokers, so check the SL distance on your symbol.
- The 60-point spread filter may be too tight for some brokers or news times. Adjust it if the EA keeps showing "Spread too high".
- Keep MT5 open, or use a VPS, so the EA keeps running.

## ❓ Troubleshooting

| Problem | What to check |
|---|---|
| EA not in Navigator | Wrong folder. Re-copy to `MQL5\Experts` and refresh Navigator |
| Sad face on chart | **Algo Trading** is off. Turn it on and tick **Allow Algo Trading** |
| No trades | Read the chart status text. Common states: *Waiting for retracement*, *Retrace too deep*, *Spread too high*, *Invalidated: price broke Point A* |
| "Invalid volume" | Lot size is below your broker's minimum |

---

## 📬 Contact & more EAs

- Telegram: [@silentcircletrader](https://t.me/silentcircletrader)
- Free EAs channel: [t.me/freexauusdforexea](https://t.me/freexauusdforexea)
- Patreon: [patreon.com/xauusdbots](https://www.patreon.com/xauusdbots)

---

<sub>Disclaimer: Trading leveraged products such as forex and gold carries a high level of risk and may not be suitable for everyone. You are responsible for your own trading decisions.</sub>
