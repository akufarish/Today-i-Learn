+++
date = '2026-01-07T15:23:42+08:00'
draft = false
title = 'Nest JS Middleware'
+++

Middleware adalah sebuah jembatan antara request dari client ke server, yang memungkinkan kita untuk menambahkan logika yang akan dijalankan sebelum request tersebut mencapai route handler atau tujuan. 

Berikut adalah diagram sederhana middleware.

<div class="img-container">
<img alt="middleware-light-diagram" src="/images/middleware-diagram.png" class="img-light img">
<img alt="middleware-dark-diagram" src="/images/middleware-diagram-dark.png" class="img-dark img">
</div>

Cara kerja middleware itu sendiri adalah setiap request yang dikirim oleh client akan melewati middleware terlebih dahulu, yang dimana pada middleware ini request tersebut dapat diperiksa atau dimodifikasi tergantung dari logic middleware. Jika request dianggap valid maka middleware akan meneruskannya ke controller, jika tidak, middleware dapat langsung menghentikan proses dan mengirim respons kembali ke client.

Untuk membuat middleware pada NestJS, kita bisa membuat class turunan dari **NestMiddleWare**, atau bisa menggunakan CLI Nest.

```terminaloutput
nest generate middleware nama_file path_file
```

> auth.middleware.ts
```typescript
import { Inject, Injectable, NestMiddleware } from '@nestjs/common';
import { type Request, type Response } from 'express';
import { WINSTON_MODULE_PROVIDER } from 'nest-winston';
import { Logger } from 'winston';

@Injectable()
export class AuthMiddleware implements NestMiddleware {
    constructor(@Inject(WINSTON_MODULE_PROVIDER) private log: Logger) {}
    use(req: Request, res: Response, next: () => void) {
        const header = req.headers.authorization;

        const token = header?.split(' ')[1];
        if (token) {
            if (token == '123') {
                next();
            } else {
                res.status(401).json({
                    message: 'Unauthorized',
                });
            }
        } else {
            res.status(401).json({
                message: `Unauthorized`,
            });
        }
    }
}
```

Kode diatas adalah contoh middleware sederhana untuk autentikasi, dimana ketika client ingin mengakses url **/api/users/auth**, client harus memberi token pada header, lalu ketika client melakukan request, middleware akan melakukan pengecekan apakah token itu ada dan apakah token itu valid, jika valid maka request akan dilanjutkan ke controller dengan function **next()**, jika tidak maka return response Unauthorized.

Decorator @Module() tidak memiliki atribut untuk meregistrasikan middleware, Untuk meregistrasikan middleware, kita harus membuat module turunan dari interface NestModule, karena pada interface NestModule terdapat method **configure(Middleware)** untuk meregistrasikan middleware dan ke route mana kita ingin middleware itu digunakan. 

> app.module.ts
```typescript
import {
  MiddlewareConsumer,
  Module,
  NestModule,
  RequestMethod,
} from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { AuthMiddleware } from './middleware/auth/auth.middleware';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(AuthMiddleware).forRoutes({
      path: '/api/users/auth',
      method: RequestMethod.ALL,
    });
  }
}
```

Catatan tambahan pada parameter **path** dapat menggunakan * (asterisk) sebagai wildcard dan akan cocok dengan kombinasi karakter apapun. Contoh ketika path nya kita buat menjadi **/api/*** berarti route apapun yang dimulai dengan **/api/** akan memiliki middleware yang sama.

Pengujian middleware

> GET /api/users/auth

Request Header:
```json
{
	"Authorization": "Bearer {token}"
}
```

Response Body:
```json
{
    "message": "Unauthorized"
}
```

> GET /api/users/auth

Request Header:
```json
{
	"Authorization": "Bearer 123"
}
```

Response Body:
```json
{
    "message": "login"
}
```
