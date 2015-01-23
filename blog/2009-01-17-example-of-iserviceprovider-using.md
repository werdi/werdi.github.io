---
title: Пример использования IServiceProvider
tags: [iserviceprovider, c#]
src: http://mindberg.blogspot.ru/2008/01/iserviceprovider.html
---
# Пример использования IServiceProvider
Задача: надо передать текст из объекта №1 в объект №2 (определен в сборке №2 , а объект №1 в сборке №1). При передаче данных объект №1 указывает тип текста, например:
```c#
public enum TextType
{
  // для обработки можно использовать IHandler1 или IHandler2
  ShortText,
  // для обработки можно использовать IHandler3 или IHandler4
  LongText,
  // для обработки можно использовать IHandler5 или IHandler6
  SecretText,
}
```
А также объект №1 передает заголовок текста и должен предоставить объекту №2 доступ к соответствующим обработчикам IHandler*, которые объект №2 может использовать как ему вздумается или может вообще не использовать. При этом все IHandler* достаточно «тяжелые» и (главное) у них нет ничего общего!

Кривое решение: создаем Data Transfer Object (DTO):
```c#
public class DTO
{
  // ссылка на Object1 необходима, потому что именно Object1 знает где взять реализацию IHandler*
  internal DTO(Object1 owner)
  {
    _Owner = owner;
  }
  private Object1 _Owner;
  public string Title { get; internal set; }
  public DataType DataType { get; internal set; }
  // создается по-запросу
  public IHandler1 Handler1
  {
    get
    {
      if(_Handler1 != null)
        _Handler1 = _Owner.CreateHandler(typeof(IHandler1));
      return _Handler1;
    }
  }
  private IHandler1 _Handler1;
  /*
   * далее идут определения остальных IHandler*
   */
}
```
Минусы: 1) DTO «привязан» к Object1, т.е. при запросе одного из IHandler* надо обязательно обращаться к Object1;  2) много кода с определениями свойств.

Решение: создаем Data Transfer Object (DTO) и реализуем в нем IServiceProvider:
```c#
public class DTO : IServiceProvider
{
    // доступен для объекта №1, но недоступен для объекта №2
    internal ServiceContainer Services { get; set; }
    // доступен для объекта №2 через IServiceProvider
    object IServiceProvider.GetService(Type serviceType)
    {
        return (this.Services != null) ? this.Services.GetService(serviceType) : null;
    }
    public string Title { get; internal set; }
    public DataType DataType { get; internal set; }
}
```
Кода стало намного меньше и нет привязки к Object1. Т.е. мы полностью избавились от минусов кривого решения.
Пример использования:
```c#
DTO dto = new DTO() { DataType = DataType.ShortText, Services = new ServiceContainer() };
switch(dto.DataType)
{
  case DataType.ShortText:
    dto.Services.Add<IHandler1>((c, t) =>Factory1.CreateHandler());
    dto.Services.Add< IHandler2>((c, t) =>Factory2.CreateHandler());
    break;
    ...
}
```
Вопрос: откуда у DTO.Services появился метод Add<T>? 

Ответ: это Extension Method, см. [ServiceModelHelpers] (http://mindberg.blogspot.com/2008/01/iserviceprovider-servicecontainer-c-30.html).
