# 领域执行上下文
每次通过领域服务模板执行领域服务的方式时，都会生成一个新的领域执行上下文对象。
对象包含了当前执行领域服务的相关数据。

开发者在使用过滤器、领域事件时，可以通过领域执行上下文获取相关数据。

## 领域上下文
领域执行上下文一定存在对应的[领域上下文](/infrastructure/domaincontext)。

通过`DomainContext`属性获取领域上下文，返回`IDomainContext`接口。

## 领域服务
领域执行上下文可能包含一个[领域服务](/domain/service)对象。

通过`DomainService`属性获取领域服务，返回`IDomainService`接口，可能返回空。

## 领域服务方法
领域执行上下文可能包含执行的方法反射信息。

通过`DomainMethod`属性获取方法信息，返回`MethodInfo`对象，可能返回空。

## 方法参数值
领域执行上下文可能包含执行的方法的参数值。

通过`ParameterValues`属性获取参数值数组，返回`object[]`数组对象，可能返回空。

## 执行结果
领域执行上下文可以获取当前执行方法返回的结果对象。

通过`Result`属性获取结果对象，可能返回空。

:::info
###### :information_source:备注
过滤器在执行下一个委托前，由于实际的领域服务方法还没有执行，所以执行结果为空。

但可能存在其它过滤器、事件处理器设置了结果，使得执行结果有值。
:::

## 是否中止
领域服务方法的执行是异步操作，可以通过`IsAborted`属性判断外部调用是否进行了取消操作。

## 是否完成
可以通过`IsCompleted`属性判断执行是否完成。

## 结束执行
在过滤器当中，可以在执行下一个委托前，通过使用`Done`方法，结束后续执行，并设置为已执行完成。

```csharp
public class HelloInterruptFilter : IDomainServiceFilter
{
    public Task OnExecutionAsync(IDomainExecutionContext context, DomainExecutionPipeline next)
    {
        context.Done();
        return Task.CompletedTask;
    }
}
```

如果领域服务方法有返回值，也可以通过调用`Done`方法是传递参数，将参数作为返回结果。

```csharp
public class HelloInterruptFilter : IDomainServiceFilter
{
    public Task OnExecutionAsync(IDomainExecutionContext context, DomainExecutionPipeline next)
    {
        context.Done("result");
        return Task.CompletedTask;
    }
}
```

:::warning
###### :warning:警告
如果设置的结果与领域服务方法返回结果类型不一致，则会抛出`InvalidCastException`异常。
:::