+++
date = '2025-12-31T13:45:27+08:00'
draft = false
title = 'Mencoba Nest Js (1)'
+++

### Introduction

Untuk menghabiskan waktu liburku, aku berencana untuk meningkatkan skill dengan belajar nestjs, sebagai seseorang yang pernah menggunakan laravel selama 2 tahun. Dimulai dari belajar nest js dasar dari video pemrograman zaman now. Yang dimana pada video itu mas Eko (Programmer Zaman Now) menjelaskan alasan kenapa menggunakan framework yang menurut saya itu menarik. Karena beliau mengatakan karena menggunakan framework lah kita sebagai developer itu memiliki sebuah standar yang harus diikuti yang memiliki nilai positif karena kemudahan untuk mewariskan project yang sudah ada karena developer di masa depan yang akan datang akan mudah beradaptasi dengan code yang sudah ada karena mengikuti standar dari suatu framework.

### Struktur Folder
Nest js memiliki struktur folder yang sederhana by default nya. Dimana cuma memiliki dua folder utama yaitu src/ (source) dan test/ . folder src yang berisi file-file codingan kita seperti controller, unit test dll, dan folder test yang berisi end to end testing.

### Decorator
Ini mungkin kali kedua saya menggunakan decorator dalam programming, dimana sebelumnya saya sudah pernah menggunakan decorator saat membuat aplikasi android menggunakan kotlin yang mengadaptasi clean architecture mvvm, dimana decorator digunakan untuk dagger. Jujur saya dulu belum paham fungsi dan kegunaan decorator itu sendiri untuk apa, dan baru kali ini tau decorator dari penjelasan singkat yang diberikan mas Eko. Dimana beliau mengatakan kalau decorator adalah fitur untuk menambahkan metadata (informasi tambahan) yang dikenali oleh kode program, berbeda dengan komentar yang kode program hiraukan decorator itu dibaca.

### Module
Nest js membagi struktur aplikasi menjadi modular dengan konsep module, Dalam aplikasi nest js pasti memiliki module utama atau defaultnya appliaction module, yang didalamnya nanti melakukan import module-module lainnya