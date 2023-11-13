# 集成测试
为了方便测试系统，
`ComBoost`提供了Nuget包[`Wodsoft.ComBoost.Mock`](https://www.nuget.org/packages/Wodsoft.ComBoost.Mock)供开发者在单元测试项目内进行集成测试。
```ps
Install-Package Wodsoft.ComBoost.Mock
```

## 使用通用主机
由于`ComBoost`基于依赖注入功能，推荐在单元测试内定义一个通用主机作为配置基础。  
参考[通用主机](/integration/generic)，在单元测试类里，创建一个主机字段于获取主机的属性。
在此处配置调用`AddMock`方法以添加模拟环境的配置。
```csharp
public class TestClass
{
        private IHost _host;
        protected IHost Host
        {
            get
            {
                if (_host == null)
                {
                    _host = Host.CreateDefaultBuilder()
                            //可选：增加json配置文件
                            .ConfigureAppConfiguration(builder =>
                            {
                                builder.AddJsonFile("appsettings.json", true);
                            })
                            .ConfigureServices((context, services) =>
                            {
                                services.AddComBoost()
                                    .AddLocalService(builder =>
                                    {
                                        builder.AddService<YourService>();
                                    })
                                    .AddMock();
                                //其它服务配置
                            })
                            .Build();
                }
                return _host;
            }
        }
}
```

## 使用内存分布式事件
在单元测试里，可以使用`ComBoost`提供的基于内存的分布式事件来进行分布式事件处理。
```csharp
public class TestClass
{
        private IHost _host;
        protected IHost Host
        {
            get
            {
                if (_host == null)
                {
                    _host = Host.CreateDefaultBuilder()
                            //可选：增加json配置文件
                            .ConfigureAppConfiguration(builder =>
                            {
                                builder.AddJsonFile("appsettings.json", true);
                            })
                            .ConfigureServices((context, services) =>
                            {
                                services.AddComBoost()
                                    .AddLocalService(builder =>
                                    {
                                        builder.AddService<YourService>();
                                    })
                                    .AddDistributed(builder =>
                                    {
                                        //使用基于内存的分布式事件
                                        builder.UseInMemory();
                                    })
                                    .AddMock();
                                //其它服务配置
                            })
                            .Build();
                }
                return _host;
            }
        }
}
```

### 设置分布式事件实例
`UseInMemory`扩展方法可以传递一个object对象快速设置基于内存的分布式事件实例的Key，
只有拥有相同的Key才能互相进行事件发布与处理。
```csharp
public class TestClass
{
        private static object _eventKey1 = new object();
        private IHost _host;
        protected IHost Host
        {
            get
            {
                if (_host == null)
                {
                    _host = Host.CreateDefaultBuilder()
                            //可选：增加json配置文件
                            .ConfigureAppConfiguration(builder =>
                            {
                                builder.AddJsonFile("appsettings.json", true);
                            })
                            .ConfigureServices((context, services) =>
                            {
                                services.AddComBoost()
                                    .AddLocalService(builder =>
                                    {
                                        builder.AddService<YourService>();
                                    })
                                    .AddDistributed(builder =>
                                    {
                                        //使用基于内存的分布式事件
                                        builder.UseInMemory(_eventKey1);
                                    })
                                    .AddMock();
                                //其它服务配置
                            })
                            .Build();
                }
                return _host;
            }
        }
}
```

### 设置分布式事件选项
`UseInMemory`扩展方法可以传递一个方法委托`Action<MockInMemoryEventOptions>`对基于内存的分布式事件选项进行设置。

`MockInMemoryEventOptions`对象的属性。

| 属性 | 类型 | 说明 |
| --- | --- | --- |
| InstanceKey | object | 事件实例的Key，默认为空。 |
| ThrowExceptionForMustHandleEventWhenNull | bool | 必须处理的事件没有处理器时抛出异常，默认为false。 |
| IsAsyncEvent | bool | 是否异步处理事件，同步处理时，引发事件后将等待事件处理完成，默认为false。 |

## 单元测试内的分布式调用
分布式系统会有多个服务，每一个服务都对应一个主机。
在单元测试内进行分布式测试，需要为每个分布式服务调用`AddMockService`方法进行配置。

`AddMockService`方法需要传入一个获取主机的方法委托`Func<IHost>`，以及一个配置对应主机行为的方法委托`Action<IComBoostMockServiceBuilder>`。  
通过`IComBoostMockServiceBuilder`的`AddService`方法添加该主机里的任意服务，不限于`IDomainTemplate`领域服务模板。  
也可以使用`AddServiceInAssembly`方法批量添加整个程序集的领域服务模板。  
同gRPC一样，也提供了`AddAuthenticationPassthrough`方法用于传递身份信息。
```csharp
public class TestClass
{
        private static object _eventKey1 = new object();
        private IHost _serviceAHost;//分布式服务A的主机
        private IHost _serviceBHost;//分布式服务B的主机
        protected IHost ServiceAHost
        {
            get
            {
                if (_serviceAHost == null)
                {
                    _serviceAHost = Host.CreateDefaultBuilder()
                            //可选：增加json配置文件
                            .ConfigureAppConfiguration(builder =>
                            {
                                builder.AddJsonFile("appsettings.json", true);
                            })
                            .ConfigureServices((context, services) =>
                            {
                                services.AddComBoost()
                                    .AddLocalService(builder =>
                                    {
                                        builder.AddService<IYourService>();
                                    })
                                    .AddMockService(() => _serviceBHost, builder =>
                                    {                                        
                                        builder
                                            .AddAuthenticationPassthrough()//添加身份透传
                                            .AddService<IServiceB>();//添加另一个分布式服务里的服务
                                            .AddServiceInAssembly(typeof(ServiceB.ISomeService).Assembly);//添加另一个分布式服务程序集里的领域服务模板
                                    })
                                    .AddMock();
                                //其它服务配置
                            })
                            .Build();
                }
                return _serviceAHost;
            }
        }
}
```

## 运行一段程序
`RunAsync`传递一个委托方法`Func<IServiceProvider, Task>`，通过在委托内参数`IServiceProvider`获取服务进行测试。
```csharp
[Fact]
public async Task Test1()
{
    await _host.RunAsync(async sp =>
    {
        var service = sp.GetRequiredService<IYourService>();
        await service.SomeMethod();
    });
}
```
`Run`为同步方法，委托类型为`Action<IServiceProvider>`。

方法委托内部作为一个生命周期，方法委托运行结束后会释放上下文资源。

## 作为领域服务方法运行
`RunAsDomainMethodAsync`传递一个委托方法`Func<IDomainExecutionContext, Task>`，通过在委托内参数`IDomainExecutionContext`获取上下文进行测试。
```csharp
[Fact]
public async Task Test1()
{
    await _host.RunAsDomainMethodAsync(async context =>
    {
        var service = context.DomainContext.GetRequiredService<IYourService>();
        await service.SomeMethod();
        await context.DomainContext.EventManager.RaiseEvent(context, new YourDomainServiceEventArgs());
    });
}
```
`RunAsDomainMethodAsync`有多个重载方法，通过泛型参数获取对应服务作为委托参数。
```csharp
[Fact]
public async Task Test1()
{
    await _host.RunAsDomainMethodAsync<YourService>(async (context, service) =>
    {
        await service.SomeMethod();
    });
}

[Fact]
public async Task Test2()
{
    await _host.RunAsDomainMethodAsync<YourService, OtherService>(async (context, service, otherService) =>
    {
        await service.SomeMethod();
    });
}
```

方法委托内部作为一个生命周期，方法委托运行结束后会释放上下文资源。

## 发布事件
`RaiseEvent`方法可以让开发者快速引发一个领域服务事件。
```csharp
[Fact]
public async Task Test1()
{
    await _host.RaiseEvent<YourService>(new YourDomainServiceEventArgs());
}
```

## 设置身份信息
`SetIdentity`方法允许开发者设置全局身份信息，传递委托方法`Func<ClaimsIdentity>`返回一个`ClaimsIdentity`对象。
```csharp
[Fact]
public async Task Test1()
{
    await _host.RunAsync(async sp =>
    {
        sp.SetIdentity(() => new ClaimsIdentity("Mock", ClaimTypes.Name, ClaimTypes.Role));
        var service = sp.GetRequiredService<IYourService>();
        await service.SomeMethod();
    });
}
```
`SetIdentity`提供了一个快速设置用户Id、用户名、角色的重载方法。
```csharp
[Fact]
public async Task Test1()
{
    await _host.RunAsync(async sp =>
    {
        sp.SetIdentity("userId", "userName", "role1", "role2", ...);
        var service = sp.GetRequiredService<IYourService>();
        await service.SomeMethod();
    });
}
```