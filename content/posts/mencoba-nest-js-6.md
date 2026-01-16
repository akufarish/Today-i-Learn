+++
date = '2026-01-14T20:19:38+08:00'
draft = false
title = 'Mencoba Nest Js (6)'
tags = ['Indonesia', 'NestJS', 'Programming']
+++

## Reflector

Sebelumnya pada materi guard, kita membuat sebuah guard untuk melakukan authorization berdasarkan role untuk mengecek, role yang boleh mengakses handler, yang cara pemanggilannya dengan cara membuat object **RoleGuard** baru disetiap handler.

> role.guard.ts

```typescript
@Get('/auth/user')
@UseGuards(new RoleGuard(['user', 'admin']))
auth(@Auth() user: User): Record<string, string> {
    return {
        email: user.email,
        username: user.name,
    };
}
```

Kode diatas akan menimbulkan masalah, karena **RoleGuard** bukan merupakan singleton, yang berarti setap kali kita menambahkan **RoleGuard** pada handler itu membuat object baru yang berarti akan memakan banyak memory. Nah kita bisa membuat RoleGuard menjadi singleton dengan bantuan **Reflector**.

Apa itu reflector? Reflector adalah sebuah fitur pada NestJS yang memiliki kemampuan untuk mengambil atau membaca metadata pada handler menggunakan decorator. Untuk membuat reflector bisa menggunakan method **Reflector.createDecorator**.

> role.decorator.ts

```typescript
import { Reflector } from "@nestjs/core";

export const Role = Reflector.createDecorator<string[]>();
```

Pada kode diatas kita membuat reflector yang bertipe string array, karena decorator ini akan menerima data berupa string array yang berisi role-role yang boleh mengakses handler yang dituju.

Supaya RoleGuard bisa menerima metadata dari decorator role, bisa dengan cara melakukan dependency injection reflector nya. lalu menggunakan method get yang menerima dua parameter yaitu decorator, dan function. Pada kode dibawah dua parameter tersebut diisi dengan decorator **Role** karena kita ingin membaca data dari decorator **Role**, dan **context.getHandler()** untuk mengetahui handler/function yang sedang dieksekusi.

> role,guard.ts

```typescript
import { CanActivate, ExecutionContext, Injectable } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { Observable } from "rxjs";
import { Role } from "./role.decorator";

@Injectable()
export class RoleGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    const roles: string[] = this.reflector.get(Role, context.getHandler());

    if (!roles) {
      return true;
    }
    // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment,@typescript-eslint/no-unsafe-member-access
    const user = context.switchToHttp().getRequest().user;
    // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access,@typescript-eslint/no-unsafe-argument
    return roles.indexOf(user.role) != -1;
  }
}
```

> user.controller.ts

```typescript
@Get('/auth/admin')
@UseGuards(RoleGuard)
@Role(['admin'])
admin(@Auth() user: User): Record<string, string> {
    return {
        email: user.email,
        username: user.name,
    };
}
```

### Pengujian

> Admin mengakses route admin

<div class="alert-container success"><strong>GET</strong> /api/users/auth/admin</div>

Request Header:

```json
{
  "Authorization": "Bearer {token}"
}
```

Response Body:

```json
{
  "email": "admin@gmail.com",
  "username": "admin"
}
```

## Global Provider

Pada mater-materi sebelumnya, kita sudah tau jika ingin membuat global filter, interceptor, guard, dan pipe, itu bisa menjadikannya global dengan menambahkannya ke NestJS application **main.ts**. Tapi muncul masalah baru karena penggunaan method **useGlobal** pada file **main.ts** tidak dapat melakukan dependency injection. Maka dari itu digunakannya lah global provider yang merupakan cara yang lebih efisien.

NestJS sudah menyediakan alias name untuk globel provider yaitu:

1. APP_INTERCEPTOR
2. APP_PIPE
3. APP_GUARD
4. APP_FILTER

Cara penggunaannya juga mirip dengan cara biasa saat kita menambahkan provider ke NestJS application provider, namun dengan catatatan untuk nama alias nya harus menggunakan alias name global yang sudah disediakan oleh NestJS.

> app.module.ts

```typescript
import {
  MiddlewareConsumer,
  Module,
  NestModule,
  RequestMethod,
} from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { UsersModule } from "./users/users.module";
import { PrismaModule } from "./common/prisma/prisma.module";
import { ConfigModule } from "@nestjs/config";
import { ValidationModule } from "./validation/validation.module";
import { AuthMiddleware } from "./common/auth/auth.middleware";
import { JwtModule } from "./common/jwt/jwt.module";
import { RoleGuard } from "./common/role/role.guard";
import { APP_GUARD } from "@nestjs/core";

@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    AppService,
    {
      provide: APP_GUARD,
      useClass: RoleGuard,
    },
  ],
})
```

### Pengujian

> Admin mengakses route admin

<div class="alert-container success"><strong>GET</strong> /api/users/auth/admin</div>

Request Header:

```json
{
  "Authorization": "Bearer {token}"
}
```

Response Body:

```json
{
  "email": "admin@gmail.com",
  "username": "admin"
}
```
