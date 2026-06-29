# Reconciliation

JustGold generates daily reconciliation files containing all transactions settled during the day. Files are produced once per day at midnight (Asia/Dubai time) and cover all transactions with status `Completed`, `Pending` or `Failed` recorded up to that point.

Three separate CSV files are produced per run — one for each transaction type:

| File | Contents |
| --- | --- |
| `buy_YYYY-MM-DD.csv` | All gold and silver buy transactions |
| `sell_YYYY-MM-DD.csv` | All gold and silver sell transactions |
| `delivery_YYYY-MM-DD.csv` | All physical delivery transactions |

---

## Delivery methods

### Portal download

Files are available for download from the JustGold partner portal under **Reports → Reconciliation**. Files are retained for 30 days.

### SFTP push

JustGold can push files automatically to your SFTP server immediately after each nightly run. To configure SFTP delivery, provide the following details to your JustGold account manager:

| Setting | Description |
| --- | --- |
| Host | SFTP server hostname or IP address |
| Port | SFTP port (default: `22`) |
| Username | SFTP login username |
| Public key | Your server's SSH public key for key-based authentication |
| Remote path | Target directory for file delivery (e.g. `/reconciliation/justgold/`) |

Once configured, files are deposited into the remote path using the standard naming convention above. Existing files are never overwritten — each day's file has a unique date suffix.

---

## File format

All files are UTF-8 encoded CSV with a header row. String fields that may contain commas are quoted. Numeric fields are unquoted decimal strings with full precision. Dates are ISO 8601 in UTC.

---

## Buy file columns

| Column | Type | Description |
| --- | --- | --- |
| `transactionId` | string | JustGold transaction identifier |
| `customerIdentifier` | string | Partner-scoped customer identifier |
| `metal` | string | `Gold` or `Silver` |
| `status` | string | `Completed` or `Failed` |
| `quantity` | decimal | Metal quantity in grams |
| `currency` | string | Settlement currency (e.g. `AED`) |
| `quotedPrice` | decimal | Buy price per gram at the time of the quote |
| `total` | decimal | Pre-fee total (`quantity × quotedPrice`) |
| `vat` | decimal | VAT amount |
| `platformFee` | decimal | Platform fee applied |
| `grandTotal` | decimal | Final total including all fees |
| `paymentReference` | string | Partner payment reference, if provided |
| `paymentMethod` | string | Payment method, if provided |
| `createdAt` | datetime | Transaction creation timestamp (UTC) |
| `updatedAt` | datetime | Last status update timestamp (UTC) |

### Sample

```csv
transactionId,customerIdentifier,metal,status,quantity,currency,quotedPrice,total,vat,platformFee,grandTotal,paymentReference,paymentMethod,createdAt,updatedAt
682710cc3f1b2c7a9d5e1111,cust-10293,Gold,Completed,0.1794,AED,557.36,100.00,0.00,2.00,102.00,pay_3f8a9c1b2d,Card,2026-06-28T20:10:00.000Z,2026-06-28T20:12:45.000Z
682710cc3f1b2c7a9d5e2222,cust-10401,Silver,Completed,5.0000,AED,32.10,160.50,0.00,3.21,163.71,pay_7a1b0c2e3f,BankTransfer,2026-06-28T21:05:00.000Z,2026-06-28T21:08:30.000Z
```

---

## Sell file columns

| Column | Type | Description |
| --- | --- | --- |
| `transactionId` | string | JustGold transaction identifier |
| `customerIdentifier` | string | Partner-scoped customer identifier |
| `metal` | string | `Gold` or `Silver` |
| `status` | string | `Completed` or `Failed` |
| `quantity` | decimal | Metal quantity sold in grams |
| `currency` | string | Settlement currency |
| `quotedPrice` | decimal | Sell price per gram at the time of the quote |
| `total` | decimal | Pre-fee total (`quantity × quotedPrice`) |
| `vat` | decimal | VAT amount |
| `platformFee` | decimal | Platform fee applied |
| `grandTotal` | decimal | Final total including all fees |
| `paymentReference` | string | Partner payment reference, if provided |
| `paymentMethod` | string | Payment method, if provided |
| `createdAt` | datetime | Transaction creation timestamp (UTC) |
| `updatedAt` | datetime | Last status update timestamp (UTC) |

### Sample

```csv
transactionId,customerIdentifier,metal,status,quantity,currency,quotedPrice,total,vat,platformFee,grandTotal,paymentReference,paymentMethod,createdAt,updatedAt
682710cc3f1b2c7a9d5e3333,cust-10293,Gold,Completed,0.5000,AED,540.00,270.00,0.00,5.40,275.40,pay_1d2e3f4a5b,Card,2026-06-28T19:30:00.000Z,2026-06-28T19:33:10.000Z
```

---

## Delivery file columns

| Column | Type | Description |
| --- | --- | --- |
| `transactionId` | string | JustGold transaction identifier |
| `customerIdentifier` | string | Partner-scoped customer identifier |
| `status` | string | `Completed` or `Failed` |
| `currency` | string | Settlement currency |
| `goldQuantity` | decimal | Total gold quantity redeemed in grams (`0` if none) |
| `goldVaultUsed` | decimal | Gold grams sourced from the customer's vault |
| `goldPurchased` | decimal | Gold grams purchased to fulfil the delivery |
| `goldMintingCharges` | decimal | Total minting charges for gold items |
| `goldServiceCharges` | decimal | Total service charges for gold items |
| `silverQuantity` | decimal | Total silver quantity redeemed in grams (`0` if none) |
| `silverVaultUsed` | decimal | Silver grams sourced from the customer's vault |
| `silverPurchased` | decimal | Silver grams purchased to fulfil the delivery |
| `silverMintingCharges` | decimal | Total minting charges for silver items |
| `silverServiceCharges` | decimal | Total service charges for silver items |
| `total` | decimal | Pre-fee subtotal |
| `vat` | decimal | VAT amount |
| `platformFee` | decimal | Platform fee applied |
| `grandTotal` | decimal | Final total including all fees |
| `paymentReference` | string | Partner payment reference, if provided |
| `paymentMethod` | string | Payment method, if provided |
| `createdAt` | datetime | Transaction creation timestamp (UTC) |
| `updatedAt` | datetime | Last status update timestamp (UTC) |

### Sample

```csv
transactionId,customerIdentifier,status,currency,goldQuantity,goldVaultUsed,goldPurchased,goldMintingCharges,goldServiceCharges,silverQuantity,silverVaultUsed,silverPurchased,silverMintingCharges,silverServiceCharges,total,vat,platformFee,grandTotal,paymentReference,paymentMethod,createdAt,updatedAt
682710cc3f1b2c7a9d5e4444,cust-10293,Completed,AED,10.0000,8.0000,2.0000,25.00,10.00,0.0000,0.0000,0.0000,0.00,0.00,650.00,0.00,0.00,685.00,pay_9c8d7e6f5a,Card,2026-06-28T18:00:00.000Z,2026-06-28T18:04:20.000Z
```

---

## Notes

- Files include transactions from all customers across your organization hierarchy.
- A transaction appears in exactly one file, determined by its type (`Buy`, `Sell`, or `Delivery`).
- `Failed` transactions are included so your records account for attempted activity. Verify `status` before posting to your ledger.
- `quantity` and all charge/fee columns are decimal strings with full precision. Use an arbitrary-precision library for any arithmetic — do not parse with native floating-point types.
- If no transactions occurred for a given type on a given day, an empty file (header row only) is still produced to confirm the run completed.
- Reconciliation runs at **00:05 Asia/Dubai time**. Files are available shortly after that time.
