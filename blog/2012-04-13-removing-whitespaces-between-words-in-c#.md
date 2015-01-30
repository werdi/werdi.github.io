---
title: How do I delete unwanted whitespaces between words in C#?
tags: [c#, linq, regex]
src: https://social.msdn.microsoft.com/Forums/ru-RU/3a431f8a-4860-41d7-b9a0-0b8c3d3f12e7/how-do-i-delete-unwanted-whitespaces-between-words-in-c?forum=csharpgeneral
---
# How do I delete unwanted whitespaces between words in C#?
*> I have whitespaces between words in a text file.  I need to replace them with '|' (pipe delimiter).  How do I do this?*

```c#
using System.IO;
using System.Linq;
using System.Text.RegularExpressions;

static void Transform(string sourceFile, string resultFile)
{
  var re = new Regex(@"(?<=\b)\s+(?=\b)", RegexOptions.Compiled | RegexOptions.Singleline);
  var lines = File.ReadLines(sourceFile).Select(line => re.Replace(line, "|"));
  File.AppendAllLines(resultFile, lines);
}
```
