---
title: Как лучше сделать демо версию?
tags: [c#, linq, wmi]
src: https://social.msdn.microsoft.com/Forums/ru-RU/1917cd51-9c46-45de-bc50-dde320c2e520/-wpf?forum=fordesktopru
---
# Как лучше сделать демо версию?
*> как можно получить уникальный код железа пользователя?*

с помощью WMI можно получить параметры железа 
см. [Computer System Hardware Classes](http://msdn.microsoft.com/en-us/library/windows/desktop/aa389273(v=VS.85).aspx)

например, чтобы получить версию BIOS надо:
подключить к проекту сборку System.Management.dll
```c#
using System.Collections.Generic; 
using System.Linq;
using System.Management;
...

var res = from mo in WmiSearch("SELECT * FROM Win32_BIOS")
          select mo.GetPropertyValue("Version");
var version = res.First();
...

public static IEnumerable<ManagementObject> WmiSearch(string query)
{
  var searcher = new ManagementObjectSearcher(query);
  foreach (ManagementBaseObject mbo in searcher.Get())
  {
    var mo = (mbo as ManagementObject);
    if (mo != null)
      yield return mo;
  }
}
```
*> как получить параметры не биоса а жесткого диска?*

для всех физических дисков вернет имя и серийный номер:
 (код метод WmiSearch [здесь](http://social.msdn.microsoft.com/Forums/ru-RU/fordesktopru/thread/1917cd51-9c46-45de-bc50-dde320c2e520/#3981d573-92a9-4527-a900-f513c2c5838d))
```c#
foreach(var mo in WmiSearch("SELECT * FROM Win32_DiskDrive"))
{
  var sn = mo.GetPropertyValue("SerialNumber");
  var n = mo.GetPropertyValue("Name");
  System.Diagnostics.Trace.WriteLine(n + " " + sn);
}
```
список других параметров [здесь](http://msdn.microsoft.com/en-us/library/windows/desktop/aa394132(v=VS.85).aspx). 