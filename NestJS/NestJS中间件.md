# NestJS中间件

所谓的中间件其实就是在路由处理前就会调用的函数，类似 VueRouter 的导航守位。
可以做的事情，可以执行代码逻辑，并且对响应和请求进行更改，同时，当前的中间件在没有执行结束的情况下，想要进行下一个中间件，就必须调用 `next()` 将控制权传递给下一个中间件。

## 创建中间件
首先需要引用 `NestMiddleware`，
```ts
import {NestMiddleware} from "@nestjs/common"
import { Request, Response, NextFunction } from 'express';

//下面这行注册中间件，完成后就是一个中间件了
export class Logger implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction){
        next()  //执行下一个中间件
    }
}
```

## 结合依赖注入，使用中间件

中间件是完全支持依赖注入的，但是需要特别注意的是，中间件是不可以在`@Module()` 列出，只能使用模块类的 `configure()` 方法设置中间件，也因此，包含中间件的模块就必须实现 `NestModule` 接口

用法如下：
```ts
// logger.middleware.ts
@Injectable()
export class Logger implements NestMiddleware {
    use(req: Request, res: Response, next: NextFunction){
        next()  
    }
}
```
模块引用中间件
```ts
// app.module.ts
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './logger.middleware';
import { DemoModule } from "./demo.module"
@Module({
    imports: [DemoModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        
        consumer.apply(LoggerMiddleware).forRoutes("demo"); 
    }
}
```
而且，`consumer.apply` 可以接受多个中间件：
```ts
consumer.apply(
    LoggerMiddleware, 
    LoggerMiddleware2, 
    LoggerMiddleware3, 
    //....
).forRoutes("demo"); 
```

上面的`forRoutes()`方法表示 中间件应用在了 `demo` 路由上。该方法还可以指定HTTP方法，甚至是路由通配符：

```ts
//例如
import { Module,NestModule,MiddlewareConsumer,RequestMethod } from '@nestjs/common';
@Module({
    imports: [DemoModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        
        consumer.apply(LoggerMiddleware).forRoutes({
            path: "demo",
            method: RequestMethod.GET   //指定为 GET请求方式
        }); 
    }
}
```

也可以把 Controller 做为 `forRoutes` 方法参数传进去:

```ts
import { Module,NestModule,MiddlewareConsumer } from '@nestjs/common';
import {DemoController} from "./demo.controller"
@Module({
    imports: [DemoModule],
})
export class AppModule implements NestModule {
    configure(consumer: MiddlewareConsumer) {
        consumer.apply(LoggerMiddleware).forRoutes(DemoController); 
    }
}
```

## 函数式写法

除了上述用的是class写法，还可以使用函数：
```ts
const middleAll = (req, res, next) => {
    next();
}

async const bootstrap = ()=> {
    const app = await NestFactory.create(AppModule);
    app.use(middleAll);
    await app.listen(3000)
}
```

## 全局中间件

全局的中间件只能使用函数式方式，例子参照函数式写法
