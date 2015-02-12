---
title: How to do scaling matrix C# windows form ?
tags: [c#, winforms, linq]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d480858d-df06-4c24-bbbd-c17a5ff13f5e/how-to-do-scaling-matrix-c-windows-form-?forum=csharpgeneral
---
# How to do scaling matrix C# windows form ?
*> load the array from text file for example : 3 56 234 12 167 123 26 84 ...*
```c#

using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var data = LoadData(4, "test.txt");
      var data2 = data
        .SelectMany(row => new[] { row, row })   // double row
        .Select(row => 
          row.SelectMany(cell => new[] { cell, cell }));   // double cell
      foreach (var row in data2)
        System.Diagnostics.Trace.WriteLine(String.Join(" ", row));
    }
    IEnumerable<IEnumerable<int>> LoadData(int capacity, string path)
    {
      var res = new List<int>(capacity);
      foreach (var line in File.ReadLines(path))
      {
        if (string.IsNullOrEmpty(line)) continue;
        int val;
        if (int.TryParse(line, out val) == false) continue;
        res.Add(val);
        if (res.Count == capacity)
        {
          yield return res;
          res = new List<int>(capacity);
        }
      }
    }
  }
}
```
result:
```   
3 3 56 56 234 234 12 12
3 3 56 56 234 234 12 12
167 167 123 123 26 26 84 84
167 167 123 123 26 26 84 84 
```
you can convert result to the jagged array with the following code: 
```
// jagged array (array-of-arrays)
var ja = data2.Select(c => c.ToArray()).ToArray();    
```
also you can convert result to rectangular array. but for the best performans use single-dimensional array (take a look at the [Harness the Features of C# to Power Your Scientific Computing Projects](http://msdn.microsoft.com/en-us/magazine/cc163995.aspx) -- there are benchmarks for the different array types)