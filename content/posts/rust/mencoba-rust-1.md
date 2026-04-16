+++
date = '2026-04-15T20:52:08+08:00'
draft = false
title = 'Mencoba Rust (1)'
+++

<!--kenapa belajar rust?-->

### Kenapa memutuskan untuk belajar rust?

Alasannya sebenarnya cukup sederhana, aku penasaran dengan bahasa rust ini sendiri, karena banyak mendengar informasi dari internet kalau rust ini cepat lah, memiliki memory manajemen yang bagus lah. Tapi alasan utama kenapa aku mau belajar rust yaitu ingin mencoba suatu hal baru, aku yang sudah ngoding web selama tiga tahun, mau mencoba hal baru dengan belajar rust ini tadi.

### Kenapa bukan go?

Go masih masuk dalam pertimbangan bahasa yang mau ku pelajari, cuma ya saat ini aku masih mencoba-coba kedua bahasa pemrograman itu, untuk mencari comfort zone ku dari dua bahasa itu ada dimana.

<!--target yang mau dicapai saat selesai belajar rust-->

### Target yang ingin dicapai jika selesai belajar rust

Kalau bicara soal target aku sendiri sudah ada beberapa list project yang mau ku buat setelah belajar rust untuk memperkuat fundamental dan lebih paham pada suatu teknologi, seperti membuat http server yang dimana selain aku belajar semakin mendalam mengenai rust, aku juga bakal belajar alur proses http server itu bagaimana.

Atau mencoba [tauri](https://v2.tauri.app/)? Walaupun ujung-ujungnya bakal nyentuh web framework lagi tapi aku sendiri mau mencoba untuk membuat file explorer atau music player.

<!--hal yang ku suka dari rust saat ini-->

### Hal yang ku suka dari rust untuk saat ini

Rust punya compiler yang strict dan memberikan pesan atau error itu sangat jelas, bahkan kadang diberi saran untuk memperbaiki error seperti menambah keyword A dan warning yang reasonable, seperti memberi tahu kalau ada variabel yang tidak digunakan maka langsung diberi warning, atau hal-hal lainnya yang tidak sesuai aturan compiler rust.

Untuk sekarang yang masih belum nyentuh memory manajemen, dan belum mengenal konsep ownership dan borrowing lebih dalam masih waras lah, kurang tau kedepannya gimana.

#### Catatan

<small>
    Apa yang ku tulis disini tidak mengajar secara dasar dan detail, karena aku mau share pengalaman ku saat belajar rust, dan banyak membandingi nya dengan bahasa pemrograman lain aja.
</small>

<!--apa yang sudah dipelajari saat ini-->

### Main function

Sama seperti bahasa pemrograman C, C++, Go, bahkan Java. Rust itu menggunakan main function sebagai program utama nya.

### Unit test

Rust memiliki fitur test bawaan standard library yang bisa langsung digunakan tanpa perlu nambah library unit test seperti bahasa pemrograman lainnya seperti javascript atau php.

Untuk membuat unit test bisa dengan menambahkan attribute atau kadang disebut annotation test pada function.

```rust
#[test]
fn test_hello() {
    println!("Hello Test");
}
```

### Variabel

Dalam rust untuk membuat variabel bisa menggunakan kata kunci `let`. Yang uniknya rust daripada bahasa pemrograman lain itu variabel nya by default itu immutable yang berarti nilai nya tidak bisa diubah lagi, kalau mau mengubah data pada variabel harus tambahkan kata kunci `mut` setlah kata kunci `let`.

```rust
#[test]
fn test_mutable_variable() {
    let immutable = "Ini immutable";
    // immutable = "tidak bisa diganti nilainya";
    println!("{}", immutable);

    let mut mutable = "ini mutable";
    println!("{}", mutable);
    mutable = "nilainya bisa dirubah";
    println!("{}", mutable);
}
```

Rust adalah bahasa pemrograman yang static typing, yang artinya ketika membuat variabel dengan tipe data tertentu, maka tipe data nya ya itu aja, tidak bisa dirubah, beda dengan php atau javascript yang bisa berubah-rubah tipe data-nya.

Rust juga memiliki fitur shadowing yang baru untuk aku sendiri, karena kita bisa membuat variabel dengan yang sama.

```rust
#[test]
fn test_shadowing_variable() {
    // buat variabel dengan variabel variabel dengan tipe data string
    let variabel = "Ini string";
    println!("{}", variabel);

    // buat variabel dengan variabel yang sama dengan tipe data integer
    let variabel = 13;
    // variabel sudah bukan string lagi,  tapi integer karena kena shadowing
    println!("{}", variabel);
}
```

### Comment

Sama seperti bahasa pemrograman lain dalam rust menggunakan double slash `//` untuk membuat komentar satu baris, dan slash bintang `/* */` untuk membuat komentar lebih dari satu baris.

```rust
/*
 * ini adalah komentar lebih dari satu barus
 * ini adalah komentar lebih dari satu baris
 */
#[test]
fn test_comment() {
    // ini komentar
    println!("Hello"); // ini komentar
}
```
