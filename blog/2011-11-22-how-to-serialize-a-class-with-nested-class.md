---
title: How to serialize a class with nested class
tags: [c#, xml, serialization]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1b0b9a6e-5af1-4e84-aa8d-5db731adba93/how-to-serialize-a-class-with-nested-class?forum=csharplanguage
---
# How to serialize a class with nested class
*> but after serializing, I got a xml file which all the properties I want to serialize as an element of the xml been serialized as attribute. What was worse, the List<string> hobby and Address property can't be serialized.*

use "public Address Address" instead of "internal Address Address"
```c#
using System;
using System.Collections.Generic;
using System.Web.Script.Serialization;
using System.IO;
using System.Xml.Serialization;

namespace ConsoleApplication1
{
  class Program
  {
    [STAThread]
    static void Main(string[] args)
    {
      var list = new List<Person> 
      {
        new Person {  
          Id = 1, Name = "p1", Age = 20, Sex = true,  
          Address = new Address { CompanyAddress = "ca1", HomeAddress = "ha1" },
          Hobby = new List<string> { "h1", "h2" }
        },
        new Person {  
          Id = 2, Name = "p2", Age = 20, Sex = true,  
          Address = new Address { CompanyAddress = "ca2", HomeAddress = "ha2" },
          Hobby = new List<string> { "h3", "h3" }
        },
      };

      // to xml
      var xs = new XmlSerializer(typeof(List<Person>));
      var sw = new StringWriter();
      xs.Serialize(sw, list);
      Console.WriteLine(sw.ToString());

      // to json 
      // 1) add references to System.Web.Extensions.dll; 
      // 2) the Project Properties - Application - 'Target framework' should be '.Net Framework 4'
      var json = new JavaScriptSerializer().Serialize(list);
      Console.WriteLine("\n\n" + json);
      
      Console.ReadLine();
    }
  }

  public class Person
  {
    public int Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public bool Sex { get; set; }
    public Address Address { get; set; }
    public List<string> Hobby { get; set; }
  }
  public class Address
  {
    public string CompanyAddress { get; set; }
    public string HomeAddress { get; set; }
  }
}
```
*> internal Address Address {...}   [...] Address property can't be serialized.*

for the non-public properties serialization use DataContractSerializer http://stackoverflow.com/q/420710