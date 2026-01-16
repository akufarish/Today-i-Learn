+++
date = '2026-01-08T18:03:32+08:00'
draft = false
title = 'Nest JS Exception Filter'
tags = ['Indonesia', 'NestJS', 'Programming']
+++

Secara default jika terjadi error pada aplikasi NestJS, aplikasi akan return data berupa json dengan status code **500 internal server error**.

```json
{
  "statusCode": 500,
  "message": "Internal server error"
}
```

Tapi terkadang kita sebagai developer juga ingin mengubah error yang dikembalikan, karena tidak semua error pada aplikasi itu bisa diberikan **500 internal server error**, bisa saja error **400 bad request** untuk validation request, atau **401 unauthorized** untuk memberitahu kalau user ini tidak memiliki hak akses. Dan lain-lain.

Disinilah **exception filter** berperan untuk mengatasi masalah tersebut. NestJS memiliki fitur bawaan bernama exception filter yang berfungsi untuk memproses error yang tidak dapat ditangani oleh aplikasi. Ketika ada error yang tidak dapat ditangani oleh aplikasi, maka exception filter akan menangkap error, lalu akan mengirim response yang lebih user-friendly ke pengguna aplikasi.

Untuk membuat exception filter, dapat membuat class turunan **Exception Filter**, atau menggunakan CLI Nest.

```terminaloutput
nest generate filter nama_file path
```

Saat membuat exception filter, ada decorator **@Catch(ErrorType)** untuk menentukan error apa yang mau ditangani.

Contoh dibawah adalah contoh exception filter untuk validasi request menggunakan zod.

> validation-filter.ts

```typescript
import { ArgumentsHost, Catch, ExceptionFilter } from "@nestjs/common";
import { ZodError } from "zod";
import { type Response } from "express";

@Catch(ZodError)
export class ValidationFilter implements ExceptionFilter<ZodError> {
  catch(exception: ZodError, host: ArgumentsHost) {
    const http = host.switchToHttp();
    const response = http.getResponse<Response>();

    return response.status(400).json({
      code: 400,
      errors: exception.issues,
    });
  }
}
```

Untuk menggunakan filter bisa menggunakan decorator **@UseFilters()**, decorator **@UseFilters()** dapat digunakan pada method atau di class controller, jika kita gunakan pada class controller, maka semua route yang ada didalam controller akan menggunakan exception filter yang disett di **@UseFilters()**.

> user-controller.ts

```typescript
@Post('/create')
@UseFilters(ValidationFilter)
async create(@Req() request: Request): Promise<User> {
    return await this.userRepository.save(request.body.name as string);
}
```

```typescript
@Controller("/api/users")
@UseFilters(ValidationFilter)
export class UserController {
  constructor() {}
}
```

<div class="alert-container error"><strong>POST</strong> /api/users/create</div>

Request Body:

```json
{
  "name": ""
}
```

Response Body:

```json
{
  "code": 400,
  "errors": [
    {
      "origin": "string",
      "code": "too_small",
      "minimum": 3,
      "inclusive": true,
      "path": [],
      "message": "Too small: expected string to have >=3 characters"
    }
  ]
}
```

Jika ingin exception filter digunakan disemua controller dan method, exception filter tersebut dapat dijadikan sebagai global dengan cara menambahkan exception filter di NestJS application menggunakan method **useGlobalFilters()**.

> app.module.ts

```typescript
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ValidationFilter } from "./validation/validation.filter";

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useGlobalFilters(new ValidationFilter());
  await app.listen(parseInt(configService.get("HTTP_PORT")!));
}
bootstrap();
```

Perlu dicatat global filter yang dimasukkan harus bisa menangani banyak jenis error.

## HttpException

Nest juga menyediakan class **HttpException** yang bisa kita gunakan untuk memberikan error yang bisa digunakan didalam logic aplikasi, tanpa membuat Exception filter. Contoh untuk mengembalikan error jika data tidak ada, kita bisa menggunakan **HttpException**. Class HttpException sendiri memiliki beberapa attribute seperti response HTTP dan status code HTTP.

> user-repository.ts

```typescript
async getById(id: string): Promise<User | null> {
    const user = await this.prismaService.user.findUnique({
        where: {
            id: parseInt(id),
        },
    });

    if (!user) {
        throw new HttpException('User not Found', 404);
    }

    return user;
}
```

<div class="alert-container error"><strong>GET</strong> /api/users/4000</div>

Response Body:

```json
{
  "statusCode": 404,
  "message": "User not Found"
}
```
