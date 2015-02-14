---
title: Как в коде задать richtextbox.Height?
tags: [c#, xaml, wpf, xml, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/5b19fbcd-eaa9-40ba-ba92-e0decff6eae6/-richtextboxheight?forum=programminglanguageru
---
# Как в коде задать richtextbox.Height?
*> логика программы такова: при нажатии кнопки addBtn на стеки добавляются пара richtextbox*

можно использовать ItemsControl и шаблон элемента (с двумя RichTextBox, контент которых привязан к соответствующим свойствам объекта)
```c#
using System.Collections;
using System.Collections.ObjectModel;
using System.Windows;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = LoadData();
    }

    object LoadData()
    {
      var data = new ObservableCollection<Training>();
      data.Add(new Training { Left = "left0", Right = "right0" });
      return data;
    }

    private void addBtn_Click(object sender, RoutedEventArgs e)
    {
      var list = this.DataContext as IList;
      list.Add(new Training { Left = "left"+list.Count, Right = "right"+list.Count });
    }
  }

  class Training
  {
      public string Left { get; set; }
      public string Right { get; set; }
  }
}
```
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">

  <Grid>
    <Button Name="addBtn"  Content=" Add one variant " Click="addBtn_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <ScrollViewer ScrollViewer.VerticalScrollBarVisibility="Visible" Margin="0,30,0,0">
      <ItemsControl ItemsSource="{Binding}" >
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Grid Height="40">
              <Grid.ColumnDefinitions>
                <ColumnDefinition />
                <ColumnDefinition />
              </Grid.ColumnDefinitions>
              <RichTextBox Grid.Column="0" DataContext="{Binding Left}">
                <FlowDocument>
                  <Paragraph>
                    <Run Text="{Binding Path=.}" />
                  </Paragraph>
                </FlowDocument>
              </RichTextBox>
              <RichTextBox Grid.Column="1" DataContext="{Binding Right}">
                <FlowDocument>
                  <Paragraph>
                    <Run Text="{Binding Path=.}" />
                  </Paragraph>
                </FlowDocument>
              </RichTextBox>
            </Grid>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>
    </ScrollViewer>
  </Grid>
</Window>
```
*> как такое чудо сохранять/загружать?*

если сохранять/загружать надо в xml файл, то можно использовать DataSet. примерно так
```xaml
<Window x:Class="WpfApplication6.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <Grid>
    <DockPanel VerticalAlignment="Top" HorizontalAlignment="Left">
      <Button Name="Add" Content="Add one variant" Click="Add_Click" />
      <Button Name="Load" Content="Load" Click="Load_Click" />
      <Button Name="Save" Content="Save" Click="Save_Click" />
    </DockPanel>
    <ScrollViewer ScrollViewer.VerticalScrollBarVisibility="Visible" Margin="0,30,0,0">
      <ItemsControl ItemsSource="{Binding Trainings}" >
        <ItemsControl.ItemTemplate>
          <DataTemplate>
            <Grid>
              <Grid.ColumnDefinitions>
                <ColumnDefinition />
                <ColumnDefinition />
              </Grid.ColumnDefinitions>
              <RichTextBox Grid.Column="0" DataContext="{Binding Left}" x:Name="rl">
                <FlowDocument>
                  <Paragraph>
                    <Run Text="{Binding Path=.}" />
                  </Paragraph>
                </FlowDocument>
              </RichTextBox>
              <RichTextBox Grid.Column="1" DataContext="{Binding Right}">
                <FlowDocument>
                  <Paragraph>
                    <Run Text="{Binding Path=.}" />
                  </Paragraph>
                </FlowDocument>
              </RichTextBox>
            </Grid>
          </DataTemplate>
        </ItemsControl.ItemTemplate>
      </ItemsControl>
    </ScrollViewer>
  </Grid>
</Window>
```
```c# 
using System.Collections;
using System.Collections.ObjectModel;
using System.Windows;
using System.Data;
using System.IO;

namespace WpfApplication6
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      this.DataContext = LoadData();
    }
    
    object LoadData()
    {
      var file = "data.xml";
      var data = new DataSet();
      if(File.Exists(file))
      {
        data.ReadXml(file);
      }
      else
      {
        var dt = new DataTable("Trainings");
        dt.Columns.Add("Left", typeof(string));
        dt.Columns.Add("Right", typeof(string));
        data.Tables.Add(dt);
        dt.Rows.Add("left test", "right test");
      }
      return data;
    }

    private void Add_Click(object sender, RoutedEventArgs e)
    {
      var dt = (this.DataContext as DataSet).Tables["Trainings"];
      dt.Rows.Add("left"+dt.Rows.Count, "right");
    }

    private void Save_Click(object sender, RoutedEventArgs e)
    {
      var ds = this.DataContext as DataSet;
      ds.AcceptChanges();
      ds.WriteXml("data.xml", XmlWriteMode.WriteSchema);
    }

    private void Load_Click(object sender, RoutedEventArgs e)
    {
      this.DataContext = LoadData();
    }
  }
}
```