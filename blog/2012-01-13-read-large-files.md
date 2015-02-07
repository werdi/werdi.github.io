---
title: Reading Large Files
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4f861585-598d-4d69-92b3-6965fafc7a35/reading-large-files?forum=csharplanguage
---
# Reading Large Files
*> I have two large files (new file = 500000kb and old file = 92000kb) stored in a .DAT file, but really it is just a CSV file with quotes as text qualifiers. My goal is to compare both datasets/files and output all data that is in the new file not in the old file. I have the two methods below but i run out of memory.*

try using the following example  
```c#
using System;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      foreach(var line in Process("file1.txt", "file2.txt"))
        System.Diagnostics.Trace.WriteLine(line);
    }

    IEnumerable<string> Process(string path1, string path2)
    {
      Func<string, int> hc = line => Regex.Replace(line, "\\s+", "").GetHashCode();
      var hs = new HashSet<int>();
      foreach(var line in System.IO.File.ReadLines(path1))
        hs.Add(hc(line));
      foreach(var line in System.IO.File.ReadLines(path2))
      {
        if(hs.Contains(hc(line)) == false)
          yield return line;
      }
    }
  }
}
```
*> However on my two largest files it missed some lines [...] ANy idea why, i thought hashes are unique?*

"The default implementation of the GetHashCode method does not guarantee unique ...".
but you can use your implementation.
take a look at the [Guidelines and rules for GetHashCode] (http://blogs.msdn.com/b/ericlippert/archive/2011/02/28/guidelines-and-rules-for-gethashcode.aspx).
