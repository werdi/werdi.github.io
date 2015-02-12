---
title: Header attributes
tags: [c#, winforms, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d62a5efc-1718-49d3-a6e7-6b82c55512a5/header-attributes?forum=xmlandnetfx
---
# Header attributes
*> I am trying to extrac all the attributes from a xml file. [...] What I want are the following attributes: TransactionTypeCode="1" ServicerNumber="xxxx187" ServicerLoanNumber="xxx32" ProcessingStatusCode*

below is an example, which extracts all the attributes using ReadAll method.
 and a specified attributes using the Read Method
```c#
using System;
using System.Collections.Generic;
using System.IO;
using System.Text;
using System.Windows.Forms;
using System.Xml;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ret = Read("test.txt", System.Text.Encoding.UTF8,
        "TransactionTypeCode", "ServicerNumber", "ServicerLoanNumber", "ProcessingStatusCode");
      foreach (var kv in ret)
        System.Diagnostics.Trace.WriteLine(kv);

      foreach (var kv in ReadAll("test.txt", System.Text.Encoding.UTF8))
        System.Diagnostics.Trace.WriteLine(kv);
    }

    Dictionary<string, string> Read(string path, Encoding e, params string[] names)
    {
      var ret = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
      foreach (var name in names) ret.Add(name, null);
      var xr = new XmlTextReader(new StreamReader(path, e));
      xr.MoveToContent();
      while (xr.Read())
      {
        if (xr.MoveToFirstAttribute())
        {
          do
            if (ret.ContainsKey(xr.LocalName))
              ret[xr.LocalName] = xr.Value;
          while (xr.MoveToNextAttribute());
          xr.MoveToElement();
        }
      }
      return ret;
    }
    IEnumerable<KeyValuePair<string, string>> ReadAll(string path, Encoding e)
    {
      var xr = new XmlTextReader(new StreamReader(path, e));
      xr.MoveToContent();
      while (xr.Read())
      {
        if (xr.MoveToFirstAttribute())
        {
          do yield return new KeyValuePair<string, string>(xr.LocalName, xr.Value);
          while (xr.MoveToNextAttribute());
          xr.MoveToElement();
        }
      }
    }
  }
}
```