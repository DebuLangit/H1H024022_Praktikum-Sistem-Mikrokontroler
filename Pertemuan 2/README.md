# PERCOBAAN 2A: SEVEN SEGMENT 

## 1. Rangkaian Schematic
Gambarkan rangkaian schematic yang digunakan pada percobaan:

<p align="center">
  <img width="704" alt="Schematic Seven Segment" src="https://github.com/user-attachments/assets/0a2f618c-bc1c-4227-8d56-fb49938a7539" />
</p>

---

## 2. Apa yang terjadi jika nilai `num` lebih dari 15?

Jika nilai `num` pada fungsi `displayDigit(num)` lebih dari 15 (misalnya 16, 17, dst.), maka akan terjadi **array out of bounds**.

- Array `digitPattern` hanya memiliki **16 indeks (0–15)**  
- Jika mengakses indeks di luar itu:
  - Mikrokontroler membaca **data acak di memori**
  - Tampilan 7-segment menjadi **tidak beraturan**
  - Berpotensi menyebabkan **program crash**

---

## 3. Jenis Seven Segment yang digunakan

Program ini menggunakan **Common Anode**.

### Alasan:
- **Logika Negasi (`!`)**
  ```cpp
  digitalWrite(segmentPins[i], !digitPattern[num][i]);

---

## 4.  Modifikasi program agar tampilan berjalan dari F ke 0 dan penjelasan disetiap baris kode
   
```cpp
// 7-Segment Common Anode

// Mendeklarasikan pin Arduino yang terhubung ke segmen a, b, c, d, e, f, g, dp
const int segmentPins[8] = {7, 6, 5, 11, 10, 8, 9, 4};

// Membuat matriks 16x8 berisi pola nyala segmen untuk angka 0-9 dan huruf A-F
// Angka 1 merepresentasikan segmen yang seharusnya "Aktif"
byte digitPattern[16][8] = {
  {1,1,1,1,1,1,0,0}, // Indeks 0: Pola angka 0
  {0,1,1,0,0,0,0,0}, // Indeks 1: Pola angka 1
  {1,1,0,1,1,0,1,0}, // Indeks 2: Pola angka 2
  {1,1,1,1,0,0,1,0}, // Indeks 3: Pola angka 3 
  {0,1,1,0,0,1,1,0}, // Indeks 4: Pola angka 4
  {1,0,1,1,0,1,1,0}, // Indeks 5: Pola angka 5
  {1,0,1,1,1,1,1,0}, // Indeks 6: Pola angka 6
  {1,1,1,0,0,0,0,0}, // Indeks 7: Pola angka 7
  {1,1,1,1,1,1,1,0}, // Indeks 8: Pola angka 8
  {1,1,1,1,0,1,1,0}, // Indeks 9: Pola angka 9
  {1,1,1,0,1,1,1,0}, // Indeks 10: Pola huruf A
  {0,0,1,1,1,1,1,0}, // Indeks 11: Pola huruf b
  {1,0,0,1,1,1,0,0}, // Indeks 12: Pola huruf C
  {0,1,1,1,1,0,1,0}, // Indeks 13: Pola huruf d
  {1,0,0,1,1,1,1,0}, // Indeks 14: Pola huruf E
  {1,0,0,0,1,1,1,0}  // Indeks 15: Pola huruf F
};

// Fungsi kustom untuk menampilkan digit ke 7-segment berdasarkan parameter 'num'
void displayDigit(int num)
{
  // Memulai perulangan sebanyak 8 kali untuk mengakses ke-8 pin segmen
  for(int i=0; i<8; i++)
  {
    // Mengirimkan sinyal ke pin. Operator '!' membalik nilai logika array 
    // (1 menjadi LOW/0, 0 menjadi HIGH/1) karena Common Anode aktif dengan logika LOW.
    digitalWrite(segmentPins[i], !digitPattern[num][i]); 
  }
}

// Fungsi inisialisasi yang dijalankan satu kali saat program pertama dimulai
void setup()
{
  // Memulai perulangan untuk mengatur mode pin
  for(int i=0; i<8; i++)
  {
    // Mengatur seluruh pin yang ada di array segmentPins sebagai OUTPUT
    pinMode(segmentPins[i], OUTPUT);
  }
}

// Fungsi utama yang akan dijalankan berulang-ulang tanpa henti
void loop()
{
  // [MODIFIKASI] Perulangan hitung mundur, dimulai dari indeks 15 (F) hingga 0
  // i-- berarti nilai i akan dikurangi 1 setiap perulangan selesai
  for(int i=15; i>=0; i--)
  {
    // Memanggil fungsi displayDigit untuk menampilkan karakter sesuai nilai i saat ini
    displayDigit(i);
    // Menahan program selama 1000 milidetik (1 detik) sebelum lanjut ke karakter berikutnya
    delay(1000);
  }
}
```
# PERCOBAAN 2B: KONTROL COUNTER DENGAN PUSH BUTTON

## 1. Rangkaian Schematic
Gambarkan rangkaian schematic yang digunakan pada percobaan:

<p align="center">
  <img width="673" height="345" alt="image" src="https://github.com/user-attachments/assets/737b9897-e80f-4e65-9c2c-b2f2cb39753d" />
</p>

---

## 2. Mengapa pada push button digunakan mode INPUT_PULLUP pada Arduino Uno? Apa keuntungannya dibandingkan rangkaian biasa?
Mode INPUT_PULLUP digunakan untuk mengaktifkan resistor pull-up internal yang ada di dalam mikrokontroler Arduino.
- Saat tombol tidak ditekan, pin akan membaca nilai HIGH (1).
- Saat tombol ditekan (dihubungkan ke GND), pin akan membaca nilai LOW (0).
Ini mencegah pin dalam kondisi floating (mengambang/tidak menentu) yang bisa menangkap noise elektromagnetik dan menyebabkan pembacaan acak.

Keuntungannya:
- Keuntungan utamanya adalah menghemat komponen hardware. Kamu tidak perlu repot-repot menambahkan resistor pull-up atau pull-down eksternal di breadboard.
- Rangkaian menjadi jauh lebih sederhana karena tombol hanya perlu dihubungkan langsung antara pin digital Arduino dan pin GND (Ground).

---

## 3. Jika salah satu LED segmen tidak menyala, apa saja kemungkinan penyebabnya dari sisi hardware maupun software?
Jika hanya satu segmen yang mati sementara yang lain normal, masalahnya bersifat spesifik pada jalur segmen tersebut.
- Dari sisi Hardware:
  - LED Segmen Putus/Terbakar: Dioda LED di dalam komponen 7-segment untuk segmen tersebut sudah rusak.
  - Kabel Jumper Kendur/Putus: Kabel yang menghubungkan pin Arduino ke pin segmen tersebut tidak menancap dengan baik di breadboard atau putus di dalam.
  - Resistor Rusak: Jika kamu menggunakan resistor pembatas arus pada masing-masing pin segmen, resistor untuk jalur tersebut mungkin rusak atau tidak terhubung sempurna.
  - Pin Arduino Rusak: Pin digital pada Arduino yang ditugaskan untuk segmen tersebut (misal pin 7 untuk segmen 'a') mengalami kerusakan internal.
- Dari sisi Software:
  - Kesalahan Deklarasi Array digitPattern: Terdapat kesalahan pengetikan pola bit (angka 0 atau 1) pada matriks digitPattern. Misalnya, segmen yang seharusnya menyala (angka 1) malah tertulis mati (angka 0).
  - Kesalahan Pin Mapping: Urutan pin pada segmentPins tertukar atau ditulis ke pin yang tidak terhubung dengan benar.
  - Lupa di-pinMode: Jika menggunakan modifikasi kode lain yang mendefinisikan pin satu per satu (bukan looping), bisa jadi pin tersebut terlewat diset sebagai OUTPUT.







