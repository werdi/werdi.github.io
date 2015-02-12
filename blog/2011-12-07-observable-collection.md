---
title: ObservableColection, INotifyPropertyChanged Master/Detail
tags: [c#, xaml, wpf, xml, binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/55d457fb-b256-4a58-b38f-6c8025f227dc/observablecolection-inotifypropertychanged-masterdetail?forum=wpf
---
# ObservableColection, INotifyPropertyChanged Master/Detail
*> I'm using VS2010/wpf[c#]. I have 2 class namely, Player and Team.Its Like Master and Detail. [...] I want to Add/update/Delete. I have one button name, save. [...] I dont know how to call INotifyPropertyChanged in Save Button.*

in your case there is no need to use INotifyPropertyChanged.
 below is an complete example. it creates teams/players editor based on a DataSet.
 (DataSet you can populate from xml string, from xml file, or from database)
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication8"
  Title="MainWindow" Height="500" Width="500" FontSize="16">
  <StackPanel>
    <DockPanel HorizontalAlignment="Left">
      <Button Content="Add" Click="Team_Add_Click" />
      <Button Content="Delete" Click="Team_Delete_Click" />
    </DockPanel>
    <ListBox ItemsSource="{Binding Team}" IsSynchronizedWithCurrentItem="True" 
      x:Name="lb" DisplayMemberPath="Name" Height="150" />
    <DockPanel HorizontalAlignment="Left">
      <Button Content="Add" Click="Player_Add_Click" />
      <Button Content="Delete" Click="Player_Delete_Click" />
    </DockPanel>
    <DataGrid ItemsSource="{Binding ElementName=lb, Path=SelectedItem.Team_Player}" x:Name="dg" 
      Height="230" CanUserAddRows="False" AutoGenerateColumns="False" ColumnWidth="*" 
      VerticalGridLinesBrush="Silver" HeadersVisibility="Column">
      <DataGrid.Columns>
        <DataGridTextColumn Header="First Name" Binding="{Binding FirstName}" Width="Auto" />
        <DataGridTextColumn Header="Last Name" Binding="{Binding LastName}" />
      </DataGrid.Columns>
    </DataGrid>
    <DockPanel  HorizontalAlignment="Left">
      <Button Content="Save" Click="Save_Click" />
      <Button Content="Load" Click="Load_Click" />
    </DockPanel>
  </StackPanel>
</Window>
```
```c# 
using System.Data;
using System.IO;
using System.Windows;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = LoadData();
    }
    private object LoadData()
    {
      var data = new DataSet();
      // populating from a string; this is for test; 
      data.ReadXml( new StringReader(@"
      <Data>
        <Team Name='LA Lakers'>
          <Player FirstName='Kobe' LastName='Bryant' />
          <Player FirstName='Pau' LastName='Gasol' />
        </Team>
        <Team Name='Detroit Pistons'>
          <Player FirstName='Richard' LastName='Hamilton' />
          <Player FirstName='Chauncey' LastName='Billups' />
        </Team>
      </Data>"));
      // data.Tables["Player"].Columns.Add("FullName", typeof(string), "FirstName + ' ' + LastName");
      return data;
    }
    private void Team_Add_Click(object sender, RoutedEventArgs e)
    {
      var dv = lb.ItemsSource as DataView;
      var dr = dv.AddNew();
      dr["Name"] = "New Team";
      dr.EndEdit();
    }
    private void Team_Delete_Click(object sender, RoutedEventArgs e)
    {
      var ds = this.DataContext as DataSet;
      var dr = lb.SelectedItem as DataRowView;
      if(dr != null)
        dr.Delete();
    }
    private void Player_Add_Click(object sender, RoutedEventArgs e)
    {
      var dv = dg.ItemsSource as DataView;
      var dr = dv.AddNew();
      dr["FirstName"] = "First";
      dr["LastName"] = "Last";
      dr.EndEdit();
    }
    private void Player_Delete_Click(object sender, RoutedEventArgs e)
    {
      var ds = this.DataContext as DataSet;
      var dr = dg.SelectedItem as DataRowView;
      if(dr != null)
        dr.Delete();
    }
    private void Save_Click(object sender, RoutedEventArgs e)
    {
      var ds = this.DataContext as DataSet;
      ds.WriteXml("Data.xml");     // \bin\Debug\save.xml
    }
    private void Load_Click(object sender, RoutedEventArgs e)
    {
      var ds = this.DataContext as DataSet;
      ds.Clear();
      ds.ReadXml("Data.xml");
    }
  }
}
```