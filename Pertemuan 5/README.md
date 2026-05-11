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
3. Modifikasilah program dengan menambah sensor (misalnya potensiometer), lalu gunakan nilainya untuk mengontrol kecepatan LED! Bagaimana hasilnya? Jelaskan
   program pada file README.md.

   Jawab:
   Program ini memodifikasi Percobaan 5A (Multitasking) dengan menambahkan komponen sensor analog berupa **Potensiometer**. Nilai resistansi dari potensiometer
   dibaca oleh Arduino (melalui pin ADC) dan dikonversi menjadi rentang waktu. Waktu tersebut kemudian diumpankan ke dalam fungsi `vTaskDelay()` untuk
   mengontrol seberapa cepat sebuah LED berkedip secara dinamis, tanpa menghentikan eksekusi task lainnya.

   ## Hasil Pengamatan Modifikasi Percobaan 5A

   | Kondisi Potensiometer | Nilai Pembacaan ADC (A0) | Estimasi Jeda (dynamicDelay) | Hasil Pengamatan LED 1 (Pin 8) | Hasil Pengamatan LED 2 (Pin 7) | Output Serial (TaskPrint) |
   | :--- | :--- | :--- | :--- | :--- | :--- |
   | **Putar Kiri Penuh (Min)** | 0 | 50 ms | Berkedip sangat cepat | Berkedip stabil (jeda 300ms) | Counter berjalan normal |
   | **Putar Tengah** | ~512 | ~525 ms | Berkedip dengan kecepatan sedang | Berkedip stabil (jeda 300ms) | Counter berjalan normal |
   | **Putar Kanan Penuh (Max)** | 1023 | 1000 ms | Berkedip sangat lambat | Berkedip stabil (jeda 300ms) | Counter berjalan normal |

   ## Analisis Hasil Modifikasi

   - **Efektivitas Fungsi Map**: Program berhasil melakukan pemetaan (*mapping*) nilai mentah ADC dari potensiometer (0-1023) menjadi nilai variabel waktu jeda    (*dynamicDelay*) antara 50ms hingga 1000ms secara *real-time*.
   - **Independensi Task**: Meskipun kecepatan kedip pada LED 1 berubah secara drastis mengikuti putaran potensiometer, durasi kedip pada LED 2 dan kecepatan       pengiriman data *counter* pada Serial Monitor sama sekali tidak terganggu. Hal ini membuktikan bahwa setiap *task* berjalan secara independen dalam alokasi    memorinya sendiri.
   - **Mekanisme Non-Blocking**: Penggunaan `vTaskDelay()` terbukti sangat efektif. Saat `TaskBlink1` berada dalam kondisi menunggu (*blocked*) karena durasi       *delay* yang panjang (1 detik), *Kernel Scheduler* segera mengalihkan sumber daya CPU untuk mengeksekusi `TaskBlink2` dan `Taskprint`. Inilah alasan
     mengapa LED 2 tidak ikut melambat meskipun LED 1 sedang dalam jeda waktu yang lama.
   - **Kesimpulan Analisis**: Implementasi RTOS pada modifikasi ini berhasil menangani input analog yang bersifat dinamis tanpa menimbulkan *blocking* pada          sistem secara keseluruhan, yang merupakan keunggulan utama dibandingkan metode `delay()` konvensional pada Arduino non-OS.



---

## B. Percobaan 5B: Komunikasi Task


1. Apakah kedua task berjalan secara bersamaan atau bergantian? Jelaskan mekanismenya!

3. Apakah program ini berpotensi mengalami race condition? Jelaskan!

4. Modifikasilah program dengan menggunakan sensor DHT sesungguhnya sehingga informasi yang ditampilkan dinamis. Bagaimana hasilnya? Jelaskan program pada file README.md.
