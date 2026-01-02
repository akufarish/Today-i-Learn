+++
date = '2026-01-02T10:22:59+08:00'
draft = false
title = 'Mencoba Nest Js (3)'
+++

## Introduction
Memasuki hari ketiga belajar nest-JS disini saya akan melakukan recap dari apa yang saya pelajari hari ini yaitu unit test, integration test, provider, dependency injection, dan custom provider.

## Unit Test
Saat membuat controller menggunakan CLI nest, secara otomatis nest juga akan membuatkan unit test untuk controller tersebut. File unit test biasanya berformat *.spec.ts.

> Contoh kode unit test
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { UserController } from './user.controller';
import { UserService } from './user.service';

describe('UserController', () => {
  let controller: UserController;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UserController],
      providers: [
        UserService,
      ],
    }).compile();

    controller = module.get<UserController>(UserController);
  });

  it('should be return GET 1', () => {
    expect(controller.getById('1')).toBe('GET 1');
  });

  it('should be return MongoDB', () => {
    expect(controller.getConnection()).toBe('MongoDB');
  });
});

```

Pada kode unit test, bagian beforeEach nya mirip dengan file module, kita harus memasukkan controller atau provider yang digunakan pada file controller supaya unit test ini dapat mengenali dependency apa saja yang digunakan.

> Command untuk menjalankan unit test
```terminaloutput
bun test
```

## Integration Test
Berbeda dengan unit test yang hanya melakukan pengujian testing pada satu controller atau model, integration test dapat melakukan testing ke seluruh aplikasi. Secara default file untuk integration test diakhiri dengan *.e2e.spec.ts, e2e merupakan singkatan dari end to end. Perubahan konfigurasi dapat dilakukan dengan mengubah isi file test/jest-e2e.json.

> Contoh kode integration test
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication } from '@nestjs/common';
import request from 'supertest';
import { App } from 'supertest/types';
import { AppModule } from './../src/app.module';

describe('AppController (e2e)', () => {
  let app: INestApplication<App>;

  beforeEach(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('Should can say hello', async () => {
    const result = await request(app.getHttpServer())
      .get('/api/users/hello')
      .query({
        name: 'Farish',
      });

    expect(result.status).toBe(200);
    expect(result.text).toBe('Hello Farish');
  });

  it('Should return GET 1', async () => {
    const result = await request(app.getHttpServer()).get('/api/users/1');

    expect(result.status).toBe(200);
    expect(result.text).toBe('GET 1');
  });
});
```
> Command untuk menjalankan e2e testing
```terminaloutput
bun test:e2e
```

## Provider
Provider adalah sebuah object yang bisa berupa service, repository, factory, helper, dan lain-lain yang dapat di inject sebagai dependency ke object lainnya seperti controller atau bahakn provider lain.

Untuk membuat provider, kita bisa membuat class lalu tambahkan decorator @Injectable(). Selanjutnya provider yang baru dibuat harus diregistrasikan pada module agar bisa dikenali oleh NestJS. Dibandingkan membuat file dan memasukkan provider secara manual, kita bisa menggunakan CLI NestJS dengan command

```terminaloutput
nest generate provider nama_file path_file
``` 

dan secara otomatis Nest akan membuat file provider sekaligus meregistrasikan provider tersebut pada module.

Atau jika ingin membuat provider sebagai service Nest juga menyediakan CLI nya dengan command
```terminaloutput
nest generate service nama_service path_file
```

> Contoh file provider
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class Connection {
  getName(): string | null {
    return null;
  }
}
```

> Contoh file provider sebagai service
```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  sayHello(name: string): string {
    return `Hello ${name}`;
  }
}
```

## Dependency Injection
Dependency injection adalah design pattern dimana object membutuhkan object lain, Contohnya pada UserController yang memerlukan UserService. NestJS memiliki tiga cara untuk melakukan dependency injection.

Pertama dapat lewat constructor dan by default juga NestJS jika ingin melakukan dependency injection menggunakan constructor.
```typescript
@Controller('/api/users')
export class UserController {
    constructor(
        private service: UserService,
    ) {}
}
```
Kedua Property-based Injection
Kita dapat membuat property didalam class lalu tambahkan decorator **@Injectable()** pada property tersebut.
```typescript
@Controller('/api/users')
export class UserController {
    @Inject()
    private userService: UserService;
}
```
Terakhir Optional Dependency jika ingin membuat dependency yang opstional, maka dapat menambahkan decorator **@Optional()** pada property, karena NestJS secara default dependency itu wajib diisi, jika dependency yang diperlukan itu tidak ada maka akan langsung dianggap error oleh NestJS.
```typescript
@Controller('/api/users')
export class UserController {
    @Optional()
    @Inject()
    private userService: UserService;
}
```