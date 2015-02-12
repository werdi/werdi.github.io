---
title: Автозамена HTML тегов
tags: [c#, winforms, html, webbrowser]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ba4d39cd-54b2-45b5-9a5d-679111d05a1f/-html-?forum=programminglanguageru
---
# Автозамена HTML тегов
*> Есть HTML документ, необходимо заменить теги в нем на свои. [...] Например, в исходном документе есть <div id="123"><p class="mb20">исходный текст</p></div>и надо это заменить на<p class="1">исходный текст</p>*
```c#
using System.Windows.Forms;
using System;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var html = @"
      <html>
        <head><style>.x1 { border:1px solid navy;} </style></head>
        <body>
          <div id=""123"" style='border:1px solid red;'>
            <p class=""mb20"">исходный текст</p>
          </div>
        </body>
      </html>";
      Action<HtmlDocument> update = doc =>
        {
          var div = doc.GetElementById("123");
          if(div == null) return;
          var p = div.FirstChild;
          div.InsertAdjacentElement(HtmlElementInsertionOrientation.AfterEnd, p);
          p.SetAttribute("className", "x1");
          ((dynamic)div.Parent.DomElement).removeChild(div.DomElement);
        };
      var wb = new WebBrowser { Parent = this, Dock = DockStyle.Fill, DocumentText = html };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test1", (s, e) => 
        {
          wb.DocumentText = html;
          while(wb.ReadyState != WebBrowserReadyState.Complete)
            Application.DoEvents();
          update(wb.Document);
        });
      this.Menu.MenuItems.Add("Test2", delegate 
      {
        new WebBrowser { DocumentText = html }
          .DocumentCompleted += (s, e) =>
          {
            var b = s as WebBrowser;
            update(b.Document);
            wb.DocumentText = b.Document.Body.Parent.OuterHtml;
          };
      });
      this.Menu.MenuItems.Add("Reset html", (s, e) => wb.DocumentText = html);
    }
  }
}
```