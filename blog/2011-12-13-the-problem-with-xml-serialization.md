---
title: Проблема с XML сериализацией
tags: [c#, wpf, xml, serialization]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ff4629b7-a6c7-4322-a6a4-49845835439a/-xml-?forum=fordataru
---
# Проблема с XML сериализацией
*> сериализует только публичные свойства и поля, но у меня есть приватное поле которое хранит значение и выдает его через свойство с методом доступа только GET*

приватные поля можно сериализовать, если реализовать интерфейс ISerializable.
```c#
using System;
using System.IO;
using System.Runtime.Serialization;
using System.Windows;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      var t1 = new Test(12345);
      var xs = new NetDataContractSerializer(); //или new DataContractSerializer(typeof(Test));
      var ms = new MemoryStream();
      xs.Serialize(ms, t1); //или xs.WriteObject(ms, t1);
      ms.Flush();
      ms.Position = 0;
      var str = System.Text.Encoding.Default.GetString(ms.ToArray());
      System.Diagnostics.Trace.WriteLine(str);

      var t2 = (Test)xs.Deserialize(ms); //или (Test)xs.ReadObject(ms);
      System.Diagnostics.Trace.WriteLine(t2);
    }
  }

  [Serializable]
  public class Test : System.Runtime.Serialization.ISerializable
  {
    public int Data { get; private set; }

    public Test(int data)
    {
      Data = data;
    }
    protected Test(SerializationInfo info, StreamingContext context)
    {
      this.Data = info.GetInt32("Data");
    }
    public void GetObjectData(SerializationInfo info, StreamingContext context)
    {
      info.AddValue("Data", this.Data);
    }
    public override string ToString()
    {
      return "{" + this.Data + "}";
    }
  }
}
```  
результат сериализации:
```xml
<Test z:Id="1" 
   z:Type="WpfApplication9.Test" 
   z:Assembly="WpfApplication9, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" 
   xmlns="http://schemas.datacontract.org/2004/07/WpfApplication9" 
   xmlns:i="http://www.w3.org/2001/XMLSchema-instance" 
   xmlns:x="http://www.w3.org/2001/XMLSchema" 
   xmlns:z="http://schemas.microsoft.com/2003/10/Serialization/">
   <Data z:Id="2" z:Type="System.Int32" z:Assembly="0" xmlns="">12345</Data>
</Test>
```
или
```xml
<Test
  xmlns="http://schemas.datacontract.org/2004/07/WpfApplication9"
  xmlns:i="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:x="http://www.w3.org/2001/XMLSchema">
  <Data i:type="x:int" xmlns="">12345</Data>
</Test>
```
