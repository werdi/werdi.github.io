---
title: C# writing XML entry
tags: [c#, xml, winforms, binding]
src: https://social.msdn.microsoft.com/Forums/ru-RU/f65fc802-4b29-4bc6-bab4-f535a459cc18/c-writing-xml-entry?forum=winforms
---
# C# writing XML entry
*> doc.DocumentElement.AppendChild(Subfaqs);//faqs  doc.DocumentElement.AppendChild(Subgic);//GIC             doc.DocumentElement.AppendChild(Neworder);//Topics*

before   doc.Save("C:\\GICorder.xml");   write the following:
```
doc.DocumentElement.AppendChild(Neworder);//Topics
Subgic.AppendChild(Subfaqs);//faqs
Neworder.AppendChild(Subgic);//GIC
```
*> i want to direct it to be above the first one since it is about "Apple".*

try to use InsertBefore instead of AppendChild

*> The gFAQCombobox shows "Apple Pie" and the gFAQrichTextBox shows "add 1 cups of milk and syrup + apple concentrate and slices + stir for 45 minutes @ 235." in 3 lines respectively.*

if you'll change schema of your xml to the following:
```xml   
<?xml version="1.0" encoding="utf-8" ?>
<root>
  <FAQ>
    <QUESTION>Oranges Juice</QUESTION>
    <ANSWER index='1'>add 2 cups of water</ANSWER>
    <ANSWER index='2'>orange concentrate</ANSWER>
    <ANSWER index='3'>stir for 2 minutes.</ANSWER>
  </FAQ>
  <FAQ>
    <QUESTION>Apple Pie</QUESTION>
    <ANSWER index='1'>add 1 cups of milk and syrup</ANSWER>
    <ANSWER index='2'>apple concentrate and slices</ANSWER>
    <ANSWER index='3'>stir for 45 minutes @ 235.</ANSWER>
  </FAQ>
</root>
``` 
then you'll can use databindings. and have a sorting in combobox:
```c#   
using System.Data;
using System.Text;
using System.Windows.Forms;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var ds = new DataSet();
      ds.ReadXml("test.txt");
      var bs = new BindingSource() { DataSource = ds, DataMember = "FAQ" };
      new DataGrid { Parent = this, Dock = DockStyle.Fill, DataSource = bs };
      var b = new RichTextBox { Parent = this, Dock = DockStyle.Top }
                .DataBindings.Add("Text", bs, "QUESTION", true, DataSourceUpdateMode.Never);
      b.Format += (s, e) =>
      {
        var bc = this.BindingContext[ds, "FAQ.FAQ_ANSWER"] as CurrencyManager;
        var sb = new StringBuilder();
        foreach (dynamic c in bc.List)
          sb.Append(c["ANSWER_Text"]).Append("\n");
        e.Value += sb.ToString();
      };
      new ComboBox { Parent = this, Dock = DockStyle.Top, DropDownStyle = ComboBoxStyle.DropDownList, 
        DataSource = bs, DisplayMember = "QUESTION" };

      this.Menu = new MainMenu();
      this.Menu.MenuItems.Add("sort ASC", (s, e) => bs.Sort = "QUESTION ASC");
      this.Menu.MenuItems.Add("sort DESC", (s, e) => bs.Sort = "QUESTION DESC");
    }
  }
}
```