---
title: Retrive List(of string) values from Dictionary of (string, List(of string))
tags: []
src: https://social.msdn.microsoft.com/Forums/ru-RU/b4391913-2594-46c2-922b-89544867ac83/retrive-listof-string-values-from-dictionary-of-string-listof-string?forum=csharpgeneral
---
# Retrive List(of string) values from Dictionary of (string, List(of string))
*> Dictionary<string, List<of string>> I'm trying to store the a.Categories in data.Attributes and then print all the keys (strings) and values (lists of string)*

use SelectMany
below is an example

```c#
using System.Collections.Generic;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var testdata = new Dictionary<string, List<string>>()
      {
        { "k1", new List<string> { "1", "2" }},
        { "k2", new List<string> { "3", "4" }},
      };

      foreach (var str in testdata.SelectMany(kvp => kvp.Value))
        System.Diagnostics.Trace.WriteLine(str);
    }
  }
}
```
*> textBox1.Text = str + Environment.NewLine; [...] Dictionary<string, List<of string>> I'm trying to store the a.Categories in data.Attributes and then print all the keys (strings) and values (lists of string) [...]*

if you need to concatenate a values, then use SelectMany and Join:
textbox1.Text = string.Join(Environment.NewLine, testdata.SelectMany(kvp => kvp.Value));
  
but if you want to use foreach like you do, then try using the following example:
```c#
