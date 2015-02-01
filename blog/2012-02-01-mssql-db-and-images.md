---
title: База MSSQL и картинки
tags: [c#, sql server, image, ef]
src: https://social.msdn.microsoft.com/Forums/ru-RU/62916518-0bb9-4356-9f90-d64b07f9fff6/-mssql-?forum=fordataru
---
# База MSSQL и картинки
*>  загрузить картинку в базу [...] SQL Express*

пример создания базы данных на основе EF Code First. 
загрузка изображения с диска в базу; чтение рисунка из базы и вывод в UI.
```c#
using System;
using System.Data.Entity;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Windows.Forms;
using System.Data.Common;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(500, 300);
      
      CreateImage("test.bmp");     // test image
      CreateDatabase("test.bmp", true);

      var d = new Data();
      new PictureBox { Parent = this, Dock = DockStyle.Fill }
        .Image = Image.FromStream(new MemoryStream(d.Pictures.First().Bytes));
    }
  
    static void CreateImage(string imgpath)
    {
      var bmp = new Bitmap(400, 200);   
      using(var g = Graphics.FromImage(bmp))
      using(var fnt = new Font("verdana", 12, FontStyle.Bold))
      {
        g.Clear(Color.Red);
        g.DrawString("Hello from database\n" + DateTime.Now, 
          fnt, Brushes.White, new PointF(10, 10));
      }
      bmp.Save(imgpath);
    }
  
    static void CreateDatabase(string imgpath, bool delete)
    {
      var d = new Data();
      if(delete) 
        d.Database.Delete();
      if (d.Database.CreateIfNotExists())
      {
        var img = File.ReadAllBytes(imgpath);
        var p = new Picture { Bytes = img };
        d.Pictures.Add(p);
        d.SaveChanges();
      }
    }
  }
  
  class Picture
  {
    public int Id { get; set; }
    public byte[] Bytes { get; set; }   // varbinary(MAX) в sql server;
  }
  
  class Data : System.Data.Entity.DbContext  // EntityFramework.dll
  {
    public DbSet<Picture> Pictures { get; set; }
    public Data() : base() { }   //  .\SQLEXPRESS
    public Data(DbConnection dc) : base(dc, true) { }
    protected override void OnModelCreating(DbModelBuilder m)
    {
      base.OnModelCreating(m);
      m.Entity<Picture>().HasKey(t => t.Id).ToTable("Pictures");
    }
  }
}
```
