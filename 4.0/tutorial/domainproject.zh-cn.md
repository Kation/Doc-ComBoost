# 领域项目
领域项目一般实现对应业务领域的服务代码、实体对象、数据对象等。

## 项目结构
```
Project.ServiceA.Domain
├─Entities
│  ├─SomethingEntity.cs
│  └─...
├─Models
│  ├─SomethingModel.cs
│  └─...
├─Services
│  ├─SomethingService.cs
│  └─...
├─DataContext.cs
└─ServiceAModule.cs
```

> 同领域项目命名空间通常保持一致
> ```
> <PropertyGroup>
>   <RootNamespace>Project.ServiceA</RootNamespace>
> </PropertyGroup>
> ```

### Entities文件夹
如果使用ORM或实体仓储进行数据库操作，一般建立`Entities`文件夹并将实体类放入此文件夹。

### Models文件夹
一般情况下，领域服务的输入输出对象不会是实体对象，而是使用数据对象（DTO、VO、VM等）作为输入输出对象。将他们存入`Models`文件夹。

### Services文件夹
存放领域服务、领域模板接口、领域服务事件、一般服务接口、一般服务实现。

### 其它
使用EF Core时，建立`DataContext`继承`DbContext`用作数据库上下文。

建立`ServiceAModule`继承`DomainModule`，方便承载项目（Web、单元测试等）快速引入此领域。