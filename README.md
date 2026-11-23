# BIODATA
Nama : Peryanur
NPM : 2410010282
Kelas : 5N REG BJM

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
