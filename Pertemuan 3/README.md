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

Jika nilai `num` pada fungsi `displayDigit(num)` lebih dari 15 (misalnya 16, 17, dst.), maka akan terjadi **array out of bounds**.

- Array `digitPattern` hanya memiliki **16 indeks (0–15)**  
- Jika mengakses indeks di luar itu:
  - Mikrokontroler membaca **data acak di memori**
  - Tampilan 7-segment menjadi **tidak beraturan**
  - Berpotensi menyebabkan **program crash**

---

## 3. Modifikasi program agar LED berkedip (blink) ketika menerima input '2' dengan kondisi jika ‘2’ aktif maka LED akan terus berkedip sampai perintah selanjutnya diberikan dan berikan penjelasan disetiap baris kode nya dalam bentuk README.md!


Program ini menggunakan **Common Anode**.

### Alasan:
- **Logika Negasi (`!`)**
  ```cpp
  digitalWrite(segmentPins[i], !digitPattern[num][i]);

---

## 4.  Tentukan apakah menggunakan delay() atau milis()! Jelaskan pengaruhnya terhadap sistem
   
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
---

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

## 4. Modifikasi rangkaian dan program dengan dua push button yang berfungsi sebagai penambahan (increment) dan pengurangan (decrement) pada sistem counter dan berikan penjelasan disetiap baris kode nya

Modifikasi Rangkaian:
Karena kita butuh dua tombol dan masih ingin menggunakan fitur Interrupt (yang sangat responsif), kita akan menggunakan pin 2 dan pin 3. Pada Arduino Uno, hanya pin 2 dan 3 yang mendukung External Interrupt.
- Tombol 1 (Increment) dihubungkan ke Pin 2 dan GND.
- Tombol 2 (Decrement) dihubungkan ke Pin 3 dan GND.

```cpp
// Pin mapping segment: a b c d e f g dp
const int segmentPins[8] = {7, 6, 5, 11, 10, 8, 9, 4};
const int btnUp = 2;   // Interrupt pin 0 untuk tombol Increment
const int btnDown = 3; // Interrupt pin 1 untuk tombol Decrement

volatile int currentDigit = 0;      // Digit saat ini (volatile karena diakses oleh ISR)
volatile bool upPressed = false;    // Flag untuk tombol Increment
volatile bool downPressed = false;  // Flag untuk tombol Decrement

// Pola nyala segmen (1 = Aktif, akan dibalik jadi LOW nanti)
byte digitPattern[16][8] = {
  {1,1,1,1,1,1,0,0}, // 0
  {0,1,1,0,0,0,0,0}, // 1
  {1,1,0,1,1,0,1,0}, // 2
  {1,1,1,1,0,0,1,0}, // 3 
  {0,1,1,0,0,1,1,0}, // 4
  {1,0,1,1,0,1,1,0}, // 5
  {1,0,1,1,1,1,1,0}, // 6
  {1,1,1,0,0,0,0,0}, // 7
  {1,1,1,1,1,1,1,0}, // 8
  {1,1,1,1,0,1,1,0}, // 9
  {1,1,1,0,1,1,1,0}, // A
  {0,0,1,1,1,1,1,0}, // b
  {1,0,0,1,1,1,0,0}, // C
  {0,1,1,1,1,0,1,0}, // d
  {1,0,0,1,1,1,1,0}, // E
  {1,0,0,0,1,1,1,0}  // F
};

// Fungsi untuk menampilkan digit (dibalik dengan '!' untuk Common Anode)
void displayDigit(int num) {
  for(int i=0; i<8; i++) {
    digitalWrite(segmentPins[i], !digitPattern[num][i]); 
  }
}

// Interrupt Service Routine (ISR) untuk tombol naik
void btnUpISR() {
  upPressed = true;
}

// Interrupt Service Routine (ISR) untuk tombol turun
void btnDownISR() {
  downPressed = true;
}

void setup() {
  for(int i=0; i<8; i++) {
    pinMode(segmentPins[i], OUTPUT);
  }
  
  pinMode(btnUp, INPUT_PULLUP);    // Tombol naik menggunakan pull-up internal
  pinMode(btnDown, INPUT_PULLUP);  // Tombol turun menggunakan pull-up internal
  
  // Mengaitkan pin dengan fungsi ISR, dipicu saat sinyal FALLING (HIGH ke LOW)
  attachInterrupt(digitalPinToInterrupt(btnUp), btnUpISR, FALLING);
  attachInterrupt(digitalPinToInterrupt(btnDown), btnDownISR, FALLING);
}

void loop() {
  // Mengecek apakah tombol Increment ditekan
  if(upPressed) {
    currentDigit = (currentDigit + 1) % 16; // Ditambah 1. Jika 15+1 akan kembali ke 0
    upPressed = false; // Reset flag
    delay(200);        // Debouncing sederhana agar tidak terjadi multiple trigger
  }
  
  // Mengecek apakah tombol Decrement ditekan
  if(downPressed) {
    currentDigit = (currentDigit - 1); // Dikurangi 1
    if(currentDigit < 0) {
      currentDigit = 15; // Jika kurang dari 0, putar kembali ke F (15)
    }
    downPressed = false; // Reset flag
    delay(200);          // Debouncing sederhana
  }
  
  // Menampilkan digit saat ini di 7-segment secara terus menerus
  displayDigit(currentDigit);
}
```


