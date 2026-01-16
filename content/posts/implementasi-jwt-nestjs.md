+++
date = '2026-01-16T20:57:26+08:00'
draft = false
title = 'Implementasi JWT NestJS'
tags = ['Indonesia', 'NestJS', 'Programming']
+++

Setelah sebelumnya sudah belajar NestJS dasar, dan juga JWT. Pada kali ini kita akan mencoba untuk mengimplementasikan JWT kedalam aplikasi NestJS.

## Setup Project

Membuat project baru dengan command:

```terminaloutput
nest new nestjs-jwt
```

Masuk ke folder project.

```terminaloutput
cd nestjs-jwt
```

Karena pada praktek ini menggunakan database, maka perlu install library prisma.

```terminaloutput
bun add @prisma/client @prisma/adapter-mariadb
bun add -d prisma
```

Bcrypt untuk melakukan enkripsi password, karena secara default NestJS tidak menyediakan fitur untuk enkripsi.

```terminaloutput
bun add bcrypt
bun add -d @types/bcrypt
```

Zod untuk validasi request yang dikirim.

```terminaloutput
bun add zod
```

Terakhir jose yang merupakan library untuk menggunakan jwt.

```terminaloutput
bun add jose
```

## Konfigurasi Prisma

Membuat module dan service prisma.

```terminaloutput
nest generate module prisma common
nest generate service prisma common
```

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

PrismaService extends **PrismaClient** untuk membuat singleton object dari prisma client yang nantinya bisa di-inject ke file lain.

```typescript
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
```

Konfigurasi adapter mysql, dan disini untuk config mysql seperti host, port dan lain-lain berasal dari .env, maka dari itu perlu melakukan dependency injection ke **ConfigService** supaya bisa mengakses .env.

```typescript
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
```

Immplementasi lifecycle NestJS dimana ketika semua module di load, maka koneksikan database, dan jika aplikasi di-shutdown maka putuskan koneksi database.

```typescript
async onModuleInit() {
    console.info("Connected prisma");
    await this.$connect();
}
async onModuleDestroy() {
    console.info("Disconnected Prisma");
    await this.$disconnect();
}
```

## Konfigurasi Service JWT

Membuate service JWT yang berisi fungsi untuk membuat token, dan validasi token.

```terminaloutput
nest generate service jwt common
```

> jwt.service.ts

```typescript
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import * as jose from "jose";
import { AuthUser } from "src/model/user-model";

@Injectable()
export class JwtService {
  protected secret: Uint8Array;
  protected issuer: string;
  protected audience: string;

  constructor(private configService: ConfigService) {
    this.secret = new TextEncoder().encode(
      this.configService.get<string>("SECRET")
    );
    this.issuer = this.configService.get<string>("ISSUER") as string;
    this.audience = this.configService.get<string>("AUDIENCE") as string;
  }

  async validate(token: string) {
    const { payload } = await jose.jwtVerify(token, this.secret, {
      issuer: this.issuer,
      audience: this.audience,
    });

    return payload;
  }

  async createToken(payload: Record<keyof AuthUser, string | number>) {
    return await new jose.SignJWT(payload)
      .setProtectedHeader({ alg: "HS256" })
      .setIssuedAt()
      .setIssuer(this.issuer)
      .setAudience(this.audience)
      .sign(this.secret);
  }
}
```

Pada fungsi **createToken** menerima satu parameter yaitu payload yang bertipe object, dengan key value **AuthUser** yang berisi id, role, dan tipe value berupa string atau number. Untuk membuat JWT bisa membuat object dari class **SignJWT** yang memiliki constructor berupa object yang ingin dijadikan **claims**. Pada kode dibawah id, dan role yang dijadikan **claims**.

Lalu dilanjut dengan chain method **setProtectedHeader** untuk konfigurasi header disini menggunakan algoritma HS256, set issuer (pembuat jwt) dan audience (penerima jwt), dan diakhiri dengan sign.

```typescript
async createToken(payload: Record<keyof AuthUser, string | number>) {
    return await new jose.SignJWT(payload)
      .setProtectedHeader({ alg: 'HS256' })
      .setIssuedAt()
      .setIssuer(this.issuer)
      .setAudience(this.audience)
      .sign(this.secret);
}
```

Untuk fungsi validasi cukup menerima satu parameter yaitu token, dengan return any, yang berfungsi untuk validasi jwt. Di dalam fungsi ini memanggil fungsi jwtVerify yang menerima tiga parameter yaitu token jwt, secret, dan options.

```typescript
async validate(token: string): Promise<any> {
    const { payload } = await jose.jwtVerify(token, this.secret);

    return payload;
}
```

## Konfigurasi ZOD

Sama seperti prisma dan jwt, untuk membuat project lebih terstruktur, rapi, dan _reusable_. Dibuatlah module dan service untuk zod itu sendiri.

```terminaloutput
nest generate module zod common
nest generate service zod common
```

> validation.service.ts

```typescript
import { Injectable } from "@nestjs/common";
import { ZodType } from "zod";

@Injectable()
export class ValidationService {
  validate<T>(schema: ZodType<T>, data: T): T {
    return schema.parse(data);
  }
}
```

Validation service memiliki method validate dengan tipe generic yang menerima dua parameter, yaitu schema bertipe ZodType generic, dan data dengan tipe generic. dan balikan dari fungsi ini adalah memanggil fungsi parse dari zod untuk melakukan parsing berdasarkan schema yang dikirim.

