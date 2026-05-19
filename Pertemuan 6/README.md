# <p align="center">Dokumentasi Praktikum</p>
https://github.com/user-attachments/assets/66e88dc7-947a-4f9c-a93c-eb9a4faaf1e2

---

# <p align="center">Menjawab Pertanyaan Praktikum</p>

## Percobaan 6A: External Interrupt

1. Jelaskan proses bagaimana tombol dapat mengubah kondisi LED menggunakan
interrupt!

   **Jawab:**
   Tombol dapat mengubah kondisi LED menggunakan hardware interrupt. Pin 2 diatur sebagai INPUT_PULLUP, sehingga normalnya bernilai HIGH. Saat tombol ditekan, tegangan berubah dari HIGH ke LOW dan memicu mode FALLING. Mikrokontroler kemudian menghentikan sementara fungsi loop() untuk menjalankan ISR tombolInterrupt() yang mengubah nilai ledState. Setelah itu, program kembali ke loop() dan menjalankan digitalWrite(13, ledState) untuk mengubah kondisi LED.
   
2. Apa fungsi attachInterrupt() pada program tersebut?

   **Jawab:**
   Fungsi attachInterrupt() digunakan untuk menghubungkan interupsi pada Pin 2 dengan fungsi tombolInterrupt. Saat terjadi perubahan sinyal turun (FALLING), mikrokontroler langsung menjalankan fungsi ISR tersebut.
  
3. Mengapa pada ISR tidak disarankan menggunakan delay() dan Serial.print()?

   **Jawab:**
   ISR harus dibuat sesingkat mungkin karena proses utama program akan berhenti sementara saat ISR berjalan. Fungsi seperti delay() dan Serial.print() sebaiknya tidak digunakan karena bersifat lambat dan blocking, sehingga dapat menyebabkan mikrokontroler hang atau melewatkan interupsi penting lainnya.

4. Apa fungsi keyword volatile pada variabel ledState?

   **Jawab:**
   Keyword volatile berfungsi agar compiler selalu membaca nilai variabel terbaru dari RAM karena nilainya dapat berubah sewaktu-waktu, misalnya oleh interupsi hardware. Hal ini mencegah compiler menggunakan nilai lama yang tersimpan di cache.
   
5. Pada percobaan digunakan mode interrupt FALLING. Modifikasikan program menggunakan mode interrupt lain (RISING, CHANGE, atau LOW) kemudian:

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

## B. Percobaan 5B: Komunikasi Task


1. Apakah kedua task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!

   **Jawab:**
   Secara fisik pada tingkat perangkat keras, kedua task (read_data dan display) berjalan secara bergantian. Arduino Uno menggunakan mikrokontroler single
   core, sehingga ia hanya dapat memproses satu instruksi pada satu waktu. Namun, pada tingkat perangkat lunak, keduanya terlihat berjalan secara bersamaan
   (concurrent).

   Mekanismenya:
   Hal ini dimungkinkan oleh Kernel Scheduler dari FreeRTOS. Scheduler membagi waktu prosesor dengan sangat cepat (time-slicing) dan melakukan context
   switching.
   - Saat task read_data sedang mengemas data dan mengirimkannya ke Queue, task tersebut memegang kendali CPU.
   - Setelah mengeksekusi vTaskDelay() atau ketika task display sedang menunggu data dengan xQueueReceive (menggunakan parameter portMAX_DELAY), task tersebut
     masuk ke status Blocked (menunggu).
   - Pada momen Blocked inilah, Scheduler langsung mengalihkan kendali CPU ke task lain yang statusnya sudah Ready, sehingga keduanya seolah-olah berjalan
     paralel tanpa saling memblokir.

2. Apakah program ini berpotensi mengalami race condition? Jelaskan!

   **Jawab:** Tidak, program ini tidak berpotensi mengalami race condition.
   Race condition terjadi ketika beberapa task mengakses dan mengubah variabel yang sama secara bersamaan sehingga data menjadi tidak valid. Pada Percobaan 5B,
   masalah ini dicegah dengan penggunaan Queue (xQueueSend dan xQueueReceive) yang bersifat thread-safe. Queue mengatur pertukaran data antara task secara aman
   sehingga tidak ada akses langsung ke variabel yang sama pada waktu bersamaan.
   
3. Modifikasilah program dengan menggunakan sensor DHT sesungguhnya sehingga informasi yang ditampilkan dinamis. Bagaimana hasilnya? Jelaskan program pada file
   README.md.

   **Jawab:**
   ```cpp
   #include <Arduino_FreeRTOS.h>
   #include <queue.h>
   #include <DHT.h>

   // Konfigurasi pin dan tipe sensor DHT
   #define DHTPIN 2
   #define DHTTYPE DHT22

   // Inisialisasi objek DHT
   DHT dht(DHTPIN, DHTTYPE);

   // Struktur data untuk menyimpan suhu dan kelembapan
   // Menggunakan float karena DHT22 membaca nilai desimal
   struct readings {
     float temp;
     float h;
   };

   QueueHandle_t my_queue;

   void setup() {
     Serial.begin(9600);
     dht.begin();

     // Membuat antrean (Queue) berkapasitas 1 elemen dengan ukuran struktur 'readings'
     my_queue = xQueueCreate(1, sizeof(struct readings));

     // Validasi: Cek apakah Queue berhasil dibuat sebelum menjalankan task
     if (my_queue != NULL) {
       xTaskCreate(read_data, "Read DHT", 128, NULL, 1, NULL);
       xTaskCreate(display, "Display", 128, NULL, 1, NULL);
     } else {
       Serial.println("Gagal membuat Queue!");
     }
   }

   void loop() {
     // Dikosongkan karena eksekusi diambil alih oleh Scheduler FreeRTOS
   }

   // Task 1: Membaca Sensor (Produsen)
   void read_data(void *pvParameters) {
     struct readings x;
  
     for (;;) {
       // Membaca data kelembapan dan suhu dari sensor
       x.h = dht.readHumidity();
       x.temp = dht.readTemperature();

       // Validasi pembacaan sensor (mencegah pengiriman data error)
       if (isnan(x.h) || isnan(x.temp)) {
         Serial.println("Gagal membaca dari sensor DHT22!");
       } else {
         // Jika berhasil, kirim struktur data ke Queue
         xQueueSend(my_queue, &x, portMAX_DELAY);
       }
    
       // Spesifikasi hardware DHT22 mewajibkan jeda minimal 2000 ms antar-pembacaan
       vTaskDelay(2000 / portTICK_PERIOD_MS);
     }
   }

   // Task 2: Menampilkan Data (Konsumen)
   void display(void *pvParameters) {
     struct readings x;
  
     for (;;) {
       // Menunggu hingga ada data yang masuk ke Queue, lalu mengambilnya
       if (xQueueReceive(my_queue, &x, portMAX_DELAY) == pdPASS) {
         Serial.print("Suhu = ");
         Serial.print(x.temp);
         Serial.print(" °C  |  Kelembapan = ");
         Serial.print(x.h);
         Serial.println(" %");
       }
     }
   }
   ```
   | Waktu Eksekusi | Aksi Task `read_data` (Produsen) | Output Task `display` (Konsumen di Serial Monitor) | Status Antrean (Queue) |
   | :--- | :--- | :--- | :--- |
   | **0 detik** | Membaca sensor & mengirim data | `Suhu = 28.50 °C | Kelembapan = 65.20 %` | Kosong (Data langsung diambil) |
   | **2 detik** | Membaca sensor & mengirim data | `Suhu = 28.50 °C | Kelembapan = 65.20 %` | Kosong (Data langsung diambil) |
   | **4 detik** | Membaca sensor & mengirim data | `Suhu = 28.60 °C | Kelembapan = 65.30 %` | Kosong (Data langsung diambil) |
   | **Simulasi Error**| Deteksi fungsi `isnan()` | `Gagal membaca dari sensor DHT22!` | Tetap Kosong |
