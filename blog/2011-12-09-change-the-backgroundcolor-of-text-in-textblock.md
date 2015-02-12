---
title: How To change the backgroundcolor for particular text in Textblock ?
tags: [c#, xaml, wpf, regex]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1be1ea52-a16c-4000-9461-33f687519f70/how-to-change-the-backgroundcolor-for-particular-text-in-textblock-?forum=wpf
---
# How To change the backgroundcolor for particular text in Textblock ?
*> How To change the color for particular text in Textblock ? [...]  example textblock1 : Hai how are You ? textblock2 : Hai Where are You ? i wanna change Background color for word how in textblock1*

you can get TextRange and ApplyPropertyValue. 
below is an example.
```xaml
<Window x:Class="WpfApplication8.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="clr-namespace:WpfApplication8"
  Title="MainWindow" Height="500" Width="500" FontSize="16">
  <StackPanel>
    <Button Content="Test" Click="Button_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <TextBlock Text="Hai how are You ?" x:Name="tb" />
    <TextBlock>
      Hai <TextBlock Background="Silver">Where</TextBlock> are You ?
    </TextBlock>
  </StackPanel>
</Window>
```
```c# 
using System.Windows;
using System.Windows.Controls;
using System.Windows.Documents;
using System.Windows.Media;

namespace WpfApplication8
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var tr = Find("how", tb);
      tr.ApplyPropertyValue(TextElement.BackgroundProperty, Brushes.Red);
      tr.ApplyPropertyValue(TextElement.ForegroundProperty, Brushes.White);
      tr.ApplyPropertyValue(TextElement.FontWeightProperty, FontWeights.Bold);
    }

    TextRange Find(string w, TextBlock b)
    {
      var si = tb.Text.IndexOf(w);
      var sp = tb.ContentStart.GetPositionAtOffset(si+1);
      var ep = tb.ContentStart.GetPositionAtOffset(si+w.Length+1);
      return new TextRange(sp, ep);
    }
  }
}
```
*> i want to select mismatched characters*

try using the following code.
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="250" Width="500" FontSize="16">
  <StackPanel x:Name="tst">
    <Button Content="Test" Click="Button_Click" 
      HorizontalAlignment="Left" VerticalAlignment="Top" />
    <TextBlock Text="012abc678" />
    <TextBlock Text="012" />
    <TextBlock Text="abc345" />
    <TextBlock Text="012abc" />
  </StackPanel>
</Window>
```
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Documents;
using System.Windows.Media;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      foreach (var tb in tst.Children.OfType<TextBlock>())
      {
        // find the beginning of the text
        TextPointer s = tb.ContentStart;
        while (s.GetPointerContext(LogicalDirection.Forward) != TextPointerContext.Text)
          s = s.GetNextInsertionPosition(LogicalDirection.Forward);

        // get indices and create TextPointers
        var trs =
          from x in Find(tb.Text, "abc")
          select new TextRange(s.GetPositionAtOffset(x[0]), s.GetPositionAtOffset(x[1]+1));

        foreach (var tr in trs.ToArray()) // .ToArray because we should have all TextPointers before ApplyPropertyValue
        {
          tr.ApplyPropertyValue(TextBlock.TextDecorationsProperty, TextDecorations.Strikethrough);
          tr.ApplyPropertyValue(TextElement.ForegroundProperty, Brushes.Silver);
        }
      }
    }
    IEnumerable<int[]> Find(string txt, string s)
    {
      var si = txt.IndexOf(s, StringComparison.OrdinalIgnoreCase);
      if (si > -1)
      {
        if (si - 1 > 0) yield return new[] { 0, si - 1 };
        var ei = si + s.Length;
        if (txt.Length - ei > 0)
          yield return new[] { ei, txt.Length - 1 };
      }
      else
        yield return new[] { 0, txt.Length - 1 };
    }

    void FindTest()
    {
      var arr = new[] { "012", "012hai678", "hai345", "012hai" };
      foreach (var str in arr)
      {
        var res = Find(str, "hai");
        System.Diagnostics.Trace.Write("\ntext \"" + str + "\": ");
        foreach (var tr in res)
          System.Diagnostics.Trace.Write("{" + String.Join(";", tr) + "}");
      }
    }
  }
}
```
*> but my input is only ... highlight unmatched characters ... haiiiiiiiiiiiii hai only color change for iiiiiiiiiiii these characters*

if i understood correctly, you want highlight repetitive characters at the end of a word. 
so below is an example. 
```c#
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Documents;
using System.Windows.Media;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var ro = RegexOptions.IgnoreCase | RegexOptions.Singleline| RegexOptions.Compiled;
      var re = new Regex(@"(\w)\1+\b", ro);
      var m = re.Match(tb.Text);

      // find the beginning of the text
      TextPointer s = tb.ContentStart;
      while (s.GetPointerContext(LogicalDirection.Forward) != TextPointerContext.Text)
        s = s.GetNextInsertionPosition(LogicalDirection.Forward);

      var tr = new TextRange(s.GetPositionAtOffset(m.Index+1), s.GetPositionAtOffset(m.Index+m.Length));
      tr.ApplyPropertyValue(TextElement.ForegroundProperty, Brushes.Red);
    }
  }
}
```
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="250" Width="500" FontSize="16">
  <StackPanel x:Name="tst">
    <Button Content="Test" Click="Button_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <TextBlock Text="abcccccccccccc abc" x:Name="tb" />
  </StackPanel>
</Window>
```
*> two strings in 2 different textblocks consider textblock1 contains :haiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiiii textblock2 contains :hai*
```xaml
<Window x:Class="WpfApplication9.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="250" Width="500" FontSize="16">
  <StackPanel x:Name="tst">
    <Button Content="Test" Click="Button_Click" HorizontalAlignment="Left" VerticalAlignment="Top" />
    <TextBlock Text="abcccccccccccc abc" x:Name="tb" />
    <TextBlock Text="abccc" />
    <TextBlock Text="abc" />
  </StackPanel>
</Window>
```
```c# 
using System.Text.RegularExpressions;
using System.Windows;
using System.Windows.Documents;
using System.Windows.Media;
using System.Linq;
using System.Windows.Controls;

namespace WpfApplication9
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
    }

    private void Button_Click(object sender, RoutedEventArgs e)
    {
      var ro = RegexOptions.IgnoreCase | RegexOptions.Singleline | RegexOptions.Compiled;
      var re = new Regex(@"(\w)\1+\b", ro);
      foreach (var t in tst.Children.OfType<TextBlock>())
      {
        var m = re.Match(t.Text);
        if(m.Success == false)
          continue;

        // find the beginning of the text
        TextPointer s = t.ContentStart;
        while (s.GetPointerContext(LogicalDirection.Forward) != TextPointerContext.Text)
          s = s.GetNextInsertionPosition(LogicalDirection.Forward);

        var tr = new TextRange(s.GetPositionAtOffset(m.Index + 1), s.GetPositionAtOffset(m.Index + m.Length));
        tr.ApplyPropertyValue(TextElement.ForegroundProperty, Brushes.Red);
      }
    }
  }
}
```
*> your code not working for all the casestext block1: dataabababababa text block2:data*

sure, you should specify an errors, because the code above cannot detect errors in words.
if you want detect errors automatically, then you have to use a spellchecker or dictionary service.