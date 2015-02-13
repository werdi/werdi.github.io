---
title: Создание дочерней формы в отдельном потоке
tags: [c#, threading, interop, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/59a6d357-aae7-448c-a3d7-45755bdd071a/-?forum=fordesktopru
---
# Создание дочерней формы в отдельном потоке
*> форма, которую нужно вывести очень долго создается*

на форме много контролов или тормозит из-за загрузки данных? 
если второй вариант, то загружать данные надо в отдельном потоке, затем передать данные в основной поток. примерно так:
```c#
Task.Factory.StartNew(() => {    // загрузка данных
  var wc = new WebClient();
  return wc.DownloadString("http://microsoft.com"); })
.ContinueWith(task => { // выполняется в основном потоке
	tb.Text = task.Result; 
}, TaskScheduler.FromCurrentSynchronizationContext());      // вместо Dispatcher.Invoke
```
перед Task.Factory.StartNew показываете сообщение, которое можно убрать после выполнения tb.Text = task.Result;

*> проблема как раз в том, что данных много, и тормозит сама инициализация формы. А Task - это Framework 4.0? Просто мне нельзя использовать выше v3.5 в этом проекте.*

т.е. в контрол выводится сразу много строк? например, в список выводится 100 тыс. строк. 
если так, то элементы можно выводить в контрол порциями. 
вместо [Task](http://msdn.microsoft.com/en-us/library/system.threading.tasks.task.aspx) можно использовать AsyncOperation. 
см. пример [здесь](http://social.msdn.microsoft.com/Forums/eu/winforms/thread/4823af90-e083-4fce-9edf-d9a47700d705/#7caada1f-1f29-429f-9155-b388a983574f)

*> в дальнейшем могут написать что-то тяжелое и мне необходимо это предусмотреть. Поэтому и хочется сделать инициализацию формы в отдельном потоке.*
```c#
using System;
using System.Drawing;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Text = "Main form";
      new Button { Parent = this, Text = "test" }
        .Click += (s, e) =>
        {
          var lbl = new Label { 
            Parent = this, Text = "Loading...", BackColor = Color.Yellow, Location = new Point(10, 100) };

          ThreadPool.QueueUserWorkItem(state =>
          {
            Thread.Sleep(1000);    // "долгая загрузка"
            var frm = new Form() { Text = "Child form" };
            this.BeginInvoke(new Action(() => this.Controls.Remove(lbl)));
            Application.Run(frm);
          });
        };
    }
  }
}
```

*> _showForm.Parent = panelAnswers; // здесь вываливается исключение* 

т.е. в основную форму надо вставить форму, созданную в отдельном потоке? обычными способами не получится. т.к. форма хранит информацию о потоке, в котором она была создана. и при прямом или косвенном обращении из другого потока будет ошибка.
можете попробовать встроить одну форму в другую с помощью функции Win API [SetParent](http://msdn.microsoft.com/en-us/library/windows/desktop/ms633541(v=vs.85).aspx). при этом встраиваемая форма должна быть примерно такой:
```c#
public class Child : Form
{
  IntPtr _Parent;

  public Child(IntPtr parent)
  {
    this.TopLevel = false;
    _Parent = parent;
  }
  protected override CreateParams CreateParams
  {
    get
    {
      CreateParams cp = base.CreateParams;
      cp.Parent = _Parent;
      return cp;
    }
  }
}
```
