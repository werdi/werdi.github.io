---
title: XmlTextReader
tags: [c#, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/8bb3a8f6-b7f3-422b-a4eb-fd637e6b66f9/xmltextreader?forum=fordataru
---
# XmlTextReader
*> ... (имеется ли вложенность) c помощью XmlTextReader*

см. [XmlTextReader.Depth](http://msdn.microsoft.com/ru-ru/library/system.xml.xmltextreader.depth.aspx)

*> Не понимаю, он мне выдает 1 когда я нахожусь на таком элементе(object name="Hero"): <object name="Hero" x="1425" y="499" a="0" s="1" f="0" z="0"> [...] что на таком(object name="level1_land03"):*

если теги на одном уровне вложенности, то Depth равны; но LineNumber отличаются.
```c#

using System;
using System.IO;
using System.Xml;

namespace ConsoleApplication1
{
  class Program
  {
      [STAThread]
    static void Main(string[] args)
    {
      var xml = @"
        <root>
          <item1>
            <item2 />
          </item1>
          <item3 />
        </root>";
      var xr = new XmlTextReader(new StringReader(xml));
      xr.MoveToContent();
      Trace(xr);
      while (xr.Read())
        Trace(xr);
    }
    static void Trace(XmlTextReader xr)
    {
      if(xr.NodeType != XmlNodeType.Element) return;
      var str = String.Concat(xr.LocalName, " depth=", xr.Depth, " line=", xr.LineNumber, " empty=", xr.IsEmptyElement);
      System.Diagnostics.Trace.WriteLine(str);
    }
  }
}
```
результат: 
```
root depth=0 line=2 empty=False
item1 depth=1 line=3 empty=False
item2 depth=2 line=4 empty=True
item3 depth=1 line=6 empty=True
```