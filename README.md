# ABCD Fib Reverse EA for MetaTrader 5

**An MT5 Expert Advisor that finds the latest ZigZag A→B leg, waits for price to retrace into the 0.705 Fibonacci zone, and opens 3 split positions with a pre-set stop loss and three take-profit targets.**

> ⚠️ Trading involves substantial risk. This EA is provided for education and testing. It does not guarantee profits. Always test on a demo account first.

---

## Features

- Non-repainting ZigZag pivots built into the EA (no extra indicator needed)
- Automatic A→B leg detection, with optional Higher High / Higher Low filter
- Entry when price retraces into the **0.705 – 0.886** zone of the A→B leg
- **Flip switch:** trade the retracement direction or the reverse direction
- 3 positions per setup with TP1 / TP2 / TP3, lot split equally
- Close-opposite option, max-spread filter and magic number
- On-chart status panel that shows exactly why the EA is waiting or not trading

---

## How It Works

1. **Find the leg.** The EA builds ZigZag pivots from closed candles. The latest confirmed pivot is **B** and the one before it is **A**.
2. **Measure the retracement.** After B forms, it measures how much of the A→B range price has given back.
3. **Enter in the zone.** When the retracement is between `InpEntryFib` (0.705) and `InpMaxDepthFib` (0.886), it opens 3 market orders.
4. **Manage the trade.** Each order has the same stop loss and its own take profit.
5. **Cancel if invalid.** If price breaks Point A first, or a new pivot changes the leg, the setup is dropped. One entry is taken per A→B leg.

### Direction modes (`InpFlipDirection`)

| | Flip OFF | Flip ON (default) |
|---|---|---|
| Up leg, price retraces down | BUY | SELL |
| Down leg, price retraces up | SELL | BUY |
| Stop loss | Beyond Point A + buffer | Beyond Point B + buffer |
| Take profits | 0.8 / 1.2 / 1.6 × AB range from entry, in the trade direction | Same multipliers, in the flipped direction |

---

## Installation

1. Open MetaTrader 5.
2. Click **File → Open Data Folder**.
3. Go to **MQL5 → Experts** and copy `ABCD_Fib_Failure_Reverse_EA_Flip.mq5` into it.
4. Open **MetaEditor** (press `F4`), open the file and press `F7` to compile. It should say `0 errors`.
5. Back in MT5, right-click **Expert Advisors** in the Navigator and choose **Refresh**.
6. Turn on **Algo Trading** in the top toolbar.

---

## Quick Start (XAUUSD, M15)

1. Open an **XAUUSD** chart and set the timeframe to **M15**.
2. Drag **ABCD_Fib_Failure_Reverse_EA_Flip** from the Navigator onto the chart.
3. On the **Common** tab, tick **Allow Algo Trading**.
4. On the **Inputs** tab, apply the settings below and click **OK**.
5. A status panel appears in the top-left corner of the chart. A setup shows `A`, `B` and the entry level, then `Waiting for retracement`.

### Recommended settings for M15

| Group | Input | Value |
|---|---|---|
| Trade | `InpTotalLot` | 0.06 (split into 3 × 0.02) |
| Trade | `InpCloseOpposite` | true |
| Trade | `InpMagic` | 884422 |
| Trade | `InpSlippage` | 30 |
| Trade | `InpMaxSpread` | 60 |
| ZigZag | `InpZZ_Depth` | 13 |
| ZigZag | `InpZZ_Confirm` | 3 |
| ZigZag | `InpZZ_Deviation` | 150 |
| ZigZag | `InpReqFractal` | false |
| ZigZag | `InpWarmupBars` | 1000 |
| Entry | `InpEntryFib` | 0.705 |
| Entry | `InpMaxDepthFib` | 0.886 |
| Entry | `InpRequireHHHL` | false (set true for a stricter trend filter) |
| Entry | `InpFlipDirection` | true |
| SL / TP | `InpSL_BufferPoints` | 150 |
| SL / TP | `InpSL_Fib` | 1.0 |
| SL / TP | `InpTP1_Mult` / `InpTP2_Mult` / `InpTP3_Mult` | 0.8 / 1.2 / 1.6 |
| Display | `InpShowStatus` | true |

> **Point size:** these values assume gold with 2 digits (1 point = $0.01). If your broker quotes 3 digits, multiply the point-based inputs (Deviation, SL Buffer, Max Spread) by 10.

---

## Input Reference

**Trade Settings**
- `InpTotalLot`: total lot, split equally into 3 positions
- `InpCloseOpposite`: close positions of the opposite direction when a new trade opens
- `InpMagic`: magic number used to identify the EA's trades
- `InpSlippage`: maximum slippage in points
- `InpMaxSpread`: no entry if the spread is higher than this (points)

**ZigZag**
- `InpZZ_Depth`: bars to the left of a pivot that must be lower (or higher)
- `InpZZ_Confirm`: bars to the right needed to confirm a pivot (the EA does not repaint)
- `InpZZ_Deviation`: minimum distance in points between opposite pivots
- `InpReqFractal`: only accept pivots that are also 5-bar fractals
- `InpWarmupBars`: history bars used to build pivots at start

**Entry on Retracement**
- `InpEntryFib`: minimum retracement of A→B before entering
- `InpMaxDepthFib`: skip if price has retraced deeper than this
- `InpRequireHHHL`: require Higher Low + Higher High (up leg) or Lower High + Lower Low (down leg)
- `InpFlipDirection`: reverse the trade direction (see table above)

**Stop Loss / Take Profit**
- `InpSL_BufferPoints`: buffer in points beyond the stop reference
- `InpSL_Fib`: stop reference when Flip is OFF (1.0 = Point A)
- `InpTP1_Mult`, `InpTP2_Mult`, `InpTP3_Mult`: take profits as multiples of the A→B range, measured from entry

---

## Reading the Status Panel

| Message | Meaning |
|---|---|
| `Waiting for enough ZigZag pivots` | Not enough history yet. Wait, or raise `InpWarmupBars`. |
| `Waiting for retracement` | A leg is found and the EA is waiting for price to reach the zone. |
| `Retrace too deep, waiting` | Price went past `InpMaxDepthFib`. |
| `Invalidated: price broke Point A` | The setup was cancelled. |
| `Spread too high` | The spread is above `InpMaxSpread`. |
| `3 positions already open` | The EA will not open another set until positions close. |
| `Trade opened for this A-B leg` | Entry done for the current leg. |

---

## Testing Before Going Live

1. Open **View → Strategy Tester** (`Ctrl+R`).
2. Select the EA, symbol **XAUUSD**, period **M15**, and a recent date range.
3. Set **Modeling** to **Every tick based on real ticks** if your broker provides them.
4. Run the test and compare drawdown and trade count, not only profit.
5. Forward-test on a **demo account** for at least one to two weeks before using real money.

---

## Tips

- Start with the lowest lot your broker allows.
- Re-check the point-based inputs if you change symbol or broker.
- Compare the EA's trades with the ABCD Fib Failure + Fractal ZigZag indicator on the same chart to see the pivots the EA uses.
- Avoid running the EA across major news releases, where spreads widen.

---

## Disclaimer

This software is provided "as is", without warranty of any kind. Past performance does not predict future results. You are solely responsible for your trading decisions and any losses. Never risk money you cannot afford to lose.

---

**Links:** [patreon.com/xauusdbots](https://patreon.com/xauusdbots)
