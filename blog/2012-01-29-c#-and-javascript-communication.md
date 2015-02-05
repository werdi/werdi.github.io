---
title: C# and javascript communication?
tags: [c#, javascript, linq, winforms, ie, automation]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d4d8655f-4cb2-42ef-9966-8c1cf8ce2e25/c-and-javascript-communication?forum=csharpgeneral 
---
# C# and javascript communication?
*> I want communicate between C# desktop app and javascript [...] C# app have a button, when that button is pushed, call the hello function in the page that write hello world.*

you can use UI Automation.
below is an example

create a winforms project; and add references: 
'C:\Program Files\Reference Assemblies\Microsoft\Framework\v3.0\UIAutomationClient.dll'
'C:\Program Files\Reference Assemblies\Microsoft\Framework\v3.0\UIAutomationTypes.dll'
```c#
using System;
using System.Diagnostics;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Threading;
using System.Windows.Automation;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      this.TopMost = true;
      start = new Button { Text="Start", Location= new Point(10, 10), Parent = this };
      start.Click += Start_Click;
      press = new Button { Text="Press", Enabled = false, Location= new Point(10, 50), Parent = this };
      press.Click += Press_Click;
    }

    Button press;
    Button start;

    private void Press_Click(object sender, EventArgs e)
    {
      try
      {
        _Js.Invoke();
      }
      catch (Exception exc)
      {
        press.Enabled = false;
        start.Enabled = true;
        MessageBox.Show(exc.Message);
      }
    }

    InvokePattern _Js;

    private void Start_Click(object sender, EventArgs e)
    {
      start.Enabled = false;
      var file = @"C:\Users\" + SystemInformation.UserName + @"\AppData\LocalLow\Temp\testpage.htm";
      File.WriteAllText(file, @"
        <html>
        <head>
        <script type=""text/javascript"">
          function test() {
              document.getElementById(""demo"").innerHTML= ""hello world "" + new Date(); }
        </script>
        </head>
        <body>
          <button onclick=""test()"">js button</button>
          <p id=""demo"">...</p>
        </body>
        </html>");

      var p = Process.Start(@"C:\Program Files\Internet Explorer\iexplore.exe", file);
      while (p.MainWindowHandle == IntPtr.Zero) Thread.Sleep(10);
      p.WaitForInputIdle(30000);

      _Js = AutomationElement.FromHandle(p.MainWindowHandle)
              .FindAll(TreeScope.Descendants, new AndCondition(
                  new PropertyCondition(AutomationElement.NameProperty, "js button"),
                  new PropertyCondition(AutomationElement.IsInvokePatternAvailableProperty, true)))
              .OfType<AutomationElement>()
              .Select(c => c.GetCurrentPattern(InvokePattern.Pattern) as InvokePattern)
              .FirstOrDefault();

      if (_Js != null)
        press.Enabled = true;
    }
  }
}
```
