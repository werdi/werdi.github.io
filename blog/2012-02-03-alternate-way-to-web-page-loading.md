---
title: Alternate way to allow a web page to load
tags: [c#, web, winforms, html]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8c714806-db02-4fcc-8a94-4a52506f1fc4/alternate-way-to-allow-a-web-page-to-load?forum=csharpgeneral
---
# Alternate way to allow a web page to load
*> I have recenty been experimenting with website automation. I have been waiting for the web page to load when I navigate to it by using Thread.sleep(5000); but sometimes this doesn't work.*
```c#
using System.Data;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(900, 500);
      var wb = new WebBrowser { Parent = this, Dock = DockStyle.Fill };
      wb.Navigate("http://bing.com");

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Run", delegate 
      {
        var input = GetElementById(wb, "sb_form_q");
        var go = GetElementById(wb, "sb_form_go");
        input.value = "malobukv";
        go.click();

        // a second page processing
        input = GetElementById(wb, "sb_form_q");
        input.value = "malobukv's threads";
        go = GetElementById(wb, "sb_form_go");
        go.click();
      });
    }
    static dynamic GetElementById(WebBrowser wb, string id)
    {
      var doc = WaitDocument(wb);
      var m = doc.GetType().GetMethod("getElementById");
      return m.Invoke(doc, new object[] { id });
    }
    static dynamic WaitDocument(WebBrowser wb)
    {
      dynamic doc = wb.Document.DomDocument;
      var rsp = doc.GetType().GetProperty("readyState");
      while (true)
      {
        var rs = rsp.GetValue(doc, null) as string;
        if (rs == "complete")
          return doc;
        Application.DoEvents();
      }
    }
  }
}
```
instead of WebBrowser you can automate IE. an example is [here] (http://social.msdn.microsoft.com/Forums/en-us/csharpgeneral/thread/89dbc8b2-df69-4afb-9813-acee113c20ea/#4f88e5ed-22b9-4ead-9db3-2a4dfcf023ad).

there is another approach based on the UI Automation. an example is [here] (http://social.msdn.microsoft.com/Forums/en-us/csharpgeneral/thread/d4d8655f-4cb2-42ef-9966-8c1cf8ce2e25/#6228adf9-a0dd-432a-98aa-be17a31dba91).
