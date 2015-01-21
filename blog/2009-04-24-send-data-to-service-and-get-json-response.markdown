---
title: Как передать данные методу сервиса и получить json-ответ
tags: [asp.net, json, rest, c#]
src: http://cognitex.blogspot.ru/2009/04/json.html
---
# Как передать данные методу сервиса и получить json-ответ
Предположим, что требуется к существующему сервису [Providers.DataProvider] (http://cognitex.blogspot.com/2009/04/blog-post_24.html) добавить метод, доступный по адресу "/Now/<name>=<number>", где вместо <name> должно быть указано какое-то слово, а вместо <number> - какое-то число. В ответ сервис пришлет ответ в json-формате: 
{"Name":"\"<name>\"","Value":<value>}

Для решения задачи надо в класс Providers.DataProvider добавить следующий метод:
```c#
[OperationContract]
[WebGet(UriTemplate = "/Now/{name}={number}", ResponseFormat = WebMessageFormat.Json)]
public Info GetInfo(string number, string name)
{
    	int value = 0;
    	if (int.TryParse(number, out value))
    	{
        	value = 10 * value;
    	}
    	return new Info() { Value = value, Name = string.Format("\"{0}\"", name) };
}
```
И следующий класс:
```c#
public class Info
{
    	public string Name { get; set; }
    	public int Value { get; set; }
}
```
Откомпилировать проект и полученную сборку \Debug\bin\Service.dll скопировать в папку \bin на сайте.

Если сайт доступен по адресу http://localhost:3840, то по адресу http://localhost:3840/DataProvider.svc/Now/site=10 получим следующий json: {"Name":"\"site\"","Value":100}

Надо заметить, что в методы сервиса можно передавать строки. Иначе, например, если метод GetInfo определить как public Info GetInfo(int number, string name), то при вызове метода получим сообщение об ошибке: 
Server Error in '/' Application. Operation 'GetInfo' in contract 'DataProvider' has a path variable named 'number' which does not have type 'string'. Variables for UriTemplate path segments must have type 'string'. Description: An unhandled exception occurred during the execution of the current web request. Please review the stack trace for more information about the error and where it originated in the code. Exception Details: System.InvalidOperationException: Operation 'GetInfo' in contract 'DataProvider' has a path variable named 'number' which does not have type 'string'. Variables for UriTemplate path segments must have type 'string'.
