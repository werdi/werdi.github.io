---
title: Function to calculate grades from multiple files
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/58107d9e-ff45-4c58-87ec-6215467a1246/function-to-calculate-grades-from-multiple-files?forum=csharpgeneral
---
# Function to calculate grades from multiple files
*> var reader = new StreamReader(categoryFileName); while(!reader.EndOfStream) [...] reader.Close();*

if you need to read a text file, just use the code below instead of code above
```c#
var text = System.IO.File.ReadAllText(categoryFileName);
```
and please provide a samples of all .txt files.

*> Is that something along the lines of what you wanted?*
yes. try using the following code:
```c#
using System.IO;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dir = @"C:\Temp\";    // path to the folder with your .txt files
      var students = (from line in File.ReadLines(dir + "students.txt")
                where string.IsNullOrWhiteSpace(line) == false
                let arr = line.Split(',')
                let id = arr[0]
                let data = File.ReadLines(dir + id + ".txt").Select(d =>
                            {
                              var itms = d.Split(',');
                              return new { A = itms[0], B = int.Parse(itms[1]), C = int.Parse(itms[2]) };
                            })
                select new
                {
                  ID = id,
                  LastName = arr[1],
                  FirstName = arr[2],
                  Data = data.ToArray()
                });

      var tb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      foreach (var s in students)
      {
        tb.AppendText(s.FirstName + " " + s.Data.Sum(a => a.C) + "\n");
      }
    }
  }
} 
```
