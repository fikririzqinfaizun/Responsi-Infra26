# Analisis Perbaikan

## Permasalahan 1 : Konfigurasi DB_HOST yang salah pada Service web1

### Gejala
Jelaskan gejala yang ditemukan.

jawab :
- Service web1 tidak dapat terhubung ke database MySQL.
- Container web1 mengalami error koneksi saat startup, misalnya: 'Unknown MySQL server host mysql'.
- Aplikasi pada web1 tidak dapat menampilkan data atau mengalami crash loop.


### Penyebab
Jelaskan akar penyebab masalah.

jawab : 
- Pada blok environment service web1, nilai DB_HOST diset ke 'mysql', sedangkan nama container database yang didefinisikan di docker-compose adalah 'db' (nama service) dengan container_name 'mysql-db'.
- Docker Compose menggunakan nama service sebagai hostname internal jaringan, bukan container_name.
- Baris bermasalah:
```
DB_HOST: mysql   # SALAH - nama service DB adalah 'db', bukan 'mysql'
```


### Solusi
Jelaskan langkah perbaikan yang dilakukan.

Jawab : 
- Ubah nilai DB_HOST pada service web1 dari 'mysql' menjadi 'db', sesuai nama service database yang terdaftar.
- Baris sebelum perbaikan:
DB_HOST: mysql
- Baris setelah perbaikan:
DB_HOST: db
- Setelah perubahan, jalankan ulang container: docker compose up -d --force-recreate web1


## Permasalahan 2 : Password Database yang salah pada Service web2

### Gejala
Jelaskan gejala yang ditemukan.

Jawab :
- Service web2 tidak dapat melakukan autentikasi ke database MySQL.
- Log container web2 menunjukkan error: 'Access denied for user 'student'@'...' (using password: YES)'.
- Fitur yang bergantung pada database tidak berfungsi pada instance web2.


### Penyebab
Jelaskan akar penyebab masalah.

Jawab : 
- Pada blok environment service web2, nilai DB_PASS diset ke 'wrongpassword', padahal password yang benar untuk user 'student' adalah 'student123' sesuai konfigurasi service db (MYSQL_PASSWORD: student123).
- Baris bermasalah:
```
DB_PASS: wrongpassword   # SALAH - password tidak sesuai
```

### Solusi
Jelaskan langkah perbaikan yang dilakukan

jawab :
- Ubah nilai DB_PASS pada service web2 dari 'wrongpassword' menjadi 'student123'.
- Baris sebelum perbaikan:
DB_PASS: wrongpassword
- Baris setelah perbaikan:
DB_PASS: student123
- Setelah perubahan, jalankan ulang container: docker compose up -d --force-recreate web2


## Permasalahan 3 : Typo Build Context dan Network yang kurang pada Service web3

### Gejala
Jelaskan gejala yang ditemukan.

Jawab : 
- Docker Compose gagal build service web3 dengan error: 'build path './web33' either does not exist or is not accessible'.
- Service web3 tidak dapat menerima trafik dari nginx load balancer meskipun container berhasil berjalan.
- Nginx melaporkan error saat mencoba meneruskan request ke web3.


### Penyebab
Jelaskan akar penyebab masalah.

Jawab : 

• Terdapat dua bug sekaligus pada service web3:
1. Typo pada build context: './web33' seharusnya './web3'. Direktori './web33' tidak ada sehingga proses build gagal.
2. Service web3 hanya terdaftar di network 'backend', tidak di network 'frontend'. Karena nginx berada di network 'frontend', nginx tidak dapat berkomunikasi dengan web3.

• Baris bermasalah:
```
context: ./web33   # SALAH - typo, direktori tidak ada
networks:          # SALAH - hanya backend, harus juga frontend
- backend
```

### Solusi
Jelaskan langkah perbaikan yang dilakukan

Jawab :
- Perbaikan 1 --> Betulkan typo pada build context:
context: ./web33  -->  context: ./web3
- Perbaikan 2 --> Tambahkan network 'frontend' pada service web3.
networks:- frontend,- backend
- Setelah perubahan, jalankan ulang: docker compose up -d --build web3


## Permasalahan 4 : Nama Volume Tidak Konsisten (Mismatch)

### Gejala
Jelaskan gejala yang ditemukan.

Jawab :
- Docker Compose memberikan warning atau error terkait volume yang tidak terdefinisi.
- Volume untuk data MySQL tidak persistent, data bisa hilang saat container di-restart.
- Perintah 'docker compose up' mungkin gagal dengan error: 'service 'db' refers to undefined volume db-data'.



### Penyebab
Jelaskan akar penyebab masalah.

Jawab :
- Terdapat inkonsistensi nama volume: service 'db' menggunakan volume bernama 'db-data' pada konfigurasi mount, namun pada blok 'volumes' di level root hanya didefinisikan 'database-data' (bukan 'db-data').
- Volume 'db-data' yang dipakai oleh service db tidak pernah dideklarasikan di bagian volumes.
- Konfigurasi bermasalah:
```
# Pada service db:
volumes:
- db-data:/var/lib/mysql   # merujuk ke 'db-data'
# Pada level root:
volumes:
database-data:           # SALAH - nama berbeda 'database-data'
```


### Solusi
Jelaskan langkah perbaikan yang dilakukan

Jawab : 
- Sesuaikan nama volume agar konsisten. Ada dua pilihan:
- Pilihan A - Ubah nama di blok volumes root menjadi 'db-data':
```
volumes:
db-data:
```
- Pilihan B - Ubah nama mount di service db menjadi 'database-data':
```
volumes:
database-data:/var/lib/mysql
```
- Pilihan A direkomendasikan karena lebih deskriptif dan tidak memerlukan migrasi data.
- Setelah perubahan, jalankan: docker compose down & docker compose up -d
