+++
date = '2026-01-15T20:59:14+08:00'
draft = false
title = 'JWT'
+++

**Json Web Token** atau yang sering disebut **JWT** adalah standar terbuka (RFC 7519) yang mendefinisikan cara untuk passing data seperti informasi user seperti (id user, role user, email user) dalam bentuk token, yang aman dan dapat diverifikasi oleh server.

Sederhananya JWT merupakan web token yang berisi object json yang digunakan untuk proses authentication atau authorization.

Kenapa kita perlu menggunakan jwt? Karena jwt itu **stateless** yang berarti server tidak menyimpan token di database. Kenapa demikian? Karena salah satu permasalahan jika menyimpan token di database, adalah server akan terbebani, karena setiap kali kita melakukan proses request yang memerlukan _authentication_ maupun _authorization_, dia akan terus melakukan query terus-menerus ke database.

Sedangkan pada JWT, server hanya perlu mengecek apakah signature dari token yang dikirimkan sudah sesuai dengan **secret key** atau tidak, untuk menentukan validitasnya.

## Struktur JWT

JWT sendiri terdiri dari 3 struktur yaitu **Header**, **Payload**, **Signature**.

### Header

Header pada jwt terdiri dari dua bagian, yaitu algoritma hashing yang digunakan, dan informasi jenis token.

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

Berisi data atau yang disebut dengan **claims**. Seperti userId, role, email.

```json
{
  "user_id": 1,
  "admin": false
}
```

### Signature

Signature adalah hasil kombinasi dari **header** dan **payload** yang sudah di encode, kemudian hashing menggunakan **secret key**.

Signature berfungsi untuk menghindari modifikasi data header maupun payload, karena jika salah satu data tersebut berubah, maka signature-nya secara otomatis akan dianggap tidak valid. Hal ini terjadi karena orang yang memodifikasi data tersebut tidak mengetahui **secret key** yang digunakan untuk mengubah signature-nya.

```typescript
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret);
```

Pada contoh diatas menggunakan **HMACSHA256** sebagai algoritma yang ditentukan pada header, lalu didalamnya berisi **base64UrlEncode(header) + "." + base64UrlEncode(payload)** lalu ditambah hashing **secret key**, yang menghasilkan jwt:

```json
eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwiaWF0IjoxNzY4Mzc4MTQwLCJpc3MiOiJ1cm46ZXhhbXBsZTppc3N1ZXIiLCJhdWQiOiJ1cm46ZXhhbXBsZTphdWRpZW5jZSJ9.m3maBAuXdZOEhs1nBkvPyEAjhqrWYFr1FmZkwn-ToVE
```

**Namun perlu catatan**, bahwa jwt tidak melakukan **enkripsi**, melainkan **encoding** saja, yang berarti header dan payload bisa di decode, oleh karena itu **jangan memasukkan data sensitif** ke payload jwt.
