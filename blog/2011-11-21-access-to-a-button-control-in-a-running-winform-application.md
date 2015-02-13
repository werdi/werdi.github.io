---
title: I Need to access a Button control in a running WinForm application from my new WinForm application
tags: [c#, winforms, linq, automation]
src: https://social.msdn.microsoft.com/Forums/ru-RU/d58bbbc4-e18d-4206-b5b2-32c2fbed1380/my-first-use-for-ui-automation-i-need-to-access-a-button-control-in-a-running-winform-application?forum=windowsaccessibilityandautomation
---
# I Need to access a Button control in a running WinForm application from my new WinForm application
*> I Need to access a Button control in a running WinForm application from my new WinForm application*

try the following example where form2 finds the process with form1 and invokes button.
```c#
using System;
using System.Diagnostics;
using System.Drawing;
using System.Linq;
using System.Windows.Automation;
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new Button { Parent = this, Text = "Test" }
        .Click += (s, e) =>
        {
          var res = new StackTrace(1).GetFrames()
            .Where(f => f.GetMethod().Name.Contains("IAccessible"))
            .FirstOrDefault();
          MessageBox.Show(res != null ? "Tested" : "Denied");
        };

      new Button { Parent = this, Text = "Open Form2", Location = new Point(0, 50) }
        .Click += delegate
        {
          var frm = new Form();
          new Button { Parent = frm, Text = "Form1.Click" }
            .Click += (s, e) =>
            {
              var p = System.Diagnostics.Process.GetProcesses()
                .Where(c => c.ProcessName.StartsWith("WindowsFormsApplication2")
                  && c.MainWindowHandle != IntPtr.Zero)
                .FirstOrDefault();

              if (p == null) return;

              var ae = AutomationElement.FromHandle(p.MainWindowHandle);
              var tst = ae.FindAll(System.Windows.Automation.TreeScope.Descendants, new AndCondition(
                new PropertyCondition(AutomationElement.NameProperty, "Test"),
                new PropertyCondition(AutomationElement.IsInvokePatternAvailableProperty, true)))
                  .OfType<AutomationElement>()
                  .Select(c => c.GetCurrentPattern(InvokePattern.Pattern) as InvokePattern)
                  .FirstOrDefault();

              if (tst != null)
                  tst.Invoke();
            };
          frm.ShowDialog();
        };
    }
  }
}
```