---
title: Изменение колекции в Parallel.ForEach
tags: [c#, async]
src: https://social.msdn.microsoft.com/Forums/ru-RU/ee5bed0b-6cd2-43ab-8c74-5f7f88b27744/-parallelforeach?forum=programminglanguageru
---
# Изменение колекции в Parallel.ForEach
*> как модицировать список в процесе прохождения по нему так, что б Parallel.ForEach знал про внесенные изменения?*

использовать потокобезопасные классы коллекций; см. [System.Collections.Concurrent] (http://msdn.microsoft.com/ru-ru/library/system.collections.concurrent.aspx)

*> Как я написал выше - мне надо удалять элемент из списка. Тот же ConcurrentBag не поддерживает методы удаления...*

можно использовать два списка: из одного брать и перекладывать в другой. если не подходит, 
то можно блокировать коллекцию с помощью lock. пример см. [ICollection.SyncRoot] (http://msdn.microsoft.com/en-us/library/system.collections.icollection.syncroot.aspx) и см. [Safe Thread Synchronization] (http://msdn.microsoft.com/en-us/magazine/cc188793.aspx)

p.s.
concurrent collections есть в [TPL DataFlow] (http://msdn.microsoft.com/en-us/devlabs/gg585582) (TDF)

*> Наверное проще извенить алгоритм так, что б не требовалось удалять элементы...*

да, см. ниже метод Filter<T>
```c#
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(300, 400);
      var rtb = new RichTextBox { Dock = DockStyle.Fill, Parent = this };
      Action<string> notify = str => rtb.AppendText(str + "\n");

      var en = LoadData();

      int test = 0;
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("START", (s, e) =>
      {
        var mi = s as MenuItem;
        mi.Enabled = false;
        test++;
        Task.Factory.StartNew(() =>
        {
          this.BeginInvoke(notify, test + ". start; count=" + en.Count());
          en = this.Filter(en, (index, value) =>
          {
            // какой-то фильтр
            return (value.ToString().Contains(test.ToString()));
          });
          this.BeginInvoke(notify, test + ". finish; count=" + en.Count());
          this.BeginInvoke(new Action(() => mi.Enabled = true));
        });
      });
    }
    
    IEnumerable<int> LoadData()
    {
      return new ConcurrentBag<int>(Enumerable.Range(0, 2000000));
    }
    
    IEnumerable<T> Filter<T>(IEnumerable<T> en, Func<long, T, bool> fun)
    {
      var ret = new ConcurrentBag<T>();
      Parallel.ForEach(en, (item, state, index) =>
      {
        var res = fun(index, item);
        if (res)
          ret.Add(item);
      });
      return ret;
    }
  }
}
```
