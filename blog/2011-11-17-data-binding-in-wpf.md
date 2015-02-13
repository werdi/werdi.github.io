---
title: Data binding in wpf
tags: [c#, xaml, wpf]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9f280f6a-4450-4157-8864-ac123f9cc146/data-binding-in-wpf?forum=wpf
---
# Data binding in wpf
*> table a only holds the codes (foreign keys)to many diffrent tables. for ex: say table a has two columns codeworker and codejob and then there are two more tables called workers and jobs and they hold the names of the corresponding code from table a. in my datagrid i need to be able to display the name from the other tables that is represented by the code in table a.*
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16" Title="Test" Height="300" Width="500"
  FocusManager.FocusedElement="{Binding ElementName=mv}">
  <Grid>
    <Grid.ColumnDefinitions>
      <ColumnDefinition />
      <ColumnDefinition />
    </Grid.ColumnDefinitions>
    <Grid.Resources>
      <Style TargetType="DataGrid">
        <Setter Property="VerticalGridLinesBrush" Value="Silver" />
        <Setter Property="HorizontalGridLinesBrush" Value="Silver" />
        <Setter Property="CanUserAddRows" Value="False" />
        <Setter Property="ColumnWidth" Value="*" />
        <Setter Property="AutoGenerateColumns" Value="False" />
        <Setter Property="HeadersVisibility" Value="Column" />
        <Setter Property="Background" Value="White" />
      </Style>
    </Grid.Resources>
    <DataGrid x:Name="mv" Grid.Column="0"
      ItemsSource="{Binding workers}" IsSynchronizedWithCurrentItem="True">
      <DataGrid.Columns>
        <DataGridTextColumn Header="workers" Binding="{Binding name}" Width="*" />
      </DataGrid.Columns>
    </DataGrid>
    <DataGrid Grid.Column="1"
      ItemsSource="{Binding ElementName=mv, Path=SelectedValue.workers_a}">
      <DataGrid.Columns>
        <DataGridTextColumn Header="jobs" Binding="{Binding job}" Width="*" />
      </DataGrid.Columns>
    </DataGrid>
  </Grid>
</Window>
```
```c#
using System.Data;
using System.Windows;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = this.LoadData();
    }

    object LoadData()
    {
      var a = new DataTable("a");
      a.Columns.Add("codeworker", typeof(int));
      a.Columns.Add("codejob", typeof(int));

      var w = new DataTable("workers");
      w.Columns.Add("id", typeof(int));
      w.Columns.Add("name", typeof(string));

      var j = new DataTable("jobs");
      j.Columns.Add("id", typeof(int));
      j.Columns.Add("name", typeof(string));

      var data = new DataSet();
      data.Tables.AddRange(new[] { a, w, j });
      data.Relations.Add("workers_a", w.Columns["id"], a.Columns["codeworker"]);
      data.Relations.Add("jobs_a", j.Columns["id"], a.Columns["codejob"]);

      // 'virtual' columns
      a.Columns.Add("worker", typeof(string), "parent(workers_a).name")
        .ColumnMapping = MappingType.Hidden;
      a.Columns.Add("job", typeof(string), "parent(jobs_a).name")
        .ColumnMapping = MappingType.Hidden;

      // test data
      for (int i = 0; i < 2; i++) w.Rows.Add(i, "worker" + i);
      for (int i = 0; i < 5; i++) j.Rows.Add(i, "job" + i);
      // worker0 jobs
      a.Rows.Add(0, 0); 
      a.Rows.Add(0, 1);
      a.Rows.Add(0, 2);
      // worker1 jobs
      a.Rows.Add(1, 2); 
      a.Rows.Add(1, 3);
      a.Rows.Add(1, 4);
      
      return data;
    }
  }
}
```
*> how would i afterwards update tablea with the new codeName if the user were to choose a diffrent name?*

simply change name of the table. and write: 
```c#
var a = new DataTable("newname");
instead of var a = new DataTable("a");
```
*> i am not using linq.*

my code above does not use linq.

*> ... SqlDataAdapter adapt = new SqlDataAdapter("select B.FirstName from tblA A inner join tblB B on(A.PersonId=B.PersonId)", conn); ...*

so requirements has apparently changed. could you please elaborate: 
what fields are into the result table?

*> if i am binding the data to a listview what is the display member path?*

in this case all you need is to use following xaml, instead of xaml in my previous posts above.
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16" Title="Test" Height="300" Width="500"
  FocusManager.FocusedElement="{Binding ElementName=mv}">
  <Grid>
    <Grid.ColumnDefinitions>
      <ColumnDefinition />
      <ColumnDefinition />
    </Grid.ColumnDefinitions>
    <Grid.Resources>
      <Style x:Key="lvhs" TargetType="GridViewColumnHeader">
        <Setter Property="Visibility" Value="Collapsed" />
      </Style>
    </Grid.Resources>
    <ListView ItemsSource="{Binding workers}" IsSynchronizedWithCurrentItem="True" x:Name="mv" Grid.Column="0">
      <ListView.View>
        <GridView ColumnHeaderContainerStyle="{StaticResource lvhs}">
          <GridView.Columns>
            <GridViewColumn Header="workers" DisplayMemberBinding="{Binding name}" />
          </GridView.Columns>
        </GridView>
      </ListView.View>
    </ListView>
    <ListView ItemsSource="{Binding ElementName=mv, Path=SelectedValue.workers_a}" Grid.Column="1">
      <ListView.View>
        <GridView ColumnHeaderContainerStyle="{StaticResource lvhs}">
          <GridView.Columns>
            <GridViewColumn Header="jobs" DisplayMemberBinding="{Binding job}" />
          </GridView.Columns>
        </GridView>
      </ListView.View>
    </ListView>
  </Grid>
</Window>
```