---
title: Как отсортировать список глобально? (для ListBox)
tags: [c#, linq, threading, silverlight]
src: https://social.msdn.microsoft.com/Forums/ru-RU/12b2e0b8-3311-411a-8a35-6155563af259/-listbox?forum=formobiledevicesru 
---
# Как отсортировать список глобально? (для ListBox)
*> мне нужно отсортировать этот список где-нибудь отдельно в глобальную переменную*
```c#
IEnumerable sortedlist;
...
sortedlist = (from p in list orderby p.Name select p).ToList();
```
*> написал sortedlist = (from p in list orderby p.Name select p).ToList(); в конструктор MainPage - производительность при первом нажатии такая же, как при вызове метода сортировки...*

сортировку можно выполнить в отдельном потоке.
примерно так:
```c#
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Windows.Controls;

namespace SilverlightApplication4
{
  public partial class MainPage : UserControl
  {
    public IEnumerable<Item> SortedList
    {
      get
      {
        _SortedListEvent.WaitOne();
        return _SortedList;
      }
      private set
      {
        _SortedList = value;
      }
    }
    IEnumerable<Item> _SortedList;
    ManualResetEvent _SortedListEvent;

    public MainPage()
    {
      InitializeComponent();
      _ListEvent = new ManualResetEvent(false);
      ThreadPool.QueueUserWorkItem(_ =>
      {
        _SortedList = (from p in list orderby p.Name select p).ToList();
        _SortedListEvent.Set();
      });
    }
  }
}
```
