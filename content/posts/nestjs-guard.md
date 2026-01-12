+++
date = '2026-01-12T21:10:27+08:00'
draft = false
title = 'Nest JS Guard'
+++

## Guard
Guard adalah class yang berfungsi untuk melakukan proses **authorization**, yaitu sebuah proses untuk mengecek apakah user berhak untuk mengakses route tersebut atau tidak. Lantas kenapa tidak melakukan proses authorization didalam middleware? Kan didalam middleware kita melakukan proses authentication. Satu alasan kenapa melakukan proses authorization didalam middleware itu susah karena middleware tidak tahu route mana yang boleh diakses dan tidak boleh, yang dia tahu hanya memanggil fungsi next() entah itu next middleware atau next route. Oleh karena itulah untuk melakukan proses authorization menggunakan guard.

Untuk membuat guard pada NestJS, bisa dengan cara membuat class turunan **CanActivate**, atau bisa menggunakan CLI Nest:

```terminaloutput
nest generate guard nama_guard path_guard
```

> role.guard.ts
```typescript
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class RoleGuard implements CanActivate {
  constructor(private roles: string[]) {}

  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment,@typescript-eslint/no-unsafe-member-access
    const user = context.switchToHttp().getRequest().user;
    // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access,@typescript-eslint/no-unsafe-argument
    return this.roles.indexOf(user.role) != -1;
  }
}
```

Guard diatas adalah guard yang berfungsi untuk melakukan authorization berdasarkan role. Dimulai dari constructor, class RoleGuard menerima data berupa string array yang nantinya berisi role-role apa saja yang boleh mengakses route ini.

Didalam guard juga ada method **canActivate()** yang berasal dari class **CanActivate** yang nantinya akan dieksekusi yang return dari fungsi tersebut bisa promise boolean atau observe boolean, yang artinya kalau return nya true maka user memiliki akses untuk route ini, jika false maka user tidak memiliki akses mengakses untuk route ini. Untuk mengambil data user yang sedang login, disini menggunakan custom decorator auth yang sudah dibuat kemaren.

```typescript
return this.roles.indexOf(user.role) != -1;
```

Kode diatas menggunakan fungsi **indexOf()** untuk melakukan pengecekan role, yang dimana cara kerja **indexOf()** berfungsi untuk mencari posisi index dari suatu element yang dapat ditemukan pada array. Contoh dari code diatas adalah jika element ditemukan maka return indeks element yang ditemukan, jika tidak ditemukan maka return -1.

Untuk menggunakan guard bisa menambahkan decorator **@UseGuard()** didalam method controller, atau jika mau dijadikan global bisa menggunakan method **useGlobalGuards()** di application module, app.module.ts

