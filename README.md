# Landslide Hazard Identification & Early Warning Edge Device

Sistem peringatan dini longsor berbasis 2 edge device (ESP32 di lereng + HP Android), AI Decision Tree, dan MQTT ke cloud.

## Arsitektur
Sensor (soil, rain, MPU6050) -> **ESP32 (Edge 1)** -> MQTT broker (cloud) -> **HP Android/Termux (Edge 2)** + Dashboard Streamlit + Telegram

## Struktur
| Folder | Isi |
|---|---|
| `firmware/esp32_edge/` | Kode ESP32 (sensor, filter, AI, MQTT, alarm lokal) + `model.h` hasil training |
| `ai/` | `train_model.py` (training Decision Tree + export C), dataset sintetis, hasil training |
| `android_edge/` | `android_edge.py` (Termux): validasi 2 tahap, notifikasi, Telegram, log CSV |
| `dashboard/` | `dashboard.py` (Streamlit realtime) |
| `docs/` | Skrip video demo |

## Wiring ESP32
| Komponen | Pin |
|---|---|
| Soil moisture capacitive (AOUT) | GPIO34 |
| Rain sensor (AO) | GPIO35 |
| MPU6050 SDA / SCL | GPIO21 / GPIO22 |
| Buzzer | GPIO25 |
| LED merah / hijau | GPIO26 / GPIO27 (+ resistor 220 ohm) |

## Cara menjalankan
1. `pip install -r requirements.txt` lalu `python ai/train_model.py` (membuat ulang `model.h`).
2. Arduino IDE: install board ESP32 + library PubSubClient, Adafruit MPU6050, Adafruit Unified Sensor, ArduinoJson. Edit WiFi dan `NODE_ID` di `esp32_edge.ino`, upload.
3. Kalibrasi `SOIL_DRY/SOIL_WET` dan `RAIN_DRY/RAIN_WET` (baca nilai ADC di Serial Monitor saat sensor kering dan basah).
4. HP Android: install Termux + Termux:API, lalu `python android_edge/android_edge.py` (opsional `TG_TOKEN`, `TG_CHAT` untuk Telegram).
5. Dashboard: `streamlit run dashboard/dashboard.py`.

## Data processing
1. **Signal processing**: moving average 10 sampel, tilt delta terhadap posisi awal, akumulasi hujan 24 jam (leaky integrator), getaran RMS.
2. **Data fusion + AI**: 5 fitur digabung, Decision Tree (depth 7) menghasilkan Aman / Waspada / Bahaya.

## Hasil model
Akurasi 86,5% pada data uji sintetis; recall kelas Bahaya 90,6%. Lihat `ai/hasil_training.txt`.
> Catatan: dataset sintetis berbasis aturan. Untuk evaluasi sebenarnya, ganti dengan data logging prototipe.

## Keunikan
- Fitur akumulasi hujan 24 jam
- Alarm lokal (buzzer/LED) tetap menyala saat internet putus
- Validasi 2 tahap di edge ke-2 untuk mengurangi alarm palsu
