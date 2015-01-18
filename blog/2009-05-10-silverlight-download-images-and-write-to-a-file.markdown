# Silverlight: загрузка изображений и запись в файл
Ситуация: надо c сервера загрузить изображение, например, Test.png и сохранить в файл, указанный пользователем на своем компьютере. 
Для этого необходимо создать Silverlight приложение. Код MainPage.xaml заменить следующим:
```xaml
<UserControl x:Class="SilverlightApplication5.MainPage"
    	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  	<Button x:Name="Load" Content="Load" Width="100" Height="30" />
</UserControl>
```
Код в MainPage.xaml.cs заменить следующим:
```c#
using System;
using System.Net;
using System.Windows;
using System.Windows.Controls;

namespace SilverlightApplication5
{
    	public partial class MainPage : UserControl
    	{
        	public MainPage()
        	{
            	InitializeComponent();
            	Load.Click += new RoutedEventHandler(Load_Click);
        	}

        	void Load_Click(object sender, RoutedEventArgs e)
        	{
            	UriBuilder ub = new UriBuilder(App.Current.Host.Source);
            	ub.Path = "/Test.png";

            	SaveFileDialog sfd = new SaveFileDialog();
            	sfd.DefaultExt = ".png";
            	if (sfd.ShowDialog() == false)
                	return;

            	WebClient wc = new WebClient();
            	wc.OpenReadCompleted += (s, a) =>
            	{
                	if (a.Cancelled || a.Error != null)
                    	return;

                	using (var trg = sfd.OpenFile())
                	{
                    	int value;
                    	while ((value = a.Result.ReadByte()) != -1)
                        	trg.WriteByte((byte)value);
                	}
            	};
            	wc.OpenReadAsync(ub.Uri);
        	}
    	}
}
```
В корень сайта добавить файл Test.png и запустить приложение.
В IE отобразится кнопка Load. После нажатия откроется диалог сохранения файла, в котором надо указать имя файла (расширение указывать не требуется) и нажать Save.
