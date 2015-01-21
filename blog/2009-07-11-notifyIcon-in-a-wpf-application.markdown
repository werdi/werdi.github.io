---
title: NotifyIcon в WPF приложении
tags: [winforms, wpf, c#]
src: http://cognitex.blogspot.ru/2009/07/notifyicon-wpf.html
---
# NotifyIcon в WPF приложении
Задача: после запуска WPF Application в system tray, т.е. в правом нижнем углу экрана должна появиться иконка; при щелчке мышью по иконке надо показать окно; закрытие окон не должно приводить к закрытию приложения.

Реализовать иконку в system tray можно с помощью NotifyIcon; для этого в References проекта WPF Application надо: 
<ol>
  <li>добавить сборку System.Windows.Forms.dll</li>
  <li>в App.xaml у тега Application убрать атрибут StartupUri</li>
  <li>в App.cs в class App добавить конструктор</li>
</ol>

```c#
public App()
{
    	// закрытие всех окон приложения не приводит к его завершению
    	this.ShutdownMode = ShutdownMode.OnExplicitShutdown;

    	// создать иконку и поместить ее в system tray
    	var ni = new System.Windows.Forms.NotifyIcon();
    	ni.Visible = true;
    	ni.Icon = WpfApplication6.Properties.Resources.Icon1;

    	ni.Click += (s, _) =>
    	{
        	// создать и показать окно
        	var uri = new Uri("Window1.xaml", UriKind.Relative);
        	var wnd = Application.LoadComponent(uri) as Window;
        	wnd.Visibility = Visibility.Visible;
        	wnd.ShowInTaskbar = false;
    	};

    	// определить контекстное меню
    	ni.ContextMenu = new System.Windows.Forms.ContextMenu(new[]
    	{
        	// завершить приложение
        	new System.Windows.Forms.MenuItem("Exit", delegate
        	{
            	ni.Visible = false;
           	this.Shutdown();
        	})
    	});
}
```
Надо заметить, что если приложение будет закрыто по ошибке или из дебаггера, то иконка останется в system tray до тех пор, пока над ней не окажется курсор мыши. 

[WpfNotifyIcon.zip](https://onedrive.live.com/self.aspx/.Public/Samples/WPF/WpfNotifyIcon.zip?cid=13085c33c4ce4359&id=documents)
