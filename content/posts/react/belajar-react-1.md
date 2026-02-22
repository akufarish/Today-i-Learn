+++
date = '2026-02-08T21:49:07+08:00'
draft = false
title = 'Belajar React (1)'
+++

Yak setelah tidak ada update setelah 18 januari, akhirnya sekarang mencoba untuk aktif lagi. Sebelumnya sudah belajar NestJS yang merupakan salah satu framework back end untuk javascript, sekarang pindah ke front end. Sebenarnya banyak framework front end itu ada banyak seperti vue, svelte, dan lain-lain. Cuma aku milih react biar nanti setelah belajar react dasar bisa lanjut belajar tanstack start karena lagi populer-populer nya itu framework. Isi dari blog ini nantinya ada; apa itu React JS, cara membuat project react, component, dan jsx. **Enjoy**

## Apa Itu React JS?

Apa itu React JS? React JS atau yang dulu dikenal dengan nama FaxJS adalah teknologi yang dikembangkan oleh tim meta di Facebook sekitar tahun 2010 dan bersifat close source. Lalu pada tahun 2013, barulah Facebook membuat React JS menjadi project open source. Saat ini React JS menjadi salah satu framework front end yang paling populer. Karena popularitas React juga jadi banyak framework yang dibuat di atas react seperti NextJS, Tanstack Start, dan Remix.

## Component

Saat belajar react atau framework front end lainnya, pastinya tidak asing dengan istilah component. Apa itu component? Component adalah kumpulan kode untuk membuat _user interface_ secara independen, yang memungkinkan component ini bisa digunakan secara berulang-ulang. Di dalam component bisa terdapat satu atau lebih element HTML, kode JavaScript, dan CSS.

Berikut adalah gambaran dari component itu sendiri

##

<div class="img-container">
<img alt="component-light-diagram" src="/images/react/component-diagram-light.png" class="img-light img">
<img alt="component-dark-diagram" src="/images/react/component-diagram-dark.png" class="img-dark img">
</div>

## Membuat Project

Ada banyak cara untuk membuat project react, bisa menggunakan vite, cli create-react-app, next js, dan react router.

Membuat project menggunakan vite

```terminaloutput
# membuat project react menggunakan javascript
bun create vite@latest nama-project --template react
# membuat project react menggunakan typescript
bun create vite@latest nama-project --template react-ts
```

Berikut adalah struktur folder project yang dibuat oleh vite.

```
├── public/
│   └── vite.svg
├── src/
│   ├── App.jsx
│   ├── assets
│   │   └── react.svg
│   ├── components
│   │   └── HelloWorld.jsx
│   ├── index.css
│   └── main.jsx
├── bun.lock
├── eslint.config.js
├── .gitignore
├── index.html
├── package.json
├── README.md
└── vite.config.js
```

## JSX

Setelah melihat struktur folder dari project react, ada file berformat .jsx yang merupakan react component. Tapi apa itu jsx? Jsx yang merupakan Javascript XML atau Javascript Extension adalah _syntax extension_ yang memungkinkan kita dapat menulis element HTML didalam kode Javascript.

Walaupun jsx memungkinkan kita dalam menulis element HTML didalam Javascript, tapi kita tidak dapat dengan mudah melakukan copy paste kode HTML ke jsx, karena jsx memiliki aturan yang tidak dimiliki oleh HTML. Contohnya adalah saat menggunakan tag element, wajib menggunakan tag penutup.

Contoh tag img

```jsx
// HTML
<img src="" alt="">

// JSX
<img src="" alt="" />
```

Dua aturan lain dari JSX ada component yang hanya boleh mengembalikan satu element, jika ingin mengembalikan beberapa element, bisa dibungkus dalam parent element.

```jsx
// Kode yang benar
<div>
  <p>Hello World</p>
</div>

// Kode yang salah
<div></div>
<p>Hello World</p>
<p>Hello World</p>
```

Aturan terakhir adalah semua attribute menggunakan camelCase. Alasan kenapa jadi camelCase karena attribute element jsx akan dikonversi ke variabel javascript, oleh karena itu nama atribute harus mengikuti aturan pembuatan variabel javascript yang tidak bisa menggunakan - (strip).

```jsx
// Kode yang benar
<p onClick="">Hello World</p>
// Kode yang salah
<p onclick="">Hello World</p>
```

## Hello World

Seperti biasa, ketika belajar bahasa pemrograman, atau framework baru dimulai dari menampilkan teks hello world, dengan membuat component baru dengan nama HelloWorld. Catatan, biasa nya component akan dibuat menggunakan format file .jsx dengan nama sesuai dengan nama komponen nya, lalu didalam file .jsx kita harus membuat default function dengan nama component nya.

Teks hello world disini juga diambil dari variabel javascript, untuk mengakses kode javascript di jsx, bisa menggunakan kurung kurawal.

> HelloWorld.tsx

```jsx
function HelloWorld() {
  const helloWorld = "Hello World";
  return (
    <div>
      <h1>{helloWorld}</h1>
    </div>
  );
}

export default HelloWorld;
```

Sebenarnya cara untuk menampilkan component memerlukan instance dari **React Root**, cuma itu nanti dibahas di blog tersendiri. Untuk sekarang modifikasi file App.jsx

> App.tsx

```jsx
import HelloWorld from "./components/HelloWorld";

function App() {
  return (
    <div>
      <HelloWorld />
    </div>
  );
}

export default App;
```

![Hello World](/images/react/helloWorld.png)
