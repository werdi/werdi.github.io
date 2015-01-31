---
title: C#. Treeview - afterselect, mysql
tags: [c#, mysql, winforms, data binding, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/969e7edc-7807-4bb8-bfb3-a5bb41b8d721/ctreeview-afterselect-mysql?forum=fordesktopru
---
# C#. Treeview - afterselect, mysql
*> нужно вместо строки string input = toolStripTextBox1.Text.Trim(); передать вinput фамилию выделенного в treeview сотрудника.*
```c#
var input = ((Person)treeView.SelectedNode).FamilyName;
```
*> в какой events мне разместить данный код, чтобы он выполнялся именно после выделения сотрудника, а не отдела или должности*
```c#
using System;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Windows.Forms;
namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 300);
      var tv = new TreeView { Dock = DockStyle.Fill, Parent = this };
      Fill(tv.Nodes);
      var d = new Details { Parent = this, Dock = DockStyle.Right, Width = 300 };
      tv.AfterSelect += (s, e) => d.Update(e.Node);
      tv.ExpandAll();
    }
    private void Fill(TreeNodeCollection nc)
    {
      var n1 = nc.Add("Отдел САПР").Nodes.Add("Инженер");
      n1.Nodes.Add(new Person { Text = "Иванов", Id = 1 });
      n1.Nodes.Add(new Person { Text = "Петров", Id = 2 });
      n1.Nodes.Add(new Person { Text = "Сидоров", Id = 3 });
    }
  }
  class Person : TreeNode
  {
    public int Id { get; set; }
  }
  class Details : UserControl
  {
    DataTable Data;
    TextBox Tb1;
    TextBox Tb2;
    DataGrid dg;
    public Details()
    {
      this.Data = new DataTable();
      Tb1 = new TextBox { Location = new Point(10, 10), Parent = this, Width = 200 };
      Tb2 = new TextBox { Location = new Point(10, 50), Parent = this, Width = 100 };
      dg = new DataGrid { Parent = this, Location = new Point(10, 100), Height = 150, Width = 270 };
    }
    public void Update(TreeNode treeNode)
    {
      var p = treeNode as Person;
      if (p == null)
        return;
      if (this.Data.Columns.Contains("id") == false || this.Data.DefaultView.Find(p.Id) == -1)
      {
        this.Fill(this.Data, p.Id);
        this.EnsureDataBinding();
      }
      // найти индекс строки, в которой есть указанный id; и сдвинуть указатель привязок;
      ((CurrencyManager)this.BindingContext[this.Data]).Position = this.Data.DefaultView.Find(p.Id);
    }
    void EnsureDataBinding()    // привязать контролы к DataTable
    {
      if (this.Tb1.DataBindings.Count > 0) 
        return;
      this.Tb1.DataBindings.Add("Text", this.Data, "val1");
      this.Tb2.DataBindings.Add("Text", this.Data, "val2");
      dg.DataSource = this.Data;
      this.Data.DefaultView.Sort = "id";   // для работы метода this.Data.DefaultView.Find;
    }
    void Fill(DataTable dt, int id)
    {
      // todo: заменить на код загрузки DataTable из базы данных
      //
      if(this.Data.Columns.Count == 0)
      {
        this.Data.Columns.Add("id", typeof(int));
        this.Data.Columns.Add("val1", typeof(string));
        this.Data.Columns.Add("val2", typeof(string));
      }
      this.Data.LoadDataRow(new object [] { id, DateTime.Now.Millisecond, id*10 }, true);
    }
  }
}
```

*> по событию afterselect должна отображаться панель [...] При нажатии на кнопку, к примеру, семья должны вывестись на экран данные из mysql*

каждый раз нажимать на кнопку может надоесть. но и загружать данные из базы при каждом AfterSelect тоже не подходит. в такой ситуации надо обращаться к базе данных с небольшой задержкой. примерно так: 
```c#
using System;
using System.Windows.Forms;
namespace WindowsFormsApplication1
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(400, 600);
      var tv = new TreeView { Dock = DockStyle.Fill, Parent = this };
      Fill(tv.Nodes);
      var deh = new Deh<Person>(500, p =>
      {
        if (p == null || p.Loaded) return;
        p.Loaded = true;
        p.Text += "- ok";
      });
      tv.AfterSelect += (s, e) => deh.Invoke(e.Node as Person);
      tv.ExpandAll();
    }
    private void Fill(TreeNodeCollection nc)
    {
      var n1 = nc.Add("Отдел САПР").Nodes.Add("Инженер");
      for (int i = 0; i < 25; i++)
        n1.Nodes.Add(new Person { Text = "фио" + i, Id = i });
    }
  }

  class Deh<T>     // Delayed Event Handler
  {
    Timer Timer;
    int Interval;
    Action<T> Action;
    public Deh(int Interval, Action<T> action)
    {
      this.Interval = Interval;
      this.Action = action;
    }
    public void Invoke(T v)
    {
      if (this.Timer != null)
        this.Timer.Stop();
  
      this.Timer = new Timer() { Interval = this.Interval };
      this.Timer.Tick += (s, e) =>
      {
        this.Timer.Stop();
        this.Action(v);
      };
      this.Timer.Start();
    }
  }

  class Person : TreeNode
  {
    public int Id { get; set; }
    public bool Loaded;
  }
}
```
