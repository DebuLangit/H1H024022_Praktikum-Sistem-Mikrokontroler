# Percobaan 3A: Komunikasi Serial (UART)

## 1. Jelaskan proses dari input keyboard hingga LED menyala/mati!
**Keyboard → UART → Arduino → Logika Program → LED**
- User mengetik input di Serial Monitor
- Data dikirim melalui komunikasi UART (USB)
- Arduino menerima data
  Data masuk ke buffer serial dan dicek menggunakan:
  ```cpp
  Serial.available()
  ```
- Arduino membaca data
  ```cpp
  Serial.available()char data = Serial.read();
  ```
- Program melakukan percabangan
  ```cpp
  if (data == '1')
  ```
  - Jika '1' → LED menyala
  - Jika '0' → LED mati
- Arduino mengontrol pin digital
  ```cpp
  digitalWrite(PIN_LED, HIGH); // nyala
  digitalWrite(PIN_LED, LOW);  // mati
  ```
- LED merespon
  - HIGH → LED ON
  - LOW → LED OFF
---

## 2. Mengapa digunakan Serial.available() sebelum membaca data? Apa yang terjadi jika baris tersebut dihilangkan?

Untuk mengecek apakah ada data yang masuk sebelum dibaca.
```cpp
if (Serial.available() > 0)
```
Jika tidak digunakan:
- Arduino bisa membaca data kosong
- Nilai yang dibaca bisa acak (garbage value)
- Program bisa tidak stabil / error logika

---

## 3. Modifikasi program agar LED berkedip (blink) ketika menerima input '2' dengan kondisi jika ‘2’ aktif maka LED akan terus berkedip sampai perintah selanjutnya diberikan dan berikan penjelasan disetiap baris kode nya dalam bentuk README.md!

**Kode Program**

```cpp
const int PIN_LED = 12;

char mode = '0'; // default OFF

unsigned long previousMillis = 0;
const long interval = 500; // 500 ms

bool ledState = LOW;

void setup() {
  Serial.begin(9600);
  Serial.println("Ketik '1'=ON, '0'=OFF, '2'=BLINK");
  pinMode(PIN_LED, OUTPUT);
}

void loop() {
  // ====== BACA INPUT UART ======
  if (Serial.available() > 0) {
    char data = Serial.read();

    if (data == '1' || data == '0' || data == '2') {
      mode = data;

      if (mode == '1') {
        digitalWrite(PIN_LED, HIGH);
        Serial.println("Mode: LED ON");
      }
      else if (mode == '0') {
        digitalWrite(PIN_LED, LOW);
        Serial.println("Mode: LED OFF");
      }
      else if (mode == '2') {
        Serial.println("Mode: LED BLINK");
      }
    }
    else if (data != '\n' && data != '\r') {
      Serial.println("Perintah tidak dikenal");
    }
  }

  // ====== MODE BLINK ======
  if (mode == '2') {
    unsigned long currentMillis = millis();

    if (currentMillis - previousMillis >= interval) {
      previousMillis = currentMillis;

      ledState = !ledState;
      digitalWrite(PIN_LED, ledState);
    }
  }
}
```
Penjelasan singkat:
- blinkMode → penanda mode berkedip
- Input '2' → mengaktifkan blink
- Loop terus berjalan → LED berkedip terus
- Input baru → mengganti mode

---

## 4.  Tentukan apakah menggunakan delay() atau milis()! Jelaskan pengaruhnya terhadap sistem
```cpp
delay()
```
- Kelebihan:
  - Mudah digunakan
  - Cocok untuk program sederhana
- Kekurangan:
  - Program berhenti sementara (blocking)
  - Tidak bisa multitasking

```cpp
millis()
```
- Kelebihan:
  - Non-blocking (program tetap jalan)
  - Bisa menjalankan banyak proses sekaligus
- Kekurangan:
  - Lebih kompleks

### Dampak pada sistem
delay()	→ Sistem berhenti sementara
millis() → Sistem tetap responsif

---

# Percobaan 3B: Inter-Integrated Circuit (I2C)

## 1. Jelaskan bagaimana cara kerja komunikasi I2C antara Arduino dan LCD pada rangkaian tersebut!
Komunikasi I2C bekerja menggunakan dua jalur utama, yaitu:
- SDA (Serial Data) → untuk data
- SCL (Serial Clock) → untuk sinkronisasi
Alur kerjanya:
- Arduino sebagai Master
  - Mengontrol komunikasi
  - Mengirim sinyal clock
- LCD sebagai Slave
  - Menerima perintah dari Arduino
  - Memiliki alamat unik (misalnya 0x27)
- Proses komunikasi:
  - Arduino mengirim alamat LCD
  - Jika alamat cocok → LCD merespon
  - Arduino mengirim data (misalnya teks atau perintah)
  - LCD menampilkan data tersebut

---

## 2. Apakah pin potensiometer harus seperti itu? Jelaskan yang terjadi apabila pin kiri dan pin kanan tertukar!
Tidak harus secara fisik kiri/kanan, yang penting konfigurasi fungsinya benar:
| Pin Potensiometer | Fungsi |
|---------|---------|
| Kiri | GND  | 
| Tengah  | Output (A0)  | 
| Kanan  | 5V  | 

Jika kiri dan kanan tertukar:
- Tegangan tetap terbaca (tidak rusak)
- Tapi hasilnya terbalik
Contoh:
- Putar kanan → nilai malah turun
- Putar kiri → nilai malah naik

---

## 3. Modifikasi program dengan menggabungkan antara UART dan I2C (keduanya sebagai output)
**Kode Program**
```cpp
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Inisialisasi LCD I2C
LiquidCrystal_I2C lcd(0x27, 16, 2);

// Pin
const int pinPot = A0;

void setup() {
  Serial.begin(9600);

  lcd.init();
  lcd.backlight();
}

void loop() {
  int adc = analogRead(pinPot);

  // Konversi ke Volt (0–5V)
  float volt = adc * (5.0 / 1023.0);

  // Konversi ke persen (0–100%)
  int persen = map(adc, 0, 1023, 0, 100);

  // Mapping bar (0–16 karakter)
  int bar = map(adc, 0, 1023, 0, 16);

  // ================= UART =================
  Serial.print("ADC: ");
  Serial.print(adc);
  Serial.print(" | Volt: ");
  Serial.print(volt, 2);
  Serial.print(" V | Persen: ");
  Serial.print(persen);
  Serial.println("%");

  // ================= LCD =================
  lcd.setCursor(0, 0);
  lcd.print("ADC:");
  lcd.print(adc);
  lcd.print(" ");

  lcd.print(persen);
  lcd.print("% ");

  // Bar di baris kedua
  lcd.setCursor(0, 1);
  for (int i = 0; i < 16; i++) {
    if (i < bar) {
      lcd.write(255);
    } else {
      lcd.print(" ");
    }
  }

  delay(200);
}
```
**Penjelasan singkat:**
- analogRead() → baca potensiometer
- Serial.print() → kirim ke UART
- lcd.print() → kirim ke I2C
- map() → ubah skala nilai

---

## 4. Lengkapi table berikut berdasarkan pengamatan pada Serial Monitor
**Rumus:**
Volt = ADC × (5 / 1023)
Persen = ADC / 1023 × 100

| ADC | Volt(V) | Persen(&) |
|---------|---------|---------|
| 1 | 0.00 | 0% |
| 21 | 0.10 | 2% | 
| 49 | 0.24 | 4% |
| 72 | 0.36 | 7% | 
| 96 | 0.47 | 9% |

