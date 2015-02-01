---
title: How to read specific text from html
tags: [c#, html, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/0ffab121-570e-4b94-8bc6-0fe86c1634fb/how-to-read-specific-text-from-html?forum=csharpgeneral
---
# How to read specific text from html
for html parsing you can use HTMLDocument (one of the IE components).

here is a sample:
```c#
using System;
using System.Runtime.InteropServices;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var html = @"<h1 class='Review'></h1><div class='customer_cmts' style='overflow-x: hidden; overflow-y: scroll; height: 147px;'>
        <ul> <li style='margin-bottom:8px;'><a href='/tripbox/reviews/travel.aspx' class='more'>It was an nice experience 
        fly with ethihad airways<br />thanks alon k larry</a></li></ul></div>";
      var txt = Helper.GetCustomerText(html);
      System.Diagnostics.Trace.WriteLine(txt);
    }
  }

  public class Helper
  {
    [ComImport, Guid("25336920-03F9-11CF-8FD0-00AA00686F13")]
    class HTMLDocument { }

    public static string GetCustomerText(string html)
    {
      dynamic doc = new HTMLDocument();
      doc.write(html);
      doc.close();
      dynamic  tags = doc.getElementsByTagName("DIV");
      for(var i=0; i < (int) tags.length; i++)
      {
        dynamic tag = tags[i];
        if(string.Equals(tag.className, "customer_cmts", StringComparison.OrdinalIgnoreCase))
          return System.Text.RegularExpressions.Regex.Replace((string) tag.innerText, "\\s{2,}", " ");
      }
      return null;
    }
  }
}
```
