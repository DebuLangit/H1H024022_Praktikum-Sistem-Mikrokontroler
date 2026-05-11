# <p align="center">Dokumentasi Praktikum</p>
## Percobaan 5A: Multitasking
https://github.com/user-attachments/assets/9f792b8f-9c9b-4bb9-adce-95ebb525d9a3
## Percobaan 5B: Komunikasi Task
(isi link video)

---

# <p align="center">Menjawab Pertanyaan Praktikum</p>

## A. Percobaan 5A: Multitasking

1. Apakah ketiga task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!

   Jawab:
   Ketiga task secara fisik berjalan secara bergantian, bukan bersamaan secara absolut. Hal ini karena Arduino Uno menggunakan mikrokontroler dengan prosesor
   inti tunggal (single-core), sehingga ia hanya bisa mengeksekusi satu instruksi pada satu waktu. Namun, ketiganya terlihat berjalan bersamaan secara kasat
   mata (seolah-olah paralel).

   RTOS menggunakan Kernel Scheduler untuk mengatur pembagian waktu CPU melalui mekanisme time-slicing. Saat sebuah task menjalankan `vTaskDelay()`, task
   tersebut berhenti sementara dan memberi kesempatan CPU menjalankan task lain yang siap dieksekusi. Pergantian antar-task (context switching) terjadi sangat
   cepat sehingga program terlihat berjalan secara bersamaan.
   
2. Bagaimana cara menambahkan task keempat? Jelaskan langkahnya!

    Jawab:
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
5. Modifikasilah program dengan menambah sensor (misalnya potensiometer), lalu gunakan nilainya untuk mengontrol kecepatan LED! Bagaimana hasilnya? Jelaskan program pada file README.md.

1. Apa fungsi perintah `analogRead()` pada rangkaian praktikum ini?

   Jawab:
   Fungsi perintah `analogRead()`: Perintah ini bertugas membaca nilai tegangan fisik yang masuk ke pin analog dari perputaran komponen potensiometer. Fungsi ini menerjemahkan sinyal listrik menjadi satuan digital dalam rentang nilai dari 0 hingga 1023 (berdasarkan kemampuan 10-bit ADC bawaan Arduino).
   
2. Mengapa diperlukan fungsi map() dalam program tersebut?

   Jawab:
   Kebutuhan fungsi `map()`: Fungsi `map()` sangat diperlukan untuk menskalakan proporsi dua tipe data yang memiliki rentang batas berbeda. Pada sistem ini, sensor potensiometer memberikan rentang input (0–1023), tetapi motor servo membutuhkan rentang nilai sudut yang lebih sempit (0–180 derajat). Jika data tidak dilakukan mapping, servo motor tidak akan mampu beroperasi secara fungsional mengikuti proporsi putaran dari potensiometer.
   
3. Modifikasi program berikut agar servo hanya bergerak dalam rentang 30° hingga 150°, meskipun potensiometer tetap memiliki rentang ADC 0-1023. Jelaskan program pada file README.md

   Jawab:
   Modifikasi Program: Untuk membatasi pergerakan dari servo di dalam rentang 30° hingga 150°, instruksi sintaks yang digunakan adalah:
   ```cpp
   pos = map(val, 0, 1023, 30, 150);
   ```
   Penjelasan: Fungsi tersebut dimodifikasi pada dua parameter terakhir batas keluarannya. Dengan format instruksi di atas, pada saat potensiometer mentransmisikan output maksimal (1023), sudut motor servo hanya akan mencapai batas maksimal 150 derajat, begitu pula untuk batas terbawah (0) yang akan direpresentasikan di sudut 30 derajat.

---

## B. Percobaan 5B: Komunikasi Task


1. Apakah kedua task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!

3. Apakah program ini berpotensi mengalami race condition? Jelaskan!

4. Modifikasilah program dengan menggunakan sensor DHT sesungguhnya sehingga informasi yang ditampilkan dinamis. Bagaimana hasilnya? Jelaskan program pada file README.md.
