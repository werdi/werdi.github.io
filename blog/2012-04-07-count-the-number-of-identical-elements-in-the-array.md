---
title: Как посчитать число одинаковых элементов в массиве?
tags: [c#, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/0a4d98ab-4aff-4436-9813-afc9db0225e6/-?forum=programminglanguageru
---
# Как посчитать число одинаковых элементов в массиве?
```c#
using System.Collections.Generic;
...
var a = new[] { 11, 11, 23, 23, 23, 23, 23, 44, 88, 88 };
var h = new Dictionary<int, int>();
foreach (var i in a)
{
  int res;
  if (h.TryGetValue(i, out res))
    h[i] += 1;
  else
    h.Add(i, 1);
}
System.Diagnostics.Trace.WriteLine("count: " + h.Count);
foreach(var kv in h)
    System.Diagnostics.Trace.WriteLine(kv.Key + " (" + kv.Value + ")");
```
или другой вариант, с помощью LINQ:

```c#
using System.Linq;
...
var a = new[] { 11, 11, 23, 23, 23, 23, 23, 44, 88, 88 };
var g = a.GroupBy(i => i);
System.Diagnostics.Trace.WriteLine("count: " + g.Count());
foreach (var k in g)
  System.Diagnostics.Trace.WriteLine(k.Key + " (" + k.Count() + ")");
```
результат:
```
count: 4
11 (2)
23 (5)
44 (1)
88 (2)
```
