---
title: Simulate typing in webbrowser form fields, how?
tags: [c#, winforms, webbrowser, html]
src: https://social.msdn.microsoft.com/Forums/ru-RU/39d18e30-d300-407b-8f54-82d87a926e81/simulate-typing-in-webbrowser-form-fields-how?forum=csharpgeneral
---
# Simulate typing in webbrowser form fields, how?
*> Is there a way i can simulate typing in the webbrowser control?*

an example is [here] (http://social.msdn.microsoft.com/Forums/us-en/csharpgeneral/thread/57a15dfe-6523-48ae-877d-117b25f9a829/#d4b4ac28-b69a-464e-9dc5-eed5219234bc)  (automate WebBrowser through HTML DOM)
if you need to automate IE, then example is [here] (http://social.msdn.microsoft.com/Forums/en-US/csharpgeneral/thread/0822751b-6bcf-4a84-9998-3be1de11f625/#6032b44a-bb71-4c27-9520-68355712c687)

*> That is not really simulating typing, its just setting value*

try using SendKeys Class.
below is an example
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var wb = new WebBrowser 
      { 
        Parent = this, 
        Dock = DockStyle.Fill,
        DocumentText = "<body><input id='inp' type='text'></input>"
      };
      this.Menu = new MainMenu();
      int i = 0;
      this.Menu.MenuItems.Add("SendKey", (s, e) => 
        {
          wb.Document.GetElementById("inp").Focus();
          SendKeys.Send(i++.ToString());
        });
    }
  }
}
```
*> Is there anyway i can execute the javascript after setting value?*

yes, you can use SendKeys.SendWait and InvokeScript.
below is an example.
```c#
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var wb = new WebBrowser 
      { 
        Parent = this, 
        Dock = DockStyle.Fill,
        DocumentText = @"
        <html>
          <head>
            <script>
              function test() { 
                inp.value += ';'; 
              } 
            </script>
          </head>
          <body>
            <input id='inp' type='text'></input>
          </body>
        </html>"};
    this.Menu = new MainMenu();
    int i = 0;
    this.Menu.MenuItems.Add("SendKey", (s, e) => 
      {
        wb.Document.GetElementById("inp").Focus();
        SendKeys.SendWait(i++.ToString());
        wb.Document.InvokeScript("test");
      });
    }
  }
}
```
*> <input type="checkbox" name="regagree" onclick="checkAgree();" id="regagree" class="check"> How do i execute the OnClick?*

you can use RaiseEvent Method.
below is an example
```c# 
using System.Windows.Forms;

namespace WindowsFormsApplication2
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var wb = new WebBrowser 
      { 
        Parent = this, 
        Dock = DockStyle.Fill,
        DocumentText = @"
        <html>
          <head>
            <script>
              function test() { 
                inp.value += ';'; 
              } 
              function checkAgree() {
                alert('ok');
              }
            </script>
          </head>
          <body>
            <input id='inp' type='text'></input>
            <input id='regagree' type='checkbox' onclick='checkAgree()'></input>
          </body>
        </html>"};
      this.Menu = new MainMenu();
      int i = 0;
      this.Menu.MenuItems.Add("SendKey", (s, e) => 
        {
          wb.Document.GetElementById("inp").Focus();
          SendKeys.Send(i++.ToString());
          wb.Document.InvokeScript("test");
          wb.Document.GetElementById("regagree").RaiseEvent("onclick");
        });
    }
  }
}
```
