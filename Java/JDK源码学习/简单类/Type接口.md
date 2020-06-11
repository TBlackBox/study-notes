# 接口源码
```
public interface Type {
    default String getTypeName() {
        return toString();
    }
}
```

# 说明

Type 是 Java 编程语言中所有类型的公共高级接口。它们包括原始类型、参数化类型、数组类型、类型变量和基本类型。`getGenericParameterTypes` 与 `getParameterTypes` 都是获取构成函数的参数类型，前者返回的是Type类型，后者返回的是Class类型，由于Type顶级接口，Class也实现了该接口，因此Class类是Type的子类，Type 表示的全部类型而每个Class对象表示一个具体类型的实例，如`String.class`仅代表String类型。由此看来Type与 Class 表示类型几乎是相同的，只不过 Type表示的范围比Class要广得多而已。当然Type还有其他子类，如：

- TypeVariable：表示类型参数，可以有上界，比如：T extends Number
- ParameterizedType：表示参数化的类型，有原始类型和具体的类型参数，比如：`List`
- WildcardType：表示通配符类型，比如：?, ? extends Number, ? super Integer