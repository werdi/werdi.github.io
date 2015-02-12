---
title: Виснет программа
tags: [c#, linq, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f1ea684c-c8de-4635-b827-404ea3f6bf5e/-?forum=fordesktopru
---
# Виснет программа
*> сканирует порты в циклеParallel.For(первый порт, [...]  нужно перепроверить эти порты с теми портами которые были открыты [...] Функция вывода в форму() [...] все начинает сильно виснуть ...*
```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
...

void Button_Click(object sender, EventArgs e)
{
  var old = new[] { 2, 4, 6 };
  Start(1, 10, old, cur =>
  {
    // выполняется в UI-потоке
    // ...
  });
}

void Start(int min, int max, IEnumerable<int> old, Action<int> progress)
{
  var ao = AsyncOperationManager.CreateOperation(null);
  SendOrPostCallback cb = state => progress((int)state);
  Task.Factory.StartNew(() =>
    Parallel.For(min, max + 1, cur =>
    {
      if (old.Contains(cur)) 
        return;

      // ...

      // переслать данные в UI-поток
      ao.Post(cb, cur);      
    }));
}
```
*> программа не заходит в этот участок =>void Start(int min, int max, IEnumerable old, Action progress) [...] в функции Start мне не удается работать с Listview и ее методами для вывода данных в форму*
```c#
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Drawing;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApplication6
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new Size(500, 300);

      var lb = new ListBox { Parent = this, Dock = DockStyle.Fill, IntegralHeight = false };
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Run", (s, e) =>
        {
          var old = new[] { 2, 4, 6 };
          Start(1, 10, old, Scan, inf =>
            lb.Items.Add("Port: " + inf.Port + " \tResult: " + inf.Result)
          );
        });
    }

    void Scan(Info info)
    {
      // здесь сканируем и сохраняем результат в info
      info.Result = (int)DateTime.Now.Ticks;
    }

    class Info
    {
      public Info(int port) { this.Port = port; }
      public readonly int Port;
      public int Result;
    }

    void Start(int min, int max, IEnumerable<int> old, Action<Info> work, Action<Info> progress)
    {
      var ao = AsyncOperationManager.CreateOperation(null);
      SendOrPostCallback cb = state => progress((Info)state);
      Task.Factory.StartNew(() => Parallel.For(min, max + 1, cur =>
      {
        if (old.Contains(cur)) return;
        var inf = new Info(cur);
        work(inf);
        ao.Post(cb, inf);
      }));
    }
  }
}
```