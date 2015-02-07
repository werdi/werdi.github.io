---
title: Как изменять логику обязательных полей
tags: [c#, winforms, compiler]
src: https://social.msdn.microsoft.com/Forums/ru-RU/93436d14-d7fc-4c78-8464-4b66a0992cbf/-?forum=fordesktopru
---
# Как изменять логику обязательных полей
*> логика описана непосредственно в коде приложения, в последнее время логику таких полей приходится всячески изменять в оперативном режиме, естественно приходится постоянно перекомпилировать само ПО*

можно создавать динамическую сборку.
примерно так:
```c#
using System;
using System.CodeDom;
using System.CodeDom.Compiler;
using System.Reflection;
using System.Windows.Forms;
using Microsoft.CSharp;

namespace WindowsFormsApplication3
{
  public partial class Form1 : Form
  {
    public Form1()
    {
      new Button { Dock = DockStyle.Top, Text = "test", Parent = this }
          .Click += (s, e) =>
            {
              Run(this);
            };
    }

    static void Run(Form frm)
    {
      var str = String.Concat(@"
          using System;
          using System.Windows.Forms;
          public static class Some {
              public static void Run(Form frm) {
                MessageBox.Show(""hello: "" + System.Reflection.Assembly.GetExecutingAssembly());
              }
          }");
      var a = Compile(str, "System.dll", "System.Windows.Forms.dll");
      var t = a.CompiledAssembly.GetType("Some");
      var m = t.GetMethod("Run", BindingFlags.Static | BindingFlags.Public);
      m.Invoke(null, new object[] { frm });
    }

    static CompilerResults Compile(string code, params string[] assemblies)
    {
      var csp = new CSharpCodeProvider();
      var ccu = new CodeCompileUnit();
      var cps = new CompilerParameters();
      cps.ReferencedAssemblies.AddRange(assemblies);
      cps.GenerateInMemory = true;
      return csp.CompileAssemblyFromSource(cps, code);
    }
  }
}
```
другой вариант: [Workflow Foundation] (http://msdn.microsoft.com/ru-ru/netframework/aa663328) (описать логику в xoml; подгружать его в runtime; для редактирования логики можно использовать редактор workflow, который встраивается в приложение)

*> По скольку версий программ много на разных компах, поэтому одномоментно, заменить файлы у всех не получится.*

на клиентах при запуске можно обращаться к серверу за сборкой.
если клиент уже работает, а сборка изменилась, то можно уведомить клиента. см. [wcf] (http://msdn.microsoft.com/ru-ru/netframework/aa663324)

*> какой из вариантов можете посоветовать Вы?*

на сервере создать wcf сервис (см. [WebServiceHost] (http://msdn.microsoft.com/ru-ru/library/system.servicemodel.web.webservicehost.aspx); пример [здесь] (http://social.msdn.microsoft.com/Forums/ru/fordataru/thread/bf8871c1-0cd2-4cad-a80e-32d1083a1dd0/#827f1e46-3034-4196-bc67-1ef79c030e93))
winforms-клиент обращается к сервису в момент загрузки, чтобы получить xoml или сборку.
если xoml или сборка на сервере изменится, то через wcf restful push service уведомить клиентов.
