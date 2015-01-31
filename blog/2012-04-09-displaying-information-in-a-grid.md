---
title: Отображение информации в Grid
tags: [grid, xaml, c#, silverlight]
src: https://social.msdn.microsoft.com/Forums/ru-RU/c7c2451b-890d-4325-88d0-e5a7671277ac/-grid?forum=aspnetru
---
# Отображение информации в Grid
*> как можно ограничить видимость объектов расположенных в элементе Grid (Panel, Canvas не суть) размерами самого контейнера.* 
с помощью привязки к размеру контейнера. примерно так: 
```xaml
<UserControl x:Class="SilverlightApplication1.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <Grid Width="200" Height="100" x:Name="grd" Background="WhiteSmoke">
    <Button Content="test" Width="{Binding ElementName=grd, Path=Height}" />
  </Grid>
</UserControl>
```
*> Пробовал использовать ListBox [...] мне его надо плавно скролить* 
в ItemsPanelTemplate надо указать StackPanel:
```xaml
<UserControl x:Class="SilverlightApplication1.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <Grid Width="200" Height="200">
    <ListBox ItemsSource="{Binding}">
      <ListBox.ItemTemplate>
        <DataTemplate>
          <Border BorderThickness="1" BorderBrush="Silver">
            <TextBlock Height="40" Text="{Binding}" Margin="2" />
          </Border>
        </DataTemplate>
      </ListBox.ItemTemplate>
      <ListBox.ItemsPanel>
        <ItemsPanelTemplate>
          <StackPanel />
        </ItemsPanelTemplate>
      </ListBox.ItemsPanel>
    </ListBox>
  </Grid>
</UserControl>
```
```c#
using System.Linq;
using System.Windows.Controls;

namespace SilverlightApplication1
{
  public partial class MainPage : UserControl
  {
    public MainPage()
    {
      InitializeComponent();
      this.DataContext = Enumerable.Range(0, 50).Select(i => new { Key = i });
    }
  }
}
```
p.s. 
для прокрутки при нажатии кнопки см. [здесь] (http://social.msdn.microsoft.com/Forums/ru-RU/formobiledevicesru/thread/58b11fa3-cbbb-4d97-ac6b-75e77d64605c/#175a8217-5e88-450a-870c-8952ba333c73)

*> А мне надо чтобы кнопка, находясь за пределами сетки grd, была невидимой. Т.е. если кнопка выступает за пределы сетки на половину то выступающая половина не видна.* 
так и есть. silverlight 5. часть кнопки за пределами Grid не видна:

![img1] (http://social.msdn.microsoft.com/Forums/getfile/90182)
```xaml
<UserControl x:Class="SilverlightApplication1.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
  <Border BorderBrush="Red" BorderThickness="1" Width="350" Height="250">
    <Grid Margin="50">
      <Grid Width="200" Height="200" x:Name="grd" Background="WhiteSmoke">
        <Button Content="Scroll" Width="300" Height="100" Margin="50,25,0,0" />
      </Grid>
    </Grid>
  </Border>
</UserControl>
```

*> А с Grid можно проделать подобный трюк?* 

для элементов придется указывать Margin или Grid.Row вместе с Grid.RowDefinitions.
проще использовать StackPanel.
