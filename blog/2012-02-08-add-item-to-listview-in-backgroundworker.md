---
title: Add item to listview in backgroundworker without refreshing the listview
tags: [c#, winforms, graphics]
src: https://social.msdn.microsoft.com/Forums/ru-RU/5bcd694e-dc57-4a53-babf-a1ace976ed9f/add-item-to-listview-in-backgroundworker-without-refreshing-the-listview?forum=winforms
---
# Add item to listview in backgroundworker without refreshing the listview
*> I have loaded many images in a listview using ImageList in c#. [...] So I call the method in backgroundworker [...] But the problem is the listview is continuously refreshing when a new item is added.*

use ListView with ControlStyles.OptimizedDoubleBuffer 

below is an example
```c#
using System.ComponentModel;
using System.Drawing;
using System.IO;
using System.Threading;
using System.Windows.Forms;

namespace WindowsFormsApplication5
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.Size = new System.Drawing.Size(600, 500);
      var il = new ImageList { ImageSize = new Size(50, 50) };
      var lv = new ListViewExt { Parent = this, Dock = DockStyle.Fill, LargeImageList = il, View = View.LargeIcon };

      var w = new BackgroundWorker { WorkerReportsProgress = true };
      w.ProgressChanged += (s, e) =>
      {
        il.Images.Add((Image)e.UserState);
        lv.Items.Add(new ListViewItem { ImageIndex = il.Images.Count-1 });
      };
      w.DoWork += delegate
      {
        foreach (var fi in Directory.EnumerateFiles(@"C:\Images", "*.jpg", SearchOption.AllDirectories))
        {
          var img = Image.FromFile(fi);
          w.ReportProgress(0, img);
        }
      };
      w.RunWorkerAsync();
    }
  }
  
  class ListViewExt : ListView
  {
    public ListViewExt()
    {
      this.SetStyle(ControlStyles.OptimizedDoubleBuffer, true);
    }
  }
}
```
