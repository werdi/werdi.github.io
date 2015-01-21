---
title: Как изменить поведение RichTextBox при нажатии Enter
tags: [richtextbox, wpf, xaml, c#]
src: http://cognitex.blogspot.ru/2009/07/richtextbox-enter.html
---
# Как изменить поведение RichTextBox при нажатии Enter
Обычно при нажатии Enter в RichTextBox создается новый параграф, который отделяется от предыдущего пустой строкой. Например, если набрать Hello, World!, поместить курсор перед World и нажать Enter, то получим:
Hello,

World! 
При этом в RichTextBox.Document будет следующий xaml:
```xaml
<FlowDocument>
  	<Paragraph xml:space="preserve">Hello, </Paragraph>
  	<Paragraph>World!</Paragraph>
</FlowDocument>
```
Чтобы избавиться от пустой строки можно подписаться на событие PreviewKeyDown и перехватывать момент нажатия:
```c#
void _Rtb_PreviewKeyDown(object sender, KeyEventArgs e)
{
    	if (e.Key == Key.Return)
    	{
        	_Rtb.CaretPosition = _Rtb.CaretPosition.InsertLineBreak();
        	e.Handled = true;
    	}
}
```
Другой способ - определить стиль. Для этого в RichTextBox надо добавить FlowDocument и FlowDocument.Resources:
```xaml
<FlowDocument>
  	<FlowDocument.Resources>
    	<Style TargetType="Paragraph">
      	<Setter Property="Margin" Value="0" />
      	<Setter Property="Padding" Value="0" />
    	</Style>
  	</FlowDocument.Resources>
</FlowDocument>
```
