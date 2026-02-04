+++
date = '2026-01-10T20:49:01+08:00'
draft = false
title = 'Nest JS Interceptor'
tags = ['Indonesia', 'NestJS', 'Programming']
+++

Secara konsep, interceptor itu mirip dengan middleware, sama-sama menjadi jembatan antara request dari client ke controller, namun pembedanya adalah interceptor dapat mengubah response yang diberikan oleh controller.

Pembeda antara middleware dan interceptor hanya terdapat pada middleware yang cuma bisa mengolah request dan meneruskan request tersebut ke middleware selanjutnya atau ke controller. Sedangkan interceptor dapat menerima Response dari controller, dan mengubah response-nya.

Berikut adalah diagram sederhana dari interceptor

<div class="img-container">
<img alt="middleware-light-diagram" src="/images/interceptor-light.png" class="img-light img">
<img alt="middleware-dark-diagram" src="/images/interceptor-dark.png" class="img-dark img">
</div>

Bisa diliat dari diagram diatas, memang mirip seperti diagram middleware, cuma yang jadi pembeda adalah request yang dikirim client akan masuk ke interceptor terlebih dahulu dan disini interceptor dapat mengubah atau melakukan pengecekkan terhadap request, lalu dikirim ke controller, dan dari controller, data yang dikembalikan juga akan masuk ke interceptor dan response tadi dapat dimodifikasi.

Untuk membuat interceptor, bisa membuat class turunan interface **NestInterceptor**, atau bisa menggunakan CLI NestJS.

```terminaloutput
nest generate interceptor nama_file path_file
```

> time.interceptor.ts

```typescript
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from "@nestjs/common";
import { map, Observable } from "rxjs";
import { type Request } from "express";

@Injectable()
export class TimeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const http = context.switchToHttp();
    const request = http.getRequest<Request>();
    return next.handle().pipe(
      map((value) => {
        // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment,@typescript-eslint/no-unsafe-member-access
        value.username = request.body["username"];
        // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access
        value.timestamp = new Date();
        // eslint-disable-next-line @typescript-eslint/no-unsafe-return
        return value;
      })
    );
  }
}
```

Diatas adalah contoh interceptor yang dimana untuk setiap response json, tambahkan field timestamp untuk mengetahui kapan request ini dijalankan, dan tambahkan field username dari request yang dikirim.

Untuk menggunakan interceptor bisa gunakan decorator **@UseInterceptor()** di controller method atau di controller, jika ingin menambahkan interceptor secara global, didalam NestJS application file **main.ts**, bisa menggunakan method **useGlobalInterceptors()**.

```typescript
@Post('/login')
@UseFilters(ValidationFilter)
@UsePipes(new ValidationPipe(loginUserRequestValidation))
@UseInterceptors(TimeInterceptor)
login( @Body() request: LoginUserRequest): any {
    return {
        data: `Hello ${request.username}`,
    };
}
```

<div class="alert-container success"><strong>POST</strong> /api/users/login</div>

Request Body:

```json
{
  "username": "admin",
  "password": "admin"
}
```

Response Body:

```json
{
  "data": "Hello admin",
  "username": "admin",
  "timestamp": "2026-01-10T13:12:29.271Z"
}
```
