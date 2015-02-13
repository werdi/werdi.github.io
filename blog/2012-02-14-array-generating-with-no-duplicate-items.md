---
title: Генерация массива из неповторяющихся элементов (некорректно работает System.Array.Сontains)
tags: [c#, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/73e9eddc-aa09-46f0-9bc7-00a94198cbb3/-systemarrayontains-?forum=fordesktopru
---
# Генерация массива из неповторяющихся элементов (некорректно работает System.Array.Сontains)
*> сгенерировать массив из неповторяющихся элементов, но с использованием Random.*
```c#
var res = Random(3, 1, 20).ToArray();
...
public IEnumerable Random(int count, int min, int max)
{
  var set = new HashSet();
  var rnd = new Random();
  while (set.Count != count)
  {
    var v = rnd.Next(min, max);
    if (set.Contains(v) == false)
    {
      yield return v;
      set.Add(v);
    }
  }
}
```
см. другие способы [здесь](http://social.msdn.microsoft.com/Forums/en-US/csharpgeneral/thread/f5dd6491-e2b3-4e77-927a-ad28aab85abc/)

*> есть массив типа string, необходимо "перемешать" элементы массива с использованием Random.*
```c#
using System.Linq;
using System.Collections.Generic;
...
var res = Random(new [] { "1", "2", "3", "4", "5" }).ToArray();
...
IEnumerable<T> Random<T>(IEnumerable<T> src)
{
  var rnd = new Random();
  return src.OrderBy(x => rnd.Next());
}
```
см. другой способ [здесь](http://stackoverflow.com/a/110570) (реализация Fisher-Yates algorithm)

*> .OrderBy(x=>rnd.Next()); [...] это рискованный способ, такая сортировка может не сойтись... Или по крайней мере, будет работать дольше чем могла бы.*

для коллекций из тысяч и более элементов можно использовать Guid.NewGuid() вместо Random.Next(), но в остальных случаях - оставить как есть.
см. [здесь] (http://stackoverflow.com/questions/6188331/shuffle-string-array-without-duplicates#comment7211957_6197291): ".OrderBy(x=>rand.Next());, and you're done. [...] really, it's fine to order by a random key." -- Eric Lippert, a principal developer on the Microsoft Visual C# compiler team.
