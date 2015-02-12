---
title: Передача деталей исключения
tags: [c#, winforms, silverlight, web service]
src: https://social.msdn.microsoft.com/Forums/ru-RU/429bca36-1af8-4dfe-a3a2-c7da6e312c19/-?forum=aspnetru
---
# Передача деталей исключения
*> ошибку я отлавливаю и мне нужно дополнить ее кучей дополнительной информации, например какие параметры в метод предавались и т.д. Как передать все это дело на клиента (Silverlight)?*

как вариант: через out; см. public string Test(out ErrorInfo error);
 ниже пример, который создает хост сервиса в форме. 
в форме также создается браузер, в который загружается .xap-файл (silverlight приложение).

[Silverlight - client]
```xaml
<UserControl x:Class="SilverlightApplication1.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16">
  <StackPanel>
    <Button Content="Test" Click="Test_Click" VerticalAlignment="Top" HorizontalAlignment="Left" />
    <TextBlock x:Name="tb" />
  </StackPanel>
</UserControl>
```
```c#
using System;
using System.Net;
using System.Windows.Controls;

namespace SilverlightApplication1
{
  public partial class MainPage : UserControl
  {
    public MainPage()
    {
      InitializeComponent();
    }
    private void Test_Click(object sender, System.Windows.RoutedEventArgs e)
    {
      var c = new WebClient();
      c.UploadStringCompleted += (_s, _e) => 
        tb.Text = _e.Error == null ? _e.Result : _e.Error.Message + "\n" + _e.Error.InnerException;
      c.UploadStringAsync(new Uri("http://localhost:8845/test/"), "data=123");
    }
  }
}
``` 
[WinForm - server] 
```c#
using System;
using System.Drawing;
using System.IO;
using System.Runtime.Serialization;
using System.ServiceModel;
using System.ServiceModel.Web;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{ 
  public partial class Form1 : Form
  {
    public Form1()
    {
      var url = "http://localhost:8845";
      var h = new WebServiceHost(new Service(), new Uri(url));
      h.Open();
      this.FormClosing += (s, e) => h.Close();
      this.Size = new Size(800, 300);
      new WebBrowser { Parent = this, Dock = DockStyle.Fill, }
        .Navigate(url);
    }
  }

  [DataContract]
  public class ErrorInfo
  {
    [DataMember]
    public string Message { get; set; }
    [DataMember]
    public int Type { get; set; }
  }

  [ServiceContract, ServiceBehavior(InstanceContextMode = InstanceContextMode.Single)]
  public class Service
  {
    [OperationContract, WebInvoke(UriTemplate = "/test/", Method = "POST", ResponseFormat = WebMessageFormat.Json, BodyStyle = WebMessageBodyStyle.Wrapped)]
    public string Test(out ErrorInfo error)
    {
      error = null;
      var dt = DateTime.Now;
      if (dt.Ticks % 2 == 0)
        error = new ErrorInfo { Message = "12345", Type = dt.Millisecond };

      return dt.Second + "." + dt.Millisecond;
    }
    [OperationContract, WebGet(UriTemplate = "/")]
    public Stream Load()
    {
      var ret = @"
      <html>
      <head>
        <script type='text/javascript'>function onSilverlightError(sender, args) { alert(args.ErrorMessage); }</script>
      </head>
      <body>
        <object data='data:application/x-silverlight-2,' type='application/x-silverlight-2' width='100%' height='100%'>
          <param name='source' value='app.xap'/>
          <param name='onError' value='onSilverlightError' />
        </object>
      </body>
      </html>";
      WebOperationContext.Current.OutgoingResponse.ContentType = "text/html";
      return ToStream(ret);
    }
    [OperationContract, WebGet(UriTemplate = "/{app}")]
    public Stream App(string app)
    {
      return File.OpenRead("SilverlightApplication1.xap"); // путь: "\bin\Debug\"
    }
    static Stream ToStream(string s)
    {
      var ret = new MemoryStream();
      var bytes = System.Text.Encoding.Default.GetBytes(s);
      ret.Write(bytes, 0, bytes.Length);
      ret.Position = 0L;
      return ret;
    }
  }
}
```