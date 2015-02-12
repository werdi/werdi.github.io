---
title: Проброс свойств через интерфейс
tags: [c#, interfaces]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a40d115a-47dd-42fd-91b9-61587ee2c75a/-?forum=fordesktopru
---
# Проброс свойств через интерфейс
*> 2-й вариант: использовать рефлексию и передавать коллекцию с ссылками на свойства. Не умею в Reflection.*

- [C# 4 and the dynamic keyword](http://www.hanselman.com/blog/C4AndTheDynamicKeywordWhirlwindTourAroundNET4AndVisualStudio2010Beta1.aspx)
- [Using Type dynamic (C# Programming Guide)](http://msdn.microsoft.com/en-us/library/dd264736.aspx)
*> 1)работа со свойствами через некий универсальный проводник - предполагается коллекция 2)свойств может быть любое количество 3)свойства могут иметь различные типы, в том числе и созданные мною*

проект под WinForms? [PropertyGrid](http://msdn.microsoft.com/ru-ru/library/system.windows.forms.propertygrid.aspx) не подходит? см. [PropertyGrid FAQ](http://www.rsdn.ru/article/dotnet/PropertyGridFAQ.xml)

*> Посмотрел, но до конца не понял, как же это приспособить? Я ограничен интерфейсом.*

например, есть интерфейс и наследники
```
interface IData { string Data { get; } }
class A : IData
{
  public string DataA { get { return "A"; } }
  string IData.Data { get { return "-"; } }
}
class B : IData
{
  public string DataB { get { return "B"; } }
  string IData.Data { get { return "-"; } }
}
```
есть метод, который возвращает IData  
```   
IData GetData()
{
 	return new B();
}
```
значение IData.Data читаем как обычно 
```
var d = GetData();
string dd = d.Data;
```
а значение B.DataB можно получить через рефлекцию
```
string db = (string)d.GetType().GetProperty("DataB").GetValue(d, null);
``` 
или с помощью dynamic: 
```
string db = ((dynamic)d).DataB;
```
*> Плюс, в вашем примере только чтение, а также нужны запись и уведомление*

если добавить setter, то значение можно именить так:
```
((dynamic)d).DataB = "123";
```
для уведомлений надо добавить реализацию INotifyPropertyChanged
примерно так
```c#
class B : IData, INotifyPropertyChanged
{
  string IData.Data { get { return "-"; } }
  public string DataB
  {
    get { return "B"; }
    set
    {
      // ... 
      this.PropertyChanged(this, new PropertyChangedEventArgs("DataB"));
    }
  }
  public event PropertyChangedEventHandler PropertyChanged = delegate {};
}
```