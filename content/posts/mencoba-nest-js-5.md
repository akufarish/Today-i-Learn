+++
date = '2026-01-06T13:45:02+08:00'
draft = false
title = 'Mencoba Nest Js (5)'
+++

## Introduction
Hari ke tujuh belajar NestJS materi yang dipelajari hari ini tidak kalah menarik, karena akan membahas validation pada NestJS, selain itu ada juga pembahasan untuk logging, global module, dan dynamic module.

## Logging
NestJS secara default sudah memiliki logger sendiri untuk menampilkan log ketika aplikasi dijalankan atau ketika terjadi error. Logger bawaan NestJS hanya akan melakukan logging ke console dan berupa text, terkadang beberapa developer ingin fitur logging yang lebih dari itu. Untungnya NestJS menyediakan cara untuk mengganti Logger nya dengan cara membuat object LoggerService, lalu pada file **main.ts** bisa menggunakan method **useLogger()** untuk mengganti logger pada Nest.

> main.ts

```typescript
import { NestFactory } from '@nestjs/core';
import { NestExpressApplication } from '@nestjs/platform-express';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  app.useLogger(app.get(MyLogger));
  await app.listen(parseInt(configService.get('HTTP_PORT')!));
}
bootstrap();
```

Salah satu library logger yang sering digunakan dan populer adalah **winston**. Alasan kenapa winston sering digunakan karena fleksibel dan memiliki fitur seperti transport yang memungkinkan log dikelola berdasarkan level, misal log untuk debug, dan production.

Untuk menggunakan winston pada aplikasi NestJs, kita dapat menambahkan library nest-winston.
```terminaloutput
bun add nest-winston
```
Pada file app.module.ts untuk menggunakan winston kita dapat import WinstonModule lalu gunakan method **forRoot()** karena winstonModule berupa **dynamic module**. Dalam method **forRoot()** kita dapat mengisi argument yang ada seperti **format** untuk log nya dalam bentuk apa, **level**, dan **transport** untuk menentukan log ini mau tampil dimana, mau di console atau simpan dalam bentuk file.
>app.module.ts
```typescript
import {
  MiddlewareConsumer,
  Module,
  NestModule,
  RequestMethod,
} from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './user/user.module';
import { ConfigModule } from '@nestjs/config';
import { BarangModule } from './barang/barang/barang.module';
import { PrismaService } from './prisma/prisma/prisma.service';
import { PrismaModule } from './prisma/prisma/prisma.module';
import * as winston from 'winston';
import { WinstonModule } from 'nest-winston';

@Module({
  imports: [
    WinstonModule.forRoot({
      format: winston.format.json(),
      level: 'debug',
      transports: [new winston.transports.Console()],
    }),
    UserModule,
    BarangModule,
    PrismaModule,
  ],
  controllers: [AppController],
  providers: [AppService, PrismaService],
})
export class AppModule {}
```
Secara default winston module ini defaultnya adalah global module, yang berarti module ini dapat kita gunakan dimanapun tanpa melakukan import lagi.

> main.ts
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { NestExpressApplication } from '@nestjs/platform-express';
import { WINSTON_MODULE_NEST_PROVIDER, WinstonLogger } from 'nest-winston';

async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);
  const loggerService: WinstonLogger = app.get(WINSTON_MODULE_NEST_PROVIDER);
  app.useLogger(loggerService);
  await app.listen(parseInt(configService.get('HTTP_PORT')!));
}
bootstrap();
```
Catatan tambahan yaitu pada application untuk penggunaan winston disini menggunakan **WINSTON_MODULE_NEST_PROVIDER** karena provider ini menyediakan wrapper yang mengimplementasikan LoggerService milik NestJS, sedangkan WINSTON_MODULE_PROVIDER merupakan provider milik winston.

## Global Module
Karena NestJS ini framework modular yang berarti kita harus melakukan import module untuk bisa mengakses provider ataupun service nya. Namun NestJS sudah menyediakan solusi untuk membuat module yang diperlukan oleh banyak module hanya perlu diimport satu kali oleh application module dan bisa digunakan oleh module-module lainnya, yaitu dengan membuat module tersebut menjadi global dengan cara menambahkan decorator **@Global()**.

Global module sendiri secara otomatis akan diimport di semua module, jadi kita tidak perlu lagi melakukan import terus-menerus. Satu contoh module yang ingin kita buat menjadi global adalah **PrismaModule** karena kita pasti menggunakan prisma pada banyak file, atau contoh lainnya ada seperti winston dan config module yang sudah kita gunakan sebelumnya.

> prisma.module.ts
```typescript
import { Global, Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

Karena prisma module sudah dibuat menjadi global module dengan bantuan decorator **@Global()** jadi prisma module hanya perlu kita import sekali di application module.

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
import { PrismaModule } from './prisma/prisma/prisma.module';

@Module({
  imports: [
    PrismaModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
## Dynamic Module
Sebelumnya pada beberapa module seperti **ConfigModule** dan **WinstonModule**, kedua module tersebut dibuat menggunakan method **forRoot()**. Cara pembuatan itu dinamakan dynamic module. Dynamic module adalah mekanisme membuat module secara dinamis, sehingga kita bisa menentukan **provider** atau **controller** secara runtime berdasarkan parameter yang kita berikan.

```typescript
import {
  MiddlewareConsumer,
  Module,
  NestModule,
  RequestMethod,
} from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from '@nestjs/config';
import * as winston from 'winston';
import { WinstonModule } from 'nest-winston';

@Module({
  imports: [
    WinstonModule.forRoot({
      format: winston.format.json(),
      level: 'debug',
      transports: [new winston.transports.Console()],
    }),
    ConfigModule.forRoot({
      isGlobal: true,
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Bisa dilihat pada contoh kode dynamic module diatas, pada WinstonModule kita set beberapa parameter yang disediakan seperti format untuk logger nya itu apa, level apa, dan transports yang mau digunakan itu apa. Begitu juga pada ConfigModule kita set parameter isGlobal mau dibuat true atau false.

Untuk membuat dynamic module, pada module yang mau kita buat menjadi dynamic dapat menambahkan **method** yang return object **DynamicModule**. Apa yang kita buat pada module biasa, bisa kita lakukan juga dengan dynamic module, seperti menentukan module ini perlu module apa saja, provider apa saja dan lain-lain.

Untuk contoh pembuatan dynamic module akana dipraktekkan pada bagian validation.

## Validation
Sama seperti database. NestJS sendiri tidak memiliki fitur untuk melakukan validation secara default. Oleh karena itu kita dapat menambahkan library validation seperti **zod** dengan cara membuat module yang nantinya akan ditambahkan pada aplikasi NestJS.

```terminaloutput
bun add zod
```

> validation.service.ts
```typescript
import { Injectable } from '@nestjs/common';
import { ZodType } from 'zod';

@Injectable()
export class ValidationService {
  validate<T>(schema: ZodType<T>, data: T): T {
    return schema.parse(data);
  }
}
```

Pada ValidationService kita membuat method validate dengan tipe generic **(T)** menggunakan, yang menerima dua parameter yaitu **schema** dengan tipe **ZodType\<T>**, dan **data** dengan tipe generic, dan return method sendiri yang generic. Apa itu generic dalam typescript? Generic adalah tipe data yang dinamis berbeda dengan **any** dengan tipe generic typescript dapat mengenali tipe data yang diberikan secara otomatis.

> validation.module.ts
```typescript
import { DynamicModule, Module } from '@nestjs/common';
import { ValidationService } from './validation.service';

@Module({})
export class ValidationModule {
  static forRoot(isGlobal: boolean): DynamicModule {
    return {
      module: ValidationModule,
      global: isGlobal,
      providers: [ValidationService],
      exports: [ValidationService],
    };
  }
}
```

Perlu catatan untuk nama method bebas tidak harus **forRoot()**. Ketika membuat return DynamicModule juga kita harus return module yang ingin digunakan dan global yang berarti module ini mau dibuat global atau tidak.