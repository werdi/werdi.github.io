---
title: Windows Forms и DataBindings
tags: [c#, winforms, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/7de49a57-ad49-4f19-b980-9948c00df434/windows-forms-databindings?forum=fordataru
---
# Windows Forms и DataBindings
*> свойство TestStringProp не реагирует на изменения в поле! Как это делаете вы?*

надо уведомить привязку о том, что данные изменились.
делается с помощью INotifyPropertyChanged.
```c#
public class Data : INotifyPropertyChanged
{
  public event PropertyChangedEventHandler PropertyChanged = delegate {};

  public string TestStringProp 
  {
    get {return _TestStringProp;}
    set
    {
      _TestStringProp = value;
      this.PropertyChanged(this, new PropertyChangedEventArgs("TestStringProp"));
    }
  }
  string _TestStringProp;
}
```  
другой способ: использовать TypeDescriptionProvider и вернуть свой PropertyDescriptor.
см. [здесь](http://social.msdn.microsoft.com/Forums/ru-RU/programminglanguageru/thread/3c68579f-a86f-4a9e-abf7-38e239907625/#dc739281-aae4-4b6d-bc8e-c6314ca931ba).

*> У меня же не работает прямая привязка: поменял в контроле - отразилось на объект.*

TextBox передает изменения в источник данных, если теряет фокус
или если у привязки указан DataSourceUpdateMode.OnPropertyChanged;

```c#
using System.ComponentModel;
using System.Drawing;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  public class Data : INotifyPropertyChanged
  {
    public event PropertyChangedEventHandler PropertyChanged = delegate { };

    public string TestStringProp
    {
      get { return _TestStringProp; }
      set
      {
        _TestStringProp = value;
        System.Diagnostics.Trace.WriteLine(value);
        this.PropertyChanged(this, new PropertyChangedEventArgs("TestStringProp"));
      }
    }
    string _TestStringProp;
	}

	public partial class Form1 : Form
	{
    public Form1()
    {
      this.Height = 100;
      new TextBox { Parent = this, Dock = DockStyle.Top }
       .DataBindings.Add("Text", new Data(), "TestStringProp")
       .DataSourceUpdateMode = DataSourceUpdateMode.OnPropertyChanged;
    }
  }
}
```
*> как биндить при несоответствии типов? Не могу связать TextBox и свойство с типом URI.*

с помощью событий Format и Parse.
```c#
using System;
using System.Data;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var dt = new DataTable();
      dt.Columns.Add("url", typeof(Uri));
      dt.Rows.Add(new Uri("http://social.msdn.microsoft.com"));
      new Button { Parent = this, Dock = DockStyle.Top, Text = "Show" }
        .Click += (s, e) => MessageBox.Show("" + dt.Rows[0]["url"]);

      var b = new Binding("Text", dt, "url", true);
      b.Format += (s, e) => e.Value = (e.Value as Uri).Host;
      b.Parse += (s, e) => e.Value = new Uri("http://" + e.Value as string + "?test");
      new TextBox { Parent = this, Dock = DockStyle.Top }.DataBindings.Add(b);
    }
  }
}
```
*> почему привязка TextBox.Lines к полю типа string[] не работает пока оно не станет неравным null?*

Binding подключается к указанному свойству контрола/объекта и узнает тип свойства. затем обращается к источнику данных, в ответ - null. 
так как тип свойства контрола/объекта может быть любым, например, WebBrowser, то Binding'у не знает как его создать, поэтому создается exception.

создать объект можно в обработчике события Format:
```c#
new TextBox { Parent = this, Dock = DockStyle.Fill, Multiline = true }
 .DataBindings.Add("Lines", BlankJobData, "QueryStrings", true,
                           DataSourceUpdateMode.OnValidation)
 .Format += (s, e) => 
   { 
     if (e.Value == null)  e.Value = new string[0]; 
   };
```