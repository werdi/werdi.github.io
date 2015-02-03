---
title: LINQ запрос на объединение
tags: [c#, xaml, linq, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/fb91505e-648e-4e42-ae21-d38e9531ef78/linq-?forum=fordataru
---
# LINQ запрос на объединение
*> во второй таблице будет поле с ID на третью таблицу. Как в таком случае поменяется запрос?*

если надо создать дерево, то как вариант, так:
```c#
using System.Linq;
...
// 
var t1 = new[] 
{ 
  new { Id = 0, Name = "n1" },
  new { Id = 1, Name = "n2" },
};
var t2 = new[] 
{ 
  new { Id = 0, Name = "n1", Parent = 0 },
  new { Id = 1, Name = "n2", Parent = 1 },
  new { Id = 2, Name = "n3", Parent = 0 },
};
var t3 = new[] 
{
  new { Id = 0, Name = "n1", Parent = 0 },
  new { Id = 1, Name = "n1", Parent = 1 },
};

var res = from x in t1
          select new 
          { 
            Row = x, 
            Childs = (from y in t2 
                      where x.Id == y.Parent 
                      select new 
                      { 
                        Row = y, 
                        Childs = (from z in t3 
                                  where y.Id == z.Parent      
                                  select z)
                      })
          };

foreach (var x in res)
{
  System.Diagnostics.Trace.WriteLine(x.Row.Id + " " + x.Row.Name, "t1");
  foreach (var y in x.Childs)
  {
    System.Diagnostics.Trace.WriteLine("\t" + y.Row.Id + " " + y.Row.Name, "t2");
    foreach (var z in y.Childs)
      System.Diagnostics.Trace.WriteLine("\t\t" + z.Id + " " + z.Name, "t3");
  }
}
```
если данные хранятся в базе данных, то проще загрузить их в DataSet и привязать его к UI.
примерно так:
```c#
using System.Data;
using System.Windows;

namespace WpfApplication2
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();

      //
      var t1 = new DataTable("t1");
      t1.Columns.Add("id", typeof(int)).AutoIncrement = true;
      t1.Columns.Add("name", typeof(string));
      // 
      var t2 = new DataTable("t2");
      t2.Columns.Add("id", typeof(int)).AutoIncrement = true;
      t2.Columns.Add("name", typeof(string));
      t2.Columns.Add("t1id", typeof(int));
      //
      var ds = new DataSet();
      ds.Tables.AddRange(new [] { t1, t2 });
      ds.Relations.Add("t1_t2", t1.Columns["id"], t2.Columns["t1id"]);
      //
      t1.Rows.Add(0, "n1");
      t1.Rows.Add(1, "n2");
      t2.Rows.Add(0, "n1", 0);
      t2.Rows.Add(1, "n2", 0);
      t2.Rows.Add(2, "n3", 1);

      this.DataContext = ds;
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication2.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="250" FontSize="16" SizeToContent="Width">
  
  <StackPanel Orientation="Horizontal">
    <ListBox ItemsSource="{Binding t1}" IsSynchronizedWithCurrentItem="True" x:Name="lb1" 
      Width="250" DisplayMemberPath="name" />
    <ListBox ItemsSource="{Binding ElementName=lb1, Path=SelectedItem.t1_t2}"
      Width="250" DisplayMemberPath="name" />
  </StackPanel>
</Window>
```
