+++
date = '2026-01-11T19:10:05+08:00'
draft = false
title = 'Nest JS Custom Decorator'
+++

NestJS sudah banyak menyediakan decorator bawaan yang bisa kita gunakan di dalam controller seperti **@Req()**, **@Body()**, dan lain-lain. Namun, terkadang kita perlu menambahkan atribut ke object request, lalu datanya bisa diakses di controller.

Satu contoh kenapa kita perlu custom decorator adalah ketika untuk mendapatkan data user yang sedang login, biasanya kodenya akan seperti ini.

> user.controller.ts

```typescript
@Get('/auth')
auth(@Req() request: Request): Record<string, string> {
    const user = request.user;
    return {
        email: user.email,
        username: user.name,
    };
}
```

kode diatas dapat dijadikan custom decorator untuk membuat kode lebih mudah untuk dibaca. Untuk membuat custom decorator sendiri bisa menggunakan function **createParamDecorator()**, atau bisa gunakan CLI NestJS dengan command.

```terminaloutput
nest generate decorator nama_file path_folder
```

> auth.decorator.ts

```typescript
import { createParamDecorator, ExecutionContext } from "@nestjs/common";
import { User } from "../../../generated/prisma/client";

export const Auth = createParamDecorator(
  (data: User, context: ExecutionContext) => {
    // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment
    const request = context.switchToHttp().getRequest();
    // eslint-disable-next-line @typescript-eslint/no-unsafe-return,@typescript-eslint/no-unsafe-member-access
    return request.user;
  }
);
```

Pada kode diatas, kita membuat decorator dengan nama Auth dengan fungsi **createParamDecorator()** yang menerima dua paramter, yaitu data yang diberi tipe data User karena kita mau mendapatkan data user yang sedang login, dan context digunakan untuk mendapatkan response atau request.

> user.controller.ts

```typescript
@Get('/auth')
auth(@Auth() user: User): Record<string, string> {
    return {
        email: user.email,
        username: user.name,
    };
}
```

<div class="alert-container success"><strong>GET</strong> /api/users/login/auth</div>

Request Header:

```json
"Authorization": "Bearer eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MiwiaWF0IjoxNzY4MTMxODQzLCJpc3MiOiJ1cm46ZXhhbXBsZTppc3N1ZXIiLCJhdWQiOiJ1cm46ZXhhbXBsZTphdWRpZW5jZSJ9.xQTQXU6H2d-RpsBLwZZwC4WCfolCKZm9cRJytHhnEcc"
```

Response Header:

```json
{
  "email": "admin@gmail.com",
  "username": "admin"
}
```
