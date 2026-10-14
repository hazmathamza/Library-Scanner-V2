# Self Checkout

One scanner for the library and the school store. The cashier opens this page on a computer with a webcam, picks a mode at the top, and students scan their ID barcodes.

- **Library** — check books out and in, add books, see active checkouts. The camera feed is shown on the page.
- **Store** — the page sits blank on "Ready to scan" with the camera running hidden in the background. A student holds up their ID, their name and photo appear, the cashier taps items, confirms, and the sale is recorded. No sign-in.

## How it works

One ZXing barcode reader runs continuously. Every scan is routed by the active mode:

- Library → student lookup → check out (selected book) or check in (pick books to return).
- Store → student lookup → item picker → confirm purchase.

Repeat scans of the same ID are ignored for a few seconds so one presentation of an ID can't create duplicate checkouts or purchases.

## Data (Firestore, project `library-system-61e86`)

Matched to the live database — the code follows the database, not the other way around.

| Collection | Document ID | Fields |
|---|---|---|
| `students` | mixed: some use the barcode as the doc ID, others use an auto-ID | `barcode` or `barcode_value` (the scanned value), `name`, `email`. Optional: `photoUrl` (or `photo` / `imageUrl`), `photoPath`. No photo on file yet — the card shows initials until one is added. |
| `books` | barcode string (e.g. the number on the book) | `title`, `author`, `barcode`, `genre` (array), `total_copies`, `available_copies`, `type`, plus catalog fields (`isbn`, `dewey`, `publisher`, `description`, …) |
| `checkouts` | auto-ID | `book_id`, `book_isbn`, `book_title`, `checkout_date` (ISO string), `returned` (boolean), `returned_at` (ISO string or null), `student_email`, `student_id`, `student_name`. One record per book. No due dates in this database. |
| `storePurchases` | auto-ID | `studentId`, `studentName`, `studentEmail`, `items` (`[{id, name, price, qty}]`), `total`, `createdAt` |
| `activity_log` | auto-ID | `action`, `details`, `timestamp` — best-effort audit trail written on checkouts, check-ins, and store purchases |

Student lookup tries three keys in order: document ID, then `barcode_value`, then `barcode` — so it finds students regardless of which keying the record uses.

Photos: Cloud Storage is **not** provisioned on this project, so photos must come from a public `photoUrl` on the student record. `photoPath` is attempted if present, but will fail gracefully until Storage is enabled.

Other collections in the same project (`users`, `field_trips`, `field_trip_buses`, `field_trip_roster`, `student_locations`, `activity_log`) belong to other features and are left untouched, except `activity_log` which this app appends to.

## Deploy

Serve over **HTTPS** (the browser only allows the webcam on a secure origin): GitHub Pages (Settings → Pages → Deploy from branch `main`) or any HTTPS static host. `localhost` works for testing. Firebase Hosting is not set up on this project.

Store items are the `STORE_ITEMS` array in `index.html`. Edit names/prices there.

## Files

- `index.html` — the whole app: Library + Store, scanner, Firebase calls
