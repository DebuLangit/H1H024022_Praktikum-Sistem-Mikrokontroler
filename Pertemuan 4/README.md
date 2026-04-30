# <p align="center">Menjawab Pertanyaan Praktikum</p>

## A. Percobaan 1: Analog to Digital Converter (ADC)

1. Apa fungsi perintah `analogRead()` pada rangkaian praktikum ini?

   Jawab:
   Fungsi perintah `analogRead()`: Perintah ini bertugas membaca nilai tegangan fisik yang masuk ke pin analog dari perputaran komponen
   potensiometer. Fungsi ini menerjemahkan sinyal listrik menjadi satuan digital dalam rentang nilai dari 0 hingga 1023 (berdasarkan
   kemampuan 10-bit ADC bawaan Arduino).
   
3. Mengapa diperlukan fungsi map() dalam program tersebut?

   Jawab:
   Kebutuhan fungsi map(): Fungsi map() sangat diperlukan untuk menskalakan proporsi dua tipe data yang memiliki rentang batas berbeda. Pada sistem ini, sensor potensiometer memberikan rentang input (0–1023), tetapi motor servo membutuhkan rentang nilai sudut yang lebih sempit (0–180 derajat). Jika data tidak dilakukan mapping, servo motor tidak akan mampu beroperasi secara fungsional mengikuti proporsi putaran dari potensiometer.
5. Modifikasi program berikut agar servo hanya bergerak dalam rentang 30° hingga 150°, meskipun potensiometer tetap memiliki rentang ADC 0-1023. Jelaskan program pada file README.md
