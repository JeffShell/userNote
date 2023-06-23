# TS 内置类总结

## ClassDecorator 

ClassDecorator 是 TypeScript 中的一种装饰器（Decorator）类型，用于修饰类的声明。

如：
```TS
export const CustomRepository = <T>(entity: ObjectType<T>): ClassDecorator =>
    SetMetadata(CUSTOM_REPOSITORY_METADATA, entity);
```

在上面的方法中，`ClassDecorator` 的意思是 返回一个修饰符函数。