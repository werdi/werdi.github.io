# Тестируем DTE
Предположим, что требуется протестировать открытие определенного проекта в Visual Studio. Для этого надо:
<ol>
  <li>создать UnitTest проект</li>
  <li>добавить в References: Microsoft.VSSDK.TestHostFramework (С:\Program Files\Microsoft Visual Studio 9.0\Common7\IDE\PublicAssemblies\Microsoft.VSSDK.TestHostFramework.dll)</li>
  <li> у тестового метода определить атрибут HostTypeAttribute:</li>
</ol>
```c#
[TestMethod()]
[HostType("VS IDE")]
public void InvokeTest()
{
    	var dte = VsIdeTestHostContext.Dte;
    	dte.Events.SolutionEvents.Opened += () =>
    	{
    	};
    	dte.ExecuteCommand("File.OpenProject", @"""C:\Projects\Test.csproj""");
}
```
В результате во время выполнения теста в отдельной Visual Studio откроется проект Test.csproj, а также будет вызван обработчик события Opened.

Если не указать HostTypeAttribute, то VsIdeTestHostContext.Dte вернет null.
Параметры для HostTypeAttribute определены в реестре: "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\9.0\EnterpriseTools\QualityTools\HostAdapters\"
