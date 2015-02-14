---
title: Adding New XML element
tags: [c#, winforms, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4539cdeb-edcf-49b0-90d7-15e171ca0b90/adding-new-xml-element?forum=csharpgeneral
---
# Adding New XML element
*> IS there a way to add a new PROJECT element with the same indentation as shown for all the new elements?*

you can clone the project node and insert in the root. below is an example.
```c#
using System.Xml.Linq;
...
var xml = @"<?xml version='1.0' encoding='utf-8'?>
<TOPROOT>
  <Project>
    <Name>Project No.1</Name>
    <Client>Project Client Name</Client>
    <Started>11/1/2011</Started>
    <Description>Project number 1 description</Description>
    <Code>2300</Code>
    <Completed></Completed>
    <Manager>Smith</Manager>
    <Analyst>Jones</Analyst>
  </Project>
</TOPROOT>";
var root = XElement.Parse(xml);
var np = Clone(root.FirstNode);
root.Add(np);
...
XNode Clone(XNode src)
{
  using (var r = src.CreateReader())
  {
    r.MoveToContent();
    return XElement.ReadFrom(r);
  }
}
```
*> Clone doesn't exist. Not much help Malobukv!*

please, take a look more carefully above. there is method XNode Clone(XNode src) { ... }
*> I created the subroutine CLONE, but process failed. Not working like I thought it would.*

try check out your code. and take a look at the following. works fine.
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
      var xml = @"<?xml version='1.0' encoding='utf-8'?>
                  <TOPROOT>
                    <Project>
                      <Name>Project No.1</Name>
                      <Client>Project Client Name</Client>
                      <Started>11/1/2011</Started>
                      <Description>Project number 1 description</Description>
                      <Code>2300</Code>
                      <Completed></Completed>
                      <Manager>Smith</Manager>
                      <Analyst>Jones</Analyst>
                    </Project>
                  </TOPROOT>";

      var root = XElement.Parse(xml);
      var np = Clone(root.FirstNode);
      root.Add(np);

      this.Size = new System.Drawing.Size(600, 600);
      new RichTextBox
      {
        Parent = this,
        Text = root.ToString(),
        Dock = DockStyle.Fill
      };
    }
    XNode Clone(XNode src)
    {
      using (var r = src.CreateReader())
      {
        r.MoveToContent();
        return XElement.ReadFrom(r);
      }
    }
  }
}
```