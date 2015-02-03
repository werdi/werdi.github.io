---
title: Помогите форматировать RTF(Rich Text Format) файлы
tags: [c#, rtf, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/001989a8-b47b-4c29-a2f3-b251643256cc/-rtfrich-text-format-?forum=aspnetru
---
# Помогите форматировать RTF(Rich Text Format) файлы
*> шаблон договора в формате rtf [...] прописаны формулы [...] \placeholder1\ [...] необходимо заменить эти формулы*

rtf можно загрузить в RichTextBox.
```
var rtb= new System.Windows.Forms.RichTextBox();
rtb.Rtf = File.ReadAllText(@"c:\temp\test.rtf");
```
найти фрагменты в тексте и заменить.
сохранить с помощью метода:
```
rtb.SaveFile(@"c:\temp\result.rtf", RichTextBoxStreamType.RichText);
```
*> и если можно примерчики?*
```c#
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 400);
      this.Font = new System.Drawing.Font("verdana", 12);

      var rtb1 = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      var rtb2 = new RichTextBox { Parent = this, Dock = DockStyle.Right, Width = 300 };

      var ph = @"\placeholder\";
      rtb1.Text = @"hello " + ph + " ... " + ph + " ...\n" + ph;
      rtb1.SelectionStart = rtb1.Find(ph);
      rtb1.SelectionLength = ph.Length;
      rtb1.SelectionBackColor = Color.Yellow;
      rtb1.SelectionStart = rtb1.SelectionLength = 0;

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("run", delegate
      {
        rtb2.Rtf = rtb1.Rtf;
        var s = -1;
        while((s = rtb2.Find(ph, s+1, RichTextBoxFinds.None)) > -1)
        {
          rtb2.SelectionStart = s;
          rtb2.SelectionLength = ph.Length;
          rtb2.SelectedText = "world";
        }
      });
    }
  }
}
```
если на хостинге запрещено использовать RichTextBox, то 
RTF можно открыть как обычный текстовый файл; найти \placeholder\ с помощью Regex и заменить.
