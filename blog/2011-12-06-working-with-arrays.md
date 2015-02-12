---
title: Работа с массивами
tags: [c#, winforms]
src: https://social.msdn.microsoft.com/Forums/ru-RU/4222a425-08a6-402c-b692-c40552a07879/-?forum=programminglanguageru
---
# Работа с массивами
*> объявляю массив в классе и хочу чтобы он был виден всем функциям этого класса [...] передаю функции этого класса заполненный массив из вне класса и пытаюсь перезаписать эти значения*

в примере в форме Check определен массив mas2 (виден из все методов определенных в Check).
 форма Form1 - для теста; создает Check и передает в нее массив, заполненный случайными значениями.
```c#
using System;
using System.Windows.Forms;

namespace WindowsFormsApplication9
{
  static class Program
  {
    [STAThread]
    static void Main()
    {
      Application.Run(new Form1());
    }
  }

  class Check : Form
  {
    int[] mas2;
    Label lbl;
    public Check(int nubmer)
    {
      mas2 = new int[nubmer];
      lbl = new Label { Parent = this, Dock = DockStyle.Top };
    }
    public void st(int[] mas)
    {
      Array.Copy(mas, mas2, mas.Length);
      lbl.Text = String.Join("; ", mas2);
    }
  }

  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("Test", (s, e) =>
        {
          var c = new Check(10);
          var a = Create(10);
          c.st(a);
          c.Show();
        });
    }
    int[] Create(int length)
    {
      var rnd = new Random();
      var a = new int[length];
      for (int i = 0; i < length; i++) a[i] = rnd.Next(0, 100);
      return a;
    }
  }
}
```