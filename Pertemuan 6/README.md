# <p align="center">Dokumentasi Praktikum</p>
https://github.com/user-attachments/assets/66e88dc7-947a-4f9c-a93c-eb9a4faaf1e2

---

# <p align="center">Menjawab Pertanyaan Praktikum</p>

## A. Percobaan 5A: Multitasking

1. Apakah ketiga task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!

   **Jawab:**
   Ketiga task secara fisik berjalan secara bergantian, bukan bersamaan secara absolut. Hal ini karena Arduino Uno menggunakan mikrokontroler dengan prosesor
   inti tunggal (single-core), sehingga ia hanya bisa mengeksekusi satu instruksi pada satu waktu. Namun, ketiganya terlihat berjalan bersamaan secara kasat
   mata (seolah-olah paralel).

   RTOS menggunakan Kernel Scheduler untuk mengatur pembagian waktu CPU melalui mekanisme time-slicing. Saat sebuah task menjalankan `vTaskDelay()`, task
   tersebut berhenti sementara dan memberi kesempatan CPU menjalankan task lain yang siap dieksekusi. Pergantian antar-task (context switching) terjadi sangat
   cepat sehingga program terlihat berjalan secara bersamaan.
   
2. Bagaimana cara menambahkan task keempat? Jelaskan langkahnya!

   **Jawab:**
   - Deklarasi Prototipe Fungsi: Tambahkan nama fungsi baru di bagian atas kode program sebelum `setup().`
     Contoh: `void TaskBlink3(void *pvParameters);`
   - Registrasi Task ke Scheduler: Di dalam blok setup(), panggil perintah xTaskCreate() untuk mengalokasikan memori dan mendaftarkan task keempat tersebut
     sebelum `vTaskStartScheduler()` dipanggil.
     Contoh: `xTaskCreate(TaskBlink3, "Task4", 128, NULL, 1, NULL);`
   - Definisi Logika Task: Buat blok fungsi di luar `setup()` dan `loop()` yang memuat logika program. Blok ini wajib memiliki perulangan tak terbatas seperti
     `while(1)` atau `for(;;)` dan wajib memuat penundaan non-blocking menggunakan `vTaskDelay()` agar task tidak memonopoli (hogging) CPU.
     Contoh:
     ```cpp
     void TaskBlink3(void *pvParameters) {
        pinMode(6, OUTPUT);
        while(1) {
          digitalWrite(6, HIGH);
          vTaskDelay(400 / portTICK_PERIOD_MS);
          digitalWrite(6, LOW);
          vTaskDelay(400 / portTICK_PERIOD_MS);
        }
      }
     ```
3. Modifikasilah program dengan menambah sensor (misalnya potensiometer), lalu gunakan nilainya untuk mengontrol kecepatan LED! Bagaimana hasilnya? Jelaskan
   program pada file README.md.

   **Jawab:**
   Program ini memodifikasi Percobaan 5A (Multitasking) dengan menambahkan komponen sensor analog berupa **Potensiometer**. Nilai resistansi dari potensiometer
   dibaca oleh Arduino (melalui pin ADC) dan dikonversi menjadi rentang waktu. Waktu tersebut kemudian diumpankan ke dalam fungsi `vTaskDelay()` untuk
   mengontrol seberapa cepat sebuah LED berkedip secara dinamis, tanpa menghentikan eksekusi task lainnya.

   ```cpp
   #include <Arduino_FreeRTOS.h>

   void TaskBlink1( void *pvParameters );
   void TaskBlink2( void *pvParameters );
   void Taskprint( void *pvParameters );

   void setup() {
     // initialize serial communication at 9600 bits per second:
     Serial.begin(9600);
  
     xTaskCreate(
       TaskBlink1
       , "task1"
       , 128
       , NULL
       , 1
       , NULL );
    
     xTaskCreate(
       TaskBlink2
       , "task2"
       , 128
       , NULL
       , 1 
       , NULL );
    
     xTaskCreate(
       Taskprint
       , "task3"
       , 128
       , NULL
       , 1
       , NULL );
    
     vTaskStartScheduler();
   }

   void loop()
   {
   }

   void TaskBlink1(void *pvParameters) {
     pinMode(8, OUTPUT);
     while(1)
     {
       Serial.println("Task1");
    
       // 1. Membaca nilai dari Potensiometer di pin A0 (0 - 1023)
       int potValue = analogRead(A0);
    
       // 2. Memetakan nilai ADC menjadi delay waktu (misal: rentang 50 ms hingga 1000 ms)
       int dynamicDelay = map(potValue, 0, 1023, 50, 1000);
    
       digitalWrite(8, HIGH);
       // 3. Mengganti nilai statis 200 dengan dynamicDelay
       vTaskDelay( dynamicDelay / portTICK_PERIOD_MS ); 
    
       digitalWrite(8, LOW);
       vTaskDelay( dynamicDelay / portTICK_PERIOD_MS );
     }
   }

   void TaskBlink2(void *pvParameters)
   {
     pinMode(7, OUTPUT);
     while(1)
     {
       Serial.println("Task2");
       digitalWrite(7, HIGH);
       vTaskDelay( 300 / portTICK_PERIOD_MS );
       digitalWrite(7, LOW);
       vTaskDelay( 300 / portTICK_PERIOD_MS );
     }
   }

   void Taskprint(void *pvParameters) {
     int counter = 0;
     while(1)
     {
       counter++;
       Serial.print("Counter: "); // Ditambahkan teks agar lebih rapi di Serial Monitor
       Serial.println(counter);
       vTaskDelay(500 / portTICK_PERIOD_MS); 
     }
   }
   ```
   **Analisis Hasil Modifikasi:**
   Hasil modifikasi menunjukkan bahwa fungsi map berhasil mengubah nilai ADC potensiometer menjadi dynamicDelay 50–1000 ms secara real-time. Perubahan tersebut
   memengaruhi kecepatan kedip LED 1 sesuai posisi potensiometer.

   Setiap task pada RTOS berjalan secara independen sehingga perubahan pada LED 1 tidak memengaruhi LED 2 maupun pengiriman data Serial Monitor. Penggunaan
   vTaskDelay() memungkinkan sistem bekerja secara non-blocking, karena saat satu task menunggu, CPU dapat menjalankan task lain.

   Dengan demikian, RTOS berhasil menangani proses dinamis secara efisien dibandingkan metode delay() biasa pada Arduino.


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
