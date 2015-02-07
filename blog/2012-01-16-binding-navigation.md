---
title: Binding navigation problem 
tags: [c#, winforms, data binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f06bcfbb-5e1a-480e-b91f-bbf3ff025b2a/binding-navigation-problem?forum=winformsdatacontrols 
---
# Binding navigation problem 
*> i have two radio button male and female and i don't know how to add binding to the radio buttons*

you can bind two radio buttons to the one property with Binding.Parse and Binding.Format events
below is an example
```c#
using System.ComponentModel;
using System.Windows.Forms;
using System.Drawing;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public bool Test { get; set; }

    public Form1()
    {
      this.Padding = new Padding(10);

      // test: label
      new Label { Parent = this, Location = new Point(10, 70) }
        .DataBindings.Add("Text", this, "Test")
        .Format += (s, e) => e.Value = "this.Test=" + e.Value;

      // radiobutton 1
      var b = new Binding("Checked", this, "Test", true, DataSourceUpdateMode.OnPropertyChanged);
      b.Parse += (s, e) => e.Value = !((bool)e.Value);
      b.Format += (s, e) => e.Value = !((bool)e.Value);
      new RadioButton { Parent = this, Dock = DockStyle.Top, Text = "False" }
        .DataBindings.Add(b);

      // radiobutton 2
      new RadioButton { Parent = this, Dock = DockStyle.Top, Text = "True" }
        .DataBindings.Add("Checked", this, "Test");

      this.Test = true;

      // test: button for toggle
      new Button { Parent = this, Location = new Point(10, 100), Text = "Toggle" }
        .Click += (s, e) => TypeDescriptor.GetProperties(this)["Test"].SetValue(this, !this.Test);
    }
  }
}
```

WPF-example is [here] (http://social.msdn.microsoft.com/Forums/en-us/wpf/thread/64765fe5-ff98-445c-879d-798aeecbefff/#6806c674-119a-4051-9ba1-2e455b24d755)
