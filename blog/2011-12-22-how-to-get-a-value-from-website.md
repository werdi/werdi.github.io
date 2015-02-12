---
title: C# .Net how to get a value from website
tags: [c#, html, winforms, web]
src: https://social.msdn.microsoft.com/Forums/ru-RU/76f5ab6b-f9cb-405c-a3e6-2402b6029e7a/c-net-how-to-get-a-value-from-website?forum=csharpgeneral
---
# C# .Net how to get a value from website
*> I have a webbrowser in my WinForm. [webbrowser1] I need to get these values [ 0111255H and XW475A4#UUZ ]*
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var html = @"
      <table>
        <tr>
          <td class='label'><span id='dlg1_ctrl1_ctl00_Label1'>Art-Nr</span></td>
          <td class='value' width='200' colSpan='3'>0111255H</td>
          <td class='label2'><span id='dlg1_ctrl1_ctl00_Label11'>Herst-Nr</span></td>
          <td class='value' colSpan='2'>XW475A4#UUZ</td>
          <td rowSpan='12'>&nbsp;&nbsp;&nbsp;</td>
          <td rowSpan='12'><table cellpadding='0' cellspacing='0' border='0' width='100%'>
        </tr>
      </table>";
      var tb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      var wb = new WebBrowser() { DocumentText = html };
      wb.DocumentCompleted += (s, e) =>
        {
          var v1 = GetValue(wb.Document, "dlg1_ctrl1_ctl00_Label1");
          var v2 = GetValue(wb.Document, "dlg1_ctrl1_ctl00_Label11");
          tb.AppendText(v1 + "; " + v2);
        };
    }
    string GetValue(HtmlDocument doc, string id)
    {
      var td = doc.GetElementById(id).Parent;
      return td.NextSibling.InnerText;
    }
  }
}
```
*> But I want 0111255H and XW475A4#UUZ. And those <td's hasn't got an ID.*

have you tried use an example above? GetValue method solves your issue.

if you need to get "0111255H" from
```html
<tr>
  <td class='value' width='200' colSpan='3'>0111255H</td>
  <td class='label'><span id='dlg1_ctrl1_ctl00_Label1'>Art-Nr</span></td>
</tr>
```
then use the following code:
```c# 
dynamic p = wb.Document.GetElementById("dlg1_ctrl1_ctl00_Label1").Parent.DomElement;
string v = p.previousSibling.InnerText; 
```