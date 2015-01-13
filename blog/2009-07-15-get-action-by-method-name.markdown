# Как получить Action по имени метода
Есть класс, в котором определен метод Test, например:
```c#
public class MyClass
{
  public void Test() 
  {
  }
}
```
Чтобы получить Action на основе имени метода Test, надо выполнить следующий код:
```c#
var mc = new MyClass();
var a = mc.CreateAction("Test");
a(); // будет вызван метод Test
```
Весь "секрет" находится в методах расширения:
```c#
public static class MetodInfoExtension
{
  public static Action CreateAction(this object instance, string methodName)
  {
    return instance.GetType().GetMethod(methodName).CreateAction(instance);
  }

  public static Action CreateAction(this MethodInfo methodInfo, object target)
  {
    var call = System.Linq.Expressions.Expression.Call(System.Linq.Expressions.Expression.Constant(target), methodInfo);
    var lambda = System.Linq.Expressions.Expression.Lambda<Action>(call);
    var ret = lambda.Compile();
    return ret;
  }
}
```
В остальных случаях можно создать Action с помощью следующих строк:
```c#
Expression<Action> exa = () => Test();
Action a = exa.Compile();
```
