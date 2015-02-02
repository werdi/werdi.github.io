---
title: WMI и трафик
tags: [wmi, html, hta]
src: https://social.msdn.microsoft.com/Forums/ru-RU/2c656bd5-f64d-470a-8203-57b33ff3a151/wmi-?forum=desktopprogrammingru
---
# WMI и трафик
*> Где, если это возможно, в WMI узнать общее количество скачанного и отданного [...] нужно на Яваскрипте*

см. [Win32_PerfRawData_Tcpip_NetworkInterface] (http://msdn.microsoft.com/en-us/library/windows/desktop/aa394340)

[test.hta]
```html
<html>
<head />
<script type="text/javascript">
window.resizeTo(500, 600);
var wmi = GetObject("winmgmts:{impersonationLevel=impersonate}");
function getData()
{
  var en = new Enumerator(wmi.InstancesOf("Win32_PerfRawData_Tcpip_NetworkInterface"));
  var ret = "";
  for (; !en.atEnd(); en.moveNext())
  {
   var itm = en.item();
   ret += itm.Name
    + "<br/>BytesReceivedPerSec=" + itm.BytesReceivedPerSec
    + "<br/>BytesSentPerSec=" + itm.BytesSentPerSec
    + "<br/>BytesTotalPerSec=" + itm.BytesTotalPerSec
    + "<br/>CurrentBandwidth=" + itm.CurrentBandwidth 
    + "<hr/>";
   }
   return ret;
}
window.onload = function ()
{
  setInterval(function () { document.body.innerHTML = getData(); }, 1000);
}
</script>
<body />
</html>
```
