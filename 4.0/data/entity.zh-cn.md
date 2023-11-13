# 实体
`ComBoost`提供了一套基础工具帮助开发者快速搭建以及使用实体相关的功能。

项目需要添加Nuget包[`Wodsoft.ComBoost.Data.Core`](https://www.nuget.org/packages/Wodsoft.ComBoost.Data.Core)。
```ps
Install-Package Wodsoft.ComBoost.Data.Core
```

## 实体定义
`实体`是领域当中常用的对象，其相当于领域的聚合根。

### 实体接口
`ComBoost`定义了`IEntity`接口用于实体逻辑。

```csharp
namespace Wodsoft.ComBoost.Data.Entity
{
    public interface IEntity
    {
        void OnCreating();
        void OnCreateCompleted();
        void OnEditing();
        void OnEditCompleted();
        bool IsRemoveAllowed { get; }
        bool IsEditAllowed { get; }
    }
}
``` 

以及带有主键定义的`IEntity<TKey>`泛型接口。

```csharp
namespace Wodsoft.ComBoost.Data.Entity
{
    public interface IEntity<TKey> : IEntity
    {
        TKey Id { get; set; }
    }
}
```

### 实体基类
`ComBoost`提供了实现了这些接口的实体基础类。
`EntityBase`继承了`EntityDTOBase`基类，实现了`IEntity`接口。
并在`OnCreateCompleted`方法内为`CreationDate`、`ModificationDate`赋值当前时间，
在`OnEditCompleted`方法内为`ModificationDate`赋值当前时间。

`EntityBase<TKey>`泛型类继承了`EntityBase`基类，实现了`IEntity<TKey>`泛型接口，增加了一个名称为`Id`，类型为`TKey`泛型参数的属性。

### 用户基类
`ComBoost`还提供了一个`UserBase`类型，继承了`EntityBase`基类。
提供了`void SetPassword(string password)`设置密码与`bool VerifyPassword(string password)`校验密码方法。

## 实体仓储
为了解耦领域服务与数据库提供器，`ComBoost`抽象了一套实体仓储类型。

### 实体查询上下文
`IEntityQueryContext<T>`是一个泛型接口，泛型参数`T`需实现`IEntity`接口。

```csharp
public interface IEntityQueryContext<T>
    where T : IEntity
{
    IDatabaseContext Database { get; }
    IEntityMetadata Metadata { get; }
    IQueryable<T> Query();
    Task<T> GetAsync(params object[] keys);
    IQueryable<TChildren> QueryChildren<TChildren>(T item, Expression<Func<T, ICollection<TChildren>>> childrenSelector)
        where TChildren : class;
    Task LoadPropertyAsync<TProperty>(T item, Expression<Func<T, TProperty?>> propertySelector)
        where TProperty : class;
}
```

`Database`属性可以获取数据库上下文。

`Metadata`属性可以获取实体元数据。

`Query`方法返回实体查询体。

`GetAsync`方法传入主键以查询实体。

`QueryChildren`方法传入实体、子项选择器以返回子项查询体。

`LoadPropertyAsync`方法传入实体、导航属性选择器可以查询导航属性。

:::warning
###### :warning:警告
`IEntityQueryContext<T>`接口没有配置依赖注入，不能通过依赖注入的形式获取。
:::

### 实体上下文
`IEntityContext<T>`是一个泛型接口，接口实现了`IEntityQueryContext<T>`，泛型参数`T`为类并实现`IEntity`接口。

```csharp
public interface IEntityContext<T> : IEntityQueryContext<T>
    where T : class, IEntity
{
    void Add(T item);
    void AddRange(IEnumerable<T> items);
    T Create();
    void Update(T item);
    void UpdateRange(IEnumerable<T> items);
    void Remove(T item);
    void RemoveRange(IEnumerable<T> items);
}
```

`Add`方法传入实体以在后续执行插入数据库操作。

`AddRange`方法传入批量实体以在后续执行插入数据库操作。

`Create`方法创建一个实体，并执行`IEntity.OnCreating`方法，返回返回该实体。实体需要执行`Add`或`AddRange`操作才会插入数据库。

`Update`方法传入实体以在后续执行更新数据库操作。

`UpdateRange`方法传入批量实体以在后续执行插入数据库操作。

`Remove`方法传入实体以在后续执行删除数据库操作。

`RemoveRange`方法传入批量实体以在后续执行删除数据库操作。

:::info
###### :information_source:备注
`IEntityContext<TEntity>`可以通过`IServiceProvider`接口或通过构造函数、方法参数的依赖注入方式获取。
:::

### 数据库上下文
`IDatabaseContext`是一个接口，内部包含了数据库相关操作。
```csharp
public interface IDatabaseContext
{
    IEnumerable<Type> SupportTypes { get; }
    IEntityContext<T> GetContext<T>() where T : class, IEntity, new();
    Task<int> SaveAsync();
    IDatabaseTransaction CreateTransaction();
}
```

`SupportTypes`返回当前数据库上下文支持的实体类型。

`GetContext`方法传入泛型类以获取该实体上下文。

`SaveAsync`方法对未保存的数据进行操作。

`CreateTransaction`方法将创建一个事务。

:::warning
###### :warning:警告
在同一个应用当中可以同时存在多个数据库等，`IDatabaseContext`并不是唯一的，不能通过依赖注入的形式获取。  
仅能通过获取`IEntityContext<T>`后的`Database`属性拿到该实体的数据库上下文。
:::

## 查询体扩展方法
`ComBoost`为查询体提供了一些扩展方法以方便开发者使用。

:::info
###### :information_source:备注
扩展方法位于`Wodsoft.ComBoost.Data.Linq.QueryableExtensions`静态类，需要引用命名空间`Wodsoft.ComBoost.Data.Linq`。
:::

:::warning
###### :warning:警告
扩展方法需要实体上下文支持，实体上下文可能不会提供完全支持。
:::

### 异步查询
同EF与EF Core一样，提供以异步方法执行Linq查询操作。

`AllAsync`

`AnyAsync`

`AverageAsync`

`CountAsync`

`FirstAsync`

`FirstOrDefaultAsync`

`LastAsync`

`LastOrDefaultAsync`

`MaxAsync`

`MinAsync`

`SingleAsync`

`SingleOrDefaultAsync`

`SumAsync`

`ToArrayAsync`

`ToDictionaryAsync`

`ToListAsync`

### 删除与更新

`DeleteAsync`

`UpdateAsync`

### 其它扩展方法

`Include`

`ThenInclude`

`AsTracking`

`AsNoTracking`

## 配置实体上下文
`ComBoost`提供了一些ORM库的实体上下文实现，开发者可快速使用这些ORM库。

### EntityFramework

项目需要添加Nuget包[`Wodsoft.ComBoost.EntityFramework`](https://www.nuget.org/packages/Wodsoft.ComBoost.EntityFramework)。
```ps
Install-Package Wodsoft.ComBoost.EntityFramework
```

使用[`Entity Framework`](https://learn.microsoft.com/zh-cn/ef/ef6/)时，按照正常的步骤去创建`DbContext`以及注入`DbContext`实例。
然后使用`AddEFContext`扩展方法传入`DbContext`泛型类型以配置实体上下文。

```csharp
services.AddScoped<YourDbContext>(sp => new YourDbContext("数据库连接串"));
services.AddEFContext<YourDbContext>();
```

之后就能通过依赖注入获取该`DbContext`内的实体的实体上下文。

:::info
###### :information_source:备注
`DbContext`内必须包含有实体的`DbSet<TEntity>`属性，才能通过依赖注入获取实体上下文。
:::

:::warning
###### :warning:警告
Entity Framework不支持`DeleteAsync`与`UpdateAsync`扩展方法。
:::

### EntityFrameworkCore

项目需要添加Nuget包[`Wodsoft.ComBoost.EntityFrameworkCore`](https://www.nuget.org/packages/Wodsoft.ComBoost.EntityFrameworkCore)。
```ps
Install-Package Wodsoft.ComBoost.EntityFrameworkCore
```

使用[`Entity Framework Core`](https://learn.microsoft.com/zh-cn/ef/core/)时，按照正常的步骤去创建`DbContext`以及注入`DbContext`实例。
然后使用`AddEFCoreContext`扩展方法传入`DbContext`泛型类型以配置实体上下文。

```csharp
services.AddDbContext<YourDbContext>(options => ...);
services.AddEFCoreContext<YourDbContext>();
```