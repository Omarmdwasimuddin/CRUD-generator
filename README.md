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

#### `prisma.module.ts`
```bash
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma.service.js';

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```
---

#### `dto/create-user.dto.ts`
```bash
import { IsString, IsEmail, IsNotEmpty, IsOptional, MinLength } from 'class-validator';

export class CreateUserDto {
    @IsEmail()
    email!: string;

    @IsOptional()
    @IsString()
    username?: string;

    @IsString()
    @IsNotEmpty()
    @MinLength(6)
    password!: string;
}
```
---


#### `dto/update-user.dto.ts`
```bash
import { PartialType } from '@nestjs/mapped-types';
import { CreateUserDto } from './create-user.dto.js';

export class UpdateUserDto extends PartialType(CreateUserDto) {}
```
---

#### `users.service.ts`
```bash
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service.js';
import { CreateUserDto } from './dto/create-user.dto.js';
import { UpdateUserDto } from './dto/update-user.dto.js';

@Injectable()
export class UsersService {
    constructor(private prisma: PrismaService) {}

    async create(createUserDto: CreateUserDto) {
        return this.prisma.client.orm.public.User.create(data);
    }

    async findAll() {
        return this.prisma.client.orm.public.User.all();
    }

    async findOne(id: string) {
        return this.prisma.client.orm.public.User.where({ id }).first();
    }

    async update(id: string, updateUserDto: UpdateUserDto) {
        return this.prisma.client.orm.public.User.where({ id }).update(data);
    }

    async remove(id: string) {
        return this.prisma.client.orm.public.User.where({ id }).delete();
    }
}
```
---

#### ``
```bash

```
---

#### ``
```bash

```
---
