---
title: Using the scheduler
description: A guide to the Scheduler API within Velocity allowing tasks to be run.
slug: velocity/dev/scheduler-api
---

The Velocity scheduler lets you decide when and how your plugin tasks run, allowing fine control
over execution. On Velocity, there is no main thread. All tasks run using the Velocity scheduler are
thus run asynchronously.

## Running a delayed task

All scheduling works by using a [`TaskBuilder`](jd:velocity:com.velocitypowered.api.scheduler.Scheduler$TaskBuilder)
returned from the [`Scheduler`](jd:velocity:com.velocitypowered.api.scheduler.Scheduler).
This fluent builder may be chained to configure the details of the scheduling.

```java
server.getScheduler()
  .buildTask(plugin, () -> {
    // do stuff here
  })
  .delay(2L, TimeUnit.SECONDS)
  .schedule();
```

Here, we are scheduling a task to run 2 seconds later. Velocity requires the instance of your
plugin, `plugin` above. If you are scheduling a task from your main plugin class you may simply use
`this`.

Time arguments are specified as a `long` with a [`TimeUnit`](jd:java:java.util.concurrent.TimeUnit).
Using time units makes scheduling delayed tasks more readable and allows for greater precision.
`2L, TimeUnit.SECONDS` is far easier to understand than the ambiguous `2000L`.

You can also use a [`Duration`](jd:java:java.time.Duration) to specify the time arguments,
e.g.: `Duration.ofSeconds(5L)`.

## Running a repeating task

Creating a repeating task is similar to a delayed task, but you must also specify
[`repeat(long, TimeUnit)`](jd:velocity:com.velocitypowered.api.scheduler.Scheduler$TaskBuilder#repeat(long,java.util.concurrent.TimeUnit)).
This example will repeat every 5 minutes.

```java
server.getScheduler()
  .buildTask(plugin, () -> {
    // do stuff here
  })
  .repeat(5L, TimeUnit.MINUTES)
  .schedule();
```

## Running a task now

Tasks use the scheduler's cached thread pool for all execution, which reuses threads. To take
advantage of this thread pool for running async tasks which run now, simply omit calling the _delay_
and _repeat_ methods of the `TaskBuilder`.

## Cancellation

The [`schedule()`](jd:velocity:com.velocitypowered.api.scheduler.Scheduler$TaskBuilder#schedule())
method returns a [`ScheduledTask`](jd:velocity:com.velocitypowered.api.scheduler.ScheduledTask),
which may then be used to cancel the task involved via the
[`cancel()`](jd:velocity:com.velocitypowered.api.scheduler.ScheduledTask#cancel()) method.
Tasks cannot be uncancelled.

Additionally, [`task.status()`](jd:velocity:com.velocitypowered.api.scheduler.ScheduledTask#status())
returns the current status of the task.

```java
ScheduledTask task = server.getScheduler()
  .buildTask(plugin, () -> {
    // do stuff here
  })
  .repeat(5L, TimeUnit.MINUTES)
  .schedule();
// ...
task.cancel();
// ...
System.out.println(task.status());
```

You can also schedule _self-cancelling_ tasks using a `Consumer<ScheduledTask>`.

```java
AtomicInteger integer = new AtomicInteger(0);

ScheduledTask task = server.getScheduler()
  .buildTask(plugin, (selfTask) -> {
    // do stuff here, for example...
    if (integer.addAndGet(1) > 10) {
      selfTask.cancel();
    }
  })
  .repeat(Duration.ofSeconds(4L))
  .schedule();
```

## Obtaining tasks from a plugin

You can get all tasks scheduled by a plugin with
[`tasksByPlugin(Object)`](jd:velocity:com.velocitypowered.api.scheduler.Scheduler#tasksByPlugin(java.lang.Object)).

```java
Collection<ScheduledTask> tasks = server.getScheduler().tasksByPlugin(plugin);
// then you can control them, for example, cancel all task scheduled by a plugin
for (ScheduledTask task : tasks) {
  task.cancel();
}
```
