+++
date = '2026-04-05T21:45:37+08:00'
draft = false
title = 'Perkenalan Go'
+++

Go atau yang kadang disebut golang adalah bahasa pemrograman yang dibuat dan dikembangkan oleh Robbert Griesmer, Rob Pike, dan Ken Thompson di google pada tahun 2007, dan di rilis ke publik sebagai open soure pada tahun 2009.

Saat ini go menjadi salah satu bahasa pemrograman populer untuk membuat backend api, dan juga merupakan bahasa pemrograman dari teknologi yang populer seperti docker, dan cockroach db.

### **Kelebihan go**

- **Simplicity & Efficiency**. Go sendiri bisa dibilang bahasa pemrograman yang sederhana dan mudah dipelajari yang memungkinkan untuk tidak membutuhkan waktu yang lama untuk mempelajarinya, karena go tidak memiliki konsep oop.
- **Memiliki Garbage Collector**. Go memiliki garbage collector sehingga kita tidak perlu melakukan management memory secara manual sepert bahasa pemrograman C atau C++.
- **Concurrency**. Go mendukung concurrency programming dengan adanya goroutine yang memungkinkan program untuk menangani beberapa tugas dalam waktu bersamaan.

### **Proses Development Program Go**

Source code go akan dibuat menggunakan extensi file .go, yang nantinya source code tersebut akan di compile oleh go compiler untuk menghasilkan binary file untuk sistem operasi yang sedang digunakan saat ini, bisa compile untuk windows, mac os, atau linux.

<div class="img-container">
<img alt="go-light-diagram" src="/images/go/alur-go-light.png" class="img-light img">
<img alt="go-dark-diagram" src="/images/go/alur-go-dark.png" class="img-dark img">
</div>

### **Instalasi Go**

Go compiler dapat diinstal dari website [https://go.dev/doc/install](https://go.dev/doc/install) lalu pilih sesuai sistem operasi yang digunakan dan ikuti tahapan instalasi-nya, jikalau sudah selesai meng-install go compiler, untuk mencek apakah sudah terinstall atau belum bisa gunakan command `go version` pada terminal.

![Go Version](/images/go/go-version.png)
