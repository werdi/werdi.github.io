---
title: C# WebBrowser Как сохранить загруженный документ?
tags: [c#, html, xml, webbrowser, winforms, dom]
src: https://social.msdn.microsoft.com/Forums/ru-RU/e5e9ce2f-1dff-45b3-9728-d02bdb51f4ec/c-webbrowser-?forum=programminglanguageru
---
# C# WebBrowser Как сохранить загруженный документ?
*> как мне получить исходную загруженную страницу*

как вариант можно обойти DOM. примерно так:
```c#
using System;
using System.Text.RegularExpressions;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(500, 500);
      var rtb = new RichTextBox { Parent = this, Dock = DockStyle.Fill };
      Action<string, int> cb = (n, indent) => rtb.AppendText("\n" + new string(' ', indent * 4) + n);
      var html = @"
      <!doctype html>
      <html>
        <head>
          <script id='x1' language='xml'>
            <xml1><item id='1' /></xml1>
          </script>
        </head>
        <body style='background-color: whitesmoke;'>
          <div>hello</div>
          <script id='x2' language='xml'>
            <xml2><item id='1' /></xml2>
          </script>
        </body>
      </html>";
      new WebBrowser { DocumentText = html }
        .DocumentCompleted += (s, e) =>
        {
          var wb = s as WebBrowser;
          dynamic doc = wb.Document.DomDocument;
          TraverseHtml(doc, cb);
        };
  }

  void TraverseHtml(dynamic n, Action<string, int> cb, int indent = 0)
  {
    var nn = (string)n.nodeName;
    if(n.hasAttributes)
    {
      foreach (dynamic a in n.attributes)
      {
        if(a.specified)
          nn += " " + (string) a.nodeName;
      }
    }
    if (nn == "SCRIPT")
    {
      if (string.Equals((string)n.getAttribute("language"), "xml", StringComparison.OrdinalIgnoreCase))
        cb("<XML>" + Regex.Replace((string)n.XMLDocument.xml, "[\n\r]", " ") + "</XML>", indent);
    }
    else
      cb(nn, indent);

    if ((bool)n.hasChildNodes)
    {
      foreach (dynamic cn in n.childNodes)
        TraverseHtml(cn, cb, indent + 1);
      }
    }
  }
}
```

*> Откройте в ie лубое изображение и потом сохраните документ вашим способом, увидите чего он там подокидывал:)*

для следующего html:
```html
<!doctype html>
<html>
  <head/>
  <body>
    <img src='http://i1.social.s-msft.com/profile/u/avatar.jpg?displayname=malobukv' />
  </body>
</html>
```
пример выдает:
```
#document
  #comment
  HTML
    HEAD
      TITLE
    BODY
      IMG src
      #text
```
