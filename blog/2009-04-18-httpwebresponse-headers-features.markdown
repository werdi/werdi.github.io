---
title: Особенность HttpWebResponse.Headers
tags: [feature, httpwebresponse, webheadercollection, c#]
src: http://cognitex.blogspot.ru/2009/04/httpwebresponseheaders.html
---
# Особенность HttpWebResponse.Headers
HTTP-ответ - это обычный текст, в котором первая строка называется строкой состояния. Под ней могут быть указаны заголовки - строки с двоеточием. Оно делит строку на две части: слева - имя заголовка, а справа - его значение.
Тело сообщения отделяется от заголовков пустой строкой. Пример HTTP-ответа:
```
HTTP/1.1 200 OK
Date: Thu, 16 Apr 2009 08:39:02 GMT
Set-Cookie: cname=5; expires=Tue, 13-Apr-2010 23:43:04 GMT; path=/; domain=.getvaluetest.ru
Set-Cookie: cvalue=5; expires=Tue, 13-Apr-2010 23:43:04 GMT; path=/; domain=.getvaluetest.ru
Content-Length: 9
Content-Type: text/html; charset=windows-1251

hello
```
Для доступа к заголовкам используется свойство HttpWebResponse.Headers типа WebHeaderCollection. Так как WebHeaderCollection наследует NameValueCollection, то можно предположить, что поведение HttpWebResponse.Headers будет соответствующим.
```c#
// пример использования NameValueCollection
NameValueCollection col1 = new NameValueCollection();
col1.Add("key1", "value1.1");
col1.Add("key2", "value2");
col1.Add("key1", "value1.2");
// в коллекции два ключа: key1 и key2
CollectionAssert.AreEquivalent(col1.AllKeys, new[] { "key1", "key2" });
// совпадают значения, возвращаемые .GetValues(int) и .GetValues(string)
var key1values = new[] { "value1.1", "value1.2" };
CollectionAssert.AreEquivalent(col1.GetValues(0), key1values);
CollectionAssert.AreEquivalent(col1.GetValues("key1"), key1values);
```
Но, оказывается метод HttpWebResponse.Headers.GetValue(string) работает иначе.
```c#
int index = Array.IndexOf(res.Headers.AllKeys, "Set-Cookie");
var values1 = res.Headers.GetValues(index);
//    [0]: "cname=5; expires=Tue, 13-Apr-2010 23:43:04 GMT; path=/; domain=.getvaluetest.ru"
//    [1]: "cvalue=5; expires=Tue, 13-Apr-2010 23:43:04 GMT; path=/; domain=.getvaluetest.ru"
var values2 = res.Headers.GetValues("Set-Cookie");
//    [0]: "cname=5; expires=Tue"
//    [1]: "13-Apr-2010 23:43:04 GMT; path=/; domain=.getvaluetest.ru"
//    [2]: "cvalue=5; expires=Tue"
//    [3]: "13-Apr-2010 23:43:04 GMT; path=/; domain=.getvaluetest.ru"
```
Причина находится в методе WebHeaderCollection.GetValue(string), где происходит нарезка строки на части по каждой запятой. Поэтому получается неожиданный результат.
