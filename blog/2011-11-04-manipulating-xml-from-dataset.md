---
title: Manipulating XML from Dataset
tags: [c#, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b524ace6-20ef-4115-91c8-c5e5fed82cc7/manipulating-xml-from-dataset?forum=csharpgeneral
---
# Manipulating XML from Dataset
```c#
using System.Collections.Generic;
using System.Linq;
using System.Xml.Linq;
...
IEnumerable<XElement> Parse(string xml)
{
  var xe = XElement.Parse(xml);
  foreach (var tblg in xe.Elements("Table").GroupBy(t => t.Element("TankID").Value))
  {
    var na =  new XElement("TankID", new XAttribute("ID", tblg.Key));
    foreach (var tbl in tblg)
    {
      var ne = 
        new XElement("No", 
          new XAttribute("ID", tbl.Element("No").Value),
          new XElement("PosX", tbl.Element("PosX").Value),
          new XElement("PosY", tbl.Element("PosY").Value)
        );
      na.Add(ne);
    }
    var nt = new XElement("Table", na);
    yield return nt;
  }
}
...
var nxml = Parse(xml).First().ToString(); 
```
for the following xml   
```c#
var xml = @"
          <NewDataSet>
            <Table>
              <TankID>Tank_1</TankID> 
              <No>1</No> 
              <PosX>123</PosX> 
              <PosY>123</PosY> 
            </Table>
            <Table>
              <TankID>Tank_1</TankID> 
              <No>2</No> 
              <PosX>321</PosX> 
              <PosY>456</PosY> 
            </Table>
            <!-- ... -->
          </NewDataSet>";
```
result:
```xml
<Table>
  <TankID ID="Tank_1">
    <No ID="1">
      <PosX>123</PosX>
      <PosY>123</PosY>
    </No>
    <No ID="2">
      <PosX>321</PosX>
      <PosY>456</PosY>
    </No>
  </TankID>
</Table>
```
*> However, there is still more things I forgot to mention. Actually I need to produce a result something like this: ...*

in this case use following code
```c# 
XElement Parse(string xml)
{
  var xe = XElement.Parse(xml);
  var nt = new XElement("Table");
  foreach (var tblg in xe.Elements("Table").GroupBy(t => t.Element("TankID").Value))
  {
    var na = new XElement("TankID", new XAttribute("ID", tblg.Key));
    foreach (var tbl in tblg)
    {
      var ne =
        new XElement("No",
          new XAttribute("ID", tbl.Element("No").Value),
          new XElement("PosX", tbl.Element("PosX").Value),
          new XElement("PosY", tbl.Element("PosY").Value)
        );
      na.Add(ne);
    }
    nt.Add(na);
  }
  return nt;
}
...
var res = Parse(xml).ToString();
```
*> "Data at the root level is invalid" error. [...] <NewDataSet />*

try the following code, it works fine. 
```c#
var v1 = XElement.Parse("<NewDataSet />");
var v2 = XElement.Parse("<NewDataSet></NewDataSet>");
var v3 = XElement.Parse("<NewDataSet><some id='1' /></NewDataSet>");
```
