# Monolith vs Microservices dengan Flask Python

Praktik perbandingan arsitektur **Monolith** dan **Microservices** menggunakan Flask (Python). Aplikasi yang dibuat adalah sistem sederhana katalog buku dan pemesanan.

Mata Kuliah: Paradigma Sistem | Kelas: TRKJ-3C
Politeknik Negeri Lhokseumawe

## Isi Repository

| File | Keterangan | Port |
|------|------------|------|
| `monolith_app.py` | Aplikasi Monolith: fitur buku dan pesanan dalam satu proses | 5050 |
| `book_service.py` | Microservice 1: layanan data buku | 5051 |
| `order_service.py` | Microservice 2: layanan pesanan (memanggil Book Service lewat HTTP) | 5052 |

> **Catatan:** Port bawaan Flask (5000) tidak dipakai karena bentrok dengan fitur **AirPlay Receiver** di macOS (respons `403 Forbidden` dari `AirTunes`).

## Prasyarat

- Python 3.10 atau lebih baru
- pip
- curl (untuk pengujian)
- Git

## Instalasi

1. Clone repository:

   ```bash
   git clone https://github.com/Rzkyrmdhn1/Monolith-Vs-Microservice.git
   cd Monolith-Vs-Microservice
   ```

2. Buat dan aktifkan virtual environment:

   ```bash
   python3 -m venv venv
   source venv/bin/activate        # macOS / Linux
   # venv\Scripts\activate         # Windows
   ```

3. Install dependensi:

   ```bash
   pip install flask requests
   ```

4. (Opsional) Cek versi yang terpasang:

   ```bash
   python3 --version
   pip list | grep -i -E "flask|requests"
   ```

---

## Bagian 1: Menjalankan Monolith

### Menjalankan server

Di **Terminal 1**:

```bash
python3 monolith_app.py
```

Server aktif di `http://127.0.0.1:5050`.

### Pengujian

Di **Terminal 3** (terminal baru):

1. Lihat daftar buku (stok awal = 5):

   ```bash
   curl http://localhost:5050/books
   ```

2. Buat pesanan:

   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"book_id": 1}' http://localhost:5050/orders
   ```

3. Cek stok lagi. Stok sekarang **4** karena fitur pesanan langsung mengurangi variabel `books`:

   ```bash
   curl http://localhost:5050/books
   ```

Setelah selesai, hentikan server dengan `Ctrl+C` sebelum lanjut ke Microservices.

---

## Bagian 2: Menjalankan Microservices

Dibutuhkan **dua terminal** untuk server dan satu terminal untuk pengujian. Pastikan virtual environment aktif di tiap terminal.

### Menjalankan kedua layanan

**Terminal 1**, Book Service (port 5051):

```bash
python3 book_service.py
```

**Terminal 2**, Order Service (port 5052):

```bash
python3 order_service.py
```

### Pengujian

Di **Terminal 3**:

1. Cek kondisi awal Book Service (stok = 5):

   ```bash
   curl http://localhost:5051/books
   ```

2. Buat pesanan lewat Order Service:

   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"book_id": 1}' http://localhost:5052/orders
   ```

3. Cek stok di Book Service. Stok tetap **5** karena Order Service hanya membaca stok, tidak mengubahnya:

   ```bash
   curl http://localhost:5051/books/1
   ```

### Eksperimen Fault Isolation

1. Matikan Book Service dengan `Ctrl+C` di Terminal 1.
2. Kirim ulang pesanan:

   ```bash
   curl -X POST -H "Content-Type: application/json" -d '{"book_id": 1}' http://localhost:5052/orders
   ```

3. Order Service tetap hidup dan membalas dengan error yang terkendali:

   ```json
   {"error": "Book Service sedang down!"}
   ```

Ini menunjukkan **Fault Isolation**: matinya satu layanan tidak menjatuhkan layanan lain.

---

## Daftar Endpoint

### Monolith (port 5050)

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| GET | `/books` | Daftar buku |
| POST | `/orders` | Buat pesanan (body: `{"book_id": 1}`), stok berkurang |

### Book Service (port 5051)

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| GET | `/books` | Daftar buku |
| GET | `/books/<book_id>` | Detail satu buku |

### Order Service (port 5052)

| Method | Endpoint | Fungsi |
|--------|----------|--------|
| POST | `/orders` | Buat pesanan (body: `{"book_id": 1}`), memeriksa stok ke Book Service |

## Ringkasan Perbandingan

| Aspek | Monolith | Microservices |
|-------|----------|---------------|
| Struktur | Satu codebase, satu proses | Layanan terpisah |
| Akses data | Variabel global | HTTP request antarlayanan |
| Kecepatan | Cepat, tanpa latensi jaringan | Lebih lambat karena panggilan jaringan |
| Konsistensi data | Mudah dijaga | Perlu penanganan khusus |
| Ketahanan | Kegagalan fatal menjatuhkan seluruh aplikasi | Fault Isolation |
| Skalabilitas | Seluruh aplikasi digandakan | Per layanan |
| Kompleksitas | Sederhana | Lebih kompleks |

## Troubleshooting

- **Port sudah dipakai / respons `403 Forbidden` dari AirTunes** (macOS): port 5000 dipakai AirPlay Receiver. Proyek ini sudah memakai port 5050, 5051, dan 5052. Jika bentrok juga, ubah nilai `port=` di file terkait, atau matikan AirPlay Receiver di *System Settings*.
- **`zsh: command not found: pip`**: aktifkan dulu virtual environment (`source venv/bin/activate`) atau gunakan `python3 -m pip`.
- **`Book Service sedang down!`** saat membuat pesanan: pastikan `book_service.py` sudah berjalan di port 5051.

## Penulis

**Mhammad Rizki Ramadhan** (2024903430099)
Teknologi Rekayasa Komputer dan Jaringan, Politeknik Negeri Lhokseumawe, 2026
