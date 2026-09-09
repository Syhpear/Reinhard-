# Bot Control Panel — versi nyata (mineflayer)

Panel HTML yang kamu upload sebelumnya 100% simulasi: tombol "Connect" cuma
mengubah state di JavaScript, daftar pemain di-random, tidak ada koneksi
jaringan sama sekali. Paket ini menambahkan **backend Node.js** yang memakai
[mineflayer](https://github.com/PrismarineJS/mineflayer) untuk benar-benar
join ke server Minecraft, plus menyambungkan panel HTML-nya ke backend itu
lewat WebSocket supaya tombol-tombolnya mengendalikan bot sungguhan.

## Kenapa perlu backend terpisah?

Browser tidak bisa membuka koneksi TCP mentah ke server Minecraft (itu
dilarang oleh sandbox browser). Jadi arsitekturnya:

```
[ Browser: bot-control-panel.html ]  <--WebSocket-->  [ server.js (Node) ]  <--protokol Minecraft-->  [ Server Minecraft ]
```

`server.js` yang memegang koneksi Minecraft asli (lewat mineflayer);
HTML-nya cuma dashboard yang mengirim perintah & menampilkan status.

## Struktur file

```
bot-panel/
├── package.json
├── server.js              <- backend: WebSocket + mineflayer
├── README.md               <- file ini
└── public/
    └── bot-control-panel.html  <- panel yang sudah disambungkan ke backend
```

Simpan persis dengan struktur folder ini (jangan pindahkan HTML-nya keluar
dari folder `public/`), karena `server.js` mengambilnya dari situ.

## Cara menjalankan

1. Install [Node.js](https://nodejs.org) versi 18 ke atas kalau belum ada.
2. Buka terminal di folder `bot-panel/`, lalu:
   ```
   npm install
   node server.js
   ```
3. Buka `http://localhost:8080` di browser. **Jangan** dobel-klik file
   HTML-nya langsung — WebSocket-nya cuma nyambung kalau dibuka lewat
   server ini.
4. Isi IP server, port, nickname, dan password AuthMe (kosongkan kalau
   server tujuan tidak pakai AuthMe), lalu klik **Connect**.

Backend ini tetap jalan (bot tetap online) selama proses `node server.js`
tetap hidup, walau tab browsernya kamu tutup/refresh. Untuk mematikan bot
beneran, klik **Disconnect** di panel atau hentikan prosesnya (Ctrl+C di
terminal).

## Fitur yang sudah nyata

- Connect/Disconnect ke server Minecraft manapun (offline-mode + AuthMe).
- Auto Respawn, Anti-AFK, Auto Fishing, Auto Attack (interval bisa diatur)
  — semuanya benar-benar jalan lewat mineflayer.
- Auto Attack & Auto Aim sengaja **cuma menyasar mob/hewan, tidak pernah
  menyasar pemain lain** — sesuai deskripsi fitur di panel aslinya.
- WASD + Jump + Sprint + Klik Kiri (serang mob / gali blok di depan) +
  Klik Kanan (pakai item / interaksi blok) — mengirim kontrol asli ke bot.
- Tombol WASD fisik di keyboard menggerakkan **semua bot yang terhubung
  sekaligus** (mode kendalikan banyak bot bareng), sama seperti niat desain
  panel aslinya — sekarang beneran jalan, bukan cuma efek visual.
- Player Online, Health/Hunger, Kill Log, Death Log, dan riwayat Chat kini
  menampilkan data asli dari server, bukan data acak.
- Auto Aim & Auto /back tetap dikunci di belakang badge "Premium" persis
  seperti desain awal — backend-nya sudah siap, tinggal kamu buka gerbang
  premium-nya sendiri kalau mau.

## Simplifikasi yang perlu kamu tahu

- Panel ini tidak punya tampilan dunia (tidak ada first-person view), jadi
  "Klik Kanan (Pasang Blok)" disederhanakan: kalau bot sedang menghadap
  sebuah blok, itu akan diinteraksi (buka pintu/peti/dsb); kalau tidak,
  item di tangan akan dipakai (activateItem). Penempatan blok presisi
  (memilih sisi blok) belum diimplementasikan — bisa ditambah nanti dengan
  plugin seperti `mineflayer-pathfinder` kalau kamu perlu.
- Command login AuthMe ditebak otomatis (`/login <password>`, lalu
  `/register <password> <password>` kalau server bilang belum terdaftar).
  Kalau plugin auth di server tujuanmu pakai command berbeda, sesuaikan di
  `server.js` bagian `bot.chat('/login ' + password)`.
- Auto-detect versi Minecraft otomatis (default mineflayer). Kalau gagal
  connect karena error terkait versi protokol, coba set versi manual di
  `server.js`: `mineflayer.createBot({ ..., version: '1.20.1' })`.
- Login akun premium/Microsoft belum ada UI-nya (panel ini didesain untuk
  server offline-mode + AuthMe). Kalau perlu, ganti `auth: 'offline'` jadi
  `auth: 'microsoft'` di `server.js`, kosongkan field password, lalu pantau
  terminal — mineflayer akan menampilkan link + kode login Microsoft di sana.

## Catatan keamanan

Panel ini **tidak punya login/otentikasi sendiri** — siapa pun yang bisa
mengakses `http://alamat-kamu:8080` bisa mengendalikan semua bot yang
tersambung. Aman dipakai di `localhost` (komputer sendiri). Kalau mau
diakses dari jarak jauh (misalnya di-hosting di VPS), taruh di belakang
reverse proxy dengan autentikasi (basic auth di Nginx/Caddy) atau VPN —
jangan expose port 8080 langsung ke internet publik.

Perhatikan juga aturan server Minecraft yang kamu sambungi — sebagian
server melarang bot/multi-akun di rules mereka. Pastikan kamu memakai ini
di server milik sendiri atau di server yang memang mengizinkan bot.

## Troubleshooting singkat

| Gejala | Kemungkinan sebab |
|---|---|
| Toast "Backend belum terhubung" terus-menerus | `node server.js` belum jalan, atau kamu membuka file HTML-nya langsung (bukan lewat `http://localhost:8080`) |
| Gagal connect, error soal versi | Set `version` manual di `mineflayer.createBot(...)` pada `server.js` |
| Bot ke-kick langsung setelah join | Cek command login AuthMe di server tujuan, sesuaikan di `server.js` |
| Auto Fishing tidak jalan | Pastikan ada fishing rod di inventory bot sebelum menyalakan fitur ini |
