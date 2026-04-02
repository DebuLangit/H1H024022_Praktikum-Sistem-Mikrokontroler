# Pertanyaan Praktikum

1. Gambarkan rangkaian schematic 5 LED running yang digunakan pada percobaan!
   Jawab:
   <img width="460" height="280" alt="diagram schematic led 5" src="https://github.com/user-attachments/assets/cc87009c-1880-4c37-83f5-4ff49b00529c" />

   Rangkaian terdiri dari:
   - Arduino Uno
   - 5 Resistor 220 ohm
   - 5 LED 
3. Jelaskan bagaimana program membuat efek LED berjalan dari kiri ke kanan!
   Jawab:
   Program membuat efek LED berjalan dari kiri ke kanan menggunakan perulangan (for loop). LED dinyalakan satu per satu mulai dari pin 2 hingga pin 6,
   diberi jeda (delay), lalu dimatikan sebelum pindah ke pin berikutnya.
4. Jelaskan bagaimana program membuat LED kembali dari kanan ke kiri!
   Jawab: 
   Program membuat LED berjalan dari kanan ke kiri dengan cara membalik urutan. Perulangan dimulai dari pin paling kanan (Pin 6), lalu LED dinyalakan,
   diberi jeda, dan dimatikan. Setelah itu, proses berpindah ke pin di sebelah kiri hingga mencapai Pin 2, sehingga terlihat seperti cahaya bergerak ke
   kiri.
5. Buatkan program agar LED menyala tiga LED kanan dan tiga LED kiri secara bergantian dan berikan penjelasan disetiap baris kode nya dalam bentuk
   README.md!
   Jawab: 
   Karena menggunakan 5 LED, maka pembagian "3 LED Kiri" dan "3 LED Kanan" akan membuat 1 LED di tengah (Pin 4) saling beririsan.
   - 3 LED Kiri: Pin 2, 3, dan 4.
   - 3 LED Kanan: Pin 4, 5, dan 6
   ```md
   ## Program LED 

   ```cpp
   // Mendeklarasikan nama pin agar lebih mudah dibaca
   int led1 = 2; // LED paling kanan dihubungkan ke Pin 2
   int led2 = 3; // LED kedua dihubungkan ke Pin 3
   int led3 = 4; // LED ketiga (tengah) dihubungkan ke Pin 4
   int led4 = 5; // LED keempat dihubungkan ke Pin 5
   int led5 = 6; // LED paling kiri dihubungkan ke Pin 6

   void setup() {
     // Mengatur semua pin LED tersebut agar berfungsi sebagai OUTPUT (mengeluarkan arus)
     pinMode(led1, OUTPUT);
     pinMode(led2, OUTPUT);
     pinMode(led3, OUTPUT);
     pinMode(led4, OUTPUT);
     pinMode(led5, OUTPUT);
   }

   void loop() {
     // --- FASE 1: Menyalakan 3 LED Kanan ---
  
     digitalWrite(led1, HIGH); // Menyalakan LED 1 (Pin 2)
     digitalWrite(led2, HIGH); // Menyalakan LED 2 (Pin 3)
     digitalWrite(led3, HIGH); // Menyalakan LED 3 (Pin 4 - Tengah)
  
     digitalWrite(led4, LOW);  // Memastikan LED 4 mati
     digitalWrite(led5, LOW);  // Memastikan LED 5 mati
  
     delay(500);               // Menahan kondisi ini selama 500 milidetik (setengah detik)


     // --- FASE 2: Menyalakan 3 LED Kiri ---
  
     digitalWrite(led1, LOW);  // Mematikan LED 1
     digitalWrite(led2, LOW);  // Mematikan LED 2
  
     digitalWrite(led3, HIGH); // Menyalakan LED 3 (Pin 4 - Tengah). Karena ini irisan, ia tetap menyala.
     digitalWrite(led4, HIGH); // Menyalakan LED 4 (Pin 5)
     digitalWrite(led5, HIGH); // Menyalakan LED 5 (Pin 6)
  
     delay(500);               // Menahan kondisi ini selama 500 milidetik sebelum kembali ke atas (loop)
   }

