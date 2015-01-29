---
title: Как организовать доступ к данным?
tags: [data access, c#, web-service, html, json, sqlce, sql server compact]
src: https://social.msdn.microsoft.com/Forums/ru-RU/bf8871c1-0cd2-4cad-a80e-32d1083a1dd0/-?forum=fordataru
---
# Как организовать доступ к данным?
пример сервиса и клиентов (браузер,html+javascript)
```c# 
using System;
using System.ServiceModel.Web;
using System.ServiceModel;
using System.IO;

static class Program
{
  [STAThread]
  static void Main()
  {
    // создать и открыть хост сервиса
    var host = new WebServiceHost(new Service(), new Uri(Service.Url));
    host.Open();
    // открыть IE и загрузить в него UI; см. ниже Service.Load
    System.Diagnostics.Process.Start(Service.Url);
    while(true);
  }
}

// сервис
[ServiceContract, ServiceBehavior(InstanceContextMode=InstanceContextMode.Single)]
public class Service
{
  public const string Url = "http://localhost:8899";
  [OperationContract, WebGet(UriTemplate = "/")]
  public Stream Load()
  {
    // создать html (можно загружать из файла)
    var ret = @"<body>
            <script language='javascript'>
              function getsum(trg) {
                // вызвать на сервере метод Service.GetSum; результат вывести на страницу в браузере.
                var res = document.getElementById(trg)
                res.innerHTML += '<br/>sending...' 
                var x = new XMLHttpRequest();
                x.open('GET', '/Sum/?'+Math.random(), false);    // false-sync; без random() кэшируются
                x.send();
                res.innerHTML += 'result: ' + x.responseText; } </script>
            <button onclick='getsum(""sum"")'>sum</button><div id='sum'></div></body>";
    WebOperationContext.Current.OutgoingResponse.ContentType = "text/html";
    return ret.ToStream();   // метод расширения. см. ниже в классе StringHelper
  }
  [OperationContract, WebGet(UriTemplate = "/Sum/", ResponseFormat = WebMessageFormat.Xml)]
  public long GetSum() { return count++; }
  private static int count = 0;
}
public static class StringHelper
{
  public static Stream ToStream(this string s)
  {
    var ret = new MemoryStream();
    var bytes = System.Text.Encoding.Default.GetBytes(s);
    ret.Write(bytes, 0, bytes.Length);
    ret.Position = 0L;
    return ret;
  }
}
```
запускать пример надо в VS (run as administrator). или указать разрешения с помощью netsh. см. http://blogs.msdn.com/b/amitlale/archive/2007/01/29/addressaccessdeniedexception-cause-and-solution.aspx
пример использования SQL Server Compact (дополнения к предыдущему примеру).
 
ниже код клиента и сервиса:
метод Add: принимает данные от клиента (POST-запрос) и сохраняет данные в базе SQL Server Compact (создается в c:\temp\test.sdf).
метод Count: выдает количество строк хранящихся в таблице в базе данных.
```c#
// сервис
[ServiceContract, ServiceBehavior(InstanceContextMode = InstanceContextMode.Single)]
public class Service
{
  public const string Url = "http://localhost:8899";
  TestStore Store { get; set; }
  public Service()
  {
    Directory.CreateDirectory(@"c:\temp");
    this.Store = new TestStore(new SqlCeConnection(@"Data Source=C:\Temp\Test.sdf"));
    this.Store.Database.CreateIfNotExists();
  }
  [OperationContract, WebGet(UriTemplate = "/")]
  public Stream Load()
  {
    // создать html (можно загружать из файла)
    var ret = @"<body>
        <script language='javascript'>
        function add(inp1, inp2, trg) { // можно сократить, если использовать JQuery
          var res = document.getElementById(trg);
          var val = document.getElementById(inp1).value
          var com = document.getElementById(inp2).value
          var x = new XMLHttpRequest();
          x.open('POST', '" + Url + @"/Add/', false);
          x.setRequestHeader('Content-Type', 'application/json');
          x.send('{ ""Value"": '+val+', ""Comment"": ""'+com+'"" }');
          res.innerHTML += '<br/>result: ' + x.responseText; } 
        </script>
        value:<input type='text' id='val' value='0' />
        comment: <input type='text' id='com' /><button onclick=""add('val', 'com', 'res')"">add</button>
        <div id='res'><div><a href='/count'>count</a></body>";
    WebOperationContext.Current.OutgoingResponse.ContentType = "text/html";
    return ret.ToStream();  // метод расширения. см. выше в классе StringHelper
  }
  [OperationContract, WebGet(UriTemplate = "/Count/", ResponseFormat = WebMessageFormat.Xml)]
  public string Count()
  {
    return this.Store.Tests.Count().ToString();
  }
  [OperationContract, WebInvoke(UriTemplate = "/Add/", RequestFormat = WebMessageFormat.Json)]
  public string Add(Test data)
  {
    this.Store.Tests.Add(data);
    this.Store.SaveChanges();
    return "ok: " + data.ToString();
  }
}
[Table("Tests"), DataContract]
public class Test
{
  [Key, DatabaseGenerated(DatabaseGeneratedOption.Identity), Column(TypeName = "bigint")]
  public int Id { get; set; }
  [DataMember]
  public int Value { get; set; }
  [DataMember]
  public string Comment { get; set; }
  public override string ToString() { return String.Concat("{Id=", this.Id, " Value=", this.Value, " Comment=", this.Comment, "}"); }
}
class TestStore : DbContext
{
  public TestStore(SqlCeConnection c) : base(c, true) { }
  public DbSet<Test> Tests { get; set; }
}
```
для компиляции примера надо подключить сборки: 
* C:\Program Files\Microsoft ADO.NET Entity Framework 4.1\Binaries\EntityFramework.dll
* C:\Windows\Microsoft.NET\Framework\v4.0.30319\System.ComponentModel.DataAnnotations.dll
* C:\Program Files\Microsoft SQL Server Compact Edition\v4.0\Desktop\System.Data.SqlServerCe.dll
* C:\Windows\Microsoft.NET\Framework\v4.0.30319\System.Data.Entity.dll
* C:\Windows\Microsoft.NET\Framework\v4.0.30319\System.Runtime.Serialization.dll
