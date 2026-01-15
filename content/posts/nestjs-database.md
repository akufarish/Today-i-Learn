+++
date = '2026-01-05T17:18:59+08:00'
draft = false
title = 'Nest JS Database'
+++

Secara default NestJS tidak menyediakan fitur untuk database atau ORM (Object Relational Mapping), namun karena itu juga kita sebagai developer diberi kebebasan untuk menggunakan berbagai macam ORM yang ada seperti prisma, drizzle, dan lain-lain.

Pada praktek ini akan menggunakan prisma sebagai ORM nya.

> install library prisma

```terminaloutput
bun add -d prisma
bun add @prisma/client
bun add @prisma/adapter-mariadb
```

Disini saya cuma akan membuat schema sederhana dimana cuma mempunyai satu table **users** yang cuma berisi id dan nama.

> init prisma

```terminaloutput
bunx prisma init
```

command diatas akan membuat file .env dan folder prisma yang berisi **schema prisma**. Pada file **schema.prisma** lah kita membuat schema database nya, seperti membuat table dan memberi relasi.

> schema.prisma

```prisma
generator client {
  provider     = "prisma-client"
  output       = "../generated/prisma"
  moduleFormat = "cjs"
}

datasource db {
  provider = "mysql"
}

model User {
  id   Int    @id @default(autoincrement())
  nama String @db.VarChar(100)

  @@map("users")
}
```

Pada file .env kita konfigurasi DATABASE_URL dengan memasukkan user, password, host, port, dan database yang mau digunakan.

> .env

```terminaloutput
DATABASE_URL=mysql://USER:PASSWORD@HOST:PORT/DATABASE
```

Untuk melakukan migration dapat menggunakan command:

```terminaloutput
bunx prisma migrate dev
bunx prisma generate
```

Dilanjut untuk membuat module untuk prisma.

> prisma.module.ts

```typescript
import { Module } from "@nestjs/common";
import { PrismaService } from "./prisma.service";

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

Terakhir membuat service untuk menggunakan **PrismaClient** dengan cara prismaService extend ke PrismaClient. Lalu didalam constructor nya buat instance Prisma ORM driver nya, disini karena saya menggunakan mysql jadi instance yang dibuat yaitu **PrismaMariaDB()**, lalu pada object parameter **PrismaMariaDB()** diisi dengan konfigurasi dari mysql seperti host, port, user, dan database yang digunakan.

```typescript
import { Injectable } from "@nestjs/common";
import { PrismaClient } from "../../../generated/prisma/client";
import { ConfigService } from "@nestjs/config";
import { PrismaMariaDb } from "@prisma/adapter-mariadb";

@Injectable()
export class PrismaService extends PrismaClient {
  constructor(private configService: ConfigService) {
    const adapter = new PrismaMariaDb({
      host: configService.get("HOST"),
      port: configService.get("PORT"),
      connectionLimit: configService.get("CONNECTION_LIMIT"),
      user: configService.get("USER"),
      database: configService.get("SELECT_DATABASE"),
    });
    super({ adapter });
    console.log("Create PrismaService");
  }
}
```

Lalu untuk pengujian saya akan modifikasi **user-repository.ts** dengan menghapus code untuk membuat factory provider, dan menggantinya dengan code untuk melakukan insert dan get data menggunakan prisma.

> user-repository.ts

```typescript
import { PrismaService } from "../../prisma/prisma/prisma.service";
import { Injectable } from "@nestjs/common";
import { User } from "../../../generated/prisma/client";

@Injectable()
export class UserRepository {
  constructor(private prismaService: PrismaService) {}

  async save(nama: string): Promise<User> {
    return await this.prismaService.user.create({
      data: {
        nama: nama,
      },
    });
  }

  async get(): Promise<User[]> {
    return await this.prismaService.user.findMany();
  }

  async getById(id: string): Promise<User | null> {
    return await this.prismaService.user.findUnique({
      where: {
        id: parseInt(id),
      },
    });
  }
}
```

Perlu diketahui juga untuk menggunakan prisma memang harus menggunakan **asynchronous function** oleh sebab itulah method-method diatas menggunakan async await. Untuk melakukan insert bisa menggunakan method **create()** dan masukkan datanya, untuk method get bisa menggunakan **findMany()** untuk mengambil semua data, atau **findUnique()** untuk mengambil data menggunakan where.

> user.controller.ts

```typescript
import {
  Controller,
  Get,
  Header,
  HttpCode,
  type HttpRedirectResponse,
  Inject,
  Param,
  Post,
  Query,
  Redirect,
  Req,
  Res,
} from "@nestjs/common";
import { type Request, type Response } from "express";
import { UserRepository } from "./user-repository/user-repository";
import { User } from "../../generated/prisma/client";

@Controller("/api/users")
export class UserController {
  constructor(private userRepository: UserRepository) {}

  @Post("/create")
  async create(@Req() request: Requst): Promise<User> {
    return await this.userRepository.save(request.body.name as string);
  }

  @Get("/:id")
  async getById(@Param("id") id: string): Promise<User | null> {
    return await this.userRepository.getById(id);
  }
}
```

Note untuk sekarang tipe request menggunakan **type Request** dari **express**. Kedepannya tidak akan menggunakan type dari express lagi.

> POST /api/users/create

Request Body:

```terminaloutput
{
    "name": "Hina Youmiya"
}
```

Response Body:

```terminaloutput
{
    "id": 5,
    "nama": "Hina Youmiya"
}
```

> GET /api/users/

Response Body

```terminaloutput
[
    {
        "id": 3,
        "nama": "Youmiya"
    },
    {
        "id": 5,
        "nama": "Hina Youmiya"
    }
]
```

> GET /api/users/5

Response Body

```terminaloutput
{
    "id": 5,
    "nama": "Hina Youmiya"
}
```
