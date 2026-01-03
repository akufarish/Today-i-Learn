+++
date = '2026-01-03T18:05:17+08:00'
draft = false
title = 'Custom Provider'
+++

## Introduction
Seharusnya **custom provider** ini ada di postingan sebelumnya, tapi karena aku merasa ini adalah materi yang rumit jadi aku bikin postingan berbeda yang khusus membahas **custom provider** ini. 

Kenapa harus menggunakan custom provider? Karena terkadang kita bisa saja membuat provider yang bukan instance dari NestJS yang berarti provider third party misalnya MysqlClient, MongoClient, PrismaClient, dan lain-lain.

Atau ada juga kasus dimana provider yang kita buat itu sedikit kompleks sehingga tidak bisa hanya dengan memanggil class nya saja. Oleh karena itu Nest mendukung beberapa cara pembuatan custom provider yang akan dibahas pada postingan ini.

## Standard Provider
Cara ini adalah cara default yang sudah digunakan sebelumnya. Yaitu dengan cara membuat provider lalu menyebutkan langsung nama class nya di attribute providers pada module.

> user-module.ts
```typescript
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { UserService } from './user.service';

@Module({
  controllers: [UserController],
  providers: [
    UserService
  ],
})
export class UserModule {}
```

## Class Provider
Class provider adalah cara membuat provider dengan menentukan class yang ingin digunakan, dibandingkan membuat dua file provider, kita cukup menggunakan satu file provider yang didalamnya berisi class-class dengan decorator **@injectable()** yang nantinya dapat dipanggil menggunakan attribut **useClass** untuk menentukan class mana yang ingin digunakan.

> connection.ts
```typescript
import { Injectable } from '@nestjs/common';

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
```

> user-module.ts
```typescript
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import {
  Connection,
  MongoDBConnection,
  MysqlConnection,
} from './connection/connection';

@Module({
  controllers: [UserController],
  providers: [
    {
      provide: Connection,
      useClass:
        process.env.DATABASE == 'mysql' ? MysqlConnection : MongoDBConnection,
    }
})
export class UserModule {}

```

Contoh diatas adalah satu contoh untuk menentukan database apa yang mau digunakan? Jika menggunakan mysql maka gunakan provider class **MysqlConnection** jika tidak gunakan **MongoDBConnection**.

## Value Provider
Value provider adalah cara membuat provider dari value **object** yang sudah ada. Untuk menggunakan value provider bisa menambahkan atribute **useValue**.

> mail-service.ts
```typescript
export class MailService {
    send() {
        console.log('Sending...');
    }
}

export const mailService = new MailService();
```

> user-module.ts
```typescript
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { mailService, MailService } from './mail/mail.service';

@Module({
  controllers: [UserController],
  providers: [
    {
      provide: MailService,
      useValue: mailService,
    },
  ],
})
export class UserModule {}
```
## Factory Provider
Factory provider adalah cara untuk membuat dependency secara dinamis menggunakan function yang akan dipanggil untuk membuat object dependency nya, yang bisa disebut sebagai factory method. Jika pada factory method memerlukan dependency lain, kita bisa memasukkan dependency yang diperlukan ke dalam atribut **inject**.

> user-repository.ts
```typescript
import { Connection } from '../connection/connection';

export class UserRepository {
  connection: Connection;

  save() {
    console.log(`Info save user with connection ${this.connection.getName()}`);
  }
}

export function createUserRepository(connection: Connection): UserRepository {
  const repository = new UserRepository();
  repository.connection = connection;
  return repository;
}
```

> user-module.ts
```typescript
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import {
  createUserRepository,
  UserRepository,
} from './user-repository/user-repository';

@Module({
  controllers: [UserController],
  providers: [
    {
      provide: UserRepository,
      useFactory: createUserRepository,
      inject: [Connection],
    },
  ],
})
export class UserModule {}
```

## Alias Provider
Mau membuat provider menggunakan object provider yang sama? Maka solusinya adalah menggunakan alias provider. Alias provider adalah cara untuk membuat dependency dengan nama yang berbeda untuk membuat object provider yang sama. Cara menggunakan alias provider adalah dengan cara menggunakan atribut **provide** yang berisi nama alias, dan **useExisting** yang berisi provider yang mau kita buat alias nya.

> user-module.ts
```typescript
import { Module } from '@nestjs/common';
import { UserController } from './user.controller';
import { mailService, MailService } from './mail/mail.service';
import {
  createUserRepository,
  UserRepository,
} from './user-repository/user-repository';

@Module({
  controllers: [UserController],
  providers: [
    {
      provide: 'EmailService',
      useClass: MailService,
    },
  ],
})
export class UserModule {}
```