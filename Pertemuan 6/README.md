# <p align="center">Dokumentasi Praktikum</p>
https://github.com/user-attachments/assets/66e88dc7-947a-4f9c-a93c-eb9a4faaf1e2

---

# <p align="center">Menjawab Pertanyaan Praktikum</p>

## Percobaan 6A: External Interrupt

1. Jelaskan proses bagaimana tombol dapat mengubah kondisi LED menggunakan
interrupt!

   **Jawab:**
   Tombol dapat mengubah kondisi LED menggunakan hardware interrupt. Pin 2 diatur sebagai `INPUT_PULLUP`, sehingga normalnya bernilai `HIGH`. Saat tombol ditekan, tegangan berubah dari `HIGH` ke `LOW` dan memicu mode `FALLING`. Mikrokontroler kemudian menghentikan sementara fungsi `loop()` untuk menjalankan ISR `tombolInterrupt()` yang mengubah nilai `ledState`. Setelah itu, program kembali ke `loop()` dan menjalankan `digitalWrite(13, ledState)` untuk mengubah kondisi LED.
   
2. Apa fungsi `attachInterrupt()` pada program tersebut?

   **Jawab:**
   Fungsi `attachInterrupt()` digunakan untuk menghubungkan interupsi pada Pin 2 dengan fungsi tombolInterrupt. Saat terjadi perubahan sinyal turun (`FALLING`), mikrokontroler langsung menjalankan fungsi ISR tersebut.
  
3. Mengapa pada ISR tidak disarankan menggunakan `delay()` dan `erial.print()`?

   **Jawab:**
   ISR harus dibuat sesingkat mungkin karena proses utama program akan berhenti sementara saat ISR berjalan. Fungsi seperti `delay()` dan `Serial.print()` sebaiknya tidak digunakan karena bersifat lambat dan blocking, sehingga dapat menyebabkan mikrokontroler hang atau melewatkan interupsi penting lainnya.

4. Apa fungsi keyword volatile pada variabel ledState?

   **Jawab:**
   Keyword volatile berfungsi agar compiler selalu membaca nilai variabel terbaru dari RAM karena nilainya dapat berubah sewaktu-waktu, misalnya oleh interupsi hardware. Hal ini mencegah compiler menggunakan nilai lama yang tersimpan di cache.
   
5. Pada percobaan digunakan mode interrupt `FALLING`. Modifikasikan program menggunakan mode interrupt lain (`RISING`, `CHANGE`, atau `LOW`) kemudian:

   kode modifikasi:
   ```cpp
   // Variabel volatile agar dapat diubah dalam ISR
   volatile bool ledState = false;

   // ISR: dijalankan saat kondisi sinyal berubah (ditekan maupun dilepas)
   void tombolInterrupt() {
     ledState = !ledState; // Toggle status LED
   }

   void setup() {
     // Konfigurasi pin 13 sebagai output (LED)
     pinMode(13, OUTPUT);
     // Konfigurasi pin 2 sebagai input dengan pull-up internal
     pinMode(2, INPUT_PULLUP);
    
     // Daftarkan ISR pada pin 2, dipicu CHANGE (perubahan sinyal)
     attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, CHANGE);
   }

   void loop() {
     // Tulis status LED sesuai variabel ledState
     digitalWrite(13, ledState);
   }
   ```
   untuk kode program sama, hanya ganti mode di baris:
   ```cpp
   attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, CHANGE);
   ```
   ```cpp
   attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, RISING);
   ```
   ```cpp
   attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, LOW);
   ```
   **Perbedaan cara kerja masing-masing mode interrupt tersebut**
   - **FALLING:** Interupsi dipicu hanya ketika logika sinyal berubah turun dari `HIGH` ke `LOW`. (Terjadi persis pada saat tombol **mulai ditekan**).
   - **RISING:** Interupsi dipicu hanya ketika logika sinyal berubah naik dari `LOW` ke `HIGH`. (Terjadi persis pada saat tombol **dilepaskan**).
   - **CHANGE:** Interupsi dipicu setiap kali ada perubahan sinyal, baik itu dari `HIGH` ke `LOW` maupun sebaliknya. (Terjadi pada saat tombol **ditekan DAN dilepaskan**).
   - **LOW:** Interupsi dipicu secara terus-menerus tanpa henti selama logika sinyal berada pada status `LOW`. (Terjadi **selama tombol ditahan**).
   
   **Analisis perubahan perilaku LED yang terjadi pada setiap mode**
   - Mode `CHANGE`
     **Source Code:** `attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, CHANGE);`
     **Perilaku LED:** LED akan berubah status dua kali untuk setiap satu siklus penekanan tombol. Saat tombol ditekan ke bawah, LED berubah status (misal dari OFF ke ON). Saat jari diangkat dan tombol terlepas, LED akan langsung berubah status lagi (dari ON ke OFF).
   - Mode `RISING`
     **Source Code:** `attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, RISING);`
     **Perilaku LED:** Berkebalikan dari `FALLING`. Saat tombol ditekan pertama kali, LED tidak merespons (tidak terjadi apa-apa). LED baru akan merespons (berubah status ON/OFF) tepat pada saat tekanan jari dilepaskan dari tombol.
   - Mode `LOW`
     **Source Code:** `attachInterrupt(digitalPinToInterrupt(2), tombolInterrupt, LOW);`
     **Perilaku LED:** Perilaku ini sangat tidak stabil untuk *toggling* status LED. Saat tombol ditahan/ditekan lama, sinyal berada di posisi `LOW`. Akibatnya, mikrokontroler akan menembakkan ISR secara berulang-ulang dengan sangat cepat (ribuan kali per detik). Hasil fisiknya, LED akan terlihat menyala redup karena ia berkedip dengan frekuensi yang sangat tinggi, dan program utama `loop()` akan sepenuhnya terhenti karena CPU sibuk melayani ISR secara terus-menerus.



---

## Percobaan 6B: Timer Menggunakan millis()

1. Jelaskan bagaimana fungsi `millis()` bekerja pada program tersebut!

   **Jawab:**
   Fungsi `millis()` pada Arduino berfungsi sebagai penghitung waktu sejak board dinyalakan. Program membandingkan waktu saat ini (`currentMillis`) dengan waktu perubahan LED sebelumnya (`previousMillis`). Jika selisihnya mencapai 1000 ms, LED akan berubah status dan waktu sebelumnya diperbarui.

2. Apa perbedaan utama antara `delay()` dan `millis()`?

   **Jawab:**
   `delay()` menghentikan seluruh proses program sementara waktu tertentu (blocking), sedangkan `millis()` memungkinkan program tetap berjalan sambil menghitung waktu (non-blocking). Karena itu, `millis() lebih cocok untuk menjalankan beberapa tugas secara bersamaan.
   
3. Mengapa metode `millis()` disebut non-blocking?

   **Jawab:**
   Metode non-blocking tidak menghentikan kerja CPU. Fungsi `millis()` hanya mengecek waktu tanpa membuat program menunggu, sehingga mikrokontroler tetap bisa menjalankan tugas lain secara bersamaan, seperti membaca sensor atau mengontrol LED.

4. Modifikasi program agar:
   - LED pertama berkedip setiap 1 detik
   - LED kedua berkedip setiap 500 ms
   - Tanpa menggunakan `delay()`
     
  **Jawab:**
  
  ```cpp
  #include <Arduino.h>

  // --- Variabel untuk LED 1 ---
  const int ledPin1 = 13;             // Pin untuk LED pertama
  unsigned long previousMillis1 = 0;  // Menyimpan waktu terakhir LED 1 berubah
  const long interval1 = 1000;        // Interval kedip LED 1 (1000 ms / 1 detik)
  bool ledState1 = false;             // Status saat ini dari LED 1

  // --- Variabel untuk LED 2 ---
  const int ledPin2 = 12;             // Pin untuk LED kedua
  unsigned long previousMillis2 = 0;  // Menyimpan waktu terakhir LED 2 berubah
  const long interval2 = 500;         // Interval kedip LED 2 (500 ms / 0.5 detik)
  bool ledState2 = false;             // Status saat ini dari LED 2

  void setup() {
    pinMode(ledPin1, OUTPUT); // Mengatur Pin 13 sebagai OUTPUT
    pinMode(ledPin2, OUTPUT); // Mengatur Pin 12 sebagai OUTPUT
  }

  void loop() {
    // Mengambil waktu absolut saat ini sejak Arduino menyala
    unsigned long currentMillis = millis();

    // --- Logika Kontrol LED 1 ---
    // Mengecek apakah selisih waktu sudah mencapai interval 1000 ms
    if (currentMillis - previousMillis1 >= interval1) {
      previousMillis1 = currentMillis; // Memperbarui patokan waktu terakhir LED 1
      ledState1 = !ledState1;          // Membalik status LED 1 (jika ON jadi OFF, dst)
      digitalWrite(ledPin1, ledState1);// Menuliskan status fisik ke Pin 13
    }

    // --- Logika Kontrol LED 2 ---
    // Mengecek apakah selisih waktu sudah mencapai interval 500 ms
    if (currentMillis - previousMillis2 >= interval2) {
      previousMillis2 = currentMillis; // Memperbarui patokan waktu terakhir LED 2
      ledState2 = !ledState2;          // Membalik status LED 2
      digitalWrite(ledPin2, ledState2);// Menuliskan status fisik ke Pin 12
    }
  
    // Karena tidak ada fungsi delay(), program akan kembali ke atas loop 
    // dengan sangat cepat untuk mengecek kondisi if secara terus-menerus.
  }
  ```
  Penjelasan kode:
  - Variabel waktu dibuat terpisah agar kontrol LED 1 dan LED 2 independen.
  - `unsigned long` digunakan untuk menyimpan waktu dari `millis()`.
  - `setup()` mengatur pin 13 dan 12 sebagai output.
  - `currentMillis` mengambil waktu real-time setiap loop berjalan.
  - `if` pertama mengontrol LED 1 setiap 1000 ms.
  - `if` kedua mengontrol LED 2 setiap 500 ms.
  - Tanpa `delay()`, sistem dapat menjalankan dua proses secara bersamaan (non-blocking multitasking).





















  
