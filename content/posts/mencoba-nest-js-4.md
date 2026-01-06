+++
date = '2026-01-04T19:51:47+08:00'
draft = false
title = 'Mencoba Nest Js (4)'
+++

## Introduction
Lanjut ke hari keempat belajar NestJS. Materi kali ini ada module reference, mengakses .env menggunakan configuration, dan membahas singleton object pada materi shared module.

## Module Reference
Nest menyediakan class bernama **ModuleRef** yang berfungsi untuk mengambil data provider tanpa melakukan dependency injection. Cara penggunaannya adalah menggunakan method **get()** dari class **ModuleRef** yang menerima satu parameter yaitu provider yang ingin kita gunakan.

> member-service.ts
```typescript
import { Injectable } from '@nestjs/common';
import { ModuleRef } from '@nestjs/core';
import { Connection } from '../connection/connection';
import { MailService } from '../mail/mail.service';

@Injectable()
export class MemberService {
  constructor(private moduleRef: ModuleRef) {}

  getConnectionName(): string | null {
    const connection = this.moduleRef.get(Connection);
    return connection.getName();
  }

  sendEmail() {
    const mailService = this.moduleRef.get(MailService);
    mailService.send();
  }
}
```

Tapi penggunaan module reference ini memiliki kekurangan dimana berbeda dengan dependency injection yang melakukan pengecekkan dependency di awal saat aplikasi dijalankan untuk melakukan pengecekkan dependency ini ada atau tidak, dan kalau tidak ada, akan langsung return error. Sedangkan jika menggunakan module reference, pengecekkan akan dilakukan ditengah berjalannya aplikasi, contohnya saat aplikasi sudah jalan dan mencoba memanggil route atau method yang menggunakan module reference dan ternyata dependency yang diperlukan itu tidak ada, maka otomatis akan error.

## Configuration
Menulis konfigurasi secara hard code bukanlah cara yang terbaik, jika melakukan itu maka akan orang bisa saja mengakses data-data penting seperti username dan password dari database atau bahkan api-key dari service yang  digunakan, salah satu praktik terbaik untuk mengatasi itu adalah menggunakan environtment variable (.env). Dengan menggunakan environtment variabel kita bisa memiliki data konfigurasi yang dinamis seperti username dan password database dan lain-lain.

Untuk dapat mengakses file **.env** pada project NestJS, kita perlu menginstall library tambahan yaitu:

```terminaloutput
bun add @nestjs/config
```

Setelah library tersebut sudah diinstall, langkah berikutnya adalah memasukkan ConfigModule ke AppModule project NestJS, karena library-library tambahan yang disediakan oleh NestJS merupakan module.

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './user/user.module';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    UserModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Bisa dilihat pada ConfigModule itu menggunakan method **forRoot()** yang nanti akan dibahas pada materi dynamic module. lalu pada method **forRoot** bisa menambahkan object dan ada atribute **isGlobal** yang berfungsi untuk memberi tahu NestJS kalau module **ConfigService** nantinya dapat diakses secara global, jadi kita tidak perlu lagi melakukan import ConfigModule pada file **user-module.ts** atau module lain.

Apa itu **ConfigService**? ConfigService adalah class yang memungkinkan kita mengakses konfigurasi yang ada didalam file .env menggunakan method **get()**.

> connection.ts
```typescript
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class Connection {
    getName(): string | null {
        return null;
    }
}

@Injectable()
export class MysqlConnection extends Connection {
    getName(): string | null {
        return 'MySQL';
    }
}

@Injectable()
export class MongoDBConnection extends Connection {
    getName(): string | null {
        return 'MongoDB';
    }
}

export function createConnection(configeService: ConfigService): Connection {
    if (configeService.get('DATABASE') == 'mysql') {
        return new MysqlConnection();
    } else {
        return new MongoDBConnection();
    }
}
```
## Shared Module
Dalam NestJS module itu merupakan singleton object. Bagaimana cara kerja singleton object? singleton object bekerja dengan cara membuat module itu cukup sekali saja, dan tidak berkali-kali, maksudnya bagaimana?

> app.module.ts
```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UserModule } from './user/user.module';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
    }),
    UserModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

> barang.module.ts
```typescript
import { Module } from '@nestjs/common';
import { UserModule } from '../../user/user.module';

@Module({
  imports: [UserModule],
})
export class BarangModule {}
```
Apabila tidak menggunakan singleton obejct ketika kita melakukan import **UserModule** pada dua file berbeda, umumnya program akan membuat dua UserModule yang mungkin untuk sekarang aplikasi berjalan dengan tidak ada masalah, tapi bagaimana jika aplikasinya semakin kompleks? **UserModule** akan diimport pada puluhan file yang berarti program akan membuat puluhan UserModule yang bisa memakan banyak memorry. Disinilah peran singleton object untuk mengatasi permasalahan tersebut, seperti yang sudah ditulis diatas, dengan menggunakan singleton object **UserModule** cuma dibuat sekali saja dan dapat diimport tanpa masalah.