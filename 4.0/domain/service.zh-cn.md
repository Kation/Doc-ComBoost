# 领域服务
简单一点理解，`领域服务`就是一个服务，领域只是人为定义的一个特殊的服务的泛称。

## 领域服务接口
`领域服务`必须实现接口`IDomainService`。
```csharp
public interface IDomainService
{
    IDomainExecutionContext? Context { get; }

    void Initialize(IDomainExecutionContext context);
}
```
包含一个获取`IDomainExecutionContext`的属性，一个初始化领域服务的方法。

## 基类
`ComBoost`默认提供了一个实现了`IDomainService`的基类`DomainService`，
通常编写`领域服务`时都会继承这个基类。
```csharp
public class DomainService : IDomainService
{
}
```

## 构造函数
`领域服务`的构造函数参数可以为任意内容，参数来源均为依赖注入。

## 领域服务方法
`领域服务`的方法是实现业务逻辑的关键所在，但方法是有所限制的：
- 不能有**重载**方法
- 方法返回类型必须为`Task`或`Task<>`

当调用出现违反以上限制的`领域服务`时会抛出异常。

## 方法参数
类似于`MVC`控制器方法参数，`领域服务`方法参数同样可以添加特性以标记其来源。
当没有特性时，默认使用`值来源`。
`ComBoost`提供了以下来源特性：
- `FromServiceAttribute`，通过依赖注入获取参数值
- `FromValueAttribute`，通过值提供器获取参数值
- `FromEntityAttribute`，通过值提供器获取值并从实体上下文获取对应实体（位于包`Wodsoft.ComBoost.Data.Core`）

您也可以使用自己编写的继承自`FromAttribute`基类的来源特性。

## 生命周期
`领域服务`的内部实现是基于依赖注入的，其生命周期为`暂时`。

## 示例
```csharp
public class GreeterService : DomainService
{
    public Task Hello([FromService] ILogger<GreeterService> logger, [FromValue] string text)
    {
        logger.LogInformation($"Client say {text}");
    }
}
```

## 注册领域服务
在配置依赖注入时，注入`ComBoost`方法后通过调用`AddLocalService`方法添加本地服务，然后调用`AddService`添加指定的`领域服务`。
```csharp
services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddService<GreeterService>();
    });
```
也可以调用`AddServiceFromAssembly`方法添加指定程序集内的所有`领域服务`。
```csharp
services.AddComBoost()
    .AddLocalService(builder =>
    {
        builder.AddServiceFromAssembly(typeof(GreeterService).Assembly);
    });
```

## 自动生成领域服务模板
`ComBoost`提供了源生成器用于生成所需的领域服务模板，开发者只需要简单添加一些特性即可使用。

### AutoTemplateAttribute
将`AutoTemplateAttribute`特性添加于`领域服务`类型之上，`ComBoost`会自动生成领域服务模板。  
默认会使用`I+领域服务类名`作为领域服务模板接口名，命名空间与`领域服务`相同。

:::warning
###### :warning:警告
`领域服务`类型需加上`partial`关键字以生成部分类并添加`DomainTemplateImplementerAttribute`特性。
:::

* 特性的`Namespace`属性指定生成的`领域服务模板`的命名空间。
* 特性的`TemplateName`属性指定生成的`领域服务模板`接口名。
* 特性的`Group`属性指定自动生成内容的组。

### AutoTemplateMethodAttribute
将`AutoTemplateMethodAttribute`特性添加于`领域服务`的领域服务方法之上，用于帮助`AutoTemplateAttribute`确定方法是否包含或排除于自动生成的领域服务模板内。

* 特性的`IsExcluded`属性指定方法排除于自动生成的领域服务模板。
* 特性的`IsIncluded`属性指定方法包含于自动生成的领域服务模板。
* 特性的`Group`属性指定对应的`AutoTemplateAttribute`内容。

:::warning
###### :warning:警告
`IsIncluded`优先于`IsExcluded`，任意方法指定了`IsIncluded`为`true`时，自动生成的领域服务模板仅包含指定`IsIncluded`为`true`的方法。
:::