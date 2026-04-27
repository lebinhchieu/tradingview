---
name: trading-key-entry
description: >
  Manages trading key entries (price levels and zones) in the structured template
  format used by the key-drawer Pine Script indicator. Use this skill whenever
  the user wants to add, remove, update, or list key levels or zones for a trading
  symbol — or wants to see/copy the full template to paste into TradingView.
  Trigger on any of: "add key", "new key", "add level", "add zone", "remove key",
  "delete key", "list keys", "show template", "update key", "show keys for",
  "XAUUSD key", "BTC zone", or similar trading-level management requests.
---

## What this skill does

Maintains `keys.txt` in the project root (`/home/chieule/workspace/tradingview/keys.txt`)
as the source of truth for all key entries. After every change, print the full
template content so the user can copy-paste it into the TradingView indicator's
**Template** input field.

---

## Template format

```
SYMBOL
===
YYYY-MM-DD HH:MM
price
description line 1
description line 2
---
YYYY-MM-DD HH:MM
price1-price2
description line 1
~~~
SYMBOL2
===
YYYY-MM-DD HH:MM
price
description
```

Rules:
- `~~~` separates symbol blocks (with a blank line around it for readability)
- `===` separates the symbol name from its first key entry
- `---` separates additional key entries within the same symbol block
- `===` may also be used in place of `---` as a key separator — both are valid
- **Datetime line**: `YYYY-MM-DD HH:MM` in **UTC+7** (Asia/Bangkok time)
- **Price line**: single value (`4700`) draws a horizontal line; range (`4753-4755`) draws a filled rectangle
- **Description lines**: everything after the price — SL, TP levels, notes, etc.

---

## Datetime

Always auto-fill the current UTC+7 time unless the user explicitly provides one.
Get it with:
```bash
TZ='Asia/Bangkok' date '+%Y-%m-%d %H:%M'
```

---

## Workflow

### Add a key

1. Run the datetime command to get the current UTC+7 time.
2. Read `keys.txt` (treat as empty string if the file doesn't exist).
3. Find the symbol block for the target symbol (case-insensitive match against the
   first line of each `~~~`-separated block).
4. If the symbol block exists, append a `---` separator and the new key entry at
   the end of that block.
5. If the symbol block doesn't exist, append a `~~~` separator (if there's already
   content) followed by a new symbol block.
6. Write the updated content back to `keys.txt`.
7. Print the full template.

### Remove a key

1. Read `keys.txt`.
2. Identify the key to remove by matching its price and/or datetime against what
   the user described. Ask for clarification if ambiguous.
3. Remove that key block (the content between its surrounding `---`/`===`
   separators). If it was the only key for a symbol, remove the entire symbol block
   including the `~~~` separator.
4. Write back and print the full template.

### List keys for a symbol

Parse the symbol's block and print each key with its index, datetime, price, and
first description line — a compact summary so the user can refer to them by number
for remove/update operations.

### Show full template

Just read `keys.txt` and print it. If the file is empty or missing, tell the user.

### Update a key

Identify the key (ask if ambiguous), replace only the fields the user wants to
change, write back, print the full template.

---

## Output format

After any write operation, always end your response with:

```
Keys template updated. Full template to paste into TradingView:

─────────────────────────────────
<full contents of keys.txt>
─────────────────────────────────
```

This makes it trivial for the user to copy the whole thing.

---

## Example

User: "add key for XAUUSD, sell zone 4753-4755, SL 4758, TP1 4750, TP2 4747, TP3 4744"

Resulting key entry appended to the XAUUSD block:

```
2026-04-27 08:30
4753-4755
SL : 4758
💰TP 1: 4750
💰TP 2: 4747
💰TP 3: 4744
```
