---
title: Как найти свободный порт
tags: [linq, rest, tcp/ip, web]
src: http://cognitex.blogspot.ru/2009/04/blog-post_21.html 
---
# Как найти свободный порт
Для работы REST-сервисов требуется базовый адрес(а), например, http://localhost:8888, но бывает, что порт 8888 занят. В таком случае надо найти свободный порт. Сделать это можно следующим методом:
```c#
public static int FindFreePort()
{
    	// если порт занят, то его нельзя использовать с другим адресом.

    	var listerner = from ip in IPGlobalProperties.GetIPGlobalProperties().GetActiveTcpListeners()
            	select ip.Port;
    	var active = from ip in IPGlobalProperties.GetIPGlobalProperties().GetActiveTcpConnections()
            	select ip.LocalEndPoint.Port;
    	List<int> ports = new List<int>(listerner.Union(active).Distinct());
    	for (int port = 1024; port < 65535; port++)
    	{
        	if (ports.Contains(port) == false)
            	return port;
    	}
    	return -1;
}
```
Привилегированные (или зарезервированными) порты: 0 - 1023.
Зарегистрированные порты: 1024 – 49151.
Динамические порты: 49152 – 65535.

Еще один способ [здесь] (http://cognitex.blogspot.com/2009/07/v2.html).
