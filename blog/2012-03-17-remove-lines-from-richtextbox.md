---
title: Удаление строк в RichTB
tags: [c#, winforms, richtextbox]
src: https://social.msdn.microsoft.com/Forums/ru-RU/bf76f3d6-6fc3-421d-ae0b-4ab9a55f0335/-richtb?forum=fordesktopru
---
# Удаление строк в RichTB
*> richTextBox1.Text = richTextBox1.Text.Remove(start_index, count); [...] как только начитается затирка, тут-же исчезают все цвета, и код становится равномерно монотонным*

это из-за замены значения свойства Text.
надо выделить первую строку и заменить значение SelectedText
```c#
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var tb = new RichTextBox { Dock = DockStyle.Fill, Parent = this, Font = new Font("verdana", 12) };
      this.Load += (s, e) => Run(tb);
    }

    private void Run(RichTextBox tb)
    {
      var i = 0;
      var t = new System.Windows.Forms.Timer { Interval = 500 };
      t.Tick += (s, e) =>
      {
        if (tb.Lines.Length - 1 == 2)
        {
          // убрать первую строку
          tb.Select(0, tb.Lines[0].Length+1);      
          tb.SelectedText = "";
        }
        
        var li = tb.Lines.Length - 1;
        tb.AppendText((li++ >= 0 ? "\n" : "") + "line"+i++);
        var ci = tb.GetFirstCharIndexFromLine(li);
        tb.Select(ci, 4);   // tb.Select(ci, tb.Lines[li].Length);
        tb.SelectionBackColor = Color.Red;
        tb.SelectionLength = 0;
      };
      t.Start();
    }
  }
}
```
*> "tb.SelectedText="";", при каждой попытке ее выполнить в динамиках ПК слышен звук некритической ошибки*

какая ос? у меня в win 7 - тихо. попробуйте заменить на tb.SelectedRtf = ""; или следующим кодом:
```c#
tb.SelectedText = " ";
SendKeys.SendWait("{BKSP}");
```
