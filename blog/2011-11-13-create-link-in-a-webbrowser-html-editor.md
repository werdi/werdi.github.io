---
title: How to create a link in C# webbrowser html editor?
tags: [c#, winforms, web browser, dom]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1d050260-3625-42cc-94ec-59bba0651a1c/how-to-create-a-link-in-c-webbrowser-html-editor?forum=csharpgeneral
---
# How to create a link in C# webbrowser html editor?
*> I would like to programmability create a link within an webbrowser document.*
```c#
using System.Drawing;
using System.Windows.Forms;
using System.Xml.Linq;

namespace WindowsFormsApplication0
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(900, 800);
      var wb = new WebBrowser { Parent = this, DocumentText = "select word and then click link", Dock = DockStyle.Fill };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Link", (s, e) => 
      {
        dynamic doc = wb.Document.DomDocument;
        dynamic r = doc.selection.createRange();
        if(r.text != "")
          r.ExecCommand("CreateLink", false, "http://bing.com");
      });
    }
  }
}
```
if you need to show Hyperlink Dialog, then use r.ExecCommand("CreateLink", true, ""); 

*> I would like to insert the text and the link*

```c#
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(900, 800);
      var wb = new WebBrowser 
      { 
        Parent = this, 
        DocumentText = "<body contenteditable='true'>text text text</body>", 
        Dock = DockStyle.Fill 
      };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Link", (s, e) => 
      {
        dynamic doc = wb.Document.DomDocument;
        dynamic r = doc.selection.createRange();
        r.pasteHTML("<a href='http://bing.com'>bing</a>");
      });
    }
  }
}
```
