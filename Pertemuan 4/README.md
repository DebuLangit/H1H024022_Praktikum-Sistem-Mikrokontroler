# <p align="center">Dokumentasi Praktikum</p>
## Percobaan 1: Analog to Digital Converter (ADC)
https://github.com/user-attachments/assets/b6be53a1-9c7e-46c8-985d-65e5a6afcd52
## Percobaan 2: Pulse Width Modulation (PWM)
https://github.com/user-attachments/assets/d1e65904-4c0b-4634-9a74-8413c75f2ce2

---

# <p align="center">Menjawab Pertanyaan Praktikum</p>

## A. Percobaan 1: Analog to Digital Converter (ADC)

1. Apa fungsi perintah `analogRead()` pada rangkaian praktikum ini?

   Jawab:
   Fungsi perintah `analogRead()`: Perintah ini bertugas membaca nilai tegangan fisik yang masuk ke pin analog dari perputaran komponen potensiometer. Fungsi ini menerjemahkan sinyal listrik menjadi satuan digital dalam rentang nilai dari 0 hingga 1023 (berdasarkan kemampuan 10-bit ADC bawaan Arduino).
   
3. Mengapa diperlukan fungsi map() dalam program tersebut?

   Jawab:
   Kebutuhan fungsi `map()`: Fungsi `map()` sangat diperlukan untuk menskalakan proporsi dua tipe data yang memiliki rentang batas berbeda. Pada sistem ini, sensor potensiometer memberikan rentang input (0–1023), tetapi motor servo membutuhkan rentang nilai sudut yang lebih sempit (0–180 derajat). Jika data tidak dilakukan mapping, servo motor tidak akan mampu beroperasi secara fungsional mengikuti proporsi putaran dari potensiometer.
   
5. Modifikasi program berikut agar servo hanya bergerak dalam rentang 30° hingga 150°, meskipun potensiometer tetap memiliki rentang ADC 0-1023. Jelaskan program pada file README.md

   Jawab:
   Modifikasi Program: Untuk membatasi pergerakan dari servo di dalam rentang 30° hingga 150°, instruksi sintaks yang digunakan adalah:
   ```cpp
   pos = map(val, 0, 1023, 30, 150);
   ```
   Penjelasan: Fungsi tersebut dimodifikasi pada dua parameter terakhir batas keluarannya. Dengan format instruksi di atas, pada saat potensiometer mentransmisikan output maksimal (1023), sudut motor servo hanya akan mencapai batas maksimal 150 derajat, begitu pula untuk batas terbawah (0) yang akan direpresentasikan di sudut 30 derajat.




























   
