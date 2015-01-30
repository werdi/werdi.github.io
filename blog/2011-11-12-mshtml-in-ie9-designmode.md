---
title: mshtml in ie9 designMode not worked
tags: [mshtml, winforms, ie, c#]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f010f66b-0f01-4bc5-a8d2-890c240c8244/mshtml-in-ie9-designmode-not-worked?forum=wpf
---
# mshtml in ie9 designMode not worked
*> webBrowser1.DocumentText = "<html><body></body></html>";*
that is the question of WinForms, not WPF. 
therefore below is an example for WinForms.
```c#
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var wb = new WebBrowser
      {
        Parent = this,
        Dock = DockStyle.Fill,
        DocumentText = "<html><body></body></html>"
      };
      wb.DocumentCompleted += (s, e) =>
      {
        dynamic d = wb.Document.DomDocument;
        d.designMode = "On";
      };
    }
  }
}
```
