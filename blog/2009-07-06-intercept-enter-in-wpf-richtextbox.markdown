# WPF: Перехват нажатия Enter в RichTextBox
Дано:
```xaml
<UserControl x:Class="Editor.TextEditor"
    	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"    >
  	<RichTextBox x:Name="_Rtb"
    	VerticalScrollBarVisibility="Visible" Padding="5"
    	HorizontalScrollBarVisibility="Auto"  FontSize="13.5" FontFamily="Verdana" Background="#FFFADF" Foreground="Black" >
    	<FlowDocument>
      	<Paragraph>
        	Hello, World!
      	</Paragraph>
    	</FlowDocument>
  	</RichTextBox>
</UserControl>
```
Надо перехватить нажатие Enter. Для этого в конструкторе TextEditor пишем:
```c#
EventManager.RegisterClassHandler(typeof(RichTextBox), RichTextBox.KeyDownEvent, new KeyEventHandler(_Rtb_KeyDown));
```
где _Rtb_KeyDown - это:
```c#
void _Rtb_KeyDown(object sender, KeyEventArgs e)
{
    	if (e.Key == Key.Return)
    	{
    	}
}
```
P.S.
Обычный способ:
```c#
_Rtb.KeyDown += new KeyEventHandler(_Rtb_KeyDown);
```
не работает (= Enter не перехватывает).

P.P.S.
Нажатие Enter также можно перехватить с помощью события PreviewKeyDown:
```c#
_Rtb.PreviewKeyDown += new KeyEventHandler(_Rtb_PreviewKeyDown);
```
