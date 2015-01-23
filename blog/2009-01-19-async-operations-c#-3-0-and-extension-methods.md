---
title: Асинхронные операции, C# 3.0 и Extension Methods
tags: [async operations, c#, extension methods]
src: http://mindberg.blogspot.ru/2008/01/c-30-extension-methods.html
---
# Асинхронные операции, C# 3.0 и Extension Methods
В WinForms Application есть ограничение: нельзя обращаться к методам контрола из потока, отличного от того, в котором был создан контрол. Т.е. если контрол был создан в потоке №1, затем был запущен поток №2 для длительной обработки данных, то из потока №2 нельзя обратиться к контролу – будет System.InvalidOperationException: «Cross-thread operation not valid: Control accessed from a thread other than the thread it was created on». 
Чтобы обойти это ограничение, у контролов существует метод BeginInvoke. Ниже пример использования BeginInvoke.
```c#
protected override void OnHandleCreated(EventArgs e)
{
  base.OnHandleCreated(e);
  Label lbl = new Label();
  lbl.Parent = this;
  lbl.Dock = DockStyle.Fill;
  // выполняется в основном/текущем потоке
  WaitCallback cb = delegate(object state)
  {
    lbl.Text = "Thread " + Thread.CurrentThread.ManagedThreadId + "; " + state + "; " + DateTime.Now;
  };
  // выполняется в рабочем потоке
  MethodInvoker mi = delegate()
  {
    for (int i = 0; i < 100; i++)
    {
      // "присоединиться" к основному потоку и передать в него данные
      lbl.BeginInvoke(cb, "Hello, from thread " + Thread.CurrentThread.ManagedThreadId);
      // имитация длительной обработки
      System.Threading.Thread.Sleep(1000);
    }
  };
  // запустить рабочий поток
  mi.BeginInvoke(null, null);
}
```
Этот же пример можно переписать по-другому, и «присоединение» к основному потоку перенести в делегат WaitCallback:
```c#
protected override void OnHandleCreated(EventArgs e)
{
  base.OnHandleCreated(e);
  Label lbl = new Label();
  lbl.Parent = this;
  lbl.Dock = DockStyle.Fill;
  // выполняется в основном/текущем потоке
  WaitCallback cb = null;
  cb = delegate(object state)
  {
    if (lbl.InvokeRequired)
    {
      // "присоединиться" к основному потоку и передать в него данные
      lbl.BeginInvoke(cb, state);
    }
    else
    {
      lbl.Text = "Thread " + Thread.CurrentThread.ManagedThreadId + "; " + state + "; " + DateTime.Now;
    }
  };
  // выполняется в рабочем потоке
  MethodInvoker mi = delegate()
  {
    for (int i = 0; i < 100; i++)
    {
      cb("Hello, from thread " + Thread.CurrentThread.ManagedThreadId);
      // имитация длительной обработки
      System.Threading.Thread.Sleep(1000);
    }
  };
  // запустить рабочий поток
  mi.BeginInvoke(null, null);
}
```
Теперь представим, что нам надо написать метод, который ничего не знает про контролы, при этом он должен создать рабочий поток, выполнить какую-то работу и затем вызвать callback в основном потоке. 
В такой ситуации поможет AsyncOperation:
```c#
public static void Method(WaitCallback cb)
{
  // "запомнить" основной/текущий поток
  AsyncOperation ao = AsyncOperationManager.CreateOperation(null);
  // будет вызван  в основном/текущем потоке
  SendOrPostCallback cbinner = delegate(object state)
  {
    cb(state);
  };
  // будет вызван из рабочего потока
  MethodInvoker mi = delegate()
  {
    // имитация длительной обработки
    System.Threading.Thread.Sleep(1000);
    // "присоединиться" к основному потоку (вызвать cbinner) и передать в него данные
    ao.Post(cbinner, "Hello, from thread " + Thread.CurrentThread.ManagedThreadId);
  };
  // запустить рабочий поток
  mi.BeginInvoke(null, null);
}
```
Пример использования метода Method:
(почти как в Comedy Club: – А как называется ваша книга? – Моя книга, называется «Книга» ... а как еще книга может называться? :)
```c#
protected override void OnHandleCreated(EventArgs e)
{
  base.OnHandleCreated(e);
  // создать контрол
  Label lbl = new Label();
  lbl.Dock = DockStyle.Fill;
  lbl.Parent = this;
  Method(delegate(object state)
  {
    lbl.Text = (string) state;
  });
}
```
В C# 3.0 вызов метода Method можно сократить и написать так:
```c#
Method(state =>
{
  lbl.Text = (string) state;
});
```
или так:
```c#
Method(state => lbl.Text = (string) state);
```
В C# 3.0 еще и не такое можно.
Например:
```c#
protected override void OnHandleCreated(EventArgs e)
{
  base.OnHandleCreated(e);
  // создать контрол
  Label lbl = new Label();
  lbl.Dock = DockStyle.Fill;
  lbl.Parent = this;
  lbl.Start(() =>
  {
    // имитация длительной обработки
    System.Threading.Thread.Sleep(1000);
    return "Hello, from thread " + Thread.CurrentThread.ManagedThreadId;
  },
  state =>    // в state находится значение, которое вернет return
  {
    lbl.Text = (string)state;
  });
}
```
Не пытайтесь найти описание метода Label.Start в MSDN, его там нет, и Visual Studio IntelliSense исправен. Дело в том, что Start определен как Extension Methods в отдельном классе.
Например:
(имя класса ни на что не влияет)
```c#
public static class AsyncOperationEx
{
  public static void Start(this Control c, Method mi, SendOrPostCallback cb)
  {
    AsyncOperation ao = AsyncOperationManager.CreateOperation(null);
    SendOrPostCallback cbinner = state => cb(state);
    MethodInvoker minner = () => ao.Post(cbinner, mi());
    minner.BeginInvoke(null, null);
  }
  public delegate object Method();
}
```
Когда я впервые увидел примеры с таким синтаксисом, подумал -  пора идти в садовники. Привык. Иногда бывает, конечно, взгляд прилипает ненадолго к строке кода, но это не страшно .... то ли еще будет .... c F#.
