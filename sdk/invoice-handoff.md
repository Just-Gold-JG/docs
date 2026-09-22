# Invoice share & download (host fill)

Opt-in callbacks for partners who must stamp the **customer name** on the tax invoice before the user shares or saves it.

**Not required for trading.** If you omit these callbacks, the SDK keeps in-SDK invoice preview and share.

Requires:

| Platform | Package | Version |
| --- | --- | --- |
| React Native | `@justgold/rn-sdk` | ^1.1.15 |
| Flutter | `justgold_sdk` | ^1.1.19 |
| Embedded UI | JustGold CDN | `1.1.15` / `latest` (sandbox: `sdk.stage.justgold.app`) |

Passing **either** `onInvoiceShare` or `onInvoiceDownload` opts the host in. Wrappers set `useHostInvoiceActions: true` on `INIT_SESSION`. The SDK still ensures the invoice exists (`GET /v1/customers/:id/transactions/:txnId/invoice`, then `POST /v1/customers/invoice` on 404), then emits the event. It does **not** preview the unfilled PDF and does **not** put PDF bytes on the WebView message.

---

## What you do

1. Receive a short-lived presigned `url` plus AcroForm field names.
2. Download the PDF **on the host** (your app / backend). Do not ask the WebView to send the file.
3. Fill the widget named `payload.form.fields.customerName` (PDF field name **`customerName`**) with the customer's full name.
4. Share or save the filled PDF.

| User action in the SDK | Event | Callback |
| --- | --- | --- |
| **Share this receipt** (invoice exists) | `INVOICE_SHARE` | `onInvoiceShare` |
| **Invoice** / **Download invoice** | `INVOICE_DOWNLOAD` | `onInvoiceDownload` |

---

## Payload

Same shape for share and download. `INVOICE_DOWNLOAD` uses `"action": "download"`.

```json
{
  "type": "INVOICE_SHARE",
  "payload": {
    "action": "share",
    "transactionId": "674a1b2c3d4e5f6789012345",
    "fileName": "invoice-#JG1A2B3C4D.pdf",
    "mimeType": "application/pdf",
    "url": "https://s3.amazonaws.com/customer-invoices/invoice-….pdf?X-Amz-Expires=…",
    "form": {
      "type": "acroform",
      "fields": {
        "customerName": "customerName"
      }
    }
  }
}
```

| Field | Type | Description |
| --- | --- | --- |
| `action` | `'share'` \| `'download'` | Which button the user tapped |
| `transactionId` | `string` | JustGold transaction id |
| `fileName` | `string` | Suggested PDF file name |
| `mimeType` | `'application/pdf'` | Always PDF |
| `url` | `string` | Short-lived HTTPS presigned URL. Download promptly |
| `form.type` | `'acroform'` | Fillable PDF |
| `form.fields.customerName` | `string` | **PDF widget name** to fill (value is `customerName`) |

Typed imports:

```ts
import type { InvoiceHostPayload } from '@justgold/rn-sdk';
```

```dart
import 'package:justgold_sdk/justgold_sdk.dart'; // InvoiceHostPayload
```

---

## React Native

```tsx
import { Share } from 'react-native';
import { JustGoldConnect, type InvoiceHostPayload } from '@justgold/rn-sdk';

async function fillAndHandoff(payload: InvoiceHostPayload, customerFullName: string) {
  const field = payload.form.fields.customerName; // "customerName"
  const response = await fetch(payload.url);
  const bytes = await response.arrayBuffer();
  const filled = await yourPdfLib.fillAcroForm(bytes, { [field]: customerFullName });

  if (payload.action === 'share') {
    await Share.share({ url: filled.fileUri, title: payload.fileName });
    return;
  }
  await yourSaveOrPresentPdf(filled.fileUri, payload.fileName);
}

<JustGoldConnect
  token={token}
  refreshToken={refreshToken}
  sandbox
  onInvoiceShare={payload => fillAndHandoff(payload, customerFullName)}
  onInvoiceDownload={payload => fillAndHandoff(payload, customerFullName)}
/>
```

`onSdkEvent` also receives `{ type: 'INVOICE_SHARE' | 'INVOICE_DOWNLOAD', payload }` if you prefer one handler.

---

## Flutter

```dart
import 'package:justgold_sdk/justgold_sdk.dart';

Future<void> fillAndHandoff(InvoiceHostPayload payload, String customerFullName) async {
  final field = payload.form.fields.customerName; // "customerName"
  final bytes = await downloadPdf(payload.url);
  final filled = await fillAcroForm(bytes, {field: customerFullName});

  if (payload.action == 'share') {
    await sharePdf(filled, payload.fileName);
    return;
  }
  await saveOrPresentPdf(filled, payload.fileName);
}

JustGoldConnect(
  token: token,
  refreshToken: refreshToken,
  sandbox: true,
  onInvoiceShare: (payload) => fillAndHandoff(payload, customerFullName),
  onInvoiceDownload: (payload) => fillAndHandoff(payload, customerFullName),
)
```

---

## Default (no callbacks)

Omit both callbacks. The SDK previews the invoice in-app and uses its own share sheet. Help `mailto:` / `tel:` / WhatsApp still use `OPEN_EXTERNAL_URL` (wrappers handle that automatically).

Do **not** send PDF base64 on custom `postMessage` / WebView handlers — large payloads are dropped on iOS.
