---
title: New Article on Regex speed versus XML processing
tags: []
src: https://social.msdn.microsoft.com/Forums/ru-RU/abb83c19-34c0-41a6-b2d8-80effae30881/new-article-on-regex-speed-versus-xml-processing?forum=regexp
---
# New Article on Regex speed versus XML processing
*> I have written another article on regex speed which was based on a question asked in the C# forum..Net Regex: Can Regular Expression Parsing be Faster than XmlDocument or Linq to Xml?*

i have tested Regex, XmlDocument, XElement and have got the following results:
``` 
{ XElement = 75, Regex = 42, XmlDocument = 37 }	best: XmlDocument
{ XElement = 37, Regex = 33, XmlDocument = 31 }	best: XmlDocument
{ XElement = 35, Regex = 34, XmlDocument = 31 }	best: XmlDocument
{ XElement = 34, Regex = 34, XmlDocument = 31 }	best: XmlDocument
{ XElement = 34, Regex = 33, XmlDocument = 31 }	best: XmlDocument
{ XElement = 43, Regex = 36, XmlDocument = 32 }	best: XmlDocument
{ XElement = 33, Regex = 33, XmlDocument = 31 }	best: XmlDocument
{ XElement = 36, Regex = 37, XmlDocument = 33 }	best: XmlDocument
{ XElement = 36, Regex = 34, XmlDocument = 32 }	best: XmlDocument
{ XElement = 33, Regex = 34, XmlDocument = 31 }	best: XmlDocument
{ XElement = 35, Regex = 33, XmlDocument = 33 }	best: Regex
{ XElement = 34, Regex = 33, XmlDocument = 31 }	best: XmlDocument
{ XElement = 33, Regex = 32, XmlDocument = 30 }	best: XmlDocument
{ XElement = 34, Regex = 33, XmlDocument = 30 }	best: XmlDocument
{ XElement = 33, Regex = 33, XmlDocument = 32 }	best: XmlDocument
{ XElement = 34, Regex = 37, XmlDocument = 30 }	best: XmlDocument
{ XElement = 33, Regex = 32, XmlDocument = 31 }	best: XmlDocument
{ XElement = 38, Regex = 33, XmlDocument = 30 }	best: XmlDocument
{ XElement = 33, Regex = 32, XmlDocument = 30 }	best: XmlDocument
{ XElement = 32, Regex = 33, XmlDocument = 30 }	best: XmlDocument
{ XElement = 33, Regex = 33, XmlDocument = 31 }	best: XmlDocument
{ XElement = 34, Regex = 35, XmlDocument = 30 }	best: XmlDocument
{ XElement = 33, Regex = 33, XmlDocument = 30 }	best: XmlDocument
{ XElement = 34, Regex = 33, XmlDocument = 31 }	best: XmlDocument
{ XElement = 33, Regex = 32, XmlDocument = 30 }	best: XmlDocument
{ XElement = 35, Regex = 33, XmlDocument = 30 }	best: XmlDocument
{ XElement = 33, Regex = 33, XmlDocument = 33 }	best: XElement
{ XElement = 53, Regex = 34, XmlDocument = 30 }	best: XmlDocument
{ XElement = 33, Regex = 32, XmlDocument = 30 }	best: XmlDocument
{ XElement = 32, Regex = 32, XmlDocument = 30 }	best: XmlDocument 
```
below is a test code
```c#
using System;
using System.Diagnostics;
using System.IO;
using System.Linq;
using System.Text.RegularExpressions;
using System.Xml;
using System.Xml.Linq;

namespace WindowsFormsApplication4
{
  static class Program
  {
    [STAThread]
    static void Main()
    {
      var di = new DirectoryInfo("test");
      if (di.Exists == false)
      {
        di.Create();
        var xml = @"<?xml version=""1.0"" encoding=""UTF-8""?>
        <urlset xmlns=""http://www.sitemaps.org/schemas/sitemap/0.9"">" +
          String
              .Concat(Enumerable.Range(0, 100)
                .Select(c =>
                  "<url><loc>http://localhost:" + c + "</loc></url>")) +
            "</urlset>";
        for (int i = 0; i < 100; i++)
          File.WriteAllText(Path.Combine(di.FullName, "test" + i + ".xml"), xml);
      }
      var res = Enumerable.Range(0, 30).Select(c => new
      {
          XElement = TestXElement(di),
          Regex = TestRegex(di),
          XmlDocument = TestXmlDocument(di)
      });
      foreach (var c in res)
      {
          var best = c.GetType()
              .GetProperties()
              .Select(p => new { Name = p.Name, Value = (long)p.GetValue(c, null) })
              .OrderBy(pv => pv.Value)
              .First();
          System.Diagnostics.Trace.WriteLine(c + "\tbest: " + best.Name);
      }
    }

    static long TestXElement(DirectoryInfo di)
    {
      var s = Stopwatch.StartNew();
      foreach (var file in di.EnumerateFiles())
      {
        var xe = XElement.Load(file.FullName);
        var c = xe.Elements().Count();
      }
      s.Stop();
      return s.ElapsedMilliseconds;
    }

    static long TestXmlDocument(DirectoryInfo di)
    {
      var s = Stopwatch.StartNew();
      foreach (var file in di.EnumerateFiles())
      {
        var xe = new XmlDocument();
        xe.Load(file.FullName);
        var c = xe.DocumentElement.ChildNodes.Count;
      }
      s.Stop();
      return s.ElapsedMilliseconds;
    }

    static long TestRegex(DirectoryInfo di)
    {
      var s = Stopwatch.StartNew();
      var re = new Regex(@"<url>\s*<loc>", RegexOptions.Multiline | RegexOptions.IgnoreCase);
      foreach (var file in di.EnumerateFiles())
      {
        var txt = File.ReadAllText(file.FullName);
        var c = re.Matches(txt).Count;
      }
      s.Stop();
      return s.ElapsedMilliseconds;
    }
  }
}
```
 
please note that XmlDocument has been loaded through the method .Load(file.FullName) instead of .LoadXml(File.ReadAllText(file.FullName)) 