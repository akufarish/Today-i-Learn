+++
date = '2026-01-09T20:43:24+08:00'
draft = false
title = 'Nest JS Pipe'
+++

## Pipe
Pipe adalah fitur yang ada pada NestJS yang memiliki dua fungsi, pertama bisa untuk melakukan transformasi/konversi request sebelum dikirim ke controller method, kedua berfungsi untuk melakukan validasi request.

Salah satu cara menggunakan pipe, bisa dengan cara menambahkan pipe ke decorator **@Query()**, **@Body()** dan **@Param()**.

NestJS sendiri sudah menyediakan banyak pipe bawaan seperti ValidationPipe, ParseIntPipe, ParseFilePipe, ParseArrayPipe, dan lain-lain.

> user.controller.ts
```typescript
@Get('/:id')
async getById(@Param('id', ParseIntPipe) id: number): Promise<User | null> {
    return await this.userRepository.getById(id);
}
```

Diatas adalah contoh penggunaan pipe untuk memastikan parameter id yang dikirim berupa number, karena walaupun variabel id diberi tipe number kita tetap dapat mengirim string, karena kode typescript ini nanti akan dikonversi ke javascript, yang dimana pada javascript tidak ada type safe, jika id tersebut kita kirimkan data string maka aplikasi tetap berjalan, kecuali kita melakukan komputasi yang menggunakan tipe data number.

NestJS juga menyediakan untuk pembuatan pipe custom, dengan cara membuat class turunan dari **PipeTransform**, atau bisa menggunakan CLI NestJS dengan command:

```terminaloutput
nest generate pipe nama_file path_folder
```

> validation.pipe.ts
```typescript
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';
import { ZodType } from 'zod';

@Injectable()
export class ValidationPipe implements PipeTransform {
  constructor(private zodType: ZodType) {}
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  transform(value: any, metadata: ArgumentMetadata) {
    return this.zodType.parse(value);
  }
}
```

File pipe custom diatas adalah pipe yang berfungsi untuk melakukan validasi menggunakan zod, yang menerima satu parameter yaitu schema zod.

> user.controller.ts
```typescript
@Post('/login')
@UseFilters(ValidationFilter)
login(
    @Body(new ValidationPipe(loginUserRequestValidation))
request: LoginUserRequest,
) {
    return `Hello ${request.username}!`;
}
```

<div class="alert-container error"><strong>POST</strong> /api/users/login</div>

Request Body:
```json
{
    "username": "",
    "password": ""
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
            "minimum": 1,
            "inclusive": true,
            "path": [
                "username"
            ],
            "message": "Too small: expected string to have >=1 characters"
        },
        {
            "origin": "string",
            "code": "too_small",
            "minimum": 1,
            "inclusive": true,
            "path": [
                "password"
            ],
            "message": "Too small: expected string to have >=1 characters"
        }
    ]
}
```

Selain menggunakan pipe dengan cara menambahkannya didalam decorator **@Query()**, **@Body()** dan **@Param()**. Kita juga bisa menambahkan decorator di class controller method menggunakan decorator **@UsePipes()**, atau jika ingin menambah pipe secara global, bisa ditambahkan ke application module menggunakan method **useGlobalPipes()**. Namun perlu dicatat jika menggunakan pipe secara global, kita harus memperhatikan pengecekkan karena perbedaan tipe data yang disett bisa menyebabkan error pada pipe.

> user.controller.ts
```typescript
@Post('/login')
@UseFilters(ValidationFilter)
@UsePipes(new ValidationPipe(loginUserRequestValidation))
login(
    @Body()
request: LoginUserRequest,
) {
    return `Hello ${request.username}!`;
}
```
Kode diatas akan berjalan dengan semestinya, tapi ketika menambahkan request baru contohnya menggunakan **@Param()** maka akan terjadi error.

```typescript
@Post('/login/:id')
@UseFilters(ValidationFilter)
@UsePipes(new ValidationPipe(loginUserRequestValidation))
login(
    @Body()
request: LoginUserRequest,
@Param('id') id: string,
) {
    return `Hello ${request.username}! + ${id}`;
}
```

<div class="alert-container error"><strong>POST</strong> /api/users/login/213asd</div>

Request Body:
```json
{
    "username": "admin",
    "password": "admin123"
}
```

Response Body:
```json
{
    "code": 400,
    "errors": [
        {
            "expected": "object",
            "code": "invalid_type",
            "path": [],
            "message": "Invalid input: expected object, received string"
        }
    ]
}
```

Kode diatas error karena validation pipe yang kita buat sebelumnya itu validasi schema zod, sedangkan pada request diatas, selain mengirimm body request, kita juga mengirim parameter request.

Untuk mengatasi permasalahan tersebut, kita dapat melakukan pengecekkan pada pipe nya menggunakan tipe metadata yang nanti akan dikirim ke controller.

> validation.pipe.ts
```typescript
import { ArgumentMetadata, Injectable, PipeTransform } from '@nestjs/common';
import { ZodType } from 'zod';

@Injectable()
export class ValidationPipe implements PipeTransform {
  constructor(private zodType: ZodType) {}
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  transform(value: any, metadata: ArgumentMetadata) {
    if (metadata.type == 'body') {
      return this.zodType.parse(value);
    } else {
      // eslint-disable-next-line @typescript-eslint/no-unsafe-return
      return value;
    }
  }
}
```

<div class="alert-container success"><strong>POST</strong> /api/users/login/213asd</div>

Request Body:
```json
{
    "username": "admin",
    "password": "admin123"
}
```

Response Body:
```html
Hello admin! + 213asd
```

