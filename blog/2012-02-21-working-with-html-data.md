---
title: Работа с HTML данными
tags: [c#, html, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d5ab5030-3216-4513-8c8f-41d7b19a3fb8/-html-?forum=vsru 
---
# Работа с HTML данными
*> HTML строки вытащить данный. Я патался вот таким способом, но у меня ничего не вышло. [...] wl.GetAttribute("class")*

вместо .GetAttribute("class") надо указать .GetAttribute("className") 
или ((dynamic)a.DomElement).attributes.item("class").value;
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var html = @"
        <a tabindex='5' class='Hello_World' href='http://www.site.com'>Site</a>
        <a href='http://www.site1.com'>Site1</a>";
      var wb = new WebBrowser { Parent = this, Dock = DockStyle.Fill, DocumentText = html };
      
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", delegate { Test(wb); });
    }

    private static void Test(WebBrowser wb)
    {
      foreach (HtmlElement a in wb.Document.GetElementsByTagName("A"))
      {
        var ti = a.GetAttribute("tabindex");
        var cn = a.GetAttribute("className");  // или: ((dynamic)a.DomElement).attributes.item("class").value;
        if (ti == "5" && cn == "Hello_World")
        {
          var href = a.GetAttribute("href");
          MessageBox.Show(href);
          break;
        }
      }
    }
  }
}
```
