+++
date = '2026-01-13T20:58:23+08:00'
draft = false
title = 'Nestjs Lifecycle'
+++

## Lifecycle Event

Pada aplikasi NestJS semua object yang ada seperti module, controller, dan provider itu memiliki lifecycle event atau alur hidup. Apa itu **lifecycle** dalam NestJS? Sesuai dengan namanya lifecycle yang berarti alur hidup adalah rankaian tahapan yang dilalui oleh objek dimulai dari objek itu dibuat, sampai objek tersebut dihancurkan.

Berikut adalah diagram lifecycle dalam aplikasi NestJS:

<div class="img-container">
<img alt="middleware-light-diagram" src="/images/lifecycle-light.png" class="img-light img">
<img alt="middleware-dark-diagram" src="/images/lifecycle-dark.png" class="img-dark img">
</div>

**OnModuleInit** Dipanggil ketika semua module diload, maka akan memanggil function **OnModuleInit()**.

**OnApplicationBootstrap** Dipanggil setelah semua module sudah diinisialisasi, tetapi sebelum aplikasi terkoneksi dengan server.

**OnModuleDestroy** dipanggil setelah menerima sinyal shutdown.

**BeforeApplicationShutdown** dipanggil setelah **OnModuleDestroy**, semua handler sudah selesai dan semua koneksi akan ditutup.

Terakhir ada **OnApplicationShutdown** yang dipanggil setelah semua koneksi tertutup.

Salah satu contoh penggunaan lifecycle diatas terdapat pada PrismaService atau handler yang menghandle database, misal ketika module pertama kali diload maka koneksikan database, dan jika aplikasi menerima sinyal untuk shutdown, maka putuskan koneksi ke database.

> prisma.service.ts

```typescript
import { Injectable, OnModuleDestroy, OnModuleInit } from "@nestjs/common";
import { PrismaClient } from "../../../../generated/prisma/client";
import { ConfigService } from "@nestjs/config";
import { PrismaMariaDb } from "@prisma/adapter-mariadb";

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  constructor(private configService: ConfigService) {
    const adapter = new PrismaMariaDb({
      host: configService.get("HOST"),
      port: configService.get("PORT"),
      connectionLimit: configService.get("CONNECTION_LIMIT"),
      user: configService.get("USER_DB"),
      database: configService.get("DATABASE"),
      password: configService.get("PASSWORD"),
      allowPublicKeyRetrieval: true,
    });
    super({ adapter });
  }
  async onModuleInit() {
    console.info("Connected prisma");
    await this.$connect();
  }
  async onModuleDestroy() {
    console.info("Disconnected Prisma");
    await this.$disconnect();
  }
}
```

> main.ts
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableShutdownHooks();
  await app.listen(3000);
}
bootstrap();
```

Kenapa pada file main.ts ada tambahan kode berupa method **enableShutdownHooks()**? Itu karena OnModuleDestroy, BeforeApplicationShutdown, dan OnApplicationShutdown akan dijalankan ketika kita memanggil **app.close()** untuk menghentikan aplikasi, tapi dalam beberapa kasus kita mau menghentikan aplikasi dengan cara menggunakan signal termination dengan menggunakan **CTRL + C**, maka dari itulah pada main.ts dipanggil method **enableShutdownHooks()** untuk tiga lifecycle tadi dapat dipanggil ketika menerima signal termination. 
