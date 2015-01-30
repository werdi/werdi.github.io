---
title: C# array with for loop
tags: [c#, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/c0346ad1-c361-4656-a8cc-67928470987a/c-array-with-for-loop?forum=csharplanguage
---
# C# array with for loop
*> something like this in console: 1111111111  0000000000   1111111111  0000000000*
try using the following code
```c#
for (int row = 0; row < 4; row++)
{
  int val = (row % 2 == 0) ? 1 : 0;
  for (int cell = 0; cell < 10; cell++)
    System.Diagnostics.Trace.Write(val);
  System.Diagnostics.Trace.WriteLine("");
}
```    
or you can use linq. below is an example.
```c#
using System;
using System.Collections.Generic;
using System.Data;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      foreach (var row in Create(4, 10))
        System.Diagnostics.Trace.WriteLine(String.Join("", row));
    }

    IEnumerable<IEnumerable<int>> Create(int rows, int cells)
    {
      return from row in Enumerable.Range(0, rows)
        select from cell in Enumerable.Range(0, cells)
          select (row % 2 == 0) ? 1 : 0;
    }
  }
}
```
*> i have array 10x10 so exactly it should look like this [...] and it should be done using "for" and "if"*
i think in your case no need to use "if"
```c#
using System;
using System.Windows;
using System.Windows.Input;

namespace ConsoleApplication1
{
  class Program
  {
    static void Main(string[] args)
    {
      int val = 1;
      for (int row = 0; row < 10; row++)
      {
        for (int cell = 0; cell < 10; cell++) Console.Write(val);
        Console.WriteLine("");
        val = 1 - val;
      }

      Console.ReadKey();
    }
  }
}
```
