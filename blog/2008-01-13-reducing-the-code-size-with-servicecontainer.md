---
title: Сокращаем код с помощью ServiceContainer и C# 3.0
tags: [servicecontainer, c#]
src: http://mindberg.blogspot.ru/2008/01/iserviceprovider-servicecontainer-c-30.html
---
# Сокращаем код с помощью ServiceContainer и C# 3.0
Например, есть класс BaseClass, в котором отображается список каких-то элементов. Элементы можно выделять, редактировать их лейблы, удалять и т.д. Всю логику можно реализовать в отдельных сервисах. Выделение элементов определим в сервисе – ISelectionService, редактирование – в ILabelEditor, и т.п.

Чтобы воспользоваться этими сервисами в контроле надо определить свойства, например:
```c#
public class MyClass : BaseClass
{
  public ISelectionService SelectionService
  {
    get
    {
      // сервис создается по-запросу, потому что он может вообще не пригодиться
      if(_SelectionService == null)
        _SelectionService = new MyControlSelectionService(this);
      return _SelectionService;
    }
  }
  private ISelectionService _SelectionService;
  public ILabelEditor LabelEditor
  {
    get
    {
      // сервис создается по-запросу, потому что он может вообще не пригодиться
      if(_LabelEditor == null)
        _ LabelEditor = new MyControlLabelEditor(this);
      return _LabelEditor;
    }
  }
  private ILabelEditor _LabelEditor;
  // пример использования сервисов
  public void KeyDown(KeyDownEventArgs e)
  {
    switch(e.KeyCode)
    {
      case Keys.F2:
       this.LabelEditor.BeginEdit(this.SelectedItem);
       break;
      case Keys.F5:
        this.SelectionService.SelectAll();
        break;
    }
  }
}
```
У такой реализации есть минусы: 
<ol>
  <li>много кода</li>
  <li>при использовании сервисов надо знать имена свойств для доступа к сервисам.</li>
</ol>
Попытаемся избавиться минуса №2. Для этого воспользуемся интерфейсом IServiceProvider:
```c#
public class MyClass : BaseClass, IServiceProvider
{
  // если BaseClass наследует Container, то можно использовать виртуальный метод GetService
  object IServiceProvider.GetService(Type serviceType)
  {
    If(serviceType == typeof(ISelectionService))
      return this. SelectionService;
    if(serviceType == typeof(ILabelEditor))
      return this.LabelEditor;
    return null;
  }
  public ISelectionService SelectionService
  {
    …
  }
  public ILabelEditor LabelEditor
  {
    …
  }
}
```
Минус №1 стал еще больше, т.е. количество кода увеличилось.
Чтобы от него избавиться воспользуемся ServiceContainer’ом, синтаксисом C# 3.0 и Extension Methods’ами:
```c#
using System;
using System.ComponentModel.Design;
public class BaseClass { /*...*/ }
public interface ISelectionService { /*...*/ }
public interface ILabelEditor { /*...*/ }
public class SelectionService : ISelectionService
{
  public MyClass Owner { get; internal set; }
}
public class LabelEditor : ILabelEditor
{
  public MyClass Owner { get; internal set; }
}
public class MyClass : BaseClass, IServiceProvider
{
  public MyClass()
  {
    // создаем контейнер для сервисов
    this.Services = new ServiceContainer();
    // добавляем в контейнер типы сервисов; экземпляры сервисы будут созданы один раз по-запросу.
    this.Services.Add<ISelectionService>((container, type) => new SelectionService() { Owner = this });
    this.Services.Add<ILabelEditor>((container, type) => new LabelEditor() { Owner = this });
  }
    protected ServiceContainer Services { get; private set; }
    object IServiceProvider.GetService(Type serviceType)
    {
      return this.Services.GetService(serviceType);
    }
}
public static class ServiceModelHelpers
{
  public static void Add<T>(this IServiceContainer sc, ServiceCreatorCallback cb)
  {
    sc.AddService(typeof(T), cb);
  }
  public static T Get<T>(this IServiceProvider sp)
  {
    return (T)sp.GetService(typeof(T));
  }
}
```
В результате, код MyClass сократился значительно.
