---
title: Binding to ComboBox SelectedValue and to the ItemsSource
tags: [c#, xaml, wpf, linq, xml]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b0daed1b-8a2e-49db-b824-4b163d755a98/binding-to-combobox-selectedvalue-and-to-the-itemssource?forum=wpf
---
# Binding to ComboBox SelectedValue and to the ItemsSource
as far as I could understand, you trying to make master-details binding.
 if so, the following example will be helpful
```xaml
<Window x:Class="WpfApplication1.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"  FontSize="15" 
  Title="MainWindow" Height="350" Width="525" FocusManager.FocusedElement="{Binding ElementName=cb}">
  <StackPanel>
    <ComboBox x:Name="cb" ItemsSource="{Binding Path=staff}" VerticalAlignment="Top"
      DisplayMemberPath="name" IsSynchronizedWithCurrentItem="True"  />
    <ListBox ItemsSource="{Binding ElementName=cb, Path=SelectedValue.staff_member}" HorizontalAlignment="Stretch"
      DisplayMemberPath="id"/>
  </StackPanel>
</Window>
```
```c#
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Data;
using System.IO;

namespace WpfApplication1
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = Load();
    }

    // returns dataset with two tables: staff, member; 
    // and one relation: staff_member
    DataSet Load()
    {
      var ds = new DataSet();
      // test purpose only
      ds.ReadXml(new StringReader(@"
        <data>
          <staff id='sid1' name='s1'>
            <member id='m1' />
            <member id='m2' />
          </staff>
          <staff id='sid2' name='s2'>
            <member id='m3' />
            <member id='m4' />
            <member id='m5' />
          </staff>
          <staff id='sid3' name='s3'>
            <member id='m6' />
            <member id='m7' />
          </staff>
        </data>"));
      return ds;
    }
  }
}
```
*> my combo box is full with the items = Tbl_Staff  but i can't get it to bind to the staff members(the selected item in the combobox is not the same as in the staff member table) and when i try to select another item it will not select ...*

i've modified my code above to reflect your suggestion.
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16" Title="Test" Height="400" Width="500"
  FocusManager.FocusedElement="{Binding ElementName=cb}">
  <StackPanel>
    <ComboBox x:Name="cb" 
      DataContext="{Binding Tbl_Staff}" 
      ItemsSource="{Binding}" 
      DisplayMemberPath="Staff" 
      IsSynchronizedWithCurrentItem="True"  />
    <ListBox Margin="0,20,0,0"
      DataContext="{Binding Tbl_StaffMembers}" 
      ItemsSource="{Binding ElementName=cb, Path=SelectedValue.Tbl_Staff_Childs}" 
      DisplayMemberPath="FirstName"/>
  </StackPanel>
</Window>
```
```c#
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Data;

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
      //Tbl_Staff: Id, Staff
      var s = new DataTable("Tbl_Staff");
      s.Columns.Add("Id", typeof(int));
      s.Columns.Add("Staff", typeof(string));

      //Tbl_StaffMembers: Id, FirstName, StaffParent
      var m = new DataTable("Tbl_StaffMembers");
      m.Columns.Add("Id", typeof(int));
      m.Columns.Add("FirstName", typeof(string));
      m.Columns.Add("StaffParent", typeof(int));

      var data = new DataSet();
      data.Tables.AddRange(new[] { s, m });
      data.Relations.Add("Tbl_Staff_Childs", s.Columns["Id"], m.Columns["StaffParent"]);

      // test data
      for (int i = 0; i < 3; i++)
        s.Rows.Add(i, "staff" + i);
      var mi = 0;
      for (int i = 0; i < s.Rows.Count; i++)
        for (int j = 0; j < 3; j++)
          m.Rows.Add(mi++, "name" + mi, i);

      return data;
    }
  }
}
```
*>...instead of it binding to the listbox i want it bind to the comboboxes...*

below is an example with comboboxes and optional contains textblock with multibinding to the combobox selected values.
you can use this xaml with the codebehind i have posted above.
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  FontSize="16" Title="Test" Height="300" Width="500"
  FocusManager.FocusedElement="{Binding ElementName=cb}">
  <StackPanel>
    <ComboBox x:Name="cb" 
      DataContext="{Binding Tbl_Staff}" 
      ItemsSource="{Binding}" 
      DisplayMemberPath="Staff" 
      IsSynchronizedWithCurrentItem="True" />
    <ComboBox x:Name="cb2"
      DataContext="{Binding Tbl_StaffMembers}" 
      ItemsSource="{Binding ElementName=cb, Path=SelectedValue.Tbl_Staff_Childs}" 
      IsSynchronizedWithCurrentItem="True"
      DisplayMemberPath="FirstName"/>
    <TextBlock Margin="0,10,0,0">
      <TextBlock.Text>
        <MultiBinding StringFormat="staff: {0}, members: {1}">
          <Binding ElementName="cb" Path="SelectedValue.Staff" />
          <Binding ElementName="cb2" Path="SelectedValue.FirstName" />
        </MultiBinding>
      </TextBlock.Text>
    </TextBlock>
  </StackPanel>
</Window>
```