# DTO是什么？

DTO只是一个类或接口，用于定义数据传输对象，传输的目标往往是数据访问对象从数据库检索数据。换句话说是定义数据是怎么通过网络发送的。

代码例子：
```ts
// user.dto.ts
export class UserDto {
    readonly id: number;
    readonly username: string;
    readonly email: string;
}
```

使用该代码：
```ts
//users.controller.ts
import {UserDto} from "./user.dto"

@Controller('users')
export class UsersController {
    @Get()
    findAll(): UserDto[] {
        return [
            {
                id: 1, username: 'user1', email: 'user1@example.com'
            },
            //.....
        ]
    }
}
```
并且可在服务间传输 数据，从一个服务到另外一个服务：
```ts
// users.service.ts
import {UserDto} from "./user.dto"

@Injectable()
export class UsersService {
    async findById(id: number): Promise<UserDto> {
        //在服务之间传递 UserDto对象
        return {id: 1, name: 'user1', email: 'user1@example.com'};
    }
}
```

## 与TS定义类型有什么不一样的

在看着这用法上，和TS定义类型的方式很相似，都是用于定义数据结构和类型，也让开发人员能够快速的读懂代码。从用法上看，TS定义类型甚至是可以代替DTO的，但主要不同的地方在于，DTO提供了更多的扩展功能，例如：
+ 允许进行数据处理
+ 验证数据

而TS定义类型，重点在于类型检查方面，最重要的原因，由于**TypeScript定义类型会被抹除**，导致Nest不能在运行引用TS， 因此根据需求，来选择定义类型 或 DTO。