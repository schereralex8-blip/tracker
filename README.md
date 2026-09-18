# Two-Book Bet Ledger

A single-file bet tracker for a **PrizePicks** account and a **Polymarket** account
side by side: every bet, both bankrolls, unit sizing, and P&L in dollars and units.

Open `index.html` in any browser. No install, no build, no server, no account.
Everything is stored in that browser's `localStorage`, so the ledger stays on your
machine — back it up from the **Bankroll & data** tab.

## What it tracks

**Bankroll** — per book: `starting balance + deposits − withdrawals + settled P&L`.
Deposits and withdrawals are logged separately from bets, so moving money between
your bank and a book never shows up as profit.

**Units** — each book gets its own unit size, either a flat dollar amount or a
percentage of its current bankroll. A bet stores the unit size it was sized
against, so last month's 1u bet stays 1u even after the bankroll moves.

**P&L** — realized profit in dollars and units, ROI on money risked, W–L–P record,
win rate, open exposure on pending bets, and a cumulative P&L curve per book plus
combined.

## Logging a bet

One form covers both books, because both reduce to the same thing: an amount at
risk and a multiple it returns.

| Field | PrizePicks | Polymarket |
| --- | --- | --- |
| Stake / cost | entry fee | shares × price |
| Price | entry multiplier (3×, 6×, 25×) | share price in cents, e.g. `42` → 2.38× |
| Result | win / loss / push | win / loss / void |

Prices can be entered three ways — **multiplier**, **American odds**, or **share
price in cents** — and all three are stored as a single payout multiplier.
Switching the book preselects the price mode that book uses.

**Partial or sold early** covers the two cases a plain win/loss can't:

- a PrizePicks flex play that goes 3-of-4 and pays 1.5× — enter what came back
- Polymarket shares sold before resolution — enter the sale proceeds

Both take the total returned including stake, and P&L is the difference.

## Tabs

- **Dashboard** — KPI strip, a card per book, the cumulative P&L chart (dollars or
  units, hover for any date), and the last six bets
- **Bets** — the full ledger, filterable by book, result and free text, sortable by
  date, P&L or stake; the footer totals whatever is on screen
- **Bankroll & data** — starting balances, unit sizing, deposits/withdrawals, and
  JSON backup / CSV export

## Backing up

The **Bankroll & data** tab prints your whole ledger as JSON. Copy it somewhere
safe; paste it back into the same box and press **Import JSON from box** to
restore it or move it to another browser or device. CSV export is there for
spreadsheet work.

Clearing your browser's site data deletes the ledger. Back it up first.

## Sample data

A first run shows sample bets so the numbers have something to say. **Clear
samples & start fresh** wipes them and resets both bankrolls to zero — and logging
your own first bet clears the sample flag automatically.
