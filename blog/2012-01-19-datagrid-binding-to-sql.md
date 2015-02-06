---
title: Странность при создании связки DataGrid - SQL средствами студии
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/48fe1e5d-bab3-4bc7-9fb1-5831575e8ceb/-datagrid-sql-?forum=fordesktopru 
---
# Странность при создании связки DataGrid - SQL средствами студии
*> В студии 2010 Ultimate (C# .NET 4.0, WPF) создаю форму и набор данных. Кидаю набор данных на форму (по идее должно создаться DataGrid, если верить MSDN), но в итоге студия ругается*

возможно, что это какой-то баг. попробуйте тоже самое сделать в Expression Blend.
(из моего опыта: чтобы не портить себе нервы, визуальные редакторы лучше не использовать).

*> получает из базы таблицу с двумя столбцами (num и value). Как сделать так, чтобы данные из столбца num базы данных выводились в столбец num грида соответственно?*
```xaml
<DataGrid ItemsSource="{Binding}" ColumnWidth="*" 
    AutoGenerateColumns="False">
  <DataGrid.Columns>
    <DataGridTextColumn Header="Номер" Binding="{Binding num}" />
    <DataGridTextColumn Header="Значение" Binding="{Binding value}" />
  </DataGrid.Columns>
</DataGrid>
```
```c#
...
public MainWindow()
{
  InitializeComponent();

  var dc = new System.Data.Linq.DataContext(....);
  this.DataContext = dc.GetTable<test>();

  // сохранить данные при закрытии окна
  this.Closing += (s, e) => dс.SubmitChanges();
}
```
