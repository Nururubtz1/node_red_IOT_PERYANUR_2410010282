# ESP32 IoT Monitoring - versi perbaikan history dan WebSocket

## Isi perbaikan

1. Dashboard Node-RED pada `http://localhost:1880/dashboard` sekarang menampilkan grafik, status peringatan, notifikasi browser, dan tabel **10 Data Sensor Terakhir**.
2. Tabel membaca kolom waktu `waktu` yang memang dikirim API SQLite dan menampilkan 10 data terbaru.
3. Dashboard standalone `dashboard.html` juga diperbaiki: membaca kolom `waktu`, urutan data terbaru, menampilkan pesan ketika API gagal, dan mengirim perubahan batas suhu melalui `PUT /api/threshold`.
4. Proses sensor menyimpan data ke SQLite dan mengirim update lewat WebSocket `/ws/sensor`. Notifikasi MQTT hanya dikirim ketika suhu baru saja beralih dari normal ke warning, bukan pada setiap sampel yang masih warning.
5. Simulator menggunakan Paho MQTT Callback API v2.

## Menjalankan

1. Pastikan Mosquitto aktif pada `localhost:1883`.
2. Di folder user Node-RED jalankan `npm install node-red-node-sqlite`, lalu restart Node-RED.
3. Import `flow1.json` ke Node-RED. Semua node SQLite menggunakan konfigurasi database yang sama, yaitu file `sensor.db` relatif terhadap working directory Node-RED.
4. Klik Deploy dan pastikan Debug `Status Database` tidak menampilkan error `no such table` atau error SQLite lain.
5. Jalankan simulator:
   ```powershell
   python -m pip install "paho-mqtt>=2.0.0"
   python simulasi_esp32.py
   ```
6. Buka `http://localhost:1880/dashboard`. Izinkan notifikasi browser melalui tombol di dashboard.

## API

API key demo: `KULIAH-API-2026` (harus sama pada function node validasi dan dashboard).

- `GET /api/suhu`
- `GET /api/history`
- `POST /api/kontrol`
- `PUT /api/threshold` dengan JSON `{ "threshold": 31.5 }`
- WebSocket `ws://localhost:1880/ws/sensor`

Contoh threshold (PowerShell):

```powershell
curl.exe -X PUT http://localhost:1880/api/threshold `
  -H "X-API-Key: KULIAH-API-2026" `
  -H "Content-Type: application/json" `
  -d '{"threshold":31.5}'
```

## Jika muncul error WebSocket listener duplikat

Flow ini hanya berisi **satu** konfigurasi `websocket-listener` dengan path `/ws/sensor` (ID pada file ini: `6c08b8d75fefffae`). Jika Node-RED tetap menampilkan `Cannot have two WebSocket listeners on the same path: /ws/sensor`, berarti ada listener lain dari flow/tab lama yang masih aktif. Buka menu Export > all flows, cari semua kemunculan `"path": "/ws/sensor"`, lalu hapus konfigurasi listener duplikat atau ubah salah satunya beserta alamat WebSocket pada dashboard yang menggunakannya. Jangan menghapus node `websocket out` yang memakai listener tunggal tersebut.

## Catatan

- Jika riwayat menampilkan nilai kosong, import ulang `flow1.json` dan klik Deploy. Node SQLite pada flow ini membutuhkan placeholder `$nama` agar parameter pada `msg.payload` terikat dengan benar; baris lama yang tersimpan sebagai `NULL` tidak dapat dipulihkan dan akan dilewati oleh API.
- Database pada flow ini menggunakan nama `sensor.db`, bukan `esp32_history.sqlite`. Seluruh node SQLite pada flow memakai konfigurasi database yang sama.
- Threshold suhu saat ini disimpan pada flow context Node-RED, bukan di tabel SQLite. Jika ingin threshold tetap bertahan setelah restart Node-RED, perlu menambahkan tabel pengaturan dan alur baca/tulis threshold.
- API key yang disimpan di HTML hanya cocok untuk demo/lab lokal, bukan autentikasi produksi.
