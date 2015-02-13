---
title: Парсинг многострочного текстового файла - C#
tags: [c#, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9cba1a5d-ff3c-478d-86ec-9520d9c97b5a/-c?forum=programminglanguageru
---
# Парсинг многострочного текстового файла - C#
*> текстовый файл, в котором есть некие разделы [...] нужно распарсить этот текст по заголовкам [...] выделить целиком всю часть между заголовками*

работает с файлами любого размера, т.к. не загружает файл целиком; можно фильтровать строки во время загрузки ...
```c#
using System.Collections.Generic;
using System.IO;
using System.Linq;
...
foreach (var b in Block.Load("test.txt"))
{
  var str = b.Title + "\n\t" + string.Join("\n\t", b.Body);
  System.Diagnostics.Trace.WriteLine(str);
}
...
class Block
{
  public string Title;
  public IList<string> Body;

  public static IEnumerable<Block> Load(string path)
  {
    Block ret = null;
    foreach (var line in File.ReadLines(path).Select(l => l.Trim()))
    {
      if (line.Length == 0 && ret != null)
      {
        yield return ret;
        ret = null;
        continue;
      }

      if (line.EndsWith(":"))
      {
        ret = new Block { Title = line.TrimEnd(':'), Body = new List<string>() };
        continue;
      }

      if (ret != null)
        ret.Body.Add(line);
    }
  }
}
```