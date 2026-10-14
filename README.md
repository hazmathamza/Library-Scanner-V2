# Self Checkout

One scanner for the library and the school store. The cashier opens this page on a computer with a webcam, picks a mode at the top, and students scan their ID barcodes.

- **Library** — check books out and in, add books, see active checkouts and overdue items. The camera feed is shown on the page.
- **Store** — the page sits blank on "Ready to scan" with the camera running hidden in the background. A student holds up their ID, their name and photo appear, the cashier taps items, confirms, and the sale is recorded. No sign-in.

## How it works

One ZXing barcode reader runs continuously. Every scan is routed by the active mode:

- Library → student lookup → check out (selected book) or check in (pick books to return).
- Store → student lookup → item picker → confirm purchase.

Repeat scans of the same ID are ignored for a few seconds so one presentation of an ID can't create duplicate checkouts or purchases.

## Data (Firestore, project `library-system-61e86`)

| Collection | Document ID | Fields |
|---|---|---|
| `students` | barcode value on the student ID | `name`, `grade`, `photoUrl` (optional), `photoPath` (optional, Firebase Storage path) |
| `books` | auto-ID | `title`, `author`, `genre`, `total_copies`, `available_copies` |
| `checkouts` | auto-ID | `student_id`, `book_id`, `checkout_date`, `due_date` (checkout + 14 days), `return_date` (`null` = still out) |
| `storePurchases` | auto-ID | `studentId`, `studentName`, `items` (`[{id, name, price, qty}]`), `total`, `createdAt` |

Student photos: if a student document has `photoUrl` (or `photo` / `imageUrl`), the Store mode shows it. Otherwise it shows the student's initials. No photo on file yet — the system works fine without one.

Store items are the `STORE_ITEMS` array at the top of the `<script>` in `index.html`. Edit names/prices there.

## Deploy

Serve over **HTTPS** (the browser only allows the webcam on a secure origin): GitHub Pages (Settings → Pages → Deploy from branch `main`) or any HTTPS static host. `localhost` works for testing.

## Files

- `index.html` — the whole app: Library + Store, scanner, Firebase calls
