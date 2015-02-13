---
title: WCF REST серилизация DataTable
tags: [c#, winforms, linq, rest, web service]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b317d235-0570-4d4e-b87e-04a771b4f022/wcf-rest-datatable?forum=aspnetru
---
# WCF REST серилизация DataTable
*> Хочу сделать сервисы WCF REST [...] что бы DataTable сериализовывался в понятный XML (и затем в JSON)*

чтобы вернуть JSON надо указать WebMessageFormat.Json и вернуть все DataRow.ItemArray.
 (для компиляции: в свойствах проекта указать Target Framework = .NET Framework 4;
 и подключить сборки System.ServiceModel.Web.dll и System.ServiceModel.dll)
```c#
using System;
using System.Data;
using System.Net;
using System.ServiceModel;
using System.ServiceModel.Web;
using System.Windows.Forms;
using System.Linq;
using System.Collections.Generic;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var url = new Uri("http://localhost:8845");
      var host = new WebServiceHost(typeof(Service), url);
      host.Open();
      this.FormClosing += (s, e) => host.Close();

      this.Size = new System.Drawing.Size(800, 400);
      var wb = new WebBrowser { Parent = this, Dock = DockStyle.Fill };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("xml", (s, e) => wb.Navigate(url + "/x"));
      this.Menu.MenuItems.Add("json", delegate
      {
        var c = new WebClient();
        c.DownloadStringCompleted += (s, e) => wb.DocumentText = e.Result;
        c.DownloadStringAsync(new Uri(url, "/j"));
      });
    }
  }

  [ServiceContract, ServiceBehavior(InstanceContextMode = InstanceContextMode.Single)]
  public class Service
  {
    DataSet CreateData()
    {
      var dt = new DataTable();
      dt.Columns.Add("Id", typeof(int));
      dt.Columns.Add("Text", typeof(string));
      for (int i = 0; i < 5; i++) dt.Rows.Add(i, "txt"+i);
      var ds = new DataSet();
      ds.Tables.Add(dt);
      ds.AcceptChanges();
      return ds;
    }
    [OperationContract, WebGet(UriTemplate = "/x/", ResponseFormat = WebMessageFormat.Xml)]
    public DataSet Xml()
    {
      return CreateData();
    }
    [OperationContract, WebGet(UriTemplate = "/j/", ResponseFormat = WebMessageFormat.Json)]
    public IEnumerable<object[]> Json()
    {
      return CreateData().Tables[0].Rows.OfType<DataRow>().Select(r => r.ItemArray);
    }
  }
}
```