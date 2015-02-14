---
title: Событийное программирование
tags: [c#, winforms, web browser, html]
src: https://social.msdn.microsoft.com/Forums/ru-RU/86e4ccd6-baef-499c-92e8-982cbb29e307/-?forum=fordesktopru
---
# Событийное программирование
*> C# компонент WebBrowser, необходимо выдержать следующий протокол для автоматизации действий: Загрузить страницу; Найти определенную форму заполнить поля и отправить; Полученный результат сохранить.*
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
*> сайты разные, количество действий с ними тоже отличается, но все это приходит в один DocumentCompleted, а необходимо еще до Navigate засекать время.*

можено переделать предыдущий пример.  
добавить класс Info ...
```c#
using System;
using System.Windows.Forms;
using System.Diagnostics;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(700, 500);
      var rtb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Start", (s, e) =>
        this.Run("http://bing.com", info => rtb.AppendText(info.ElapsedMilliseconds + "\n" + info.Result))
        );
    }

    class Info
    {
      public string Url { get; set; }
      public long ElapsedMilliseconds { get; set; } 
      public string Result { get; set; }
    }

    void Run(string url, Action<Info> fn)
    {
      var wb = new WebBrowser();
      var info = new Info { Url = url };
      var sw = new Stopwatch();
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
            sw.Stop();
            info.Result = wb.Document.Body.InnerText;
            info.ElapsedMilliseconds = sw.ElapsedMilliseconds;
            fn(info);
            break;
        }
      };
      sw.Start();
      wb.Navigate(url);
    }
  }
}
```