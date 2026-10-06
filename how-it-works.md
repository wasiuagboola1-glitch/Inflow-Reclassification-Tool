# How it works

## Decision order for every bank line

1. **Eligibility** - only credits whose remark starts with a prefix on `Config` are processed. Anything else is `SKIP`.
2. **Policy number** - every 10-digit number in the remark is checked against `Policy_Map` (manual exceptions) and then `Policy_Data`.
   The product on the policy record is turned into a ledger through the alias list on the `Ledgers` tab, or taken from the optional ledger column.
3. **Holder name** - if the remark has no policy number, the whole holder name is searched in `Policy_Data`.
   It is posted only when every record with that name resolves to the same ledger, and is labelled *Holder name match - verify*.
4. **Product in remark** - otherwise the product name in the remark is matched to the alias list.
   The **longest** alias found wins. If two different ledgers tie, the line is held.
5. **Nothing found** - status `REVIEW`; the Basis column explains why.

## Safeguards

| Situation | What the tool does |
|---|---|
| Policy says ledger A, remark product says ledger B | holds the line (`Config!H10` can switch to "policy wins") |
| Two aliases of equal length point to different ledgers | holds the line, asks for a more specific alias |
| Holder name matches policies on different ledgers | holds the line |
| Policy in remark but not in the database, product known | posts by product and adds a note to add the policy |
| Holder name used | posts but labels it for verification |

## Posting cases

- **Different entity from the bank** - intercompany: two Master rows and four Details rows per line
  (bank Dr, intercompany Cr, intercompany Dr, policy Cr).
- **Same entity as the bank** - direct: one Master row and two Details rows (bank Dr, policy Cr).

## Controls (Checks tab)

- Bank credits = posted + needs review + skipped
- Details debits = Details credits
- Master total = direct amounts + 2 x intercompany amounts
- Autocredit rows and amounts = Items rows and amounts
- Bank ledger and intercompany accounts present on `Config`

## Extending the tool

- **New product or spelling** - add an alias (UPPERCASE, no spaces) and ledger on `Ledgers`. The sorted list updates itself.
- **New policies** - add rows to `Policy_Data` (columns A:C). Policy numbers are best stored as text.
- **New entity** - add it to the `Config` entities table and the intercompany table.
- **New remark prefix** - add it to the prefix list on `Config`.
