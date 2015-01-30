---
title: Question on Cross Threading Marshalling.
tags: [c#, threading]
src: https://social.msdn.microsoft.com/Forums/ru-RU/46c4c99c-98d8-4958-9927-7d8fcc350cc4/question-on-cross-threading-marshalling?forum=parallelextensions
---
# Question on Cross Threading Marshalling.
*> I believe it has something to do with which TaskScheduler things are run on, but am not sure. Advice?*
```c#
Task.Factory.StartNew(() =>  // worker thread
{
  var oc = new ObservableCollction<string>();
  oc.Add("Test");
  return oc;
})
.ContinueWith(task =>  // ui thread
{
  this.DataContext = task.Result;
}
, TaskScheduler.FromCurrentSynchronizationContext());
```
another approach, based on AsyncOperations, is [here] (http://social.msdn.microsoft.com/Forums/en-us/wpf/thread/a8fbb994-9ddf-4df1-96a3-83c7bcfe05f2/#fa749837-eeb8-43b8-b2d0-5163e1abb665).
