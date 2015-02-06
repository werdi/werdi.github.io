---
title: WebClient и POST запрос
tags: [c#, xaml, wpf, webclient]
src: https://social.msdn.microsoft.com/Forums/ru-RU/6dd84bf0-e2ce-4714-abde-01fcd9679de9/webclient-post-?forum=formobiledevicesru
---
# WebClient и POST запрос
*> Ошибка 3 "System.Net.WebClient" не содержит определения для "UploadValues"*

вместо List должен быть NameValueCollection.
примерно так:
```c#
using System;
using System.Net;
using System.Collections.Specialized;
...

var w = new WebClient();
var c = new NameValueCollection();
c.Add("k1", "v1");
c.Add("k2", "v2");
w.UploadValues("http://site.net/", c)   // отправит: k1=v1&k2=v2
```
*> как перевести строку такого вида :[{"id":1,"name":"Бабслет", [...] в нормальный вид(масив или что то в это роде).*

см. [Working with JSON Data] (http://msdn.microsoft.com/en-us/library/cc197957(v=vs.95).aspx); 
[Making use of your JSON data in Silverlight] (http://timheuer.com/blog/archive/2008/05/06/use-json-data-in-silverlight.aspx)

*> Я пытался сделать так но все равно что то не то 8(*

пример сериализации коллекции объектов в json.
строка отсылается на сервер, с которого загружено silverlight app.
если требуется отсылать на другой сервер, то надо создать crossdomain.xml и clientaccesspolicy.xml
```xaml
<UserControl
	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
	x:Class="SilverlightApplication4.MainPage"
	Width="640" Height="480">
	<Grid x:Name="LayoutRoot" Background="White">
		<Button Content="Test" x:Name="btn" Click="Test_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
		<TextBox x:Name="tst" Margin="0,30,0,0" ScrollViewer.VerticalScrollBarVisibility="Visible" TextWrapping="Wrap" />
	</Grid>	
</UserControl>
```
```c#
using System;
using System.Net;
using System.Windows;
using System.Windows.Controls;
using System.Collections.Generic;
using System.Text;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Json;
using System.IO;
using System.Threading;

namespace SilverlightApplication4
{
	public class RestParametr
	{
		public string Name { get; set; }
	}
	
	[CollectionDataContract]
	public class RestParametrs : List<RestParametr>
	{
	}
	
	public partial class MainPage : UserControl
	{
		public MainPage()
		{
			InitializeComponent();
		}
		private void Test_Click(object sender, RoutedEventArgs e)
		{
			var url = Application.Current.Host.Source.GetComponents(
				UriComponents.SchemeAndServer, UriFormat.UriEscaped) + "/TestPage.html";
			tst.Text = "uploading...\n" + url;
			
			var list = new RestParametrs();
			list.Add(new RestParametr { Name = "n1" });
			list.Add(new RestParametr { Name = "n2" });
			
			string str = null;
			try
			{
				var jsr = new DataContractJsonSerializer(typeof(RestParametrs));
 				var ms = new MemoryStream();
 				jsr.WriteObject(ms, list);
 				str = Encoding.UTF8.GetString(ms.ToArray(), 0, (int)ms.Length);
				tst.Text += "\n" + str;
			}
			catch(Exception exc)
			{
				tst.Text = exc.ToString();
			}
			
			var w = new WebClient();
			w.Headers["Content-type"] = "application/json";
 			w.Encoding = System.Text.Encoding.UTF8;
			w.UploadProgressChanged += _UploadProgressChanged;
			w.UploadStringCompleted += _UploadStringCompleted;
			w.UploadStringAsync(new Uri(url), str);
		}

		private void _UploadProgressChanged(object sender, UploadProgressChangedEventArgs e)
		{
			tst.Text += "\n" + e.BytesSent + " " + e.BytesReceived;
		}
		private void _UploadStringCompleted(object sender, UploadStringCompletedEventArgs e)
		{
			tst.Text += "\n" + e.Result;
		}
	}
}
