# NestJS CRUD Generator — সহজ বাংলা ব্যাখ্যা

## সমস্যা কোথায়?

কোনো project-এ নতুন feature যোগ করতে গেলে প্রায়ই নতুন resource যোগ করতে হয়। প্রতিটা resource-এর জন্য কিছু repetitive কাজ বারবার করতে হয়।

ধরা যাক তোমার `User` আর `Product` — এই দুইটা entity-র জন্য CRUD endpoint বানাতে হবে। Best practice অনুযায়ী, প্রতিটা entity-র জন্য এই কাজগুলো করতে হয়:

1. **Module** generate করা (`nest g mo`) — code গুছিয়ে রাখা, related component-গুলোকে একসাথে গ্রুপ করার জন্য
2. **Controller** generate করা (`nest g co`) — CRUD route define করার জন্য (GraphQL app হলে query/mutation)
3. **Service** generate করা (`nest g s`) — business logic implement ও আলাদা রাখার জন্য
4. **Entity class/interface** generate করা — resource-এর data shape represent করার জন্য
5. **DTO (Data Transfer Object)** generate করা — data network-এ কীভাবে পাঠানো হবে সেটা ঠিক করার জন্য

**এতগুলো ধাপ!** প্রতিটা resource-এর জন্য এগুলো বারবার করা সময়সাপেক্ষ আর ক্লান্তিকর।

---

## সমাধান: `nest g resource`

এই repetitive কাজ কমাতে Nest CLI একটা **generator (schematic)** দেয়, যেটা সব boilerplate code automatic জেনারেট করে দেয়।

> **Note:** এই schematic দিয়ে HTTP controller, Microservice controller, GraphQL resolver (code-first ও schema-first দুটোই), আর WebSocket Gateway — সবকিছুই জেনারেট করা যায়।

### নতুন Resource তৈরি করা

Project-এর root directory-তে গিয়ে চালাও:

```bash
nest g resource
```

শুধু module/service/controller class-ই না, `nest g resource` command আরও তৈরি করে দেয়:
- Entity class
- DTO class
- Testing (`.spec`) file

---

## REST API-এর জন্য জেনারেট হওয়া Controller

```ts
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
    return this.usersService.findOne(+id);
  }

  @Patch(':id')
  update(@Param('id') id: string, @Body() updateUserDto: UpdateUserDto) {
    return this.usersService.update(+id, updateUserDto);
  }

  @Delete(':id')
  remove(@Param('id') id: string) {
    return this.usersService.remove(+id);
  }
}
```

এখানে দেখা যাচ্ছে — কোনো কিছু নিজে না লিখেই, সবগুলো CRUD endpoint-এর জন্য **placeholder automatic তৈরি হয়ে গেছে** (REST-এর জন্য route, GraphQL-এর জন্য query/mutation, Microservice/WebSocket Gateway-এর জন্য message subscribe)।

> **Note:** জেনারেট হওয়া Service class কোনো নির্দিষ্ট ORM বা data source-এর সাথে বাঁধা না। এটা এতটাই generic যে যেকোনো project-এর প্রয়োজন মেটাতে পারে। By default সবগুলো method-এ placeholder থাকে — নিজের project-এর data source অনুযায়ী সেগুলো পূরণ করে নিতে হয়।

---

## GraphQL-এর জন্য Resolver জেনারেট করা

REST controller-এর বদলে GraphQL resolver জেনারেট করতে চাইলে, transport layer হিসেবে **GraphQL (code first)** বা **GraphQL (schema first)** সিলেক্ট করলেই হবে:

```
$ nest g resource users

> ? What transport layer do you use? GraphQL (code first)
> ? Would you like to generate CRUD entry points? Yes
> CREATE src/users/users.module.ts (224 bytes)
> CREATE src/users/users.resolver.spec.ts (525 bytes)
> CREATE src/users/users.resolver.ts (1109 bytes)
> CREATE src/users/users.service.spec.ts (453 bytes)
> CREATE src/users/users.service.ts (625 bytes)
> CREATE src/users/dto/create-user.input.ts (195 bytes)
> CREATE src/users/dto/update-user.input.ts (281 bytes)
> CREATE src/users/entities/user.entity.ts (187 bytes)
> UPDATE src/app.module.ts (312 bytes)
```

> **Hint:** Test file জেনারেট না করতে চাইলে `--no-spec` flag দেওয়া যায়:
> ```bash
> nest g resource users --no-spec
> ```

জেনারেট হওয়া Resolver:

```ts
import { Resolver, Query, Mutation, Args, Int } from '@nestjs/graphql';
import { UsersService } from './users.service.js';
import { User } from './entities/user.entity.js';
import { CreateUserInput } from './dto/create-user.input.js';
import { UpdateUserInput } from './dto/update-user.input.js';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Mutation(() => User)
  createUser(@Args('createUserInput') createUserInput: CreateUserInput) {
    return this.usersService.create(createUserInput);
  }

  @Query(() => [User], { name: 'users' })
  findAll() {
    return this.usersService.findAll();
  }

  @Query(() => User, { name: 'user' })
  findOne(@Args('id', { type: () => Int }) id: number) {
    return this.usersService.findOne(id);
  }

  @Mutation(() => User)
  updateUser(@Args('updateUserInput') updateUserInput: UpdateUserInput) {
    return this.usersService.update(updateUserInput.id, updateUserInput);
  }

  @Mutation(() => User)
  removeUser(@Args('id', { type: () => Int }) id: number) {
    return this.usersService.remove(id);
  }
}
```

এখানে শুধু boilerplate mutation/query-ই তৈরি হয়নি, বরং **সবকিছু একসাথে জোড়া লাগানোই আছে** — `UsersService`, `User` entity, আর সংশ্লিষ্ট DTO (input) সবগুলোই properly wire করা।

---

## সংক্ষেপে (Quick Summary)

| বিষয় | মূল কথা |
|---|---|
| সমস্যা | প্রতিটা নতুন resource-এর জন্য module/controller/service/entity/DTO — এই ৫টা জিনিস বারবার manually বানাতে হয় |
| সমাধান | `nest g resource` — একটা command দিয়ে সবকিছু automatic জেনারেট |
| যা যা জেনারেট হয় | Module, Controller/Resolver, Service, Entity, DTO, `.spec` test file |
| Support করা transport | REST, GraphQL (code-first/schema-first), Microservice, WebSocket Gateway |
| Service-এর বিশেষত্ব | কোনো নির্দিষ্ট ORM-এ বাঁধা না — placeholder থাকে, নিজের project অনুযায়ী পূরণ করতে হয় |
| Test file বাদ দিতে | `nest g resource users --no-spec` |

**তোমার [[ecommerce-nextjs-nestjs]] project-এর সাথে সংযোগ:** তুমি এই project-এ modules/products/cart/orders/payments ইত্যাদি অনেকগুলো resource manually structure করেছ। ভবিষ্যতে নতুন resource (যেমন coupon, notification-preference) যোগ করার সময় `nest g resource <name>` দিয়ে module+controller+service+DTO-এর initial boilerplate এক কমান্ডেই বানিয়ে ফেলতে পারবে, এরপর শুধু business logic বসাতে হবে।
