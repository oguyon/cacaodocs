---
description: Using execution queues for advanced sequencing
---

# milk-fpsCTRL's execution queues

{% hint style="info" %}
Using execution queues is optional (but very powerful). You can skip this section if you don't need to sequence operations, or if you have another way to do it. Unless you change queues and priorities, all tasks will be in the queue #0, with priority=1, and will simply be executed in the order they are received (FIFO).
{% endhint %}

Tasks processed by milk-fpsCTRL's sequencer are arranged in execution queues. Each task belongs to a single queue, defined by an index (integer between 0 and 99). Each queue also has a priority index (index from 0 to 99).

## Rules

The sequencer decides which task to run according the following rules:

* Tasks with the higher priority value are executed first
* Priorities are associated to queues, not individual tasks. Changing a queue priority affects all tasks in the queue
* If queue priority = 0, no task is executed in the queue: it is paused
* Task order within a queue must be respected. Execution order is submission order (FIFO)
* Tasks can overlap if they belong to separate queues and have the same priority
* A running task waiting to be completed cannot block tasks in other queues
* If two tasks are ready with the same priority, the one in the lower queue index will be launched

## Conventions

* queue #0 is the main queue
* Keep queue 0 priority at 10
* Return to queue 0 when done working in other queues

