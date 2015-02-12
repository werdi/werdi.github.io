---
title: Reading value of element from CDATA
tags: [c#, winforms, linq, webbrowser, interop, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/67b15733-d81c-4246-b45d-3871868abfc7/reading-value-of-element-from-cdata?forum=xmlandnetfx
---
# Reading value of element from CDATA
*> I would like to get the value of admin which is inside CDATA, but I am unable to read the element using Xpath.*
```c#
using System.Windows.Forms;
using System.Xml.Linq;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var xml = @"<?xml version = '1.0' encoding = 'UTF-8'?>
        <result final = 'true' transaction-id='84WO' xmlns='http://cp.com/rules/client'>
          <client id = 'CustName'>
            <quoteback />
          </client>
          <report format = 'CP XML'>
            <![CDATA[<?xml version='1.0' encoding = 'UTF-8' standalone = 'yes'?>
              <personal_auto xmlns = 'http://cp.com/rules/client'>
              <admin>User1</admin>
              <report>Good</report>
              </personal_auto>
            ]]>
          </report> 
        </result>";

      var res = GetAdmin(xml);
      System.Diagnostics.Trace.WriteLine(res);
    }

    string GetAdmin(string xml)
    {
      var x1 = XElement.Parse(xml);
      var r1 = x1.Element("{http://cp.com/rules/client}report").Value;
      var x2 = XElement.Parse(r1);
      return x2.Element("{http://cp.com/rules/client}admin").Value;
    }
  }
}
```
*> But my xml message is not one time. I will be getting lot of xml messages and for each xml i need to extract admin*

use the GetAdmin Method above in your project.

*> I will be getting lot of xml messages and for each xml i need to extract admin and notify him for ACK.*

if you need an editor+extractor, then below is an example
```c#
using System;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var xml = @"
        <?xml version = '1.0' encoding = 'UTF-8'?>
        <result final = 'true' transaction-id='84WO' xmlns = 'http://cp.com/rules/client'>
          <client id = 'CustName'><quoteback /></client>
          <report format = 'CP XML'>
            <![CDATA[
            <?xml version='1.0' encoding = 'UTF-8' standalone = 'yes'?>
              <personal_auto xmlns = 'http://cp.com/rules/client'>
                <admin>User1</admin>
                <report>Good</report>
              </personal_auto>
            ]]>
          </report> 
        </result>";
      this.Size = new Size(500, 300);
      var rtb = new RichTextBox
      {
        Parent = this,
        Dock = DockStyle.Fill,
        Height = 250,
        Text = xml
      };
      var lbl = new Label
      {
        Parent = this,
        Dock = DockStyle.Bottom,
        Height = 35,
        TextAlign = ContentAlignment.MiddleLeft,
        BackColor = Color.FromKnownColor(KnownColor.Info)
      };
      rtb.TextChanged += (s, e) => lbl.Text = "";
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Run (F5)", (s, e) => 
        this.GetValue(rtb.Text, obj => lbl.Text = "" + obj))
        .Shortcut = Shortcut.F5;
  }

  void GetValue(string text, Action<object> callback)
  {
    var js = @"
    <script language='javascript'>
      function addNs(xml, prefix, ns)
      {
        xml.XMLDocument.setProperty('SelectionLanguage', 'XPath');
        xml.XMLDocument.setProperty('SelectionNamespaces', 'xmlns:' + prefix + '=""' + ns + '""');
      }
      function createXml(xml)
      {
        document.body.innerHTML += ""<xml>"" + xml.replace(/<\?.+?\?>/, '') + ""</xml>"";
        return document.body.lastChild;
      }
      function run(xml)
      {
        var _result = createXml(xml);
        addNs(_result, 'x', 'http://cp.com/rules/client');
        var xr = _result.XMLDocument.selectSingleNode('//x:report');
        var _report = createXml(xr.text);
        addNs(_report, 'x', 'http://cp.com/rules/client');
        return _report.XMLDocument.selectSingleNode('//x:admin').text;
      }
    </script>";
    new WebBrowser { DocumentText = "<html><head>" + js + "</head><body /></html>" }
      .DocumentCompleted += (s, e) =>
        callback((s as WebBrowser).Document.InvokeScript("run", new[] { text }));
    }
  }
}
```