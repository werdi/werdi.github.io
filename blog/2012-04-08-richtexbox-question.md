---
title: Richtextbox question
tags: [richtextbox, c#, winforms, tom, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a6a330b7-6c31-4f6b-a502-e846a748d808/richtextbox-question?forum=winforms
---
# Richtextbox question
*> Richtextbox [...] I want to change the cursor to "Hand" when the mouse is over a bold word ...* 

this is possible with TOM ([Text Object Model] (http://msdn.microsoft.com/en-us/library/windows/desktop/bb787607(v=vs.85).aspx)): ITextDocument and ITextRange interfaces.
below is an example (created in Visual Studio 11)
```c#
using System.Drawing;
using System.Runtime.InteropServices;
using System.Windows.Forms;
using tom;     // add reference to the file C:\Windows\System32\msftedit.dll

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(500, 300);
      this.Font = new System.Drawing.Font("verdana", 15f);

      var tb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      Fill(tb);
      var td = tb.GetTextDocument();
      tb.MouseMove += (s, e) =>
      {
        var ch = tb.GetCharIndexFromPosition(e.Location);
        var tr = td.Range(ch, ch + 1);
        tr.Expand(2);   // tomWord
        tb.Cursor = (tr.Font.Bold == -1) ? Cursors.Hand : Cursors.Default;

        System.Diagnostics.Trace.WriteLine(tb.Rtf);
      };
    }
    private void Fill(RichTextBox tb)   // test data
    {
      tb.Rtf = @"{\rtf1\ansi\ansicpg1251\deff0\deflang1049{\fonttbl{\f0\fnil\fcharset204 Verdana;}}
      \viewkind4\uc1\pard\f0\fs30 hello, \b world\b0 !\par}";
    }
  }

  public static class RichTextBoxExtension
  {
    [DllImport("user32.dll", CharSet = CharSet.Auto)]
    static extern int SendMessage(HandleRef hWnd, int msg, int wParam, 
      [MarshalAs(UnmanagedType.IUnknown)] out object editOle);
  
    public static ITextDocument GetTextDocument(this RichTextBox tb)
    {
      object editOle = null;
      return (SendMessage(new HandleRef(tb, tb.Handle), 0x43c, 0, out editOle) != 0)
        ? (ITextDocument)editOle
        : null;
    }
  }
}
```
