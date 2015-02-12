---
title: Reading from a file line by line
tags: [c#, linq, wpf, regex]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f8b3a8b5-8770-45de-b088-7bbd5b0e736a/reading-from-a-file-line-by-line?forum=csharpgeneral
---
# Reading from a file line by line
*> only show values such as "Violation found: type=3(Line), level=Default(0), color=5, weight=0, style=4." There are quite a number of duplicates as you can see because of the number of violations found.*

so, you want to extract substrings with "Violation found:...." and remove duplicates.
below is an example.
```c#
using System.IO;
using System.Linq;
using System.Text.RegularExpressions;
using System.Windows;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      var re = new Regex(
        @"Violation\s+found:\s+type=(.+?),\s+level=(.+?),\s+color=(.+?" +
        @"),\s+weight=(.+?),\s+style=(.+?)\.",
        RegexOptions.IgnoreCase | RegexOptions.Multiline | RegexOptions.Compiled);

      var ms =
        from line in File.ReadLines("test.txt")
        let m = re.Match(line)
        where m.Success
        select new
        {
          Type = m.Groups[1].Value,
          Level = m.Groups[2].Value,
          Color = m.Groups[3].Value,
          Weight = m.Groups[4].Value,
          Style = m.Groups[4].Value
        };

      foreach (var str in ms.Distinct())
        System.Diagnostics.Trace.WriteLine("Violation found: " + str.ToString().Trim('{', '}', ' '));
    }
  }
}
```   
result for the file \Drawings\H338088-0070-60-042-0015:
```
Violation found: Type = 4(Line string), Level = Default(0), Color = 5, Weight = 0, Style = 0
Violation found: Type = 3(Line), Level = Default(0), Color = 5, Weight = 0, Style = 0
```