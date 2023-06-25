# TypeORM 内置类总结

## TypeORM内置

### ObjectLiteral

`ObjectLiteral`是指一个普通的JavaScript对象，用于表示数据库中的一行数据或查询结果的一行。用于表示数据库中的一行数据或查询结果的一行。它是一个简单的JavaScript对象，其中的属性对应于数据库表的列名，属性的值对应于数据库中的实际数据。

内部的接口如下：
```ts
export interface ObjectLiteral {
    [key: string]: any;
}
```

### SelectQueryBuilder

`SelectQueryBuilder`是一个用于构建和执行查询的类。它是TypeORM的一部分，用于与数据库进行交互。

如下用法：
```ts
import { getRepository, SelectQueryBuilder } from 'typeorm';
import { User } from './user';

async function getUsersOverAge(age: number): Promise<User[]> {
  const userRepository = getRepository(User);   //获取与User实体相关联的存储库。
  const queryBuilder: SelectQueryBuilder<User> = userRepository.createQueryBuilder('user');
  const users = await queryBuilder
    .where('user.age > :age', { age })  // 添加查询条件
    .orderBy('user.name', 'ASC')       // 按姓名升序排序
    .getMany();                        // 执行查询并获取结果集
  return users;
}
```


### TreeRepositoryUtils

`TreeRepositoryUtils` 是工具类，可以在没有使用`TreeRepository`的情况下处理树状结构数据。`Repository`而不是`TreeRepository`，或者只需要在某些场景下处理树状结构数据，那么可以使用`TreeRepositoryUtils`来帮助处理树形数据。
如：
```ts
import { getTreeRepository, TreeRepositoryUtils } from 'typeorm';

// 定义实体类
@Entity()
@Tree("closure-table")
class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeParent()
  parent: Category;

  @TreeChildren({ cascade: true })
  children: Category[];
}

// 创建树形仓库
const categoryRepository = getTreeRepository(Category);

// 创建根节点
const rootCategory = new Category();
rootCategory.name = "Root Category";

// 保存根节点到数据库
await categoryRepository.save(rootCategory);

// 创建子节点
const childCategory = new Category();
childCategory.name = "Child Category";
childCategory.parent = rootCategory;

// 保存子节点到数据库
await categoryRepository.save(childCategory);

// 获取根节点的所有子节点
const children = await TreeRepositoryUtils.getChildren(categoryRepository, rootCategory);
console.log(children);

// 获取子节点的所有父节点
const ancestors = await TreeRepositoryUtils.getAncestors(categoryRepository, childCategory);
console.log(ancestors);
```

### FindOptionsUtils

该类 是 TypeORM 中的一个实用工具类，用于构建和处理查询选项（FindOptions），该类有如下方法：

+ applyOptions    (queryBuilder: SelectQueryBuilder<Entity>, options: FindManyOptions<Entity>): SelectQueryBuilder<Entity>：将查询选项应用到查询构建器上，构建出符合选项条件的查询。
+ applyPagination (queryBuilder: SelectQueryBuilder<Entity>, options: FindManyOptions<Entity>): SelectQueryBuilder<Entity>：将分页选项应用到查询构建器上，构建出分页查询。
+ applyRelations  (queryBuilder: SelectQueryBuilder<Entity>, options: FindManyOptions<Entity>): SelectQueryBuilder<Entity>：将关联选项应用到查询构建器上，构建出包含关联实体的查询。
+ applyConditions (queryBuilder: SelectQueryBuilder<Entity>, options: FindManyOptions<Entity>): SelectQueryBuilder<Entity>：将条件选项应用到查询构建器上，构建出符合条件的查询。
+ applySorting    (queryBuilder: SelectQueryBuilder<Entity>, options: FindManyOptions<Entity>): SelectQueryBuilder<Entity>：将排序选项应用到查询构建器上，构建出按指定字段排序的查询。

例子：
```ts
import { getRepository, FindManyOptions, FindOptionsUtils } from 'typeorm';

// 定义实体类
@Entity()
class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  age: number;

  @Column()
  email: string;
}

//创建仓库
const userRepository = getRepository(User);
//构建查询选项
const options: FindManyOptions<User> = {
  where: {
    age: MoreThan(18),
    email: Like('%@example.com')
  },
  order: {
    name: 'ASC',
  },
  take: 10,
  skip: 0
}
// 创建查询构建器
const queryBuilder = userRepository.createQueryBuilder();
// 应用查询选项到查询构建器
FindOptionsUtils.applyOptions(queryBuilder, options);
// 执行查询
const users = await queryBuilder.getMany();
// 输出查询结果
console.log(users);
```

### DataSource

该类是一个数据库连接的配置和管理对象。它负责与数据库建立连接，并提供执行 SQL 查询和操作数据库的功能。
如下：
```ts
{
  "type": "mysql",
  "host": "localhost",
  "port":3306,
  "username": "your_username",
  "synchronize": ["src/entities/*.ts"],
}
```

### ObjectType

表示对象的某个类型
```ts
export declare type ObjectType<T> = {
    new (): T;
} | Function;
```
例子使用如下：
```ts
class Person {
  name: string;
  age: number;

  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

const personType: ObjectType<Person> = Person;

const createObject<T> = (ObjectType: ObjectType<T>, ...args: any[]): T => {
  if (ObjectType instanceof Function) {
    return new ObjectType(...args) as T;
  } else {
    throw new Error('ObjectType must be a constructor function');
  }
}
const person: Person = createObject(personType, '小明', 30);
console.log(person instanceof Person); // 输出: true
console.log(person.name); // 输出: John Doe
console.log(person.age); // 输出: 30
```

### BaseEntity

该类用于定义实体类，包含了一些常用的实体方法和属性，可以作为其他实体类的基类继承使用。
如下使用：
```ts
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  age: number;
}
// 使用BaseEntity的方法进行数据库操作
const user = new User();
user.name = 'John Doe';
user.email = 'john@example.com';
await user.save(); // 保存实体到数据库

user.name = 'John Smith';
await user.save(); // 更新实体

await user.remove(); // 从数据库中删除实体
```
通过继承 `BaseEntity`， `User` 类获得了一些内置的实体方法和属性，例如 `.save()`、`.remove()`、`.findOne()` 等。


### Entity

装饰器，实体，表示数据库的表

### CreateDateColumn

装饰器，自动创建时间列

### PrimaryColumn

装饰器，用于标识一个属性作为实体的主键列。

### UpdateDateColumn

装饰器，自动更新该属性的值为每次实体被更新时的当前时间戳


### PrimaryGeneratedColumn

装饰器，主键列，与 `@PrimaryColumn()` 不同的地方在于，该装饰器是自动生成的，但该主键列的类型就必须是 `number` or `bigint`

### Column

装饰器 列

### Repository

该类是一个用于访问数据库的类，它封装了对数据库表的常见操作，如插入、更新、删除和查询。与`BaseEntity` 不一样，虽然都是用一些操作实体的方法，但是`BaseEntity` 不支持扩展新方法，想要扩展，就得使用 `Repository` ，继承该类，新增方法。

```ts
//文件一
import { EntityRepository, Repository } from 'typeorm';
import { User } from './path_to_user_entity';

@EntityRepository(User)
export class UserRepository extends Repository<User> {
  // 自定义方法：按照姓名查询用户
  async findByFullName(firstName: string, lastName: string): Promise<User | undefined> {
    return this.findOne({ firstName, lastName });
  }
  // 自定义方法：按照邮箱查询用户
 Email(email: string): Promise<User | undefined> {
    return this.findOne({ email });
  }
}
//文件二
const userRepository = new UserRepository();
const newUser = new User();
newUser.firstName = 'John';
newUser.lastName = 'Doe';
newUser.email = 'john@example.com';
await userRepository.save(newUser);
console.log('User created:', newUser);
```
### EntityRepository

`EntityRepository`是一个装饰器（Decorator），用于定义自定义的实体（Entity）`Repository`。需要搭配`Repository`类使用。


### EventSubscriber、EntitySubscriberInterface

`EventSubscriber` 用于监听和处理数据库操作事件的装饰器，通过创建自定义的 `EventSubscriber` 类，并使用相关的装饰器进行标记特定的数据库操作发生时执行自定义的逻辑。该类支持的方法有：
+ `listenTo`：用于指定`EventSubscriber`类要监听哪个实体，即，返回的实体类，就是要监听的实体。
+ `Insert`(entity: T): 在实体插入到数据库之前调用该方法。 
+ `afterInsert`(entity: T): 在实体插入到数据库之后调用该方法。
+ `beforeUpdate`(entity: T): 在实体更新到数据库之前调用该方法。
+ `afterUpdate`(entity: T): 在实体更新到数据库之后调用该方法。
+ `beforeRemove`(entity: T)：在实体从数据库删除之前调用该方法。
+ `afterRemove`(entity: T): 在实体从数据库删除之后调用该方法。

而`EntitySubscriberInterface` 是`@EventSubscriber()`装饰的类要实现的接口。

例子：
```ts
mport { EventSubscriber, EntitySubscriberInterface, InsertEvent, UpdateEvent, RemoveEvent } from 'typeorm';
import User } from './path_user_entity';

@EventSubscriber()
export class UserSubscriber implements EntitySubscriberInterface<User> {
  listenTo() {
    return User;
  }

  beforeInsert(event: Insert<User>) {
    console.log('Before user insert:', event.entity);
    // 在此处执行自定义逻
  }

  afterInsert(event: InsertEvent<User>) {
    console.log('After user insert:', event.entity);
    // 在此处执行自定义逻辑
  }
}
```

### ManyToMany

该类是装饰器，即多对多，用于在实体类之间建立多对多关系。，该关系其中A包含B的多个实例，而B包含A的多个实例。类似的，还有`@OneToOne`、`@ManyToOne`、`@OneToMany`

```ts
//实体一
@Entity()
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToMany(() => Tag)
  @JoinTable()
  tags: Tag[];
}

//实体二
@Entity()
export class Tag {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Article, article => article.tags)
  articles: Article[];
}
```

### Tree、TreeChildren、TreeParent

该类为装饰器，用于处理树结构的装饰器。

+ `@Tree()` 装饰器是用于将实体类标记为可层级的树状结构。它可以放置在实体类的顶部，并指定一些选项来定义树状结构的行为。
+ `@TreeChildren()` 装饰器是用于表示实体类中的子节点属性。它指示一个实体拥有多个子节点。
+ `@TreeParent()` 装饰器是用于表示实体类中的父节点属性。它指示一个实体具有一个父节点。

例子：

```ts
import { Entity, PrimaryGeneratedColumn, Column, Tree, TreeChildren, TreeParent} from 'typeorm';

@Entity()
@Tree("materialized-path")
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeChildren()
  children: Category[];

  @TreeParent()
  parent: Category;
}
```


### TreeRepository

`TreeRepository` 是用于处理树状结构数据的特殊Repository。它扩展自Type中的Repository类，并提供了一些针对树状数据操作的额外方法和功能。


```ts
@Entity()
@Tree("materialized-path")
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeChildren()
  children: Category[];

  @TreeParent()
  parent: Category;
}

async function example() {
  const connection = await createConnection();

  // 使用TreeRepository
  //getTreeRepository 用于获取树状结构特定的 Repository 实例的方法。传入要处理的实体类作为参数，返回一个具有树状结构特性的 Repository 对象
  const categoryRepository = getTreeRepository(Category);
  const treeCategories = await categoryRepository.findTrees();
  console.log('Tree categories:', treeCategories);
}
```


### FindTreeOptions

FindTreeOptions 是一个用于查询树状结构数据的选项对象。它用于在查询数据库时指定树状结构的加载方式和过滤条件。常用选项如下：
+ `relations?: string[]`：一个字符串数组，用于指定需要加载的关联关系。这允许你通过关联的方式加载相关的实体。
+ `where?: string | Brackets | ((qb: SelectQueryBuilder<Entity>) => string) | ObjectLiteral`：一个用于筛选结果的查询条件。你可以使用字符串、括号、函数或对象字面量来指定条件。
+ `order?: { [P in keyof Entity]?: 'ASC' | 'DESC' }`：一个用于指定结果排序顺序的对象字面量。你可以指定实体的属性和排序方式（升序或降序）。
+ `skip?: number`：一个可选的数字，用于指定要跳过的查询结果数量。
+ `take?: number`：一个可选的数字，用于指定要获取的查询结果数量。
+ `cache?: boolean | number | { id: any, milliseconds: number }`：一个可选的布尔值、数字或缓存配置对象，用于指定是否启用缓存以及缓存的配置选项。
+ `withDeleted?: boolean`：一个可选的布尔值，用于指定是否包含已被软删除的实体。

```ts
import { getRepository } from 'typeorm';
import { Category } from './category.emtity';

const categoryRepository = getRepository(Category);

const options: FindTreeOptions<Category> = {
  relations: ['children'], // 加载子类别关联
  order: { id: 'ASC' }, // 按照 ID 升序排序
};

const categories = await categoryRepository.findTrees(options);

console.log(categories);
```

### EntityNotFoundError

错误类型，用于表示在执行数据库操作时找不到指定的实体。

```ts
import { getRepository, EntityNotFoundError } from 'typeorm';

try {
  const userRepository = getRepository(User);
  const user = await userRepository.findOneOrFail(1); // 尝试查找 ID 为 1 的用户
  console.log(user);
} catch (error) {
  if (error instanceof EntityNotFoundError) {
    console.log('找不到指定的用户');
  } else {
    console.log('发生其他错误:', error);
  }
}
```

### EntityPropertyNotFoundError

错误类型，用于表示在实体映射中找不到指定的属性。

```ts
import { getRepository, EntityPropertyNotFoundError } from 'typeorm';

try {
  const userRepository = getRepository(User);
  const user = new User();
  user.invalidProperty = 'value'; // 无效的属性名称
  await userRepository.save(user);
} catch (error) {
  if (error instanceof EntityPropertyNotFoundError) {
    console.log('指定的属性在实体映射中未定义');
  } else {
    console.log('发生其他错误:', error);
  }
}
```


### QueryFailedError

错误类型，用于表示数据库查询失败的错误。

```ts
import { getRepository, QueryFailedError } from 'typeorm';

try {
  const userRepository = getRepository(User);
  await userRepository.save({}); // 保存无效的用户对象
} catch (error) {
  if (error instanceof QueryFailedError) {
    console.log('数据库查询失败:', error.message);
    console.log('数据库错误代码:', error.code);
    console.log('底层驱动程序错误:', error.driverError);
  } else {
    console.log('发生其他错误:', error);
  }
}
```


### EntityNotFoundError

错误类型，用于表示无法找到指定实体的错误。

```ts
import { getRepository, EntityNotFoundError } from 'typeorm';

try {
  const userRepository = getRepository(User);
  const user = await userRepository.findOneOrFail({ id: 1 }); // 查询 ID 为 1 的用户
  console.log('找到用户:', user);
} catch (error) {
  if (error instanceof EntityNotFoundError) {
    console.log('找不到指定的用户');
  } else {
    console.log('发生其他错误:', error);
  }
}
```

### In
In 操作符来查询具有指定 ID 的用户。In 接受一个数组参数，其中包含要匹配的值。在这种情况下，我们使用 ids 数组作为参数，以查找 ID 在该数组中的用户。
```ts
import { getRepository } from 'typeorm';

const userRepository = getRepository(User);

const ids = [1, 2, 3, 4, 5];

const users = await userRepository.find({
  where: {
    id: In(ids),
  },
});
```

### IsNull

用于指定一个字段的值为 null 的查询条件

```ts
import { getRepository } from 'typeorm';

const userRepository = getRepository(User);

const users = await userRepository.find({
  where: {
    email: IsNull(),
  },
});
```

### Not

表示不等于

```ts
import { getRepository, Not } from 'typeorm';

const userRepository = getRepository(User);

const users = await userRepository.find({
  where: {
    age: Not(25), // 不等于 25 岁的用户
  },
});
```


## 结合Nest —— `@nestjs/typeorm`

### TypeOrmModule

用于在 NestJS 应用程序中集成 TypeORM 的核心模块之一。它提供了一些静态方法和配置选项，用于配置和管理 TypeORM 在 NestJS 中的使用。可以配置 TypeORM 的连接选项、实体、数据库迁移等。

常见用法：
```ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    //forRoot 表示用来配置连接数据库选项
    TypeOrmModule.forRoot({
      type: 'mysql',
      host: 'localhost',
      port: 3306,
      username: 'root',
      password: 'password',
      database: 'mydatabase',
      entities: [User],
      synchronize: true,
    }),
  ],
})
export class AppModule {}
```

目前静态方法有如下：
+ forRoot(options: TypeOrmModuleOptions): DynamicModule: 配置 TypeORM 的根模块连接选项。
+ forFeature(entities: EntityClassOrSchema[]): DynamicModule: 导入指定的实体类或仓库。
+ forRootAsync(options: TypeOrmModuleAsyncOptions): DynamicModule: 异步配置 TypeORM 的根模块连接选项。可以使用自定义提供程序（TypeOrmModuleAsyncOptions）来获取连接选项。
+ forFeatureAsync(options: TypeOrmModuleAsyncOptions): DynamicModule: 异步导入指定的实体类或仓库。可以使用自定义提供程序（TypeOrmModuleAsyncOptions）来获取实体类或仓库。


### TypeOrmModuleOptions

用于配置 TypeORM 连接选项的接口。它包含了一系列属性，用于指定数据库连接和其他相关选项。
如下：

+ type：指定数据库类型，如 'mysql'、'postgres'、'sqlite'、'mssql' 等。
+ host：指定数据库主机名。
+ port：指定数据库端口号。
+ username：指定连接数据库的用户名。
+ password：指定连接数据库的密码。
+ database：指定要连接的数据库名称。
+ entities：指定实体类的数组，用于映射数据库表。
+ synchronize：一个布尔值，表示是否自动创建数据库表结构。在开发环境中可能会设置为 true，但在生产环境中建议设为 false。
+ migrations：指定数据库迁移的路径和选项，用于管理数据库结构变更。
+ subscribers：指定数据库事件订阅者的数组，用于监听和处理数据库事件。
+ cli：用于配置数据库命令行工具的选项，如数据库迁移命令行工具。

例子：
```ts
const options: TypeOrmModuleOptions = {
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  username: 'root',
  password: 'password',
  database: 'mydatabase',
  entities: [User, Product],
  synchronize: true,
};
```

### getDataSourceToken

是一个函数，用于获取数据源的令牌（token），为数据源创建一个唯一的标识符，以便在模块中注册和访问数据源。

