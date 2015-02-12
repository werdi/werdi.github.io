---
title: Почему ProgressBar замораживается?
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d9d67dc7-f485-4bd8-b76d-3e84ca78a361/-progressbar-?forum=fordesktopru
---
# Почему ProgressBar замораживается?
*> Как сделать окошко с ProgressBar? [...] загрузить данные в новом потоке*
``xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525" FontSize="16">
  <StackPanel>
    <Button Content="Run" Click="Run_Click" HorizontalAlignment="Left" x:Name="btn" />
    <ProgressBar x:Name="pb" Height="25" />
  </StackPanel>
</Window>
```
```c#
using System;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }
    private void Run_Click(object sender, RoutedEventArgs e)
    {
      btn.IsEnabled = false;
      
      Action<int> cb = progress =>  // вызывается в UI-потоке
        pb.Value = progress;    

      Task.Factory.StartNew(() =>    // выполняется в отдельном потоке
      {
        for (int i = 0; i <= 100; i += 10)
        {
          // передать данные в UI-поток
          this.Dispatcher.BeginInvoke(cb, i);
          Thread.Sleep(200);
        }

        // поменять значение свойства кнопки
        this.Dispatcher.Invoke(new Action(() => btn.IsEnabled = true));
      });

      Test2();
    }
    private void Test2()
    {
      var wnd = new Window()
      {
        Owner = this,
        ShowInTaskbar = false,
        Height = 60,
        Width = 400,
        WindowStyle = WindowStyle.None,
        WindowStartupLocation = System.Windows.WindowStartupLocation.CenterOwner
      };
      wnd.Loaded += delegate
      {
        var pb = new ProgressBar() { Height = 25, Margin = new Thickness(10) };
        wnd.Content = pb;
        Action<int> cb = progress => pb.Value = progress;
        Task.Factory.StartNew(() =>
        {
          for (int i = 0; i <= 100; i += 5)
          {
            wnd.Dispatcher.BeginInvoke(cb, i);
            Thread.Sleep(50);
          }
          wnd.Dispatcher.Invoke(new Action(() => wnd.Close()));
        });
      };
      wnd.ShowDialog();
    }
  }
}
```
другой пример передачи данных между потоками [здесь](http://social.msdn.microsoft.com/Forums/en/winforms/thread/4823af90-e083-4fce-9edf-d9a47700d705/#7caada1f-1f29-429f-9155-b388a983574f) (en) -- на основе AsyncOperation + CancellationToken (- для остановки потоков).

*> Окно wnd запускается в новом потоке или в основном?*

в основном.
 
*> и куда ложить загрузку данных?*

вместо "for (int i = 0; i <= 100; i += 5) { ... }"
 
*> почему в новом потоке мой класс выдает ошибку (не распознан формат RichTextBox ...*

какой класс? 

*> Есть какие-то ограничения к классам (данных) которые загружаются в отдельных потоках?*

из отдельных потоков ни прямо, ни косвенно нельзя обращаться к контролам. 
других ограничений нет.