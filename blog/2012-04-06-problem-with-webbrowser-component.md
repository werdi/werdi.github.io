---
title: Проблема с компонентом WebBrowser
tags: [c#, winforms, webbrowser, html, xsl]
src: https://social.msdn.microsoft.com/Forums/ru-RU/45d7014d-d66a-4cf2-ac59-9cadd7c6b397/-webbrowser?forum=programminglanguageru
---
# Проблема с компонентом WebBrowser
*> В компоненте WebBrowser всё отображается иначе.*

попробуйте в html добавить 'X-UA-Compatible'. должен быть примерно такой html:
```html
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv='Content-Type' content='text/html; charset=unicode' />
    <meta http-equiv='X-UA-Compatible' content='IE=9' /> 
    ...
```

*> Начало документа XSL у меня такое: <xsl:template match = "/root"><html charset="utf-8">*

т.е. <!DOCTYPE html> не выводится.
DOCTYPE можно вывести в html с помощью xsl:text
```xsl
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform"> 
  <xsl:output method="html" encoding="utf-8" indent="yes" /> 
   <xsl:template match="/"> 
    <xsl:text disable-output-escaping='yes'>&lt;!DOCTYPE html></xsl:text> 
    <html> 
    ... 
    </html> 
  </xsl:template> 
</xsl:stylesheet> 
```

*> StreamReader f = new StreamReader("new.html", enc);webBrowser1.DocumentText = ConvertExtendedAscii(f.ReadToEnd());*

другой способ: XslCompiledTransform.Transform в MemoryStream и его загрузка в WebBrowser.DocumentStream 
```c#
using System.IO;
using System.Windows.Forms;
using System.Xml;
using System.Xml.Xsl;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var xsl = @"
        <xsl:stylesheet version='1.0' xmlns:xsl='http://www.w3.org/1999/XSL/Transform'> 
          <xsl:output method='html' encoding='utf-8' indent='yes' /> 
           <xsl:template match='/'> 
            <xsl:text disable-output-escaping='yes'>&lt;!DOCTYPE html></xsl:text> 
            <html> 
              <head>
                <meta http-equiv='X-UA-Compatible' content='IE=9' /> 
              </head>
              <body>
                hello
              </body>
            </html> 
          </xsl:template> 
        </xsl:stylesheet>";
      var xslt = new XslCompiledTransform();
      xslt.Load(new XmlTextReader(new StringReader(xsl)));
      var xml = new XmlDocument();
      xml.LoadXml(@"<root />");
      var ms = new MemoryStream();
      xslt.Transform(xml.CreateNavigator(), null, ms);
      ms.Position = 0;
      new WebBrowser() { Dock = DockStyle.Fill, Parent = this, DocumentStream = ms };
    }
  }
}
```
