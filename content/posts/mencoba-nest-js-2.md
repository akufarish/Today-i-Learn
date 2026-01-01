+++
date = '2026-01-01T11:48:17+08:00'
draft = false
title = 'Mencoba Nest Js (2)'
+++

## Introduction

Memasuki hari kedua belajar nestjs, sebelum itu selamat tahun baru 2026, semoga di tahun ini lebih sering produktif dan ada peningkatan dari tahun sebelumnya. Kembali ke topik utama, disini saya akan melakukan recap untuk materi controller, http method, http request, http response, asynchronous, cookie, dan view pada materi nest js dasar dari programmer zaman now.

## Controller

Sama seperti laravel, nest js merupakan framework yang menerapkan MVC yang dimana controller disini bertugas untuk memproses HTTP request maupun response. Dalam nest js untuk membuat controller kita bisa dengan mudah menggunakan CLI nest generate controller nama_controller path. controller dalam node js dapat dikenali dengan adanya decorater @Controller() yang pada parameter decorator tersebut kita dapat melakukan konfigurasi prefix path untuk controller.

```
@Controller('/api/users')
export class UserController {
}
```

## HTTP Method

Semua tentang decorator. Mungkin itu yang saya rasakan ketika menggunakan nest js pertama kali karena semua hal yang ada disini sudah diurus oleh decorator. Kalau pada laravel jika ingin membuat routing bisa lewat web.php atau api.php, pada nest didalam controller lah kita melakukan routing, menggunakan decorator dari HTTP method yang ada.

1. Get() untuk method get.
2. Post() untuk method post.
3. @Put() untuk method put.
4. @Delete() untuk method delete.
5. @Patch() untuk method patch.
6. @Head() untuk method head.
7. @Option() untuk method option.
8. @All() untuk semua HTTP Method.

Dalam decorator juga kita bisa menambahkan path parameter untuk menentukan route nya, jika tidak diisi maka default nya akan '/' mengikuti route prefix

```
@Post()
post(): string {
  return 'POST';
}

@Get('/sample')
get(): string {
  return 'GET';
}
```

## HTTP Request

Mau mengambil query? gunakan decorator @Query(), mau mengambil data dari parameter? gunakan @Param(), request? gunakan Request(), body? gunakan Body(). Itu semua sudah diurus oleh decorator dan kita bisa dengan mudah mengakses nya saja.

```
@Get('/hello')
sayHello(@Query('name') name: string): string {
  return `Hello ${name}`;
}

@Get('/:id')
getById(@Param('id') id: string): string {
  return `GET ${id}`;
}

@Get('get-cookie')
getCookie(@Req() request: Request): string {
  return request.cookies['name'] as string;
}
```

## HTTP Response

Secara default return value yang kita set pada method yang ada didalam controller akan dijadikan response body untuk HTTP Resposne. Namun jika memerlukan express response juga bisa menggunakan decorator @Res() dan type nya Resposne dari express.

> Contoh menjadikan string sebagai HTTP Response

```
@Get('/sample')
get(): string {
  return 'GET';
}

```

> Contoh menggunakan express response

```
@Get('/set-cookie')
setCookie(@Query('name') name: string, @Res() response:Response) {
  response.cookie('name', name);
  response.status(200).send('Success set cookie');
}
```

## Asynchronous

Cukup gunakan async method dengan return value Promise<T> data

```
@Get('/async-hello')
async asyncSayHello(@Query('name') name: string): Promise<string> {
  return `Hello ${name}`;
}
```

## Cookie

By default nestJS tidak memiliki support cookie secara bawaan. Oleh karena itu kita bisa menggunakan library tambahan seperti cookie-parser yang nantinya dapat kita tambahkan di file main.ts

> Menambah library cookie-parser

```
bun add cookie-parser
bun add -d @types/cookie-parser
```

> Konfigurasi main.ts

```
import * as cookieParser from 'cookie-parser';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.use(cookieParser.default('SECRET'));

  await app.listen(process.env.PORT ?? 3000);
}
```

Untuk mengakses cookie kita dapat menggunakan request dan response dari express js

> Contoh penggunaan cookie

```
@Get('/set-cookie')
  setCookie(@Query('name') name: string, @Res() response: Response) {
    response.cookie('name', name);
    response.status(200).send('Success set cookie');
  }

@Get('get-cookie')
  getCookie(@Req() request: Request): string {
    return request.cookies['name'] as string;
  }
```

## View

Sama seperti cookie, nestJS by default juga tidak memiliki template engine bawaan, namun kita bisa menggunakan mustache sebagai template engine nya.

> Menambah library mustache

```
bun add mustache-express
bun add -d @types/mustache-express
```

> Konfigurasi main.ts

```
import * as mustache from 'mustache-express';
import { NestExpressApplication } from '@nestjs/platform-express';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.use(cookieParser.default('SECRET'));

  app.set('views', __dirname + '/../views');
  app.set('view engine', 'html');
  app.engine('html', mustache.default());
  await app.listen(process.env.PORT ?? 3000);
}
```

Seperti yang bisa diliat pada konfigurasi main.ts saat menambah library cookie dan mustache kita set NestFactory.create dengan type NestExpressApplication yang berarti project nest js yang dikerjakan saat ini adalah express application yang membuat variabel app memiliki method yang sama seperti membuat express.
