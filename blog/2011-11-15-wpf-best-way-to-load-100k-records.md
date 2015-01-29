---
title: WPF best way to load 100,000 records
tags: [wpf, c#, xaml, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/0d6d08d9-5d28-4041-bfd2-38eec37bbda0/wfp-best-way-to-load-100000-records?forum=wpf
---
# WPF best way to load 100,000 records
*> Load 100,000 records to a gridview [...] The legacy system works fast and its so quick, but the WPF application takes around 20 seconds to load. Any suggestions to make it fast?*

one of the approaches is to make a chunk loader.
something like this.
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <ListView ItemsSource="{Binding Test}">
    <ListView.View>
        <GridView>
            <GridView.Columns>
                <GridViewColumn Header="Id" DisplayMemberBinding="{Binding Text}" Width="200" />
            </GridView.Columns>
        </GridView>
    </ListView.View>
    </ListView>
</Window>
```
```c#
using System;
using System.Collections.Generic;
using System.Data;
using System.Threading;
using System.Threading.Tasks;
using System.Windows;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
        this.DataContext = LoadData(500, (ds, rows) => 
        {
          var dt = ds.Tables["Test"];
          foreach(var row in rows)
            dt.LoadDataRow(row, false);
        });
      }

      object LoadData(int chunksize, Action<DataSet, List<object[]>> fn)
      {
        var dt = new DataTable("Test");
        dt.Columns.Add("Id", typeof(int));
        dt.Columns.Add("Text", typeof(string));
        var data = new DataSet();
        data.Tables.Add(dt);
        Task.Factory.StartNew(state =>
        {
          var lst = new List<object[]>();
          for (int i = 0; i < 100000; i++)
          {
            lst.Add(new object[] { i, "text" + i });
            if(lst.Count == chunksize)
            {
              this.Dispatcher.BeginInvoke(fn, data, lst);
              lst = new List<object[]>();
              Thread.Sleep(100);
            }
          }
        }, data);
        return data;
      }
    }
}
```
