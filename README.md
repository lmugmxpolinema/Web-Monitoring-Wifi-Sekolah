# Monitoring Web Sekolah

Sistem monitoring berbasis web untuk Optical Network Terminal (ONT) pada jaringan sekolah. Menyediakan tampilan status real-time, pemeriksaan latensi ping, pelacakan gangguan, dan notifikasi ketika perangkat tidak dapat diakses atau mengalami masalah.

## Fitur

- Pemeriksaan ping berkala terhadap perangkat ONT untuk memantau latensi dan ketersediaan
- Penyimpanan data berbasis JSON untuk daftar ONT (`onts.json`), riwayat gangguan (`outages.json`), dan notifikasi (`notifications.json`)
- Dashboard web dengan tampilan daftar, peta, analitik, dan notifikasi
- Impor dan konversi data dari CSV ke JSON utama
- Skrip utilitas untuk penghapusan duplikat, penggabungan, dan pengaturan ulang status ONT
- Backup otomatis file JSON
- Unit test untuk logika ping, pembuatan notifikasi, dan pembaruan real-time

## Persiapan

Ikuti langkah berikut untuk menyiapkan dan menjalankan proyek di mesin lokal Anda.

### Persyaratan

- Python 3.10 atau versi lebih baru
- pip (pengelola paket Python)
- Windows PowerShell (untuk skrip deploy) atau Bash (di Linux/macOS)

### Instalasi

1. Clone repositori:

   ```powershell
   git clone https://github.com/Wijayayaya/Monitoring-Web-Sekolah.git
   cd Monitoring-Web-Sekolah
   ```

2. Buat dan aktifkan virtual environment:

   ```powershell
   python -m venv .venv; .\\.venv\\Scripts\\Activate.ps1
   ```

3. Instal dependensi:

   ```powershell
   pip install -r requirements.txt
   ```

## Konfigurasi

- Edit `onts.json` untuk menambahkan daftar perangkat ONT (IP dan identifier)
- Sesuaikan ambang batas ping dan jadwal di `ping_check.py` sesuai kebutuhan
- Template HTML ada di folder `templates/`; aset statis di `static/`

## Penggunaan

- **Konversi Data**: Impor data ONT dari CSV:

  ```powershell
  python convert_csv_to_main.py input.csv
  ```

- **Monitor Ping**: Jalankan pemeriksaan ping berkala:

  ```powershell
  python ping_check.py
  ```

- **Web Server**: Mulai server Flask:

  ```powershell
  python app.py
  ```

- **Deploy (Windows)**:

  ```powershell
  ./deploy.ps1
  ```

## Struktur Proyek

```
.
├── app.py                # Aplikasi Flask dan routing
├── ping_check.py         # Logika ping dan deteksi gangguan
├── convert_csv_to_main.py# Konversi CSV ke JSON
├── check_duplicates.py   # Menghapus entri ONT duplikat
├── merge_onts.py         # Menggabungkan file JSON ONT
├── reset_ont_status.py   # Reset status di file JSON
├── *.json                # File status dan backup
├── templates/            # Template HTML dashboard
├── static/               # Ikon dan gambar
├── reports/              # Dokumentasi dan laporan
├── test_*.py             # Unit test
└── requirements.txt      # Dependensi Python
```

## Menjalankan Tes

Jalankan semua unit test dengan pytest:

```powershell
pytest
```

## Kontribusi

Kontribusi sangat dipersilakan! Silakan ajukan issue atau pull request di GitHub.

## Lisensi

Proyek ini menggunakan lisensi MIT. Lihat [LICENSE](LICENSE) untuk detail.
