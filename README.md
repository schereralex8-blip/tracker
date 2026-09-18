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

**Month by month** — paired columns per month, one per book, reading off a zero
baseline so a losing month hangs below the line. Hover a month for its split, net,
bet count and money staked. The chart shows the last twelve months; the table under
it carries every month, with staked, per-book P&L, net, units and ROI, and an
all-time row at the foot. Months where nothing settled stay in the sequence as gaps
rather than being skipped — a quiet month is information, and dropping it would bend
the time axis. The dollars/units switch is shared with the cumulative chart, so the
two never disagree.

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
  units, hover for any date), the month-by-month breakdown, and the last six bets
- **Bets** — the full ledger, filterable by book, result and free text, sortable by
  date, P&L or stake; the footer totals whatever is on screen
- **Bankroll & data** — starting balances, unit sizing, deposits/withdrawals, and
  JSON backup / CSV export

## Importing from PrizePicks

**Bankroll & data → Import from PrizePicks.** PrizePicks keeps entry history behind
your login and publishes no per-account API, so unlike Polymarket there's no address
to point at and nothing to fetch. This is a paste, in either of two shapes.

### Spreadsheet or CSV

Copy straight out of Excel, Numbers or Google Sheets — tabs, commas and semicolons
all parse, quoted fields included. A header row is matched on column *names*, so
order doesn't matter; without one, this order is assumed:

```
date,bet,stake,multiplier,result,returned,note
2026-09-14,Power Play 2x - Bedard 2+ SOG / Kucherov 1+ pt,25,3,win,,
2026-09-09,Flex Play 6x - Sunday slate longshot,20,25,loss,,lottery ticket
2026-09-04,Flex Play 4x - Judge 1+ HR combo,20,6,partial,30,3 of 4 hit
2026-09-18,Flex Play 4x - Monday night props,25,6,pending,,
```

**Copy the template** drops those rows straight into the box to edit.

Header names are matched loosely — `entry fee`, `amount`, `wager` and `cost` all
read as the stake; `status`, `outcome` and `result` are the same column; `payout`,
`won` and `winnings` all mean what came back. Money can carry `$` and thousands
commas. Dates take `2026-09-14`, `9/14/2026` or `Sep 14, 2026`. Multipliers take
`3`, `3x`, or American odds like `+150`.

- **result** accepts win / loss / push / partial / pending and the obvious synonyms
  (won, lost, refunded, voided, open…). Leave it blank and it's inferred from the
  money: nothing back is a loss, the full ticket is a win, something in between is
  a part-hit.
- **returned** is only needed for a flex play that part-hits — leave it blank
  otherwise. If an entry is marked won but paid less than the ticket priced, it's
  recorded as a partial at the original multiplier rather than quietly repriced, so
  the ticket keeps saying what it was worth.
- A **book** column is honoured if present, so a mixed spreadsheet can carry
  Polymarket rows too.

Rows that can't become a bet — no stake, or no multiplier *and* no payout to work
one out from — are listed in the preview with the reason, never silently dropped.

### Entries JSON

If you can get the raw payload: log in to PrizePicks in a desktop browser, open your
entry history, and in DevTools → Network click the request that returns it, then
**Copy → Copy response**. Paste that in. Your browser is already signed in, so you
never handle a token, and nothing leaves the page.

Field names are matched by alias rather than assumed, so stakes, multipliers, results
and dates come across from most payload shapes. Pick-level detail varies, so an entry
may land as its type and pick count — `Flex Play — 4 picks` — and you can rename it
afterwards like any hand-logged bet.

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

Both importers share one preview-and-apply path, and imported bets carry a key tying
them to their source row. Re-import as often as you like: bets are refreshed in
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
