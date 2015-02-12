---
title: How to Read a line when i put cursor on Flow Document?
tags: [c#, xaml, wpf, interop]
src: https://social.msdn.microsoft.com/Forums/ru-RU/24dfc5a2-a0fc-4b25-b7ca-f16f511ed830/how-to-read-a-line-when-i-put-cursor-on-flow-document?forum=wpf
---
# How to Read a line when i put cursor on Flow Document?
*> In WPF window i have FlowDocument.In that few lines(rows) of data is there.My requirement is when i put my cursor in a particular line(row) and right click on that and show in notepad.*

a string pass through the SendKeys, so add reference to the 
system.windows.forms.dll before compile of following the example.

```xaml
<Window x:Class="WpfApplication7.MainWindow"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  Title="MainWindow" Height="350" Width="525">
  <RichTextBox x:Name="rtb" ContextMenu="{x:Null}">
    <FlowDocument>
      <Paragraph>line1</Paragraph>
      <Paragraph>line2</Paragraph>
      <Paragraph>line3</Paragraph>
    </FlowDocument>
  </RichTextBox>
</Window>
```
```c#
using System.Diagnostics;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Documents;

namespace WpfApplication7
{
  public partial class MainWindow : Window
  {
    public MainWindow()
    {
      InitializeComponent();
      rtb.ContextMenuOpening += new ContextMenuEventHandler(rtb_ContextMenuOpening);
    }

    void rtb_ContextMenuOpening(object sender, ContextMenuEventArgs e)
    {
      EditingCommands.MoveToLineStart.Execute(null, rtb);
      EditingCommands.SelectToLineEnd.Execute(null, rtb);
      if(Process.Start("notepad").WaitForInputIdle())
        System.Windows.Forms.SendKeys.SendWait(rtb.Selection.Text);
    }
  }
}
```