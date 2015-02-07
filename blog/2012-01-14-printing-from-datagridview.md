---
title: Печать из DataGridView
tags: [c#, winforms, graphics]
src: https://social.msdn.microsoft.com/Forums/ru-RU/be7ca8a4-d424-4535-ae1a-8fb32a2567eb/-datagridview?forum=fordesktopru
---
# Печать из DataGridView
*> Данные из БД загружаются в DataGridView. А сейчас мне нужно напечатать эти данные. Как послать эти данные на печать?*
```c#
using System.Data;
using System.Drawing;
using System.Drawing.Printing;
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("c1", typeof(int));
      dt.Columns.Add("c2", typeof(int));
      for(int i=0; i < 5; i++)
        dt.Rows.Add(i, i);

      var dg = new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = dt };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Print", delegate 
      {
        var pd = new PrintDocument();
        pd.PrintPage += (s, e) =>
          {
            var bmp = new Bitmap(dg.Width, dg.Height);
            dg.DrawToBitmap(bmp, dg.ClientRectangle);
            e.Graphics.DrawImage(bmp, new Point(100, 100));
          };
        pd.Print();
      });
    }
  }
}
```
+ см. [The DataGridViewPrinter Class] (http://www.codeproject.com/KB/printing/datagridviewprinter.aspx); 
[Print the data of the datagridview in C#] (http://social.msdn.microsoft.com/Forums/bg-BG/winforms/thread/5024a2af-e6d6-4a99-88af-6a2b3b0add1f); 
[Printing a DataGridView] (http://www.codeproject.com/KB/grid/GridDrawer.Net.aspx)

*> Качество печати при таком методе будет просто ужасным.*

вывод в bitmap и последующий вывод на печать - это только для примера.
в обработчике PrintPage надо использовать методы e.Graphics.
примеры можно найти по ссылкам; см. выше под кодом.
