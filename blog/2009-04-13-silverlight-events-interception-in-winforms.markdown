---
title: Перехват событий Silverlight в WinForms
tags: [out-of-browser, silverlight, winforms, c#]
src: http://cognitex.blogspot.ru/2009/04/silverlight-winforms_13.html
---
# Перехват событий Silverlight в WinForms
Silverlight можно использовать в WinForms-приложении не только для вывода Xaml. Также можно перехватывать сообщения, которые создаются в Silverlight.

За основу возьмем пример из сообщения ["Silverlight в WinForms"] (http://cognitex.blogspot.com/2009/04/silverlight-winforms.html). Добавим в него неполное определение интерфейса IXcpControl (интерфейс наследует IDispatch, поэтому необязательно импортировать все свойства и методы):
```c#
[ComImport, Guid("1FB839CC-116C-4C9B-AE8E-3DBB6496E326"),
InterfaceType(ComInterfaceType.InterfaceIsIDispatch), ComVisible(true)]
public interface IXcpControl
{
    	[DispId(0x20)]
    	string Source { get; set; }

    	[DispId(0x2c)]
    	object OnLoad { get; set; }
}
```
И добавим наследника IReflect (используется в качестве посредника между Silverlight и WinForms; также можно использовать вместе с JavaScript):
```c#
public class Handler : IReflect
{
    	public EventHandler _EventHandler;
    	public Handler(EventHandler eh)
    	{
        	_EventHandler = eh;
    	}

    	[DispId(0)]
    	public void Method()
    	{
        	if (_EventHandler != null)
            	_EventHandler(this, EventArgs.Empty);
    	}

    	object IReflect.InvokeMember(string name, BindingFlags invokeAttr, Binder binder, object target, object[] args, ParameterModifier[] modifiers, System.Globalization.CultureInfo culture, string[] namedParameters)
    	{
        	if (name.Equals("[DISPID=0]", StringComparison.OrdinalIgnoreCase))
        	{
            	this.Method();
            	return null;
        	}
        	throw new MissingMethodException(base.GetType().Name, name);
    	}

    	PropertyInfo[] IReflect.GetProperties(BindingFlags bindingAttr) { return new PropertyInfo[0]; }
    	MethodInfo[] IReflect.GetMethods(BindingFlags bindingAttr) { return new MethodInfo[0]; }
    	FieldInfo[] IReflect.GetFields(BindingFlags bindingAttr) { return new FieldInfo[0]; }
    	MemberInfo[] IReflect.GetMembers(BindingFlags bindingAttr) { return new MemberInfo[0]; }
    	MethodInfo IReflect.GetMethod(string name, BindingFlags bindingAttr, Binder binder, Type[] types, ParameterModifier[] modifiers) { return null; }
    	FieldInfo IReflect.GetField(string name, BindingFlags bindingAttr) { return null; }
    	MemberInfo[] IReflect.GetMember(string name, BindingFlags bindingAttr) { return null; }
    	MethodInfo IReflect.GetMethod(string name, BindingFlags bindingAttr) { return null; }
    	PropertyInfo IReflect.GetProperty(string name, BindingFlags bindingAttr, Binder binder, Type returnType, Type[] types, ParameterModifier[] modifiers) { return null; }
    	PropertyInfo IReflect.GetProperty(string name, BindingFlags bindingAttr) { return null; }
    	Type IReflect.UnderlyingSystemType { get { return null; } }
}
```
Теперь надо создать форму и в ее конструктор поместить следующий код:
```c#
this.Size = new Size((int)(SystemInformation.WorkingArea.Width * .50f), 400);
string html = @"
    	<SCRIPT id='_Xaml' type='text/xaml'>
        	<?xml version='1.0' encoding='utf-8' ?>
        	<Canvas
            	xmlns='http://schemas.microsoft.com/winfx/2006/xaml/presentation'
            	xmlns:x='http://schemas.microsoft.com/winfx/2006/xaml'>
                	<TextBlock FontSize='40' Foreground='Red'>Hello, World!</TextBlock>
        	</Canvas>
    	</SCRIPT>
    	<OBJECT ID='_Ag' data='data:application/x-silverlight,' type='application/x-silverlight' Width='100%' Height='100%'>
        	<PARAM NAME='Background' VALUE='#FFFADF' />
    	</OBJECT>";
WebBrowser wb = new WebBrowser();
wb.DocumentCompleted += delegate
{
    	var ag = wb.Document.GetElementById("_Ag").DomElement as IXcpControl;
    	ag.OnLoad = new Handler(delegate { MessageBox.Show("Done!"); });
    	ag.Source = "#_Xaml";
};

wb.DocumentText = html;
wb.Dock = DockStyle.Fill;
wb.Parent = this;
```
WebBrowser загрузит html и вызовет обработчик события DocumentCompleted. К событию OnLoad будет подключен обработчик, который вызовется из Silverlight после завершения загрузки xaml.
