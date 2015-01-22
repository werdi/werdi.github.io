---
title: Silverlight 2.0. ServiceReferences.ClientConfig
tags: [silverlight, xml]
src: http://mindberg.blogspot.ru/2008/11/silverlight-20-servicereferencesclientc.html
---
# Silverlight 2.0. ServiceReferences.ClientConfig
Если в .ClientConfig-файле нет необходимых данных, то попытка обратиться к wcf-сервису из Silverlight 2.0 приводит к ошибке: System.Collections.Generic.KeyNotFoundException was unhandled by user code, Message="The given key was not present in the dictionary."
Пример правильного конфигурационного файла. 
[ServiceReferences.ClientConfig]
```xml
<configuration>
  <system.serviceModel>
    <bindings>
      <basicHttpBinding>
        <binding name="WSHttpBinding_IService1">
          <security mode="None" />
        </binding>
      </basicHttpBinding>
    </bindings>
    <client>
      <endpoint address="http://localhost:3646/Service1.svc" binding="basicHttpBinding"
        bindingConfiguration="WSHttpBinding_IService1" contract="ServiceReference1.IService1"
        name="WSHttpBinding_IService1" />
    </client>
  </system.serviceModel>
</configuration>
```
