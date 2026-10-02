# Finn Sawmill US (demo) API

Invented demonstration data made by Quarri. Finn Sawmill US is not a real business; no figure describes a real
company.

The branch's order system, served as static JSON files. No key is needed.

**Base address:** `https://theoquarri.github.io/finn-sawmill-api` (this folder is the repository root; `.nojekyll`
makes GitHub Pages serve every file as it is). Data to Sunday 27 Sep 2026; generated 28 Sep 2026, 03:30 UTC.

## Resources

| Resource | Path | Page size | Incremental by |
|---|---|---|---|
| Products | `/products/` | 100 | (small: read in full) |
| Customers | `/customers/` | 100 | (small: read in full) |
| Vendors | `/vendors/` | 100 | (small: read in full) |
| Invoices, with their lines | `/invoices/` | 200 | `updatedAt` |
| Purchase receipts | `/receipts/` | 100 | `updatedAt` |

## Paging and "updated since"

Each resource has an index, `/<resource>/index.json`:

```json
{"resource": "invoices", "generated_at": "2026-09-28T03:30:00Z", "page_size": 200, "records": 4229,
 "pages": [{"href": "page-0001.json", "count": 200, "updated_from": "2024-10-01T03:08:04Z", "updated_to": "2024-11-06T03:10:04Z"}, ...]}
```

Records are in `updatedAt` order. To read only what changed since a time T, skip the pages whose `updated_to` is
before T and keep the records with `updatedAt` at or after T (a live system would take `?updated_since=T`; static
files cannot, so the index carries each page's range).

Each page, `/<resource>/page-NNNN.json`:

```json
{"resource": "invoices", "page": 1, "count": 200, "data": [ ... ], "next": "page-0002.json"}
```

`next` is null on the last page. As with offset paging on a live system, a record can appear on two pages (when
records are added while a client pages): every fourth invoices page ends with the first two invoices of the next.
Clients should keep one copy per `invoiceNo`.

## Fields

**Products:** `itemCode`, `description`, `family` (DIM dimension lumber, BRD boards, PLY plywood), `species` (SPR-EU
European spruce, PIN-EU European pine, SYP southern yellow pine), `nominal` (2x4 and so on), `lengthFt`, `grade`,
`uom` (MBF thousand board feet, or MSF thousand square feet; written in either case), `basePriceUsd` (the price list's
base, September 2024), `defaultVendor`, `activeFlag` (Y or N), `updatedAt`.

**Customers:** `custNo` (five digits, as text, with leading zeros), `custName`, `state`, `creditLimit` (text, with a
dollar sign and commas), `terms` (NET15, NET30, NET45), `activeFlag`, `updatedAt`.

**Vendors:** `vendorCode`, `vendorName`, `updatedAt`. FINN-FI is the Finnish mill (intercompany).

**Invoices:** `invoiceNo`, `docType` (INV, or CM for a credit memo, whose quantities and amounts are negative),
`status` (POSTED or VOID), `invoiceDate` (MM/DD/YYYY), `custNo`, `updatedAt`, and `lines`: `lineNo`, `itemCode`,
`qty`, `qtyUom` (MBF, MSF, or BF when a line was keyed in board feet), `unitPrice` (per `qtyUom`, as text),
`extAmountCents` and `costAmountCents` (whole cents).

**Receipts:** `poNo`, `receiptTs` (ISO 8601, UTC, with a Z), `vendorCode`, `itemCode`, `qtyReceived`, `uom`,
`unitCostUsd`, `freightUsd`, `currency` (USD), `updatedAt`.
