---
title: Реализация паттернов IoC, DI и Interception
tags: [patterns, c#]
src: http://cognitex.blogspot.ru/2009/05/ioc-di-interception.html
---
# Реализация паттернов IoC, DI и Interception
Паттерны: Inversion of Control (IoC), Dependency Injection (DI) и Interception реализованы в [Unity] (http://unity.codeplex.com/) (= Unity Application Block; есть исходники).

Перед компиляцией для папки Tests надо выполнить Unload Projects in Solution Folders (см. контекстное меню папки).

Для использования Unity в cсвоем проекте надо к подключить две сборки: Microsoft.Practices.ObjectBuilder2 и Microsoft.Practices.Unity.

Небольшой пример:
```c#
using System.Windows.Forms;
using Microsoft.Practices.Unity;

public interface ITest
{
}

public class Test : ITest
{
    	public Test(string value)
    	{
    	}

    	public void Call(int value)
    	{
    	}

    	public string Value
    	{
        	get { return _Value; }
        	set
        	{
            	_Value = value;
        	}
    	}
    	private string _Value;
}

public partial class Form1 : Form
{
    	public Form1()
    	{
        	// создать контейнер
        	IUnityContainer myContainer = new UnityContainer();

        	/// при вызове myContainer.Resolve<ITest>():
        	///   - вызван конструктор, который получит "1"
        	///   - в свойство Value пападет "3"
        	///   - в метод Call - 2;
        	myContainer.RegisterType<ITest, Test>(
            	new InjectionConstructor("1"),
            	new InjectionMethod("Call", 2),
            	new InjectionProperty("Value", "3")
            	);

        	var tst = myContainer.Resolve<ITest>();
    	}
}
```
Интересно, что информацию о типе, создаваемого объекта, а также значения передаваемые в его методы, можно определить в config-файле, а не в методе RegisterType, как это приведено в примере.

Надо заменить, что делать аналогичную инициализацию объекта немного избыточно. Намного проще реализовать метод, например, так:
```c#
myContainer.RegisterType<ITest>(() =>
{
    	var ret = new Test("1") { Value = "3" };
    	ret.Call(2);
    	return ret;
});
```
То есть в момент вызова метода myContainer.Resolve будет вызвана фабрика объекта (анонимный метод).

Подробнее о паттернах см. [здесь] (http://ru.wikipedia.org/wiki/%D0%92%D0%BD%D0%B5%D0%B4%D1%80%D0%B5%D0%BD%D0%B8%D0%B5_%D0%B7%D0%B0%D0%B2%D0%B8%D1%81%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D0%B8).
