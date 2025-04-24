# Dokumentasi ESP32-CAM MJPEG Multiclient

## Gambaran Umum

Proyek ini memungkinkan modul ESP32-CAM untuk melakukan streaming video MJPEG dengan dukungan multi-klien. Kode ini memungkinkan ESP32-CAM menyediakan stream video secara real-time melalui koneksi WiFi, dan beberapa klien dapat mengakses stream tersebut secara bersamaan.

## Struktur Proyek

```
├── camera_pins.h           - Definisi pin kamera untuk berbagai model ESP32-CAM
├── esp32_camera_mjpeg_multiclient.ino - File utama aplikasi
├── home_wifi_multi.h       - Konfigurasi kredensial WiFi
└── src/
    ├── OV2640.cpp          - Implementasi kelas kamera OV2640
    └── OV2640.h            - Header kelas kamera OV2640
```

## Komponen Utama

### 1. OV2640 Camera
Kelas `OV2640` menyediakan abstraksi untuk modul kamera, menghandle inisialisasi kamera dan pengambilan gambar.

### 2. Task RTOS
Kode menggunakan sistem multitasking FreeRTOS dengan 3 task utama:
- `tMjpeg` - Menangani koneksi klien ke webserver
- `tCam` - Mengambil frame gambar dari kamera dan menyimpannya di buffer lokal
- `tStream` - Mengirimkan frame ke semua klien yang terhubung

### 3. Webserver
Menyediakan endpoint HTTP untuk mengakses stream dan gambar JPG:
- `/mjpeg/1` - Stream MJPEG untuk dilihat di browser
- `/jpg` - Untuk mengambil satu frame gambar JPG

## Cara Kerja

1. **Inisialisasi**:
   - Kamera diinisialisasi dengan konfigurasi pin yang sesuai
   - Terhubung ke jaringan WiFi menggunakan kredensial dari `home_wifi_multi.h`
   - Webserver dimulai dan task RTOS dibuat

2. **Pengambilan Frame**:
   - Task `tCam` mengambil frame dari kamera dengan kecepatan yang ditentukan (FPS=14)
   - Frame disimpan dalam buffer lokal, menggunakan PSRAM jika tersedia

3. **Streaming**:
   - Saat klien terhubung ke endpoint `/mjpeg/1`, koneksi klien ditambahkan ke antrian
   - Task `tStream` mengambil frame terbaru dan mengirimkannya ke semua klien dalam format MJPEG
   - Sinkronisasi antara task menggunakan semaphore untuk mencegah konflik saat mengakses frame

4. **Pengelolaan Sumber Daya**:
   - Task akan mensuspend dirinya sendiri saat tidak ada klien yang terhubung untuk menghemat daya
   - Buffer gambar dialokasikan secara dinamis berdasarkan ukuran yang dibutuhkan
   - PSRAM digunakan jika tersedia untuk buffer yang lebih besar

## Penggunaan

1. **Konfigurasi Wifi**:
   - Edit file `home_wifi_multi.h` dengan SSID dan password WiFi
   
2. **Pilih Model Kamera**:
   - Di `esp32_camera_mjpeg_multiclient.ino`, pastikan model kamera yang benar sudah didefinisikan (saat ini `CAMERA_MODEL_AI_THINKER`)
   
3. **Upload**:
   - Upload kode ke modul ESP32-CAM menggunakan Arduino IDE
   
4. **Akses Stream**:
   - Buka URL yang ditampilkan pada output Serial Monitor di browser
   - Format URL: `http://[IP_ESP32]/mjpeg/1`

## Konfigurasi

Beberapa parameter yang dapat disesuaikan:
- `FPS` - Frame per detik (saat ini 14)
- `config.frame_size` - Resolusi frame (FRAMESIZE_VGA, FRAMESIZE_SVGA, dll)
- `config.jpeg_quality` - Kualitas JPEG (0-63, nilai lebih rendah = kualitas lebih tinggi)

## Catatan

- Kamera ESP32-CAM memiliki memori terbatas, jadi resolusi tinggi mungkin tidak stabil
- Semakin banyak klien yang terhubung, semakin banyak daya pemrosesan yang dibutuhkan
- Kode secara otomatis memanfaatkan PSRAM jika tersedia untuk performa lebih baik