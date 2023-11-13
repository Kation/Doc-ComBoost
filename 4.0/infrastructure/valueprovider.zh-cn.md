# 值提供器
领域执行过程中，值提供器会提供领域上下文相关的数据，比如领域方法的参数。
除了方法的参数值外，各种不同的值提供器还有各自的数据来源。

## 获取值
使用`GetValue`方法从值提供器获取值，传入名称、值类型，返回`object`对象。

```csharp
IValueProvider valueProvider = ...;
object value = valueProvider.GetValue("name", typeof(string));
```

还有泛型扩展方法可传入并获取指定类型。

```csharp
IValueProvider valueProvider = ...;
string value = valueProvider.GetValue<string>("name");
```

也有扩展方法`GetRequiredValue`，不存在值时将抛出异常。

## 获取键
`Keys`属性包含了值提供器的键的数据，通过`IValueKeyCollection`接口，可以判断是否存在键、是否存在前缀、获取包含前缀的键。

是否存在键。

```csharp
IValueProvider valueProvider = ...;
if (valueProvider.Keys.ContainsKey("name"))
{

}
```

是否存在前缀。

```csharp
IValueProvider valueProvider = ...;
if (valueProvider.Keys.ContainsPrefix("query", '.'))
{

}
```

获取包含前缀的键。

```csharp
IValueProvider valueProvider = ...;
var keys = valueProvider.Keys.GetKeysFromPrefix("query", '.'));
```

## 是否包含键
`ContainsKey`方法可以判断值提供器里是否存在某个键。

```csharp
IValueProvider valueProvider = ...;
if (valueProvider.ContainsKey("name"))
{

}
```

## 设置值
`SetValue`方法可以手动为值提供器设置键与值。

```csharp
IValueProvider valueProvider = ...;
valueProvider.SetValue("name", "value");
```

## 设置别名
`SetAlias`可以为键设置别名，当使用`GetValue`方法获取某个键的值时，如果值不存在而键又设置为了别名，那么会使用对应的键去获取值。

当值不存在时。
```csharp
IValueProvider valueProvider = ...;
valueProvider.SetValue("key", "value");
var value = valueProvider.GetValue<string>("name");//value = null
```

设置别名后再获取。
```csharp
IValueProvider valueProvider = ...;
valueProvider.SetValue("key", "value");
valueProvider.SetAlias("key", "name");
var value = valueProvider.GetValue<string>("name");//value = "value"
```