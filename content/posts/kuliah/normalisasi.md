+++
date = '2026-02-09T15:47:15+08:00'
draft = false
title = 'Normalisasi'
tags = ['Indonesia', 'Kuliah', 'Basis Data']
+++

Normalisasi 1NF, 2NF, 3NF

## Bentuk Normal ke Satu (1NF)

syarat:

1. Tidak ada set atribut yang berulang atau bernilai ganda
2. Telah ditentukanya primary key untuk tabel atau relasi.
3. Tiap atribut hanya memiliki satu pengertian.
4. Tiap atribut dapat memillki banyak nilai sebenarnya menggambarkan entitas atau relasi yang terpisah.

| NIM      | Nama | 
| -------- | ---- |
| A0213213 | Andi |
| A0213213 | Andi |

## Bentuk Normal ke Dua (2NF)

1. Bentuk data telah memenuhi kriteria bentuk normal ke satu
2. Atribut bukan konci (non-key attribute) haruslah memiliki ketergantungan fungsional sepenuhnya pada primary key
