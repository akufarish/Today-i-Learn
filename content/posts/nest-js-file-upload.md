+++
date = '2026-01-18T20:53:02+08:00'
draft = false
title = 'Nest Js File Upload'
+++

Untuk menghandle proses upload file, nest sudah mennyediakan module bawaan berbasis `multer`. Apa itu multer? Multer adalah middleware untuk node ataupun express yang menangani `multipart/form-data`, yang sering digunakan untuk upload file via HTTP `POST`.

Karena nest secara default menggunakan typescript, alangkah baiknya untuk menambah dependency types multer, untuk type safety.

```terminaloutput
bun add -d @types/multer
```

Setelah menginstall dependency diatas, maka kita bisa memberikan tipe `Express.Multer.File` ke variabel yang menyimpan file yang akan diupload nantinya.

Membuat handler degan route `/api/v1/upload` dengan http method post.

> app.controller.ts

```typescript
@Post('/api/v1/upload')
@UseInterceptors(FileInterceptor('image'))
async uploadFile(@UploadedFile() image: Express.Multer.File) {
    return this.appService.handleFileUpload(image);
}
```

Untuk mengupload satu file, bisa menggunakan interceptor `FileInterceptor` yang akan menghandle request file menggunakan decorator `@@UploadedFile`

```typescript
@UseInterceptors(FileInterceptor('image'))
async uploadFile(@UploadedFile() image: Express.Multer.File)
```

Pada service layer, buat fungsi dengan nama handleFileUpload yang menerima satu parameter image dengan tipe `Express.Multer.File`.

> app.service.ts

```typescript
async handleFileUpload(image: Express.Multer.File) {
    const uploadDir = './images';
    const fileExtension = path.extname(image.originalname).toLowerCase();
    const fileName = `${Date.now()}${fileExtension}`;
    const filePath = path.join(uploadDir, fileName);

    const compressedImage: sharp.Sharp = sharp(image.buffer)
      .png({
        quality: 80,
      })
      .jpeg({ quality: 80 });

    await compressedImage.toFile(filePath);

    return compressedImage;
}
```

Pertama-tama tentuka dimana gambar akan disimpan, disini gambar akan disimpan pada folder images, ambil extension file yang diupload, lalu generate nama flie yang _unique_ menggunakan `Date.now`, lalu gunakan fungsi join dari path untuk mendapatkan path file.

```typescript
const uploadDir = "./images";
const fileExtension = path.extname(image.originalname).toLowerCase();
const fileName = `${Date.now()}${fileExtension}`;
const filePath = path.join(uploadDir, fileName);
```

Gunakan sharp untuk melakukan proses _compress_ gambar dan turunkan kualitas gambar yang diupload menjadi 80%

```typescript
const compressedImage: sharp.Sharp = sharp(image.buffer)
  .png({
    quality: 80,
  })
  .jpeg({ quality: 80 });
```

Simpan file ke output menggunakan fungsi `ToFile` dari sharp.

```typescript
await compressedImage.toFile(filePath);
```
