---
title: Использование Extension Methods для определения event'ов
tags: [extension methods, events, snippets, xml, c#]
src: http://mindberg.blogspot.ru/2008/02/extension-methods-event.html
---
# Использование Extension Methods для определения event'ов
Например, есть класс TextView, в котором надо реализовать следующий интерфейс:
```c#
public interface IEditorService
{
  event EventHandler<KeyEventArgs> KeyDown;
}
```
Для этого в определение класса TextView добавляем IEditorService, ставим курсор на IEditorService, жмем Alt+Shift+F10 и выбираем Explicitly implement interface ‘IEditorService’. В результате получим следующий полуфабрикат:
```c#
internal class TextView : IEditorService
{
  event EventHandler<KeyEventArgs> IEditorService.KeyDown
  {
    add { throw new NotImplementedException(); }
    remove { throw new NotImplementedException(); }
  }
}
```
Немного подпиливаем, т.е. доводим до рабочего вида реализацию IEditorService.KeyDown:
```c#
event EventHandler<KeyEventArgs> IEditorService.KeyDown
{
  add { _KeyDown = (EventHandler<KeyEventArgs>)Delegate.Combine(_KeyDown, value); }
  remove { _KeyDown = (EventHandler<KeyEventArgs>)Delegate.Remove(_KeyDown, value); }
}
private EventHandler<KeyEventArgs> _KeyDown;
```
Так надо было делать раньше, в мрачные времена, еще до появления C# 3.0 и Visual Studio 2008, когда мир не знал про могучие методы расширения (Extension Methods). Теперь все изменилось и IEditorService.KeyDown можно реализовать, например, следующим образом:
```c#
event EventHandler<KeyEventArgs> IEditorService.KeyDown
{
  add { value.CombineWith(ref  _KeyDown); }
  remove { value.RemoveFrom(ref _KeyDown); }
}
private EventHandler<KeyEventArgs> _KeyDown;
```
Кода меньше, что радует пальцы и мозг. Хотя больше радуется мозг, потому что пальцам можно помочь с помощью Snippet'ов, поддержка которых есть в Visual Studio.

Про снипеты...

В папке «My Documents\Visual Studio 2008\Code Snippets\Visual C#\My Code Snippets» создаем текстовый файл:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<CodeSnippet Format="1.0.0">
  <Header>
    <Title>Event Add Remove</Title>
    <Shortcut>evem</Shortcut>
    <Description></Description>
  <SnippetTypes>
    <SnippetType>Expansion</SnippetType>
  </SnippetTypes>
  </Header>
  <Snippet>
    <Declarations>
      <Literal default="true">
        <ID>eh</ID>
        <ToolTip>handler name</ToolTip>
        <Default>_handler</Default>
      </Literal>
    </Declarations>
    <Code Language="csharp" Format="CData">
      <![CDATA[
       add { value.CombineWith(ref $eh$); }
       remove { value.RemoveFrom(ref $eh$); }
      ]]>
    </Code>
  </Snippet>
</CodeSnippet>
```
Сохраняем файл под именем, например, Event.Add.Remove.snippet (где .snippet – это расширение этого файла).

Теперь возвращаемся в редактор кода. Выделяем блоки:
```c#
add { throw new NotImplementedException(); }
remove { throw new NotImplementedException(); }
```
набираем evem (потому что такое значение указано в теге Shortcut, см. выше), жмем Tab два раза (после первого нажатия закроется окно IntelliSense), набираем _KeyDown, жмем Enter. Код создан, руки свободны J.

P.S.
Определение методов CombineWith и RemoveFrom:
```c#
public static class ExtensionMethods
{
  public static void CombineWith<T>(this EventHandler<T> value, ref EventHandler<T> target) where T : EventArgs
  {
    target = (EventHandler<T>) Delegate.Combine(target, value);
  }
  public static void RemoveFrom<T>(this EventHandler<T> value, ref EventHandler<T> target) where T : EventArgs
  {
    target = (EventHandler<T>) Delegate.Remove(target, value);
  }
}
```
