# 🎯 Tebak Angka

> Game tebak angka bergaya **terminal CLI Linux** — latar hitam, teks hijau, murni JavaScript.
> **100% offline**, tanpa library, tanpa CDN, tanpa asset eksternal.

![JavaScript](https://img.shields.io/badge/JavaScript-Vanilla-yellow?style=flat-square&logo=javascript)
![Offline](https://img.shields.io/badge/Offline-100%25-brightgreen?style=flat-square)
![License](https://img.shields.io/badge/License-Unlicense-blue?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-Browser%20%7C%20Node.js-informational?style=flat-square)

Satu file, dua mode:

- 🌐 **Browser** (desktop & mobile) → simpan sebagai `.html`
- 💻 **Terminal** (Node.js) → rename jadi `.js`

---

## 📑 Daftar Isi

- [Fitur](#-fitur)
- [Cara Menjalankan](#-cara-menjalankan)
- [Cara Bermain](#-cara-bermain)
- [Sistem Save Game](#-sistem-save-game)
- [Progresi Level](#-progresi-level)
- [Catatan Mobile](#-catatan-mobile)
- [Kompatibilitas](#-kompatibilitas)
- [Troubleshooting](#-troubleshooting)
- [Arsitektur Kode](#-arsitektur-kode)
- [Lisensi](#-lisensi)

---

## ✨ Fitur

| Fitur | Keterangan |
| :--- | :--- |
| 🖥️ **Tampilan terminal** | Latar hitam, teks hijau, kursor `_` berkedip (muncul-hilang) |
| 🎮 **Gameplay klasik** | Tebak angka → petunjuk **"Lebih besar"** / **"Lebih kecil"** |
| 📈 **Level progresif** | Level 1 = 1–5, Level 2 = 1–10, Level 3 = 1–20, Level 4 = 1–50, dst |
| ⏱️ **Timer real-time** | Format `jam:menit:detik` di samping penghitung percobaan |
| 💾 **Save terenkripsi** | Ketik `save` → dapat kode acak **512 karakter hex** (bukan plaintext) |
| 🔐 **Crypto murni JS** | XOR stream + xorshift32 PRNG + FNV-1a hash + checksum — tanpa library |
| ⌨️ **Keyboard virtual** | Ketik `keyboard` untuk muncul/sembunyikan keypad di layar |
| 🧠 **Validasi input** | Selain angka/minus/`keyboard`/`save` → muncul **"Input tidak valid"** |
| 📱 **Multi-platform** | Jalan di Chrome, Firefox, Safari, Edge, Node.js — OS apapun |
| 📦 **Zero dependency** | Tanpa internet sejak file selesai diunduh |

---

## 🚀 Cara Menjalankan

### Mode 1 — Browser (Desktop / Mobile)

1. Salin seluruh kode ke file baru bernama `tebak-angka.html`
2. Buka file tersebut dengan **klik ganda** (atau drag ke browser)
3. Selesai — game langsung jalan

> ⚠️ Tidak perlu server lokal, tidak perlu `npm install`, tidak perlu koneksi internet.

### Mode 2 — Terminal (Node.js)

1. Simpan kode yang sama ke file `tebak-angka.js`
2. Jalankan:

```bash
node tebak-angka.js
```

3. Game berjalan di CLI dengan warna ANSI
4. Keluar kapan saja dengan `Ctrl+C`

> 💡 File yang sama berfungsi di kedua mode. Tidak perlu edit apapun — kode mendeteksi environment secara otomatis.

---

## 🎮 Cara Bermain

### Menu Awal

Saat pertama kali dibuka, muncul prompt:

```text
> _
```

Ada 2 pilihan:

| Aksi | Hasil |
| :--- | :--- |
| Tekan **Enter** (kosong) | Mulai game baru dari **Level 1** |
| Tempel **kode save** lalu Enter | Lanjutkan progres dari level tersimpan |

### Gameplay

```text
=== LEVEL 1 (1-5) ===
Tebak angka antara 1 sampai 5.
```

Ketik angka lalu Enter:

| Respons | Arti |
| :--- | :--- |
| `>> BENAR!` | Naik ke level berikutnya |
| `>> Lebih besar` | Angka rahasia **lebih besar** dari tebakanmu |
| `>> Lebih kecil` | Angka rahasia **lebih kecil** dari tebakanmu |

### Perintah Khusus

| Perintah | Fungsi |
| :--- | :--- |
| `save` | Generate kode save 512 karakter → klik/tekan untuk menyalin |
| `keyboard` | Toggle keyboard virtual di layar (on/off) |
| *Enter kosong* | Tidak melakukan apa-apa saat di tengah game |

### Aturan Validasi Input

Input **hanya** menerima:

- ✅ Angka — `1`, `2`, `42`, `999`, ...
- ✅ Angka minus — `-5`, `-100`, ...
- ✅ Perintah khusus — `save`, `keyboard`
- ❌ Selain di atas → muncul `>> Input tidak valid`

---

## 💾 Sistem Save Game

### Cara Menyimpan

1. Saat bermain, ketik `save` lalu tekan Enter
2. Muncul kode seperti ini (contoh dipersingkat):

```text
=== SAVE GAME ===
Level: 4 | Percobaan: 7 | Waktu: 00:02:31
Salin kode berikut (512 karakter):

A03F7B2E9C1D...(total 512 karakter)...5F8E
```

3. **Klik kode di browser** → otomatis tersalin ke clipboard
4. Di terminal → seleksi manual lalu copy

### Cara Memuat

1. Refresh halaman / buka ulang game
2. Di prompt menu, tempel kode
3. Tekan Enter → langsung lanjut dari level & waktu tersimpan

### Detail Teknis Enkripsi

Plaintext yang disimpan:

```text
L<level>|A<attempt>|T<detik>|N<target>
```

Contoh: `L3|A2|T120|N17` → Level 3, sudah salah 2×, waktu 2 menit, target 17.

**Proses enkripsi:**

1. Setiap byte plaintext di-XOR dengan byte keystream dari **xorshift32 PRNG**
2. PRNG di-seed dari **FNV-1a hash** atas kunci rahasia internal
3. Ditambahkan **checksum 1 byte** (mod 256) untuk validasi integritas
4. Hasil di-encode hex, **di-pad dengan hex acak sampai panjang tepat 512**
5. Output: hex UPPERCASE 512 karakter

**Proses dekripsi:**

1. Ambil 4 hex pertama = panjang data
2. Ambil N byte berikutnya = ciphertext
3. Verifikasi checksum
4. XOR ulang dengan keystream → dapat plaintext
5. Parse dan validasi range (level, target, dsb)

> 🔐 Kode palsu / rusak / hasil edit tangan akan **otomatis ditolak** karena checksum tidak cocok.

---

## 📈 Progresi Level

Pola: kelipatan `5 → 10 → 20`, naik 10× setiap 3 level.

| Level | Range Angka |
| :---: | :---: |
| 1 | 1 – 5 |
| 2 | 1 – 10 |
| 3 | 1 – 20 |
| 4 | 1 – 50 |
| 5 | 1 – 100 |
| 6 | 1 – 200 |
| 7 | 1 – 500 |
| 8 | 1 – 1.000 |
| 9 | 1 – 2.000 |
| 10 | 1 – 5.000 |
| ... | ... |

---

## 📱 Catatan Mobile

- **Deteksi otomatis** — jika perangkat punya touchscreen, keyboard virtual langsung aktif
- **Keyboard OS** — tap di mana saja pada layar untuk memunculkan keyboard sistem
- **Keyboard virtual** — tombol besar ramah jari, mencakup `0-9`, `-`, `Del`, `Enter`
- **Clipboard** — tombol salin bekerja dengan `navigator.clipboard`, dengan fallback `execCommand` untuk browser lama

---

## 🧩 Kompatibilitas

| Platform | Status |
| :--- | :---: |
| Chrome / Edge (Desktop) | ✅ |
| Firefox (Desktop) | ✅ |
| Safari (macOS) | ✅ |
| Chrome (Android) | ✅ |
| Safari (iOS / iPadOS) | ✅ |
| Node.js ≥ 12 | ✅ |
| Browser tanpa JavaScript | ❌ |

---

## 🛠️ Troubleshooting

**❓ Keyboard virtual tidak muncul padahal di HP.**
> Ketik `keyboard` untuk toggle manual. Atau tap layar dulu.

**❓ Kode save tidak bisa ditempel di mobile.**
> Paste biasanya lewat long-press → Paste. Jika tidak bisa, salin kode ke notes dulu, lalu long-press input.

**❓ Kode save hilang / terpotong.**
> Kode harus **tepat 512 karakter** hex. Jika ada spasi/enter yang ikut tersalin, akan di-strip otomatis saat ditempel di browser.

**❓ Di terminal tidak ada warna.**
> Beberapa terminal tidak support ANSI. Coba Windows Terminal, iTerm2, atau terminal Linux modern.

**❓ Bisa dicurangi dengan buka DevTools?**
> Bisa (semua game client-side pasti bisa). Angka target disimpan di memori JavaScript. Ini game santai, bukan kompetisi. 😄

---

## 🧠 Arsitektur Kode

```text
createGame(io)         → core logic (state machine, level, timer, save)
  ├─ newLevel()        → setup level baru
  ├─ handleEnter()     → proses input Enter
  ├─ handleChar()      → tambah 1 karakter ke buffer
  └─ handleBackspace() → hapus 1 karakter

encryptSave(str)       → plaintext → hex 512 char
decryptSave(code)      → hex 512 char → plaintext (dengan validasi)

initBrowser()          → render DOM, keyboard, event listener
initNode()             → readline CLI, ANSI color
```

**Deteksi environment:**

```js
const IS_BROWSER = (typeof window !== 'undefined') && (typeof document !== 'undefined');
```

---

## 📄 Lisensi

Proyek ini dirilis di bawah **Unlicense** — dilepas ke **public domain**. Bebas dipakai, dimodifikasi, dan dibagikan tanpa syarat apapun.

Cocok untuk belajar:

- State machine game sederhana
- Enkripsi XOR + PRNG murni JS
- DOM API tanpa framework
- Multi-target runtime (browser + Node.js) dalam satu file

---

## 🎉 Selamat Bermain!

Ketik angka, kalahkan level, dan jangan lupa `save` sebelum tutup tab.
