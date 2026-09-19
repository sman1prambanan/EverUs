# Tabungan Digital

Prototype full-stack ringan dengan:
- HTML5
- CSS3
- Vanilla JavaScript
- Google Sheets
- Google Apps Script

## Struktur

```text
project/
├── index.html
├── login.html
├── dashboard.html
├── css/
│   ├── style.css
│   └── responsive.css
├── js/
│   ├── config.js
│   ├── api.js
│   ├── auth.js
│   └── dashboard.js
├── assets/
│   └── qris.png
└── backend/
    ├── Code.gs
    └── Seed.gs
```

## 1. Buat Google Spreadsheet

Buat spreadsheet baru. Salin Spreadsheet ID dari URL:

`https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit`

Buat 4 sheet:
- Users
- Transactions
- Sessions
- Notifications

Header persis mengikuti `Code.gs`.

## 2. Apps Script

Spreadsheet → Extensions → Apps Script.

Buat project Apps Script dan tempel:
- `backend/Code.gs`
- `backend/Seed.gs`

Di Project Settings → Script Properties tambahkan:

`SPREADSHEET_ID = ID spreadsheet Anda`

Jalankan `setupSheets()` satu kali.

Lalu jalankan `seedDemoUsers()` satu kali untuk membuat:
- usera / UserA123!
- userb / UserB123!

Ganti password sebelum penggunaan nyata.

## 3. Deploy API

Deploy → New deployment → Web app.

Atur:
- Execute as: Me
- Who has access: Anyone (untuk prototype)

Salin URL `/exec`.

Masukkan URL tersebut ke:

`js/config.js`

Contoh:
```js
window.APP_CONFIG = {
  API_URL: "https://script.google.com/macros/s/XXXX/exec",
  POLL_MS: 4000
};
```

## 4. QRIS

Simpan QRIS Anda sebagai:

`assets/qris.png`

Tidak ada QRIS asli yang disertakan dalam prototype.

## 5. Menjalankan frontend

Karena browser dapat membatasi request dari `file://`, gunakan local server.

Contoh Python:
```bash
python -m http.server 5500
```

Buka:
`http://localhost:5500/login.html`

Atau gunakan Live Server di VS Code.

## 6. Alur testing

### User B → User A
1. Login `userb`.
2. Klik Menabung.
3. QRIS terbuka dan tidak memiliki tombol close.
4. Klik Saya Sudah Membayar.
5. Masukkan Rp100.000.
6. Kirim.
7. Transaksi menjadi PENDING.
8. Login `usera`.
9. User A mendapat pending/notification.
10. Approve.
11. Saldo User B bertambah Rp100.000.

### Reject
Buat transaksi baru, lalu User A memilih Tolak.
Saldo pengirim tidak berubah.

### User A → User B
Ulangi alur dari `usera`, lalu approval dilakukan oleh `userb`.

### Anti-double approval
Setelah transaksi APPROVED, coba approve lagi.
Backend akan menolak karena status bukan PENDING. LockService juga digunakan pada operasi perubahan transaksi/saldo.

### Persistensi QRIS
Saat user membuka QRIS, status draft pembayaran disimpan server-side bila Anda menambahkan draft state pada alur produksi. Pada prototype ini, `getPendingPayment` sudah disiapkan sebagai endpoint agar mekanisme tersebut dapat dikembangkan tanpa menjadikan localStorage sebagai sumber kebenaran.

## 7. Catatan keamanan

Ini prototype, bukan sistem perbankan.

- Jangan mempercayai saldo dari frontend.
- Jangan mengubah saldo dari frontend.
- Approval selalu diverifikasi backend.
- Transaksi hanya bisa diproses saat PENDING.
- Approver diverifikasi backend.
- `LockService` mencegah dua request approval memproses transaksi bersamaan.
- Password contoh disimpan sebagai SHA-256 hash; untuk production gunakan mekanisme password hashing yang sesuai dan secret/session management yang lebih kuat.
- Jangan menganggap tombol "Saya Sudah Membayar" sebagai bukti pembayaran QRIS. Verifikasi otomatis membutuhkan payment gateway/QRIS API resmi.

## 8. Pengembangan berikutnya

Struktur frontend sengaja memusatkan komunikasi di `js/api.js`, sehingga endpoint dapat diganti dengan backend lain/WebSocket/Firebase tanpa membongkar komponen UI.

Fitur lanjutan yang cocok:
- profil
- audit log
- pagination transaksi
- session revoke/logout server-side
- rate limiting
- webhook/payment gateway
- real QRIS verification
- admin panel
- export mutasi


## Fitur tambahan v2
- Dashboard menampilkan **Saldo gabungan A + B** (jumlah saldo seluruh user aktif) dan **Tabungan pribadi** (saldo user yang login).
- Menu **Akun & Keamanan** untuk mengganti username dan/atau password. Perubahan wajib dikonfirmasi dengan password saat ini.
- Username harus unik dan 3–30 karakter; password baru minimal 8 karakter.

## v3 additions
- User A can set the live saving price from **Atur Harga**.
- Users no longer enter the saving amount manually; backend always uses the server-side active price.
- Added notification delete-one/delete-all actions and unread badge behavior.
- Added mobile hamburger drawer for navigation.
- Added lightweight loading indicator while dashboard data is fetched.
- Added `Settings` sheet automatically via `setupSheets()`; `SAVING_PRICE` is stored there.

After updating Apps Script, redeploy the Web App as a **new version**.


## V5 — optimasi performa
- Login hanya membaca data Users yang diperlukan dan menggunakan cache pendek.
- Session aktif disimpan di CacheService sehingga request dashboard tidak perlu scan Sessions setiap kali.
- Dashboard mengambil pending payment dalam request yang sama, sehingga tidak ada request kedua saat masuk dashboard.
- Polling tetap 120 detik dan tidak melakukan refresh tambahan saat tab kembali aktif.
- Cache saldo pengguna dan harga menabung di-invalidate setelah perubahan penting.
- Riwayat transaksi/notifikasi dibatasi agar response tetap ringan.

Setelah mengganti backend, deploy Web App sebagai versi baru. Tidak perlu membuat spreadsheet baru.
