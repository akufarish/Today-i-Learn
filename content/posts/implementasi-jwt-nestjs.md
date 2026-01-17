+++
date = '2026-01-16T20:57:26+08:00'
draft = false
title = 'Implementasi JWT Pada NestJS'
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

## Exception Filter

Setelah konfigurasi zod, sekarang membuat exception filter untuk menangkap error yang dihasilkan oleh zod, untuk membuat exception filter bisa menggunakan command.

```terminaloutput
nest generate filter validation commmon
```

> validation.filter.ts

```typescript
import { ArgumentsHost, Catch, ExceptionFilter } from "@nestjs/common";
import { ZodError } from "zod";
import { type Response } from "express";

@Catch(ZodError)
export class ValidationFilter implements ExceptionFilter<ZodError> {
  catch(exception: ZodError, host: ArgumentsHost) {
    const http = host.switchToHttp();
    const response = http.getResponse<Response>();

    return response.status(400).json({
      status: 400,
      message: exception.issues,
    });
  }
}
```

Decorator **@Catch()** akan menangkap error dari zod, lalu return dengan response status 400, dan message yang berisi error validasi zod.

## Register

Pertama-tama buat terlebih dahulu module, service, dan controller auth menggunakan CLI NestJS.

```terminaloutput
nest generate module user
nest generate controller user
nest generate service user
```

Setelah menjalankan command diatas, maka akan muncul folder user yang berisi file module, controller, dan service yang sudah digenerate oleh nest js

Sebelum menambahkan logic untuk register, alangkah baiknya membuat model untuk request dan response terlebih dahulu, bisa buat manual dengan membuat folder model yang didalamya berisi file-file model baik untuk request atau response yang dibuat menggunakan interface typescript.

> web-response.ts

```typescript
export interface WebResponse<T> {
  status: number;
  data: T;
}
```

Model diatas adalah struktur untuk response dari semua handler api yang berisi status code, dan data yang merupakan generic.

> user-model.ts

```typescript
export interface RegisterRequest {
  name: string;
  email: string;
  password: string;
}

export interface RegisterResponse {
  name: string;
  email: string;
}
```

Bisa diliat pada request diatas, register akan menerima request name, email, dan password, sedangkan response akan menampilkan name, dan email saja.

> user.controller.ts

```typescript
import {
  Body,
  Controller,
  Get,
  HttpCode,
  HttpStatus,
  Post,
  UseFilters,
} from '@nestjs/common';
import { UsersService } from './users.service';
import {
  type RegisterRequest,
  RegisterResponse,
} from '../model/user-model';
import { WebResponse } from '../model/web-response';
import { ValidationFilter } from '../common/validation/validation.filter';

@Controller('/api/v1')
@UseFilters(ValidationFilter)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post('/register')
  @HttpCode(HttpStatus.CREATED)
  async register(
    @Body() payload: RegisterRequest,
  ): Promise<WebResponse<RegisterResponse>> {
    return this.usersService.register(payload);
  }
```

```typescript
@Post('/register')
@HttpCode(HttpStatus.CREATED)
async register(
    @Body() payload: RegisterRequest,
  ): Promise<WebResponse<RegisterResponse>> {
    return this.usersService.register(payload);
}
```

Membuat handler register dengan method post, dan http status code 201 jika sukses. Handler ini menerima request dengan tipe RegisterRequest yang diambil menggunakan decorator **@Body()**, dengan balikan fungsi Web Reponse dengan tipe Reigster Response. Fungsi register memanggil service layer dari user service dengan method **register**.

> user.service.ts

```typescript
import { HttpException, HttpStatus, Injectable } from "@nestjs/common";
import { type RegisterRequest, RegisterResponse } from "../model/user-model";
import { WebResponse } from "../model/web-response";
import { PrismaService } from "../common/prisma/prisma/prisma.service";
import { z } from "zod";
import { ValidationService } from "../validation/validation.service";
import * as bcrpt from "bcrypt";

@Injectable()
export class UsersService {
  constructor(
    private prismaService: PrismaService,
    private validationService: ValidationService
  ) {}

  async register(
    payload: RegisterRequest
  ): Promise<WebResponse<RegisterResponse>> {
    const schema = z.object({
      email: z.email().nonempty().min(5),
      name: z.string().nonempty().min(5),
      password: z.string().nonempty().min(5),
    });

    payload = this.validationService.validate(schema, payload);

    const isUserAlreadyExist = await this.prismaService.user.findFirst({
      where: {
        email: payload.email,
      },
    });

    if (isUserAlreadyExist) {
      throw new HttpException("User already registered", HttpStatus.CONFLICT);
    }

    payload.password = await bcrpt.hash(payload.password, 10);

    const user = await this.prismaService.user.create({
      data: payload,
    });

    return {
      status: HttpStatus.CREATED,
      data: user,
    };
  }
}
```

User service melakukan dependency injection pada prisma service untuk mengakses prisma client yang memungkinkan aplikasi dapat berkomunikasi dengan database, dan validation service untuk validasi request.

```typescript
constructor(
    private prismaService: PrismaService,
    private validationService: ValidationService
) {}
```

Fungsi register menerima satu parameter bertipe RegisterRequest, dengan balikan WebResponse dengan tipe RegisterResponse.

```typescript
async register(
    payload: RegisterRequest,
  ): Promise<WebResponse<RegisterResponse>> {
    const schema = z.object({
      email: z.email().nonempty().min(5),
      name: z.string().nonempty().min(5),
      password: z.string().nonempty().min(5),
    });

    payload = this.validationService.validate(schema, payload);

    const isUserAlreadyExist = await this.prismaService.user.findFirst({
      where: {
        email: payload.email,
      },
    });

    if (isUserAlreadyExist) {
      throw new HttpException('User already registered', HttpStatus.CONFLICT);
    }

    payload.password = await bcrpt.hash(payload.password, 10);

    const user = await this.prismaService.user.create({
      data: payload,
    });

    return {
      status: HttpStatus.CREATED,
      data: user,
    };
}
```

Pada fungsi register, hal pertama yang dilakukan adalah membuat schema untuk validasi menggunakan zod object.

```typescript
const schema = z.object({
  email: z.email().nonempty().min(5),
  name: z.string().nonempty().min(5),
  password: z.string().nonempty().min(5),
});

payload = this.validationService.validate(schema, payload);
```

Untuk penjelasan validitas schema diatas bisa lihat table dibawah.

| Attribute | Validation |     | Description                               |
| --------- | ---------- | --- | ----------------------------------------- |
| email     | email      |     | request harus berupa email yang valid     |
|           | nonempty   |     | request harus berupa email yang valid     |
|           | min(5)     |     | request harus memiliki minimal 5 karakter |
| name      | string     |     | request harus berupa string               |
|           | nonempty   |     | request harus berupa email yang valid     |
|           | min(5)     |     | request harus memiliki minimal 5 karakter |
| password  | string     |     | request harus berupa string               |
|           | nonempty   |     | request harus berupa email yang valid     |
|           | min(5)     |     | request harus memiliki minimal 5 karakter |

Lakukan pengecekan email ke database, jika email dari request sudah terdaftar di-database, maka return http exception dengan pesan user already registered.

```typescript
const isUserAlreadyExist = await this.prismaService.user.findFirst({
  where: {
    email: payload.email,
  },
});

if (isUserAlreadyExist) {
  throw new HttpException("User already registered", HttpStatus.CONFLICT);
}
```

Buat data user dan return status code createed (201), dengan data user.

```typescript
const user = await this.prismaService.user.create({
  data: payload,
});

return {
  status: HttpStatus.CREATED,
  data: user,
};
```

Pengujian jika input kosong

<div class="alert-container error"><strong>POST</strong> /api/v1/register</div>

Request Body:

```json
{
  "email": "",
  "password": "",
  "name": ""
}
```

Response Body:

```json
{
  "status": 400,
  "message": [
    {
      "origin": "string",
      "code": "invalid_format",
      "format": "email",
      "pattern": "/^(?!\\.)(?!.*\\.\\.)([A-Za-z0-9_'+\\-\\.]*)[A-Za-z0-9_+-]@([A-Za-z0-9][A-Za-z0-9\\-]*\\.)+[A-Za-z]{2,}$/",
      "path": ["email"],
      "message": "Invalid email address"
    },
    {
      "origin": "string",
      "code": "too_small",
      "minimum": 1,
      "inclusive": true,
      "path": ["email"],
      "message": "Too small: expected string to have >=1 characters"
    },
    {
      "origin": "string",
      "code": "too_small",
      "minimum": 5,
      "inclusive": true,
      "path": ["email"],
      "message": "Too small: expected string to have >=5 characters"
    },
    {
      "origin": "string",
      "code": "too_small",
      "minimum": 1,
      "inclusive": true,
      "path": ["name"],
      "message": "Too small: expected string to have >=1 characters"
    },
    {
      "origin": "string",
      "code": "too_small",
      "minimum": 5,
      "inclusive": true,
      "path": ["name"],
      "message": "Too small: expected string to have >=5 characters"
    },
    {
      "origin": "string",
      "code": "too_small",
      "minimum": 1,
      "inclusive": true,
      "path": ["password"],
      "message": "Too small: expected string to have >=1 characters"
    },
    {
      "origin": "string",
      "code": "too_small",
      "minimum": 5,
      "inclusive": true,
      "path": ["password"],
      "message": "Too small: expected string to have >=5 characters"
    }
  ]
}
```

Pengujian berhasil registrasi

<div class="alert-container success"><strong>POST</strong> /api/v1/register</div>

Request Body:

```json
{
  "email": "test@gmail.com",
  "password": "testing",
  "name": "testing"
}
```

Response Body:

```json
{
  "status": 201,
  "data": {
    "id": 3,
    "name": "testing",
    "email": "test@gmail.com"
  }
}
```

## Login

Sebelum membuat fungsi login, buat terlebih dahulu interface untuk request dan response login.

> user-model.ts

```typescript
export interface LoginRequest {
  email: string;
  password: string;
}

export interface LoginResponse {
  name: string;
  email: string;
  token: string;
}
```

Pada file user.service.ts, sekarang buat function login yang menerima satu parameter yaitu payload dengan tipe LoginRequest, dengan balikan fungsi berupa WebResponse dengan tipe LoginResponse.

> user.service.ts

```typescript
async login(payload: LoginRequest): Promise<WebResponse<LoginResponse>> {
    const schema = z.object({
      email: z.email().nonempty(),
      password: z.string().nonempty(),
    });

    payload = this.validationService.validate(schema, payload);

    const user = await this.prismaService.user.findFirst({
      where: {
        email: payload.email,
      },
    });

    if (!user) {
      throw new HttpException('User not found', HttpStatus.NOT_FOUND);
    }

    const isPasswordValid = await bcrpt.compare(
      payload.password,
      user.password,
    );

    if (!isPasswordValid) {
      throw new HttpException('Incorrect Password', HttpStatus.BAD_REQUEST);
    }

    const jwt = await this.jwtService.createToken({
      id: user.id,
      role: user.role as string,
    });

    const loginResponse: LoginResponse = {
      email: user.email,
      name: user.name,
      token: jwt,
    };

    return {
      data: loginResponse,
      status: HttpStatus.OK,
    };
  }
```

Sama seperti pada fungsi register, hal pertama yang dilakukan adalah membuat schema zod untuk validasi request yang dikirim ke handler login.

```typescript
const schema = z.object({
  email: z.email().nonempty(),
  password: z.string().nonempty(),
});

payload = this.validationService.validate(schema, payload);
```

Untuk penjelasan validitas schema diatas bisa lihat table dibawah.

| Attribute | Validation |     | Description                           |
| --------- | ---------- | --- | ------------------------------------- |
| email     | email      |     | request harus berupa email yang valid |
|           | nonempty   |     | request harus berupa email yang valid |
| password  | string     |     | request harus berupa string           |
|           | nonempty   |     | request harus berupa email yang valid |

Lakukan query ke database untuk mencari data user, berdasarkan email dari request.

```typescript
const user = await this.prismaService.user.findFirst({
  where: {
    email: payload.email,
  },
});
```

Jika data user tidak ditemukan, maka return error dengan pesan "User not found", dengan http status code 404 not found.

```typescript
if (!user) {
  throw new HttpException("User not found", HttpStatus.NOT_FOUND);
}
```

Validasi password yang dihash menggunakan fungsi compare dari bcrypt yang menerima dua parameter yaitu password dari request, dan password yang di-hash, yang nantinya akan return boolean.

```typescript
const isPasswordValid = await bcrpt.compare(payload.password, user.password);
```

Jika password tidak valid, maka tampilkan pesan "Incorrect Password" dengan http code 401 bad request.

```typescript
if (!isPasswordValid) {
  throw new HttpException("Incorrect Password", HttpStatus.BAD_REQUEST);
}
```

Panggil fungsi createToken dari jwt service untuk membuat jwt, dengan id user, dan role user sebagai claims.

```typescript
const loginResponse: LoginResponse = {
  email: user.email,
  name: user.name,
  token: jwt,
};
```

Kembalikan LoginResponse dengan http status code 200, jika request sukses.

```typescript
return {
  data: loginResponse,
  status: HttpStatus.OK,
};
```

Panggil method login dari user service ke controller. Buat fungsi login dengan decorator **@Post()** untuk membuat post request, dan jika sukses, maka return http status code 200 ok.

Pada fungsi login juga karena akan mengirim request, jadi tambahkan decorator **@Body()** untuk mengambil request yang dikirim.

> user.controller.ts

```typescript
@Post('/login')
@HttpCode(HttpStatus.OK)
async login(
    @Body() payload: LoginRequest,
  ): Promise<WebResponse<LoginResponse>> {
    return this.usersService.login(payload);
}
```

Pengujian login berhasil

<div class="alert-container success"><strong>POST</strong> /api/v1/login</div>

Request Body:

```json
{
  "email": "test@gmail.com",
  "password": "testing"
}
```

Response Body:

```json
{
  "data": {
    "email": "testa@gmail.com",
    "name": "testing",
    "token": "eyJhbGciOiJIUzI1NiJ9.eyJpZCI6NCwicm9sZSI6bnVsbCwiaWF0IjoxNzY4NjUzNTI2LCJpc3MiOiJ1cm46ZXhhbXBsZTppc3N1ZXIiLCJhdWQiOiJ1cm46ZXhhbXBsZTphdWRpZW5jZSJ9._8gGXDhRwC_quSPwKRVcUVRRS7CI-pp7WJ22p3iJYkU"
  },
  "status": 200
}
```

Pengujian login akun tidak terdaftar

<div class="alert-container error"><strong>POST</strong> /api/v1/login</div>

Request Body:

```json
{
  "email": "tidakadaa@gmail.com",
  "password": "testing"
}
```

Response Body:

```json
{
  "statusCode": 404,
  "message": "User not found"
}
```

Pengujian login akun password tidak cocok

<div class="alert-container error"><strong>POST</strong> /api/v1/login</div>

Request Body:

```json
{
  "email": "testa@gmail.com",
  "password": "salah"
}
```

Response Body:

```json
{
  "statusCode": 400,
  "message": "Incorrect Password"
}
```

