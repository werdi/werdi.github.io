---
title: HTML decoding
tags: [c#, html]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8c89b2db-ecc7-4430-9cd3-ade7f9f36223/html-decoding?forum=csharpgeneral
---
# HTML decoding
*> How to decode the html into normal string? I'm using HttpUtility.HTMLdecode() method, buyt it is giving me string with html tags*

the following method works even for invalid html
```c#
using System.Windows.Forms;
...
var txt = GetText("<b><div>text1<i>text2<ul><li>item1<li>item2");
...
string GetText(string html)
{
  var wb = new WebBrowser();
  wb.DocumentText = html;
  while(wb.ReadyState != WebBrowserReadyState.Complete) 
      Application.DoEvents();
  return wb.Document.Body.InnerText;
}
```
*> It is giving me error of some active x control*

you didn't say, that you have activex in html. ok. try the following code.
if not help, then cut from the html all of the <object> tags before GetText using.
```c#
using System.Runtime.InteropServices;
...
var txt = Html.GetText("<b><div>text1<i>text2<ul><li>item1<li>item2");
...
public class Html
{
  [ComImport, Guid("25336920-03F9-11CF-8FD0-00AA00686F13")]
  class HTMLDocument { }

  public static string GetText(string html)
  {
    dynamic doc = new HTMLDocument();
    doc.write(html);
    doc.close();
    return doc.body.innerText;
  }
}
```
*> the HTML Agility Pack library is really powerful and easy to use. [...] don't forget to download the library and add reference to it.*

HTML Agility Pack is a great library, but it's redundant.
as mentioned above, html parsing can be done using HTMLDocument (this is a component of Internet Explorer).