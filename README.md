## CRUD generator

#### Generating a new resource
```bash
nest g resource [name]
```
```bash
nest g resource user
```
---

#### Prisma er service and module create koro.
```bash
nest g module prisma
```
```bash
nest g service prisma
```
---

[Connect NestJ with Prisma and Neon (Prisma v8)](https://github.com/Omarmdwasimuddin/Connect-NestJ-with-Prisma-and-Neon-Prisma-v8-)

#### Package install
```bash
npm i class-validator class-transformer
```
```bash
npm i @nestjs/mapped-types
```
---


#### `prisma.service.ts`
```bash
import { Injectable } from '@nestjs/common';
import { db } from './db.js';

@Injectable()
export class PrismaService {
    get client() {
        return db;
    }
}
```
---
