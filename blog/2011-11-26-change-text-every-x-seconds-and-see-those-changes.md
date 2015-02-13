---
title: Change Button Text every X seconds and see those changes
tags: [c#, winforms, html, webbrowser]
src: https://social.msdn.microsoft.com/Forums/ru-RU/a2f6e706-c1cd-475f-82fe-823c57db0219/change-button-text-every-x-seconds-and-see-those-changes-?forum=winforms
---
# Change Button Text every X seconds and see those changes
*> I have try the code but i have nothing , there are no numbers at the button :/Here is the code , Am i doing something wrong ?*

on the server you trying to operate with button which is in the web browser on client.
for changing button text on the client, the html should contains javascript. something like this: 
```
<script language='javascript'>
  var index = 0;
  setInterval(function () { btn.innerText = 'test' + (index++); }, 1000);
</script>
... 
<button id='btn'>test</button>
```
*> Can I read values from a array that is used in the form.aspx.cs , which is already defined ...
ArrayList lstNumbers = nbs.RandomNumbers(Total); ( 36 numbers )*

sure, if this numbers will be transferred to the client.
something like this:
```
<script language="javascript">
  var lstNumbers = [ 1, 2, 3, 4, 5 ];
  ...
```
below is an example (based on winforms) shows how it could be done
```c# 
using System.Windows.Forms;
using System.Collections;
using System.Collections.Generic;

namespace WindowsFormsApplication4
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      var arr = new List<int>();
      for (int i = 0; i < 20; i++)
        arr.Add(i);

      var jsarr = "[" + string.Join(", ", arr) + "]";
      var html = @"
      <html>
      <head>
        <script language='javascript'>
          var arr = " + jsarr + @";
          arr.ci = 0;   // current index
          var iid;       // intervalId
          var fn = function () { 
            if(arr.ci < arr.length) {
              res.innerHTML += arr[arr.ci++] + '; ';
            }
            else {
              clearInterval(iid);
              res.innerHTML += ' ok';
            }
          }
          window.onload = function() { iid = setInterval(fn, 250); };
        </script>
      </head>
      <body>
        <div id='res'></div>
      </body>
      </html>";
      new WebBrowser { Parent = this, Dock = DockStyle.Fill, DocumentText = html };
    }
  }
}
```