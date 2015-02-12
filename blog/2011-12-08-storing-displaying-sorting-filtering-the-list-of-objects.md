---
title: Хранение, отображение, сортировка, фильтрация списка объектов
tags: [c#, winforms, linq, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/b542e6c7-0fd9-4d12-a584-00623291c0d6/-?forum=fordataru
---
# Хранение, отображение, сортировка, фильтрация списка объектов
*> предлагают создать структуру в DataTable похожую на мой класс и потом производить с ним эти самые фильтрации и выборки. Но мне нужны не только хранимые поля, но и методы их обработки и статические данные [...] планирую потом эти данные еще и отображать (со всеми последствиями мигание при перерисовке, "сбивание" курсора)*

вместо DataTable можно использовать BindingList.
данные можно привязать к DataGridView. работает без проблем.
ниже пример привязки 100 тыс. элементов.
```c#
using System.ComponentModel;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  class Data
  {
    public string Value1 { get; set; }
    public string Value2 { get; set; }
  }

  public partial class Form1 : Form
  {
    public Form1()
    {
      var bl = new BindingList<Data>();
      for (int i=0; i < 100000; i++) bl.Add(new Data { Value1 = i.ToString() });
      new DataGridView { Parent = this, Dock = DockStyle.Fill, DataSource = bl };
    }
  }
}

```
*> У биндинг листа при привязке к DataGridView не работает фильтрация и сортировка. Нужно реализовывать какой-то свой интерфейс*

в linq есть OrderBy и OrderByDescending. 
ниже пример, который хранит данные в базе данных sql server (express версия бесплатна).
база данных создается автоматически.   
в примере есть фильтрация и сортировка. есть возможность редактирования значений в DataGridView.
изменения сохраняются в базе данных при закрытии формы.
```c#
using System.Data.Entity;
using System.Linq;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  class Item
  {
    public int Id { get; set; }
    public string Value { get; set; }
  }
  class DataStore : DbContext
  {
    public DbSet<Item> items { get; set; }
    protected override void OnModelCreating(DbModelBuilder m)
    {
      base.OnModelCreating(m);
      m.Entity<Item>().HasKey(e => e.Id).ToTable("Items");
    }
  }

  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataStore();
      if (ds.Database.CreateIfNotExists())
      {
        for (int i = 0; i < 30; i++)
          ds.items.Add(new Item { Value = "v" + i });
        ds.SaveChanges();
      }
      var res = from x in ds.items where x.Value.EndsWith("2") select x;  // фильтрация
      var dg = new DataGridView
      {
        Parent = this,
        Dock = DockStyle.Fill,
        AutoGenerateColumns = false,
        AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill,
        DataSource = res.ToList()
      };
      dg.Columns.AddRange(
        new DataGridViewTextBoxColumn
        {
          HeaderText = "Value",
          DataPropertyName = "Value",
          SortMode = DataGridViewColumnSortMode.Programmatic
        });
      this.FormClosing += (s, e) => ds.SaveChanges();
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("sort asc", delegate { dg.DataSource = res.OrderBy(i => i.Value).ToList(); });
      this.Menu.MenuItems.Add("sort desc", delegate { dg.DataSource = res.OrderByDescending(i => i.Value).ToList(); });
    }
  }
}
```