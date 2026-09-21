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
        return this.prisma.client.orm.public.User.create(createUserDto);
    }

    async findAll() {
        return this.prisma.client.orm.public.User.all();
    }

    async findOne(id: string) {
        return this.prisma.client.orm.public.User.where({ id }).first();
    }

    async update(id: string, updateUserDto: UpdateUserDto) {
        return this.prisma.client.orm.public.User.where({ id }).update(updateUserDto);
    }

    async remove(id: string) {
        return this.prisma.client.orm.public.User.where({ id }).delete();
    }
}
```
---

#### `users.controller.ts`
```bash
import { Controller, Get, Post, Patch, Delete, Param, Body } from '@nestjs/common';
import { UsersService } from './users.service.js';
import { CreateUserDto } from './dto/create-user.dto.js';
import { UpdateUserDto } from './dto/update-user.dto.js';

@Controller('users')
export class UsersController {
    constructor(private readonly usersService: UsersService) {}

    @Post()
    create(@Body() createUserDto: CreateUserDto) {
        return this.usersService.create(createUserDto);
    }

    @Get()
    findAll() {
        return this.usersService.findAll();
    }

    @Get(':id')
    findOne(@Param('id') id: string) {
        return this.usersService.findOne(id);
    }

    @Patch(':id')
    update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
        return this.usersService.update(id, updateUserDto);
    }

    @Delete(':id')
    remove(@Param('id') id: string) {
        return this.usersService.remove(id);
    }
}
```
---

#### `users.module.ts`
```bash
import { Module } from '@nestjs/common';
import { UsersService } from './users.service.js';
import { UsersController } from './users.controller.js';
import { PrismaModule } from '../prisma/prisma.module.js';

@Module({
  imports: [PrismaModule],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```
---


#### `app.module.ts`
```bash
import { Module } from '@nestjs/common';
import { AppController } from './app.controller.js';
import { AppService } from './app.service.js';
import { ConfigModule } from '@nestjs/config';
import { PrismaModule } from './prisma/prisma.module.js';
import { UsersModule } from './users/users.module.js';

@Module({
  imports: [ConfigModule.forRoot({ isGlobal: true }), PrismaModule, UsersModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
---

#### Install koro
```bash
npm install @js-temporal/polyfill
```
---

#### `main.ts`
```bash
import { Temporal } from '@js-temporal/polyfill';
(globalThis as any).Temporal = Temporal;

import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module.js';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```
---
