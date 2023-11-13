# 领域上下文
领域上下文是用于领域服务执行的关键对象，领域上下文为领域服务提供了依赖注入、过滤器、事件管理器、值提供器、取消令牌等。

每次获取领域服务模板时，`IDomainTemplate.DomainContext`属性都将是从`IDomainContextProvider`获取的全新的领域上下文。

## 服务提供器
领域上下文实现了`IServiceProvider`，一般是创建了作用域的`IServiceProvider`。
可以通过领域上下文获取其它服务对象。

```csharp
IDomainContext context = ...;
var someService = context.GetRequiredService<SomeService>();
```

## 过滤器
可以为领域上下文单独设置过滤器，这样设置的过滤器仅对使用该领域上下文的领域服务生效。

```csharp
IDomainContext context = ...;
context.Filters.Add(new YourFilter());
```

## 事件管理器
可以为领域上下文单独设置领域事件处理器，这样设置的事件处理器仅对使用该领域上下文的领域服务生效。

```csharp
IDomainContext context = ...;
context.EventManager.AddEventHandler<YourEventArgs>((context, e) => Task.Completed);
```

## 数据字典
领域上下文包含一个`dynamic`类型的属性`DataBag`，可以为其设置索引或属性。
一般用于设置标识后供过滤器或事件处理器使用。

```csharp
IDomainContext context = ...;
context.DataBag.Somethings = true;
```

## 值提供器
领域执行过程中，值提供器会提供领域上下文相关的数据，比如领域方法的参数。
除了方法的参数值外，各种不同的值提供器还有各自的数据来源。

可以通过领域上下文拿到值提供器。

```csharp
IDomainContext context = ...;
context.ValueProvider.SetValue("value", true);
```

## 用户主体声明
`User`属性可以获取当前上下文的用户主题声明。

```csharp
IDomainContext context = ...;
ClaimsPrincipal user = context.User;
```