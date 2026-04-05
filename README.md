# lenskart-accounting
# Lenskart → Tally Prime Import Tool

A standalone, browser-based tool that converts Lenskart franchise accounting data into **Tally Prime Excel import format**. No installation, no server, no npm — just open the HTML file in any modern browser.

Built for **SG Traders** (GSTIN: 32DBNPG5619A1Z1), Kottarakkara, Kerala — a Lenskart franchise operating under the GST margin scheme.

---

## Features

- Processes 4 source files in one click — GSTR-1, GSTR-2B, Lenskart Wallet CSV, Cash Book
- Outputs Tally Prime official Excel format (matches Tally's own sample template column headers exactly)
- Handles both revenue streams: B2C walk-in margin sales and B2B online commission orders
- Splits GSTR-2B credit notes (Purchase Return) and debit notes (Purchase - Debit Note) into separate voucher types
- Computes TDS u/s 194H split by Lenskart Haryana and Rajasthan entities
- Timezone-safe date handling (IST-safe Excel serial to DD-MM-YYYY conversion)
- Per-section preview + individual download + combined workbook download
- Ledger Master sheet included in every download for first-time Tally setup

---

## Business Model

SG Traders operates as a Lenskart franchise with two income streams:

**Stream 1 — B2C Walk-in Sales (GST Margin Scheme)**
- SG raises margin invoice to customer under SG's GSTIN → appears in GSTR-1 B2C
- Lenskart charges product cost to SG → appears in GSTR-2B as purchase invoice
- Lenskart reimburses SG's margin via wallet → `COLLECTION_REIMBURSEMENT`
- Net: SG retains ~30% margin, Lenskart retains ~70% product cost

**Stream 2 — B2B Commission (Online Orders)**
- Customer orders on Lenskart app, Lenskart invoices customer directly
- SG raises commission invoice to Lenskart → appears in GSTR-1 B2B
- Lenskart credits commission to wallet → `ORDER_COMMISSION`

**Cash Flow**
- All customer payments collected at store (cash + digital via Lenskart POS)
- Cash collected → `Dr Cash / Cr Lenskart Wallet`
- SG remits Lenskart's share back → `INTER_WALLET_TRANSFER`
- Commission bank payout → `Dr Lenskart Payout Receivable / Cr Lenskart Wallet` (bank side booked separately during bank reconciliation)

---

## How to Use

1. **Download** `lenskart_tally_tool_v3.html`
2. **Open** it in Chrome, Edge, or Firefox (double-click the file)
3. **Fill in** Capital Account name and Bank Account name (asked each run)
4. **Upload** the four source files:
   - GSTR-1 Workings `.xlsx` (e.g. `250508_GSTR-1_Workings_April_SG.xlsx`)
   - GSTR-2B `.xlsx` (e.g. `042025_32DBNPG5619A1Z1_GSTR2B_16052025.xlsx`)
   - Lenskart Wallet CSV (e.g. `walletTransactionHistory2025-04-01to2025-04-30.csv`)
   - Cash Book `.xlsx` (e.g. `cash_april.xlsx`)
5. **Click** Generate Tally Import Files
6. **Preview** each section, then download individually or as a combined workbook
7. **Import** into Tally Prime via `Gateway of Tally → Import → Data → Accounting Vouchers`

---

## Output Sections

| Section | Tally Voucher Type | Source |
|---|---|---|
| B2B Sales | Sales | GSTR-1 B2B sheet |
| B2B Credit Notes | Credit Note | GSTR-1 B2B CDNR sheet |
| B2C Sales | Sales | GSTR-1 B2C Invoices sheet |
| B2C Credit Notes | Credit Note | GSTR-1 B2C return sheet |
| Purchases (GSTR-2B) | Purchase | GSTR-2B B2B sheet |
| GSTR-2B Credit-Debit Notes | Debit Note / Purchase | GSTR-2B B2B-CDNR sheet |
| Cash Book | Receipt / Payment | Cash Book 2025 CR sheet |
| Wallet Journal | Journal / Receipt | Wallet CSV |
| TDS Entry | Journal | Computed from GSTR-1 B2B |
| Month-end Set-off | Journal | Placeholder — fill from Trial Balance |

---

## Journal Entry Logic

### B2C Sales
```
Dr  B2C Customers              (invoice value)
Cr  Sales - B2C Kerala         (taxable value)
Cr  Output CGST / SGST / IGST  (tax)
```

### B2B Commission
```
Dr  Lenskart Solutions Pvt Ltd - Haryana/Rajasthan  (invoice value)
Cr  Sales - Commission (B2B)                         (taxable value)
Cr  Output IGST                                      (tax)
```

### GSTR-2B Purchase
```
Dr  Purchase - Lenskart (B2B)   (taxable value)
Dr  Input IGST / CGST / SGST    (tax)
Cr  Lenskart Solutions H/R      (invoice value)
```

### GSTR-2B Credit Note (supplier gives discount) → Tally Debit Note
```
Dr  Lenskart Solutions H/R         (note value)
Cr  Purchase Return - Lenskart     (taxable value)
Cr  Input IGST                     (tax)
```

### GSTR-2B Debit Note (supplier charges extra) → Tally Purchase
```
Dr  Purchase - Debit Note (Lenskart)  (taxable value)
Dr  Input IGST                         (tax)
Cr  Lenskart Solutions H/R             (note value)
```

### Wallet — COLLECTION_REIMBURSEMENT (closes B2C debtors)
```
Dr  Lenskart Wallet A/c   (amount)
Cr  B2C Customers         (amount)
```

### Wallet — ORDER_COMMISSION (closes B2B debtors)
```
Dr  Lenskart Wallet A/c              (amount)
Cr  Lenskart Solutions - Haryana     (amount)
```

### Wallet — COMMISSION_BANK_PAYOUT
```
Dr  Lenskart Payout Receivable   (amount)
Cr  Lenskart Wallet A/c          (amount)

-- Book bank side separately when reconciling bank statement:
Dr  Federal Bank Current A/c     (amount)
Cr  Lenskart Payout Receivable   (amount)
```

### TDS u/s 194H (split by entity)
```
Dr  TDS Receivable - Lenskart (194H)       (2% of net Haryana B2B taxable)
Cr  Lenskart Solutions Pvt Ltd - Haryana

Dr  TDS Receivable - Lenskart (194H)       (2% of net Rajasthan B2B taxable)
Cr  Lenskart Solutions Pvt Ltd - Rajasthan
```
*Base = GSTR-1 B2B taxable gross less B2B CDNR credit notes, per entity*

### Month-end Set-off (manual — fill from Trial Balance)
```
Dr  Lenskart Wallet A/c                    (Haryana closing balance)
Cr  Lenskart Solutions Pvt Ltd - Haryana

Dr  Lenskart Wallet A/c                    (Rajasthan closing balance)
Cr  Lenskart Solutions Pvt Ltd - Rajasthan
```

---

## Ledger Master

Every downloaded file includes a **Ledger Master** sheet listing all ledger names with their Tally Groups. Create these ledgers in Tally before importing for the first time.

| Ledger | Tally Group |
|---|---|
| Lenskart Solutions Pvt Ltd - Haryana | Sundry Debtors |
| Lenskart Solutions Pvt Ltd - Rajasthan | Sundry Debtors |
| B2C Customers | Sundry Debtors |
| Lenskart Payout Receivable | Current Assets |
| Lenskart Wallet A/c | Current Assets |
| TDS Receivable - Lenskart (194H) | Current Assets |
| Sales - Commission (B2B) | Sales Accounts |
| Sales - B2C Kerala | Sales Accounts |
| Output IGST / CGST / SGST | Duties & Taxes |
| Input IGST / CGST / SGST | Duties & Taxes |
| Purchase - Lenskart (B2B) | Purchase Accounts |
| Purchase Return - Lenskart | Purchase Accounts |
| Purchase - Debit Note (Lenskart) | Purchase Accounts |
| Margin Adjustment - Lenskart | Indirect Income |
| Employee Training Expense | Indirect Expenses |
| EDC Charges | Indirect Expenses |
| Petty Cash Expenses | Indirect Expenses |
| Cash | Cash-in-Hand |
| Federal Bank Current A/c | Bank Accounts |
| Capital Account | Capital Account |

Non-Lenskart GSTR-2B suppliers (MBM Communications, Zoho, etc.) are auto-detected and added to the Ledger Master as **Sundry Creditors** with their exact trade name from GSTR-2B.

---

## Source File Requirements

| File | Sheet Name Required | Key Columns Used |
|---|---|---|
| GSTR-1 Workings | `B2B ` (with trailing space), `B2B CDNR`, `B2C Invoices`, `B2C return` | GSTIN, Invoice No, Date, Taxable Value, Tax |
| GSTR-2B | `B2B`, `B2B-CDNR` | GSTIN, Trade Name, Invoice No, Date, Note Type, Values |
| Wallet CSV | — (header row required) | `transaction_type`, `operation_type`, `amount`, `created_at` |
| Cash Book | `2025 CR` | col A=Date, col C=Cash, col J=Deposit, col K=Expense, col L=Drawings |

---

## Technical Details

**Framework:** Vanilla HTML5 + JavaScript — no build step, no npm

**Libraries (loaded via CDN):**
- [SheetJS 0.18.5](https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js) — reads and writes Excel files
- [PapaParse 5.4.1](https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js) — parses wallet CSV
- [Plus Jakarta Sans + JetBrains Mono](https://fonts.google.com) — UI typography

**Date handling:** Excel serial numbers are converted to DD-MM-YYYY using pure UTC arithmetic `(serial − 25569) × 86400000` — bypasses browser timezone entirely, safe for IST (UTC+5:30)

**GSTIN detection:** Simple two-part check — starts with 2 digits AND exactly 15 characters — avoids brittle regex, handles all GSTIN formats correctly

**Dr = Cr enforcement:** First ledger entry amount in each voucher is auto-computed as the balancing figure — every voucher is guaranteed balanced

**Output format:** Tally Prime official Excel multi-line format — one row per Dr/Cr entry, first row of each voucher carries Date/Type/Number/Narration/Reference, subsequent rows are blank in those columns

---

## Known Limitations

- Month-end set-off entries are **placeholders with amount ₹1** — you must replace with actual closing balances from the Trial Balance before importing that section
- COMMISSION_BANK_PAYOUT books to `Lenskart Payout Receivable` only — bank side must be entered manually when reconciling bank statement
- All wallet ORDER_COMMISSION credits to Lenskart Haryana by default — wallet CSV does not carry GSTIN-level split
- Non-Lenskart supplier purchase ledger names default to trade name from GSTR-2B — rename in Tally after first import if needed

---

## File Structure

```
lenskart-tally-tool/
│
├── lenskart_tally_tool_v3.html   ← The entire tool (single file)
└── README.md                     ← This file
```

---

## License

Private use only. Built for SG Traders, Kottarakkara, Kerala.
Not licensed for redistribution or commercial use.t 
