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
      this.configService.get<string>("SECRET"),
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
    private validationService: ValidationService,
  ) {}

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

## Setup Middleware, Custom Decorator dan Role Guard

Setelah berhasil membuat handler untuk melakukan register dan login, sekarang masuk ke pembuatan middleware, untuk penjelasan middleware sendiri sudah pernah dibahas, bisa dicek ke postingan [middleware](https://akufarish.my.id/posts/middleware/), dan untuk custom decorator juga sama bisa cek ke [custom decorator](https://akufarish.my.id/posts/nestjs-custom-decorator/).

Untuk membuat middleware, bisa menggunakan CLI Nest dengan perintah.

```terminaloutput
nest generate middleware auth common
```

Setelah menjalankan command diatas, maka nest akan mengenerate file auth.middleware didalam folder common/auth.

> auth.middleware.ts

```typescript
import {
  HttpException,
  HttpStatus,
  Injectable,
  NestMiddleware,
} from "@nestjs/common";
import { type Response } from "express";
import { JwtService } from "../jwt/jwt.service";
import { AuthUser } from "src/model/user-model";

@Injectable()
export class AuthMiddleware implements NestMiddleware {
  constructor(private jwtService: JwtService) {}

  async use(req: any, res: Response, next: () => void) {
    // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment,@typescript-eslint/no-unsafe-member-access
    const header = req?.headers.authorization;
    // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment,@typescript-eslint/no-unsafe-call,@typescript-eslint/no-unsafe-member-access
    const token = header?.split(" ")[1];

    if (!token) {
      throw new HttpException("Token missing", HttpStatus.UNAUTHORIZED);
    }

    try {
      const validate = await this.jwtService.validate(token as string);

      const user: AuthUser = {
        id: validate.id as number,
        role: validate.role as string,
      };

      // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access
      req.user = user;
      next();
    } catch (e) {
      console.error(e);
      throw new HttpException("Unauthorized", HttpStatus.UNAUTHORIZED);
    }
  }
}
```

Karena token yang dikirim itu memiliki teks **`Bearer`** didepan, maka saat membaca nilai yang dikirim pada header authorization, harus dipisah terlebih dahulu. Untuk memisah string pada javascript, bisa menggunakan method **split()**, disini menggunakan **split(' ')** yang berarti memisah string berdasarkan spasi, yang nantinya untuk bisa diakses dengan memanggil urutannya menggunakan indeks array. Maka dari itu, disini token mengambil indeks ke 1 karena indeks 1 mengambil data token saja, dan bukan `bearer`.

```typescript
const header = req?.headers.authorization;
// eslint-disable-next-line @typescript-eslint/no-unsafe-assignment,@typescript-eslint/no-unsafe-call,@typescript-eslint/no-unsafe-member-access
const token = header?.split(" ")[1];
```

Lakukan pengecekan apakah token itu diinclude didalam request atau tidak, jika tidak ada maka tampilkan pesan Token missing.

```typescript
if (!token) {
  throw new HttpException("Token missing", HttpStatus.UNAUTHORIZED);
}
```

Lanjut ke verifikasi token JWT dengan memanggil method validate dari jwt service, lalu buat data object dengan tipe `AuthUser` yang berisi id, dan role dari user yang sedang login. Lalu masukkan data user tadi ke dalam objek request, supaya data-nya dapat diakses oleh custom decorator nanti.

Jika token jwt yang dikirimkan tidak valid, maka tampilkan pesan **`Unauthorized`**.

```typescript
try {
  const validate = await this.jwtService.validate(token as string);

  const user: AuthUser = {
    id: validate.id as number,
    role: validate.role as string,
  };

  // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access
  req.user = user;
  next();
} catch (e) {
  console.error(e);
  throw new HttpException("Unauthorized", HttpStatus.UNAUTHORIZED);
}
```

Setelah selesai membuat middleware, sekarang akan dilanjut dengan membuat custom decorator. Kenapa perlu menggunakan custom decorator? Itu supaya bisa mengakses objek user dari objek request yang sudah disetting middleware sebelumnya.

Untuk membuat custom decorator, bisa menggunakan CLI Nest dengan commmand.

```terminaloutput
nest generate decorator auth common
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
  },
);
```

DIdalam decorator auth, ambil request dari handler yang dituju, dan balikkan objek user dari request.

```typescript
const request = context.switchToHttp().getRequest();
return request.user;
```

Setelah selesai membuat middleware dan custom decorator untuk mengambil data user yang sedang login, sekarang tinggal menggunakan middleware dan decorator auth yang baru saja dibuat.

Untuk menggunakan middleware, pada application module implement **NestModule** untuk mengakses method **configure** untuk menambahkan middleware. Pada praktek ini gunakan middleware auth untuk routes /api/v1/auth/\*, dan request method nya semua method.

> app.module.ts

```typescript
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(AuthMiddleware).forRoutes({
      path: "/api/v1/auth/*",
      method: RequestMethod.ALL,
    });
  }
}
```

Terkahir membuat guard untuk proses _auhtorization_ menggunakan guard dengan reflector untuk membuat guard singleton. Untuk membuat guard dapat menggunakan CLI Nest dengan command.

```terminaloutput
nest generate guard role common
```

Sedangkan untuk reflector, dapat dibuat dengan memanggil method **createDecorator** dari class **Reflector**. Disini diberi tipe string array, karena reflector ini nantinya dapat diberi nilai string array berupa role seperti ['admin', 'user'] dan lain-lain.

> role.decorator.ts

```typescript
import { Reflector } from "@nestjs/core";

export const Role = Reflector.createDecorator<string[]>();
```

> role.guard.ts

```typescript
import { CanActivate, ExecutionContext, Injectable } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { Observable } from "rxjs";
import { Role } from "./role.decorator";

@Injectable()
export class RoleGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(
    context: ExecutionContext,
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

Pertama-tama ambil data string array dari reflector menggunakan reflector yang sudah di-dependency injection pada constructor dengan method **get()** yang menerima dua parameter, yaitu reflector-nya disini berupa Role, dan handler dari routes yang dituju. Jika data role tidak ada maka return true, yang berarti handler boleh diakses.

```typescript
const roles: string[] = this.reflector.get(Role, context.getHandler());

if (!roles) {
  return true;
}
```

Ambil data user dari object request, lalu cek apakah role user dari objek user ada atau tidak didalam array string dari reflector.

```typescript
const user = context.switchToHttp().getRequest().user;
return roles.indexOf(user.role) != -1;
```

Setelah selesai semua, sekarang tinggal membuat route yang memerlukan authorization, yang menggunakan guard, reflector dan custom decorator yang dibuat tadi, dengan routes harus `/auth/*`.

> user.controller.ts

```typescript
@Get('/auth/user')
@UseGuards(RoleGuard)
@Role(['admin', 'user'])
auth(@Auth() user: AuthUser): Record<keyof AuthUser, string | number> {
    return {
      id: user.id,
      role: user.role,
    };
}

@Get('/auth/admin')
@UseGuards(RoleGuard)
@Role(['admin'])
admin(@Auth() user: AuthUser): Record<keyof AuthUser, string | number> {
    console.info(user);
    return {
      id: user.id,
      role: user.role,
    };
}
```

## Pengujian

Pengujian token kosong

<div class="alert-container error"><strong>GET</strong> /api/v1/auth/admin</div>

Request Header:

```json
{
  "Authorization": "Bearer"
}
```

Response Body:3

```json
{
  "statusCode": 401,
  "message": "Token missing"
}
```

Pengujian token invalid

<div class="alert-container error"><strong>GET</strong> /api/v1/auth/admin</div>

Request Header:

```json
{
  "Authorization": "Bearer asdasdasdasdasdasd12312"
}
```

Response Body:

```json
{
  "statusCode": 401,
  "message": "Unauthorized"
}
```

Pengujian token valid

<div class="alert-container error"><strong>GET</strong> /api/v1/auth/admin</div>

Request Header:

```json
{
  "Authorization": "Bearer {token}"
}
```

Response Body:

```json
{
  "id": 1,
  "role": "admin"
}
```
