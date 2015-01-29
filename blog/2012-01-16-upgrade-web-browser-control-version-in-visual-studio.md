---
title: Как "обновить" версию компонента WebBrowser в Visual Studio 2010?
tags: [c#, winforms, web browser, html5]
src: https://social.msdn.microsoft.com/Forums/ru-RU/17d1a95e-ae19-48de-96f9-5c117e44d121/-webbrowser-visual-studio-2010?forum=desktopprogrammingru
---
# Как "обновить" версию компонента WebBrowser в Visual Studio 2010?
*> компонент WebBrowser, но у него движок от ИЕ 7... Как мне обновить его до ИЕ 8 или ИЕ 9?*
в html указать <meta http-equiv='X-UA-Compatible' content='IE=9' />.
пример html5 в WebBrowser:
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 500);
      new WebBrowser { Dock = DockStyle.Fill, Parent = this }
        .DocumentText = 
           @"<!DOCTYPE html>
            <html>
            <head>
              <meta http-equiv='Content-Type' content='text/html; charset=unicode' />
              <meta http-equiv='X-UA-Compatible' content='IE=9' /> 
              <title></title>
            </head>
            <body>
              <div>
                 <video autoplay='autoplay' preload='metadata'>
                    <source src='http://html5demos.com/assets/dizzy.mp4' type='video/mp4' />
                </video>
              </div>
            </body>
            </html>";
    }
  }
}
```
