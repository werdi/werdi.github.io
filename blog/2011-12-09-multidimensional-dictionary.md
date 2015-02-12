---
title: Многомерный словарь
tags: [c#, winforms, linq, data]
src: https://social.msdn.microsoft.com/Forums/ru-RU/9bc6e894-5a7b-40fa-85d1-aa221d812637/-?forum=programminglanguageru
---
# Многомерный словарь
*> Нужен словарь, Dictionary, с меняющейся мерностью.*

слова можно объединять в группы 
и хранить в таблице в базе данных (создается автоматически на основе классов).
 
```c#
using System.Linq;
using System.Windows.Forms;
using System.Data.Entity;
...

class Word
{
  public int Id { get; set; }
  public int Group { get; set; }
  public string Value { get; set; }
}

class DataStore : DbContext
{
  public DbSet<Word> Words { get; set; }
  protected override void OnModelCreating(DbModelBuilder m)
  {
    base.OnModelCreating(m);
    m.Entity<Word>().HasKey(e => e.Id).ToTable("Words");
  }
}
...

// пример использования
var ds = new DataStore();
// автоматически создать базу данных
if (ds.Database.CreateIfNotExists())
{
  // для теста ...
  ds.Words.Add(new Word { Group = 1, Value = "w1" });
  ds.Words.Add(new Word { Group = 1, Value = "w2" });
  ds.Words.Add(new Word { Group = 2, Value = "w3" });
  // сохранить в базе 
  ds.SaveChanges();
}

// получить слова из группы 1
var res = from w in ds.Words where w.Group == 1 select w;

```