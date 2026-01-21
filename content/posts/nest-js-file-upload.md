+++
date = '2026-01-18T20:53:02+08:00'
draft = false
title = 'Nest Js File Upload'
+++

Untuk menghandle proses upload file, nest sudah mennyediakan module bawaan berbasis `multer`. Apa itu multer? Multer adalah middleware untuk node ataupun express yang menangani `multipart/form-data`, yang sering digunakan untuk upload file via HTTP `POST`.

Karena nest secara default menggunakan typescript, alangkah baiknya untuk menambah dependency types multer, untuk type safty.

```terminaloutput
bun add -d @types/multer
```


