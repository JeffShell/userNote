# Nest的内置类

## `@nestjs/core`

### NestFactory

想要创建Nest实例，就得用该类，该类暴露了一些静态方法用于创建应用实例。
```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```


### BaseExceptionFilter

如果想要完全自定义异常过滤器，可以继承该类并调用`catch()` 方法

```ts
import { Catch, ArgumentsHost } from '@nestjs/common';
import { BaseExceptionFilter } from '@nestjs/core';

@Catch()
export class AllExceptionsFilter extends BaseExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    super.catch(exception, host);
  }
}
```

### APP_FILTER

全局过滤器

```ts
import { Module } from '@nestjs/common';
import { APP_FILTER } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_FILTER,
      useClass: HttpExceptionFilter,
    },
  ],
})
export class AppModule {}
```

### APP_INTERCEPTOR

全局拦截器

```ts
import { Module } from '@nestjs/common';
import { APP_INTERCEPTOR } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_INTERCEPTOR,
      useClass: LoggingInterceptor,
    },
  ],
})
export class AppModule {}
```

### APP_PIPE

全局管道

```ts
import { Module } from '@nestjs/common';
import { APP_PIPE } from '@nestjs/core';

@Module({
  providers: [
    {
      provide: APP_PIPE,
      useClass: ValidationPipe
    }
  ]
})
export class AppModule {}
```

## `@nestjs/platform-fastify`

`@nestjs/platform-fastify` 是Nestjs适配fastify框架的HTTP驱动

+ FastifyAdapter
+ NestFastifyApplication

NestJS默认使用的库是 Express，如果想改成fastify，就按下面的写法：
```ts
import { NestFactory } from '@nestjs/core';
import { NestFastifyApplication, FastifyAdapter } from '@nestjs/platform-fastify';
import { AppModule } from './app.module';
import { join } from 'path';

async function bootstrap() {
    const app = await NestFactory.create<NestFastifyApplication>(AppModule, new FastifyAdapter());
    // 做一些事情
    await app.listen(3000);
}
bootstrap();
```

## `@nestjs/common`

### DynamicModule

表示动态模块，一般用作类型，如下代码，该方法表示返回一个动态模块

```ts
import { Module, DynamicModule } from '@nestjs/common';
import { createDatabaseProviders } from './database.providers';
import { Connection } from './connection.provider';

@Module({
  providers: [Connection],
})
export class DatabaseModule {
  static forRoot(entities = [], options?): DynamicModule {  //这行代码
    const providers = createDatabaseProviders(options, entities);
    return {
      module: DatabaseModule,
      providers: providers,
      exports: providers,
    };
  }
}
```

### Module

装饰类，表示模块，可接受的属性对象有：
+ `providers`：由 Nest 注入器实例化的提供者，并且可以至少在整个模块中共享
+ `controllers`：必须创建的一组控制器
+ `imports`：导入模块的列表，这些模块导出了此模块中所需提供者
+ `exports`：由本模块提供并应在其他模块中可用的提供者的子集。

例子：
```ts
import { Module } from '@nestjs/common';
import { CatsController } from './cats.controller';
import { CatsService } from './cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
  exports: [CatsService]
})
export class CatsModule {}
```

### Provider

类型，表示 可以作为应用程序的依赖注入容器中的提供者。在NestJS中，提供者被用于提供可被注册到类中的实例、值、工厂函数等等。如

```ts
//文件 service.provider.ts
import { Injectable, Provider } from '@nestjs/common';

@Injectable()
class MyService {
  // ...
}

const myServiceProvider: Provider = {
  provide: MyService,
  useClass: MyService,
};

//另一个文件

import { Module } from '@nestjs/common';
import { myServiceProvider } from './service.provider.ts';

@Module({
  providers: [myServiceProvider], // 注入了
})
export class AppModule {}
```

### Type

Type 是 @nestjs/common 模块中出的一个泛型类型。

Type 泛型类型用于表示构造函数的类型。它是 TypeScript 内置的一个类型，在 NestJS 中使用它可以帮助进行静态类型检查和引用类构造函数。

如：

```ts
import { Injectable } from '@nestjs/common';
import { Type } from '@nestjs/common';

@Injectable()
class MyService {
  // ...
}

//用法一  定义提供者（）：当你使用提供者时，它通常指定为提供者对象中的 useClass 属性的类型
const myServiceProvider = {
  provide: 'MyServiceProvider',
  useClass: MyService as Type<MyService>,
};

//用法二  类型引用：你使用 Type 类型来引用类的构造函数，在声明、装饰器或其他需要构造函数引用地方使用。
const myServiceConstructor: Type<MyService> = MyService;
```

### SetMetadata

Nest 提供了通过 `@SetMetadata()` 装饰器将定制元数据附加到路由处理程序的能力。例子

```ts
//用法一：
@Post()
@SetMetadata('roles', ['admin'])    //这行
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
//用法二：
export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
@Roles('admin')
async create(@Body() createCatDto: CreateCatDto) {
  this.catsService.create(createCatDto);
}
```

### Injectable

装饰类，表示服务，具体使用不必多说

### Controller

装饰类，表示控制器，具体使用不必多说
```ts
@Controller()
export class ControllerClass {
    @Get()
    async getData(){
        return //.....
    }
}
```

### 请求方式与接受方式

#### `@Body()` 、`@Param()`、`@Query()`等
  
```ts
//....
async postData(@Body() data: DataDto){
    //做一些数据
    return //返回一些数据
}
```


#### `@Get()`、`@Post` 等

```ts
@Get()
async getData(){
    return //....
}
```


### 管道

#### ParseUUIDPipe

解析字符串并验证是否为UUID

```ts
@Get(':uuid')
async findOne(@Param('uuid'), new ParseUUIDPipe() uuid: string) {
    return this.catsService.findOne(uuid);
}
```

#### ValidationPipe

主要用于自动执行输入验证和转换。它可以与装饰器 `@Body()`、`@Query()`、`@Param()` 等共同使用，对传入的数据进行校验和转换，并将无效或不符合预期的数据抛出异常。

当应用程序接收到输入数据时，ValidationPipe 会根据预定义的验证规则对数据进行验证，若不满足验证规则，就会抛出错误信息的异常。

用法需要与DTO搭配使用，如：
```ts
import { Controller, Post, Body, ValidationPipe } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  @Post()
  createUser(@Body(ValidationPipe) createUserDto: CreateUserDto) {
    // 处理创建用户的逻辑
  }
}
```
同时，`ValidationPipe` 也会接受选项来决定验证，常用的选项有：
+ `transform`：布尔值，指定是否启用数据转换，默认为 true，为true时，会把传入的数据自动转换成目标DTO的类型。
+ `disableErrorMessages`：布尔值，指定是否禁用错误消息的生成和显示，默认为 false。
+ `exceptionFactory`：一个自定义的异常工厂函数，用于创建验证失败时抛出的异常。该函数接收一个 ValidationError 数组作为参数，返回处理后的异常实例。
+ `skipMissingProperties`：布尔值，指定是否跳过缺失的属性验证，默认为 false。当设置为 true 时，将不会对缺失的属性进行验证。
+ `whitelist`：布尔值，指定是否禁止非白名单属性的传递，默认为 false。当设置为 true 时，如果传入了未在白名单中声明的属性，则会抛出异常。
+ `groups`：字符串数组，指定要使用的验证组列表。只有与指定的验证组相关联的验证器才会执行。
+ 其他

如：
```ts
import { Controller, Post, Body, ValidationPipe } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users')
export class UsersController {
  @Post()
  createUser(
    @Body(
      new ValidationPipe({
          transform: true,
          whitelist: true,
          forbidNonWhitelisted: true,
          forbidUnknownValues: true,
          validationError: { target: false },
      }),
    ) createUserDto: CreateUserDto
  ) {
    // 处理创建用户的逻辑
  }
}

```

### 序列化

#### SerializeOptions

装饰类，序列化选项，是基于 class-transformer 库提供的装饰器，根据接收的配置对象，来决定如何去序列化类或属性。如：
```ts
//序列化类
@SerializeOptions({
  excludePrefixes: ['_'],   //表示忽略开头带 “_” 的属性
})
export class User {
  id: number;
  name: string;
  _secretField: string;
}

//序列化属性
export class User {
  id: number;
  @SerializeOptions({ name: 'username' })  //序列化后，原本叫 name 的属性，变成了username
  name: string;
}
```
SerializeOptions的常用选项：
  
+ `exclude`: 布尔值，指定是否排除属性，默认为 false。
+ `excludeIfEmpty`: 布尔值，指定如果属性为空，则是否排除属性，默认为 false。
+ `excludePrefixes`: 字符串数组，指定要排除的属性前缀。
+ `groups`: 字符串或字符串数组，指定属性所属的分组。可以根据分组来选择或排除特定的属性。
+ `sinceVersio`n: 数字，指定自从哪个版本开始包含属性。
+ `toClassOnly`: 布尔值，指示在反序列化时是否应用此选项，将属性从来源对象转换为类实例。
+ `name`: 字符串，定属性在序列化时的字段名称。
+ `transform`: 函数，定义自定义的属性转换逻辑。函数接受原始值作为参数，并返回转换后的值。
+ 等其他的选项

#### ClassSerializerInterceptor

基础类，是内置拦截器类，该拦截器用于在响应过程中自动将对象进行序列化，以指定的类的形式返回响应数据。通常用于转换对象为 JSON 或其他格式的字符串输出。

可在全局或特定的路由处理程序上使用该类，如：
```ts
@Controller('user')
@UseInterceptors(ClassSerializerInterceptor)
export class UsersController {
    @Get()
    getAllUsers(): User[] {
        //....
    }
}
```


如果创建的类想要执行一些序列化的操作，可以继承 `ClassSerializerInterceptor` 类，如下面想要创建一个全局拦截器：
```ts
//app.interceptor.ts
export class AppIntercepter extends ClassSerializerInterceptor {
    serialize(){}
}
```

### 拦截器

#### NestInterceptor

自定义拦截器的接口，只要是想自己根据要求创建一个拦截器，就必须要实现该接口。

NestInterceptor 接口主要定义了一个 `intercept` 方法，该方法在请求被路由处理程序执行前后被触发。它接收两个参数：`context`（当前的上下文对象）和 `next`（下一个中间件或处理程序）。

接口的定义：
```ts
export interface NestInterceptor<T = any, R = any> {
  intercept(context: ExecutionContext, next: CallHandler<T>): Observable<R>;
}
```
+ `context` 参数是当前请求的执行上下文对象，其中包含有关当前请求的信息，例如请求、响应、路由处理程序等。
+ `next` 参数是一个表示链中下一个中间件或处理程序的 CallHandler 对象。通过在 intercept 方法中调用 `next.handle()` 可以继续该链并将请求传递给下一个处理程序。
+ `Observable<R>` 是一个可观察对象，它会返回最终处理结果。

例子：
```ts
// app.interceptor.ts
import { CallHandler, ExecutionContext, Injectable, NestInterceptor } from '@nestjs/common';

@Injectable()
export class AppInterceptor implements NestInterceptor {

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().
      pipe(map(data => ({ data }))); // map操作符与Array.prototype.map类似
  }
}
```

### UseInterceptors
用于设置拦截器，如：

```ts
import { AppIntercepter } from './app.interceptor';

@UseInterceptors(AppIntercepter)
@Controller('posts')
export class PostController {
    //。。。。。
}
```


### PlainLiteralObject

`PlainLiteralObject` 是一个类型别名，表示具有字符串键和任意值的普通 JavaScript 对象，类似于 `{ [key: string]: any }表示一个没有特定结构或约束的对象。如下：

```ts
import { Injectable, PlainLiteralObject } from '@nestjs/common';

@Injectable()
export class MyService {
  private config: PlainLiteralObject;

  constructor() {
    this.config = {
      key1: 'value1',
      key2: 'value2'
    }
  }
}
```


### StreamableFile

流式文件类，是一个持有要返回的流的类。你可以传入一个Buffer或者Stream到StreamableFile类的构造函数来创建一个新的StreamableFile实例。如：
```ts
import { Controller, Get, StreamableFile } from '@nestjs/common';
import { createReadStream } from 'fs';
import { join } from 'path';

@Controller('file')
export class FileController {
  @Get()
  getFile(): StreamableFile {
    const file = createReadStream(join(process.cwd(), 'package.json'));
    return new StreamableFile(file);
  }
}
```

### ForbiddenException

自定义异常类，表示客户端请求被服务拒绝或禁止执行特定操作的情况，例如当应用程序需要向客户端发送 403 响应时，可以使用该类，通常发生在用户没有足够的权限或者身份验证失败的情况，如：
```ts
import { Controller, Get, ForbiddenException } from '@nestjs/common';

@Controller('users')
export class UsersController {
    @Get(':id')
    getUser(@Param('id') id: string): User{
        const user = getUserById(id);

        if (!user) {
          throw new NotFoundException(`User with ID ${id} not found`);
        }
        if (!user.isAdmin) {
          throw new ForbiddenException('You do not have permission to access this resource');
        }
        return user;
    }
}
```


### ValidatorOptions

所有可用的`class-validator`选项，都继承自 `ValidatorOptions`，是用于配置验证器行为的选项对象。它是由底层的 class-validator 库提供支持，用于验证数据的有效性。

例如：
```ts
//文件一
export const DtoValidation = (
    options?: ValidatorOptions,
) => {
    //。。做一些事情
};
//文件二

@DtoValidation({ groups: ['create'] })
export class CreateCategoryDto {
    // ...
}
```


### ClassTransformOptions

ClassTransformOptions 是用于配置Class-Transformer行为的选项对象。

以下是一些常见的 ClassTransformOptions 属性：

+ `excludeExtraneousValues`（可选）: 设置为 true，表示排除目标对象中未定义的属性。当将源对象转换为目标对象时，只有来自源对象的属性才会被包含在目标对象中。
+ `ignoreDecorators`（可）: 设置为 true，表示忽略基于装饰器的元数据配置。默认情况下，Class Transformer 会根据装饰器进行对象的转换，通过设置该选项为 true，可以禁用装饰器的影响。
+ `enableImplicitConversion`（可选）: 设置为 true，表示启用隐式转换。例如，在转换过程中，如果目标属性类型与源属性类型不匹配，Class Transformer 将尝试执行适当的隐式类型转换。

```ts
import { Controller, Post, Body } from '@nestjs/common';
import { CreateUserDto } from './dto/create-user.dto';
import { plainToClass } from 'class-transformer';

@Controller('users')
export class UsersController {
  @Post()
  createUser(@Body() createUserDto: CreateUserDto) {
    // 创建 ClassOptions 对象来配置转换选项
    const transformOptions = {
      excludeExtraneousValues: true,
    };

    // plainToClass 方法进行转换，传入相应的 ClassTransformOptions
    const user = plainToClass(CreateUserDto, createUserDto, transformOptions);

    // 在此处执行创建用户的逻辑
    console.log(user);
  }
}
```

### Paramtype

枚举类型，用于表示声明类型，即，路由参数里的类型，都有
```ts
export declare type Paramtype = 'body' | 'query' | 'param' | 'custom';
```



### ArgumentMetadata

用于表示元数据的类型，一般用于自定义管道中的参数，如下：
```ts
import { PipeTransform, Injectable, ArgumentMetadata, BadRequestException } from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToInstance } from 'class-transformer';

@Injectable()
export class ValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) { //这行
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }
    const object = plainToInstance(metatype, value);
    const errors = await validate(object);
    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }
    return value;
  }
}
```
### Catch

用在异常过滤器的修饰符，用在的那个类，就必须实现 `ExceptionFilter ` 接口
```ts
import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
import { Request, Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();
    const status = exception.getStatus();

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Date().toISOString(),
        path: request.url,
      });
  }
}
```

### ArgumentsHost

用于异常过滤器里的`catch` 方法第二个参数的类型，如上

### HttpException

Nest的HttpException是NestJs提供的基础异常类，扩展自Error对象。例子：
```ts
import { HttpException, HttpStatus } from '@nestjs/common';
import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface';
@Injectable()
export class CatsService {
  private readonly cats: Cat[] = [];
  create(cat: Cat) {
    this.cats.push(cat);
  }
  findAll(): Cat[] {
    if (this.cats.length === 0) {
      throw new HttpException('No cats found', HttpStatus.NOT_FOUND);
    }
    return this.cats;
  }
}
```


### HttpStatus

HttpStatus是NestJs提供的一个枚举，它包含HTTP状态代码的常量，如下：
+ CONTINUE
+ SWITCHING_PROTOCOLS
+ PROCESSING
+ EARLY_HINTS
+ OK
+ CREATED
+ ACCEPTED
+ NON_AUTHORITATIVE_INFORMATION
+ NO_CONTENT
+ RESET_CONTENT
+ PARTIAL_CONTENT
+ MULTI_STATUS
+ 等


## `@nestjs/swagger`
+ PartialType