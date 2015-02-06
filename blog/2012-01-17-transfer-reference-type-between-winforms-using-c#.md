---
title: Transfer Reference Type (string) between WinForms using C#
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b550b7c0-03cb-4f7c-a3e6-db983588ba4d/transfer-reference-type-string-between-winforms-using-c?forum=winforms 
---
# Transfer Reference Type (string) between WinForms using C#
*> would like for a Reference Type (string) to be accessible across my WinApp. [...] frmMain.UsrID [...] But the Types remain blank (string.Emptry).*

use Application.OpenForms
below is an example.
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Name = "Main";
      new Button { Parent = this, Text = "Logon" }
        .Click += (s, e) =>
          {
            var f = new Logon();
            f.ShowDialog();
            System.Diagnostics.Trace.WriteLine(this.UsrID);
          };
    }

    public string UsrID { get; set; }
  }
  
  class Logon : Form
  {
    public Logon()
    {
      var f = Application.OpenForms["Main"] as Form1;
      f.UsrID = "OK";
    }
  }
}
```
