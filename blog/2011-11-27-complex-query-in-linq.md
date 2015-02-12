---
title: Сложный запрос в Linq
tags: [c#, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/777fa5da-fc22-4657-b9fe-60c22b972be0/-linq?forum=fordataru
---
# Сложный запрос в Linq
*> Есть товар(номер), он продаётся в разных городах [...] много магазинов, где его продают. [...] Есть список городов [...]  получить список городов из первой таблицы(чтобы не повторялись) и соответствующие id им из второй таблицы*

т.е. надо получить список городов, в которых хоть что-то продается.
```c#
var products = new[] 
{
  new { Name = "p0", Id = 0 },
  new { Name = "p1", Id = 1 },
  new { Name = "p2", Id = 2 },
};
var cities = new[] 
{
  new { Name = "c0", Id = 0 },
  new { Name = "c1", Id = 1 },
  new { Name = "c2", Id = 2 },
  new { Name = "c3", Id = 3 },
  new { Name = "c4", Id = 4 },
};
var shops = new[] 
{
  new { City = 0, Products = new [] { 0, 1 } },
  new { City = 0, Products = new [] { 1, 2 } },
  new { City = 1, Products = new [] { 0 } },
};

var cids = from s in shops group s by s.City into g select g.Key;
var res = from cid in cids join c in cities on cid equals c.Id select c;
foreach (var c in res)
  System.Diagnostics.Trace.WriteLine(c);
```
или так 
```c#
var ret = from cid in (from s in shops select s.City).Distinct() 
          join c in cities on cid equals c.Id
          select c;
foreach (var c in res)
  System.Diagnostics.Trace.WriteLine(c);
``` 
результат:
```
{ Name = c0, Id = 0 }
{ Name = c1, Id = 1 }
```