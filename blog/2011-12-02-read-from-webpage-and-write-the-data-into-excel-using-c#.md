---
title: Read from webpage and write the data into excel using C#
tags: [c#, winforms, web browser]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b66b2459-7262-44b8-828c-23b2c361e22c/read-from-webpage-and-write-the-data-into-excel-using-c?forum=ieextensiondevelopment
---
# Read from webpage and write the data into excel using C#
*> Now i want to give some extra functionality to it by reading data from webpage (or website)*

you can load web page into the WebBrowser and extract data through the DOM methods.
for example the code below loads the web page of the NASDAQ Real Time Stock Quotes for MSFT 
and retrieves the Last Sale value.
```c#
using System;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(200, 100);
      this.TopMost = true;
      var lbl = new Label { Parent = this, Dock = DockStyle.Fill, TextAlign = ContentAlignment.MiddleLeft };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Load", (s, e) => LoadData(str => lbl.Text = str));
    }
    void LoadData(Action<string> callback)
    {
      var url = new Uri("http://www.nasdaq.com/symbol/msft/real-time");
      var wb = new WebBrowser();
      wb.DocumentCompleted += (s, e) =>
      {
        if (wb.ReadyState == WebBrowserReadyState.Complete && url == e.Url)
        {
          var el = wb.Document.GetElementById("_LastSale");
          if (el != null)
            callback(el.InnerText);
        }
      };
      wb.Navigate(url);
    }
  }
}
```
