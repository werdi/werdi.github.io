---
title: Unable to POST to an aspx page
tags: [c#, winforms, web browser, html]
src: https://social.msdn.microsoft.com/Forums/ru-RU/57a15dfe-6523-48ae-877d-117b25f9a829/unable-to-post-to-an-aspx-page?forum=csharpgeneral
---
# Unable to POST to an aspx page
*> I am working on a program that will use a webBrowser in a forms app. From here it needs to launch a website and login to it then redirect based on login info.*

if i've understood correctly you need to automate webbrowser. if so, then 
below is an example.
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(700, 500);
      var rtb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Start", (s, e) => this.Run("http://bing.com", rtb.AppendText));
    }

    void Run(string url, Action<string> fn)
    {
      var wb = new WebBrowser();
      wb.DocumentCompleted += (s, e) =>
      {
        switch (e.Url.LocalPath)
        {
          case "/":
            dynamic input = wb.Document.GetElementById("sb_form_q").DomElement;
            dynamic go = wb.Document.GetElementById("sb_form_go").DomElement;
            input.value = "malobukv msdn";
            go.click();
            break;
          case "/search":
            fn(wb.Document.Body.InnerText);
            break;
        }
      };
      wb.Navigate(url);
    }
  }
}
```