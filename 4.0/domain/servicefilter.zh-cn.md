# 领域服务过滤器
领域服务过滤器是一种委托。
每个过滤器可以选择是否执行下一个委托，在执行前后可获取执行上下文并执行其它代码。

## 过滤器实现
领域服务过滤器是一个类，必须实现`IDomainServiceFilter`接口。
过滤器可以是一个特性，`ComBoost`提供了`DomainServiceFilterAttribute`基类以直接继承并编写过滤器。

```csharp
public class YourFilter : DomainServiceFilterAttribute
{
    public override async Task OnExecutionAsync(IDomainExecutionContext context, DomainExecutionPipeline next)
    {
        //执行前代码
        await next();
        //执行后代码
    }
}
```

## 使用过滤器
领域服务过滤器有多种使用方法，生效范围各不相同。

### 全局领域服务过滤器
在[配置`ComBoost`服务](/integration/generic#添加全局领域服务过滤器)时添加。

```csharp
builder.Services.AddComBoost()
    .AddGlobalFilter<YourFilter>();
```

### 为特定领域服务使用过滤器
在[配置`ComBoost`领域服务](/integration/generic#为领域服务设置过滤器)时添加。

```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>()
            .UseFilter<YourFilter>();
    });
```

或在领域服务类添加过滤器特性。
```csharp
[YourFilter]
public class YourService : DomainService
{

}
```

### 为特定领域服务的特定方法使用过滤器
在[配置`ComBoost`领域服务](/integration/generic#为领域服务设置过滤器)时添加。
```csharp
builder.Services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<YourService>()
            .UseFilter<YourFilter>("Method1", "Method2", ...);
    });
```

或在领域服务的方法添加过滤器特性。
```csharp
public class YourService : DomainService
{
    [YourFilter]
    public Task Method1()
    {
        //...
    }
    
    [YourFilter]
    public Task Method2()
    {
        //...
    }
}
```

### 为当前上下文添加过滤器
通过依赖注入获取领域服务模板时，通过`IDomainTemplate.Context`接口属性获取领域服务上下文。
将过滤器添加入上下文内的`Filters`属性即可。

```csharp
YourServiceTemplate service = ...;
service.Context.Filters.Add(new YourFilter());
```

## 过滤器顺序
领域服务过滤器是有执行顺序的，执行顺序如下：

1. 领域上下文添加的过滤器
2. 配置的领域服务方法过滤器
3. 领域服务方法的过滤器特性
4. 配置的领域服务过滤器
5. 领域服务的过滤器特性
6. 配置的全局过滤器

:::info
###### :information_source:备注
同一级别下，后添加的过滤器先执行。
:::