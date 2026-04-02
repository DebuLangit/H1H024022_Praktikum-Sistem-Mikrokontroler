# 📘 Pertanyaan Praktikum

## 1. Gambarkan rangkaian schematic 5 LED running!
**Jawab:**

<img width="460" height="280" alt="diagram schematic led 5" src="https://github.com/user-attachments/assets/cc87009c-1880-4c37-83f5-4ff49b00529c" />

**Rangkaian terdiri dari:**
- Arduino Uno  
- 5 Resistor 220 ohm  
- 5 LED  

---

## 2. Jelaskan bagaimana program membuat efek LED berjalan dari kiri ke kanan!
**Jawab:**

Program membuat efek LED berjalan dari kiri ke kanan menggunakan **perulangan (`for loop`)**.  
LED dinyalakan satu per satu mulai dari **pin 2 hingga pin 6**, diberi jeda (`delay`), lalu dimatikan sebelum berpindah ke pin berikutnya.

---

## 3. Jelaskan bagaimana program membuat LED kembali dari kanan ke kiri!
**Jawab:**

Program membuat LED berjalan dari kanan ke kiri dengan cara **membalik urutan perulangan**.  
Perulangan dimulai dari **pin 6 ke pin 2**, LED dinyalakan, diberi jeda, lalu dimatikan, sehingga terlihat seperti cahaya bergerak ke kiri.

---

## 4. Buat program LED menyala 3 kiri dan 3 kanan secara bergantian!
**Jawab:**

Karena menggunakan 5 LED, maka terdapat **1 LED tengah (pin 4)** yang menjadi irisan:

- **3 LED kiri**: pin 2, 3, 4  
- **3 LED kanan**: pin 4, 5, 6  

---

## 💡 Program LED Bergantian

```cpp
// Mendeklarasikan pin LED
int led1 = 2; // LED 1 (kanan)
int led2 = 3; // LED 2
int led3 = 4; // LED 3 (tengah)
int led4 = 5; // LED 4
int led5 = 6; // LED 5 (kiri)

void setup() {
  // Mengatur semua pin sebagai OUTPUT
  pinMode(led1, OUTPUT);
  pinMode(led2, OUTPUT);
  pinMode(led3, OUTPUT);
  pinMode(led4, OUTPUT);
  pinMode(led5, OUTPUT);
}

void loop() {

  // ===== FASE 1: 3 LED KANAN MENYALA =====
  digitalWrite(led1, HIGH); // LED 1 ON
  digitalWrite(led2, HIGH); // LED 2 ON
  digitalWrite(led3, HIGH); // LED tengah ON

  digitalWrite(led4, LOW);  // LED 4 OFF
  digitalWrite(led5, LOW);  // LED 5 OFF

  delay(500); // Delay 0.5 detik

  // ===== FASE 2: 3 LED KIRI MENYALA =====
  digitalWrite(led1, LOW);  // LED 1 OFF
  digitalWrite(led2, LOW);  // LED 2 OFF

  digitalWrite(led3, HIGH); // LED tengah tetap ON
  digitalWrite(led4, HIGH); // LED 4 ON
  digitalWrite(led5, HIGH); // LED 5 ON

  delay(500); // Delay 0.5 detik
}
