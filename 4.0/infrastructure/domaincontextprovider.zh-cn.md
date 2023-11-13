# 领域上下文提供器
领域上下文提供器用于`ComBoost`在获取领域服务模板时，创建领域上下文。

在调用`AddComBoost`配置`ComBoost`框架时，会注入一个`IDomainContextProvider`接口，实现类型为`CompositeDomainContextProvider`。

## 领域上下文提供器接口
领域上下文提供器需实现`IDomainContextProvider`接口。

### 是否可提供领域上下文
`IDomainContextProvider.CanProvide`属性返回当前领域上下文提供器是否可以获取领域上下文。
如可以获取返回true，否则返回false。

### 获取领域上下文
`IDomainContextProvider.GetContext`方法返回一个全新的领域上下文，如无法获取会抛出`NotSupportedException`异常。

## 复合型领域上下文提供器
`CompositeDomainContextProvider`内可以包含多个领域上下文提供器，通过后续的`ComBoost`进行配置添加。
如使用`AddAspNetCore`会添加`HttpDomainContextProvider`领域上下文提供器。

### 手动添加领域上下文提供器
也可以通过`TryAddContextProvider`扩展方法，手动添加领域上下文提供器。

```csharp
builder.Services.AddComBoost()
    .TryAddContextProvider<YourDomainContextProvider>(500);
```

### 领域上下文提供器顺序
添加领域上下文提供器时，需要传递顺序参数。

复合型领域上下文提供器获取领域上下文时，会按照顺序值从小到大，依次判断领域上下文提供器是否可提供领域上下文。
如果可以提供，则获取领域上下文后返回，否则继续用下一个领域上下文提供器进行判断。

:::warning
###### :warning:警告
如果所有领域上下文提供器都无法提供领域上下文，则会抛出`NotSupportedException`异常。
:::

## 常用的领域上下文提供器
列举在`ComBoost`当中常用的领域上下文提供器。

### HttpDomainContextProvider
使用`AddAspNetCore`配置ASP.NET Core时添加，顺序值为500。

### MvcDomainContextProvider
使用`AddMvc`配置ASP.NET Core Mvc时添加，顺序值为200。

### EmptyDomainContextProvider
使用`AddEmptyContextProvider`配置空上下文提供器时添加，顺序值为1000。