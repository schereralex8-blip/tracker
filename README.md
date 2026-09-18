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

## Importing from Polymarket

**Bankroll & data → Import from Polymarket.** Paste the wallet address from your
profile URL (`polymarket.com/profile/0x…` — the proxy wallet), and the tracker reads
your public trade history off Polymarket's data API and turns each market into a bet.
Read-only public data: no keys, no signing, nothing that can move funds.

Set the Polymarket starting bankroll *before* importing — each imported bet snapshots
the unit size it was sized against. If you get the order wrong, **Resize all Polymarket
bets** re-stamps them at the current unit.

### Two routes to the same import

- **Fetch from Polymarket** — one click, when the page can reach polymarket.com
  directly. That means `index.html` opened from your own machine.
- **Paste JSON** — the tracker builds the two API links, you open them and paste
  the responses back. Needed on claude.ai, where the page is sandboxed and cannot
  call other sites. Same parser, same result.

### What it does with the data

Trades are aggregated per market and outcome, so a position filled across five orders
becomes one bet with a blended entry price. Each market can produce a closed bet, an
open bet, or both:

| Polymarket | Becomes |
| --- | --- |
| Bought, redeemed at $1 | Win — stake is your cost, payout multiple is `1 / avg price` |
| Bought, sold before resolution | Partial — the sale proceeds are the return |
| Bought, resolved against you | Loss |
| Bought, still holding | Pending, counted as open exposure |
| Sold part, still holding the rest | Two bets — cost basis split pro-rata by shares |
| Resolved your way, not yet claimed | Win, at $1 a share |

**Paste both responses when you can.** A market you lost leaves no trade when it
resolves — the position simply disappears — so nothing in your trade history says you
lost. The positions list is what lets those get booked as losses instead of sitting
open forever. Import with trade history alone and they stay pending; the preview says
so when that happens.

Splits, merges, conversions and rewards are skipped — they don't map to a single bet —
and the preview counts them so nothing vanishes silently.

### Re-importing

Every import is previewed before it touches your ledger, and imported bets carry a key
tying them to their market. Re-import as often as you like: bets are refreshed in
place rather than duplicated, and a row the new data no longer supports — an open
position that has since resolved — is replaced rather than left behind. Anything
imported stays editable by hand like any other bet.

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
