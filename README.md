# 🎭 Behavioral Design Patterns

## Understanding Patterns Through a To-Do List Application

## Table of Contents

- [Introduction](#introduction)
- [What are Behavioral Patterns?](#what-are-behavioral-patterns)
- [Our Example: To-Do List Application](#our-example-to-do-list-application)
- [Pattern 1: Observer Pattern](#pattern-1-observer-pattern)
- [Pattern 2: Strategy Pattern](#pattern-2-strategy-pattern)
- [Pattern 3: Command Pattern](#pattern-3-command-pattern)
- [Best Practices](#best-practices)
- [Summary](#summary)

---

## Introduction

In our previous lesson, we learned about **Creational Patterns** - patterns that help us create objects smartly. Now we'll explore **Behavioral Patterns** - patterns that help objects communicate and work together effectively.

> **"Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects."** - Gang of Four

### Why Learn Behavioral Patterns?

- ✅ **Better Communication** - Objects interact in flexible, maintainable ways
- ✅ **Loose Coupling** - Components don't need to know too much about each other
- ✅ **Flexible Workflows** - Easy to change behavior without changing code
- ✅ **Reusable Actions** - Encapsulate behaviors that can be swapped or reused

---

## What are Behavioral Patterns?

**Behavioral patterns** focus on how objects communicate and distribute responsibilities.

When building applications, objects need to work together. Without proper patterns, you might end up with:

- Objects tightly coupled to each other (hard to change one without breaking others)
- Duplicate code for similar behaviors
- No way to undo actions
- Notifications scattered everywhere

Behavioral patterns solve these problems by providing structured ways for objects to interact and share responsibilities.

### The Three Patterns We'll Learn:

| Pattern      | Purpose                                    | Problem It Solves                                          | Real-World Analogy                                            |
| ------------ | ------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------- |
| **Observer** | Notify multiple objects when things change | Hard-coded notifications, tight coupling                   | YouTube subscriptions - you get notified when channels upload |
| **Strategy** | Swap algorithms/behaviors at runtime       | Complex if-else logic, rigid behavior                      | Different payment methods (cash, card, online)                |
| **Command**  | Encapsulate actions as objects             | No undo/redo, actions scattered, hard to log/queue actions | TV remote - each button is a command object                   |

---

## Our Example: To-Do List Application

We'll continue building our **To-Do List Application** from the creational patterns lesson. Now we'll add behavioral capabilities!

```
📱 To-Do List App (Extended)
├── 📋 Task Manager (Singleton)
├── 🏭 Notification Factory (Factory Pattern)
├── 👷 Task Builder (Builder Pattern)
│
└── NEW BEHAVIORAL FEATURES:
    ├── 👀 Task Observers (Observer Pattern)
    ├── 🔄 Sorting Strategies (Strategy Pattern)
    └── ⏮️ Undo/Redo Commands (Command Pattern)
```

### What We're Adding:

- **Observers** to notify users when tasks change (completed, overdue, etc.)
- **Strategies** to sort/filter tasks in different ways
- **Commands** to implement undo/redo for task operations

---

## Pattern 1: Observer Pattern

### 🎯 **Purpose**

Establish a one-to-many relationship where multiple objects (observers) automatically get notified when another object (subject) changes state.

### 📖 **Concept in Plain English**

Imagine you subscribe to a YouTube channel (like Jacksepticeye). When that channel uploads a video, YouTube notifies ALL subscribers automatically. You don't need to constantly check "has this channel uploaded yet?", but instead the notification comes to you.

In our to-do app: When a task changes (completed, priority changed, deadline approaching), we want to notify everyone interested - the UI, notification system, achievement tracker, etc.

**Without Observer:** Each place (i.e. file) that changes a task must manually call every notification method.
**With Observer:** The task itself notifies all registered observers automatically.

### 🎨 **Visual Diagram**

```
WITHOUT Observer:
═════════════════
When task is completed, must manually notify everyone:

┌───────────────────┐
│ Task.complete()   │
│   method          │
└──────┬────────────┘
       │
       ├──> Manually call: ui.update() #this updates the screen to show task completion
       ├──> Manually call: achievements.track() #tracks achievements when task is  completed
       ├──> Manually call: notifier.send() #this sends a notification to the user
       └──> Manually call: stats.log() #logs statistics for the task

❌ Task object needs to manually notify the other objects
❌ Easy to forget one of the manual calls
❌ Task must know about all these systems
❌ Adding new feature => modify task code



WITH Observer:
══════════════
Task just says "I changed!" and observers handle themselves:

┌──────────────────────┐
│   Task (Subject)     │
│   state changed!     │
└──────────┬───────────┘
           │ notify_all() #this is a Task class method looping through all observers │ and calls their update() method
           ▼
    ┌──────────────┐
    │  Observers   │
    └──────┬───────┘
           ├──> 📱 UI Observer         → updates screen
           ├──> 🏆 Achievement Observer → tracks achievements
           ├──> 🔔 Notifier Observer   → sends notification
           └──> 📊 Stats Observer       → logs statistics

✅ Task doesn't know WHO is listening.
✅ Easy to add new observers.
✅ Loose coupling.
```

### 📝 **Pseudocode**

```
PSEUDOCODE:
-----------
CLASS Subject:
    LIST observers = []

    FUNCTION attach(observer):
        ADD observer to observers list

    FUNCTION detach(observer):
        REMOVE observer from observers list

    FUNCTION notify():
        FOR EACH observer in observers:
            CALL observer.update(self)

CLASS Observer:
    FUNCTION update(subject):
        // React to the change

USAGE:
    subject = create Subject
    observer1 = create Observer
    observer2 = create Observer

    subject.attach(observer1)
    subject.attach(observer2)

    subject.notify()  // Both observers get notified!
```

### 🚫 **Anti-Example: Without Observer Pattern**

```python
# ❌ PROBLEMATIC: Task must manually notify everything (the ui_display, achievement_tracker, notifier)

class Task:
    # ❌ First problem here is that Task has to know about ALL the systems that care about it (UI, achievements, notifications)
    # so when you add a new system, you have to modify Task code to include it (i.e adding stats_logging would make this constructor even more crowded)
    def __init__(self, name, ui_display, achievement_tracker, notifier):
        # reminder: the self here is the Task Object
        self.name = name
        self.priority = "medium"
        self.completed = False

        # ❌ Task must know about all these systems
        self.ui_display = ui_display
        self.achievement_tracker = achievement_tracker
        self.notifier = notifier

    def complete(self):
        """Mark task as complete"""
        self.completed = True

        # ❌ Must manually call each object to notify them of the change
        self.ui_display.update_task_display(self)
        # Reminder: the self here is the Task object, but we're calling the update_task_display() method of the ui_display object (which is passed as a parameter to the Task constructor)
        self.achievement_tracker.track_task_completed(self)
        self.notifier.send_completion_notification(self)
        # ❌ What if we forget one?
        # ❌ What if we want to add new tracking later?

    def set_priority(self, priority):
        """Change priority"""
        self.priority = priority

        # ❌ Duplicate notification code --- one here, one in complete()
        self.ui_display.update_task_display(self)
        self.achievement_tracker.track_priority_changed(self)
        # ❌ Easy to forget notifier here -- in fact, im not even gonna add it.

    # ❌ If we add dueDateNow() or assignedTo(), we'd have to remember to add (self.ui_display.update_task_display(self) and self.achievement_tracker.track_due_date_changed(self) and self.notifier.send_due_date_changed_notification(self)) in those methods too!

# ❌ These are the "observers" that need to be manually notified by Task
class UIDisplay:
    def update_task_display(self, task):
        # reminder: the self here is the UIDisplay object
        print(f"📱 UI: Updated display for '{task.name}'")

class AchievementTracker:
    def track_task_completed(self, task):
        print(f"🏆 Achievements: Task '{task.name}' completed")

    def track_priority_changed(self, task):
        print(f"🏆 Achievements: Priority changed for '{task.name}'")

class Notifier:
    def send_completion_notification(self, task):
        print(f"🔔 Notification: Task '{task.name}' done!")
print("=== Without Observer Pattern ===\n")

# ❌ Must pass all dependencies to task
# this means Task is tightly coupled to these specific implementations (i.e. if we want to change how notifications work, we have to modify Task code)
ui = UIDisplay()
achievements = AchievementTracker()
notifier = Notifier()

# In this example, we create a Task named "Write report" and pass in the UI, achievements tracker, and notifier objects that it needs to notify when things change.
task = Task("Write report", ui, achievements, notifier)

print("Completing task...")
task.complete()
# task.complete() manually calls:
# self.ui_display.update_task_display(self) → prints "📱 UI: Updated display for 'Write report'
# self.achievement_tracker.track_task_completed(self) → prints "🏆 Achievements: Task 'Write report' completed"
# self.notifier.send_completion_notification(self) → prints "🔔 Notification: Task 'Write report' done!

print("\nChanging priority...")
task.set_priority("high")
# ❌ Notice: No notification sent since we forgot to add it in set_priority() method!
# set_priority() manually calls:
# self.ui_display.update_task_display(self) → prints "📱 UI: Updated display for 'Write report'
# self.achievement_tracker.track_priority_changed(self) → prints "🏆 Achievements: Priority changed for 'Write report
# ❌ but we forgot to add notifier.send_priority_changed_notification(self), so no notification is sent for priority change

print("\n❌ Problems:")
print("  • Task is tightly coupled to UI, Achievements, Notifier")
print("  • Easy to forget to notify some systems")
print("  • Adding new observer = modify Task class")
print("  • Can't add/remove observers at runtime")
```

**Output:**

```
=== Without Observer Pattern ===

Completing task...
📱 UI: Updated display for 'Write report'
🏆 Achievements: Task 'Write report' completed
🔔 Notification: Task 'Write report' done!

Changing priority...
📱 UI: Updated display for 'Write report'
🏆 Achievements: Priority changed for 'Write report'

❌ Problems:
  • Task is tightly coupled to UI, Achievements, Notifier
  • Easy to forget to notify some systems
  • Adding new observer = modify Task class
  • Can't add/remove observers at runtime
```

**Problems:**

- 🔴 **Tight Coupling**: Task must know about every system that cares about it
- 🔴 **Not Flexible**: Can't add/remove observers without changing Task code (i.e. remove achievements observer, update Task class)
- 🔴 **Error Prone**: Easy to forget to notify someone
- 🔴 **Violates SRP**: Task handles both task logic AND notification logic
- Note: SRP is "Single Responsibility Principle" - a class should have only one reason to change. Here, Task has two reasons to change: if task logic changes OR if notification logic changes.

### ✅ **Solution: With Observer Pattern**

```python
# ✅ SOLUTION: Observer pattern decouples subject from observers

# Step 1: Define Observer base class
class TaskObserver:
    """Base class that all observers should follow"""

    def update(self, task, event_type):
        # this is important since the Task class will call this method on all observers when something changes, so we need to make sure every observer implements this method to avoid errors
        # this is basically the "interface" as discussed in the slides that all observers must follow. All observers must have an update() method that takes in the task and the type of event that happened (e.g. "completed", "priority_changed", etc.)
        """Called when task changes -- this is overridden by concrete observers"""
        raise NotImplementedError("Each observer must implement update()")


# Step 2: Define Subject class (the object being observed)
class Task:
    def __init__(self, name):
        self.name = name
        self.completed = False
        self.priority = "medium"

        # ✅ Task just maintains a list of observers
        # this is the only portion of the Task class that needs to be modified to add observer support, and it's pretty minimal - just a list to track observers and methods to attach/detach/notify them
        self._observers = []

    def attach(self, observer):
        # basically "subscribe" an observer to this task
        """Add an observer"""
        if observer not in self._observers:
            self._observers.append(observer)
            print(f"✅ Attached {observer.__class__.__name__} to '{self.name}'")

    def detach(self, observer):
        # this is the "unsub" method
        """Remove an observer"""
        if observer in self._observers:
            self._observers.remove(observer)
            print(f"❌ Detached {observer.__class__.__name__} from '{self.name}'")

    def notify(self, event_type):
        # this is the magic method that tells all observers "Hey, something happened!"
        """Notify all observers of a change"""
        print(f"\n🔔 Task '{self.name}': Notifying observers about '{event_type}'")
        # Loop through all observers and call their update method
        for observer in self._observers:
            observer.update(self, event_type)

    # Business methods that trigger notifications
    def complete(self):
        """Mark task as complete"""
        self.completed = True
        self.notify("completed")  # Tell observers we're done!

    def set_priority(self, priority):
        """Change priority"""
        old_priority = self.priority
        self.priority = priority
        self.notify(f"priority_changed:{old_priority}->{priority}")

    # we can add more methods like assign_to() or dueDateNow() and just call self.notify() with different event types, and observers can choose to react to those events


# Step 3: Create concrete observers (note: each one reacts differently)
class UIObserver:
    """Updates the user interface"""

    def update(self, task, event_type):
        # This gets called whenever the task changes
        print(f"   📱 UIObserver: Updating display for '{task.name}' ({event_type})")


class AchievementObserver:
    """Tracks achievements and milestones"""

    def __init__(self):
        self.completed_count = 0

    def update(self, task, event_type):
        # track task completions
        if "completed" in event_type:
            self.completed_count += 1
            print(f"   🏆 AchievementObserver: {self.completed_count} tasks completed!")
            if self.completed_count == 3:
                print(f"   🎉 Achievement Unlocked: Complete 3 tasks!")
            elif self.completed_count == 5:
                print(f"   🎉 Achievement Unlocked: Task Master - 5 tasks completed!")
        # later on, we can also track events such as finishing tasks before the deadline
        # it would look like:
        # if "early_completion" in event_type:
        #     self.early_completion_count += 1


class NotificationObserver:
    """Sends notifications to users"""

    def update(self, task, event_type):
        # Only send notification when task is completed
        if "completed" in event_type:
            print(f"   🔔 NotificationObserver: Sending 'Task Done!' notification")
            # this is where we'd usually call some notification service to send a real notification

task = Task("Write report")

# Create observers (instances of the observer classes we defined above)
# ✅ Notice: Task doesn't need to know about these specific observers, it just knows they are "observers" that have an update() method
ui = UIObserver()
achievements = AchievementObserver()
notifier = NotificationObserver()

# ✅ Attach observers (can be done at runtime!)
# think of it as hitting the subscribe button on Youtube channels, making it easier to add/remove observers whenever we want without changing Task code
print("--- Attaching Observers ---")
task.attach(ui)
task.attach(achievements)
task.attach(notifier)

# ✅ When task changes, everyone is notified automatically!
print("\n--- Changing Task State ---")
print("1. Changing priority...")
task.set_priority("high")

print("\n2. Completing task...")
task.complete()

# ✅ Can detach observers at runtime!
# this is basically the unsub button on youtube
# in terms of resources, this helps because if we have a lot of observers, we can detach the ones we don't need anymore to avoid unnecessary notifications and processing
print("\n--- Detaching Notification Observer ---")
task.detach(notifier)

# Testing with achievements observer only (meaning the UI won't update and no notifications will be sent, but achievements will still be tracked; in a real app, makes is easier to enable "Do Not Disturb" mode where you still want to track achievements but don't want notifications or UI updates)
print("\n--- Testing with More Tasks ---")
task2 = Task("Code review")
task2.attach(achievements)  # Only attach achievement tracker
task2.complete()

task3 = Task("Team meeting")
task3.attach(achievements)
task3.complete()  # Should unlock achievement!

print("\n✅ Benefits:")
print("  • Task doesn't know about specific observers")
print("  • Easy to add/remove observers at runtime")
print("  • Each observer handles its own logic")
print("  • No modification to Task needed for new observers")
```

**Output:**

```
=== With Observer Pattern ===

--- Attaching Observers ---
✅ Attached UIObserver to 'Write report'
✅ Attached AchievementObserver to 'Write report'
✅ Attached NotificationObserver to 'Write report'

--- Changing Task State ---
1. Changing priority...

🔔 Task 'Write report': Notifying observers about 'priority_changed:medium->high'
   📱 UIObserver: Updating display for 'Write report' (priority_changed:medium->high)
   🏆 AchievementObserver: Priority updated for 'Write report'

2. Completing task...

🔔 Task 'Write report': Notifying observers about 'completed'
   📱 UIObserver: Updating display for 'Write report' (completed)
   🏆 AchievementObserver: 1 tasks completed!
   🔔 NotificationObserver: Sending 'Task Done!' notification

--- Detaching Notification Observer ---
❌ Detached NotificationObserver from 'Write report'

--- Testing with More Tasks ---
✅ Attached AchievementObserver to 'Code review'

🔔 Task 'Code review': Notifying observers about 'completed'
   🏆 AchievementObserver: 2 tasks completed!
✅ Attached AchievementObserver to 'Team meeting'

🔔 Task 'Team meeting': Notifying observers about 'completed'
   🏆 AchievementObserver: 3 tasks completed!
   🎉 Achievement Unlocked: Complete 3 tasks!

✅ Benefits:
  • Task doesn't know about specific observers
  • Easy to add/remove observers at runtime, no need to modify Task class
  • Each observer handles its own logic
  • No modification to Task needed for new observers
```

### 🔧 **TL;DR: How to Implement Observer Pattern**

**In 4 Simple Steps:**

**Step 1: Define Observer Interface**

This helps create a common interface that all observers must follow, so that the Subject can call update() on all observers without worrying about their specific types

```python
class Observer:
     """Base class that all observers should follow"""
    def update(self, subject, event):
        raise NotImplementedError("Implement this!")
```

**Step 2: Make Your Subject Observable**

Add observer support to your subject class (the object being watched):

```python
class Subject:
    def __init__(self):
        # this is basically the only time we need to modify the Subject class to add observer support
        self._observers = []  # List to track observers

    def attach(self, observer):
        """Subscribe an observer"""
        self._observers.append(observer)

    def detach(self, observer):
        """Unsubscribe an observer"""
        self._observers.remove(observer)

    def notify(self, event_type):
        """Tell all observers about changes"""
        for observer in self._observers:
            observer.update(self, event_type)
```

**Step 3: Trigger Notifications When State Changes**

Call `notify()` in methods in Subject that change the Subject's state:

```python
def complete(self):
    self.completed = True
    self.notify("completed")  # Notify everyone!

def set_priority(self, new_priority):
    self.priority = new_priority
    self.notify("priority_changed")  # Notify again!
```

**Step 4: Create Concrete Observers**

Implement the `update()` method to react to changes:

```python
class UIObserver(Observer):
    # remember, the Observer base class has an update() method that all observers must implement
    def update(self, subject, event):
        print(f"UI updated for {subject.name} - event: {event}")
        # this is where you'd put code to actually update the UI

class NotificationObserver(Observer):
    def update(self, subject, event):
        if "completed" in event:
            print(f"🔔 Task '{subject.name}' is done!")
            # this is where you'd call your notification service to send a real notification
```

**That's it!** Now your objects can communicate without tight coupling.

### 🔑 **Key Takeaways**

- ✅ **Loose Coupling**: Subject doesn't know about concrete observers
- ✅ **Dynamic Relationships**: Add/remove observers at runtime
- ✅ **Open/Closed Principle**: Add new observers without modifying subject
- ✅ **Broadcast Communication**: One event notifies many observers
- ⚠️ **Watch Out**: Too many observers can slow down notifications

### 🤷‍♀️ My Take

The Observer pattern can be difficult to set up at first, but once you have it in place, it makes your code much more flexible and maintainable. It’s especially useful in applications with lots of user interactions or real-time updates, like our to-do list app. Just remember to keep an eye on performance if you have a large number of observers.

---

## Pattern 2: Strategy Pattern

### 🎯 **Purpose**

Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### 📖 **Concept in Plain English**

Imagine you have a list of tasks and want to sort them. Sometimes you want to sort by due date, sometimes by priority, sometimes by category. The sorting algorithm changes based on what you need at that moment.

**Without Strategy:** You have a giant if-else or switch statement that picks which sorting algorithm to use. Every time you add a new sorting method, you modify this code.

**With Strategy:** Each sorting algorithm is its own class. You can swap them in and out easily without changing the task list code!

**Think of it like payment methods:** Whether you pay with cash, credit card, or mobile wallet, the checkout process is the same - only the payment strategy changes.

### 🎨 **Visual Diagram**

```
WITHOUT Strategy:
═════════════════
TaskList has giant if-else for every sorting method:

┌────────────────────────────────────────┐
│ TaskList.sort(method):                 │
│                                        │
│   IF method == "priority":             │
│       [priority sorting code]          │  ❌ All sorting
│   ELIF method == "date":               │     logic mixed
│       [date sorting code]              │     together!
│   ELIF method == "alphabetical":       │
│       [alphabetical sorting code]      │  ❌ Hard to add
│   ELIF method == "category":           │     new methods!
│       [category sorting code]          │
│   ...                                  │
└────────────────────────────────────────┘


WITH Strategy:
══════════════
Each sorting algorithm is separate, swappable:

┌──────────────┐
│  TaskList    │
└──────┬───────┘
       │ uses
       ▼
┌──────────────────┐
│ SortStrategy     │◀─┐
│ (interface)      │  │ implements
└──────────────────┘  │
                      │
    ┌─────────────────┼─────────────────┬──────────────────┐
    │                 │                 │                  │
┌───▼─────┐  ┌────────▼────┐  ┌────────▼───┐  ┌──────────▼──┐
│Priority │  │   Date      │  │Alphabetical│  │  Category   │
│Strategy │  │  Strategy   │  │  Strategy  │  │  Strategy   │
└─────────┘  └─────────────┘  └────────────┘  └─────────────┘

✅ Easy to swap strategies!
✅ Easy to add new strategies!
✅ Each strategy is independent!
```

### 📝 **Pseudocode**

```
PSEUDOCODE:
-----------
INTERFACE SortStrategy:
    FUNCTION sort(tasks):
        // Define how to sort

CLASS PrioritySortStrategy IMPLEMENTS SortStrategy:
    FUNCTION sort(tasks):
        RETURN tasks sorted by priority

CLASS DateSortStrategy IMPLEMENTS SortStrategy:
    FUNCTION sort(tasks):
        RETURN tasks sorted by date

CLASS TaskList:
    CONSTRUCTOR(strategy):
        self.strategy = strategy
        self.tasks = []

    FUNCTION set_strategy(strategy):
        self.strategy = strategy

    FUNCTION sort_tasks():
        RETURN self.strategy.sort(self.tasks)

USAGE:
    task_list = TaskList(PrioritySortStrategy())
    sorted = task_list.sort_tasks()  // Sorted by priority

    task_list.set_strategy(DateSortStrategy())
    sorted = task_list.sort_tasks()  // Now sorted by date!
```

### 🚫 **Anti-Example: Without Strategy Pattern**

```python
# ❌ PROBLEMATIC: All sorting logic mixed in one class

from datetime import datetime

class Task:
    def __init__(self, name, priority, due_date, category):
        self.name = name
        self.priority = priority  # "low", "medium", "high"
        self.due_date = due_date
        self.category = category

    def __repr__(self):
        return f"Task('{self.name}', {self.priority}, {self.due_date}, {self.category})"


class TaskList:
    def __init__(self):
        self.tasks = []

    def add_task(self, task):
        self.tasks.append(task)

    def sort_tasks(self, method):
        """
        ❌ PROBLEM: Giant if-else with all sorting logic!
        """
        if method == "priority":
            # Priority sorting logic
            priority_order = {"low": 1, "medium": 2, "high": 3, "critical": 4}
            self.tasks.sort(key=lambda t: priority_order.get(t.priority, 0), reverse=True)

        elif method == "date":
            # Date sorting logic
            def date_key(task):
                if task.due_date:
                    return datetime.strptime(task.due_date, "%Y-%m-%d")
                return datetime.max
            self.tasks.sort(key=date_key)

        elif method == "alphabetical":
            # Alphabetical sorting logic
            self.tasks.sort(key=lambda t: t.name.lower())

        elif method == "category":
            # Category sorting logic
            self.tasks.sort(key=lambda t: t.category)

        else:
            print(f"❌ Unknown sorting method: {method}")
        # ❌ Want to add new sorting? Modify this function!

    def display(self, title="Tasks"):
        print(f"\n{title}:")
        for i, task in enumerate(self.tasks, 1):
            print(f"  {i}. {task.name} [{task.priority}] - {task.due_date} ({task.category})")


print("=== Without Strategy Pattern ===\n")

# Create tasks
task_list = TaskList()
task_list.add_task(Task("Write report", "high", "2026-03-20", "work"))
task_list.add_task(Task("Buy groceries", "low", "2026-03-16", "personal"))
task_list.add_task(Task("Team meeting", "medium", "2026-03-18", "work"))
task_list.add_task(Task("Call dentist", "medium", "2026-03-17", "personal"))

print("Original order:")
task_list.display()

print("\n--- Sorting by Priority ---")
task_list.sort_tasks("priority")
task_list.display("Sorted by Priority")

print("\n--- Sorting by Date ---")
task_list.sort_tasks("date")
task_list.display("Sorted by Date")

print("\n--- Sorting Alphabetically ---")
task_list.sort_tasks("alphabetical")
task_list.display("Sorted Alphabetically")

print("\n❌ Problems:")
print("  • All sorting logic in ONE giant if-else")
print("  • Adding new sort method = modify TaskList")
print("  • Can't reuse sorting logic elsewhere")
print("  • Violates Single Responsibility Principle")
print("  • Hard to test individual sorting methods")
```

**Output:**

```
=== Without Strategy Pattern ===

Original order:

Tasks:
  1. Write report [high] - 2026-03-20 (work)
  2. Buy groceries [low] - 2026-03-16 (personal)
  3. Team meeting [medium] - 2026-03-18 (work)
  4. Call dentist [medium] - 2026-03-17 (personal)

--- Sorting by Priority ---

Sorted by Priority:
  1. Write report [high] - 2026-03-20 (work)
  2. Team meeting [medium] - 2026-03-18 (work)
  3. Call dentist [medium] - 2026-03-17 (personal)
  4. Buy groceries [low] - 2026-03-16 (personal)

--- Sorting by Date ---

Sorted by Date:
  1. Buy groceries [low] - 2026-03-16 (personal)
  2. Call dentist [medium] - 2026-03-17 (personal)
  3. Team meeting [medium] - 2026-03-18 (work)
  4. Write report [high] - 2026-03-20 (work)

--- Sorting Alphabetically ---

Sorted Alphabetically:
  1. Buy groceries [low] - 2026-03-16 (personal)
  2. Call dentist [medium] - 2026-03-17 (personal)
  3. Team meeting [medium] - 2026-03-18 (work)
  4. Write report [high] - 2026-03-20 (work)

❌ Problems:
  • All sorting logic in ONE giant if-else
  • Adding new sort method = modify TaskList
  • Can't reuse sorting logic elsewhere
  • Violates Single Responsibility Principle
  • Hard to test individual sorting methods
```

**Problems:**

- 🔴 **God Method**: `sort_tasks()` knows about ALL sorting algorithms
- 🔴 **Not Extensible**: Adding new sort = modify existing sort_tasks() code
- 🔴 **Not Reusable**: Can't use these sorting algorithms elsewhere (they're nested in the sort_tasks())
- 🔴 **Hard to Test**: Must test all sorting methods through one function
- 🔴 **Violates OCP**: Open/Closed Principle - should be open for extension, closed for modification

### ✅ **Solution: With Strategy Pattern**

```python
# ✅ SOLUTION: Each sorting algorithm is its own strategy

from datetime import datetime

class Task:
    def __init__(self, name, priority, due_date, category):
        self.name = name
        self.priority = priority
        self.due_date = due_date
        self.category = category

    def __repr__(self):
        return f"Task('{self.name}', {self.priority}, {self.due_date}, {self.category})"


# Step 1: Define base Strategy class (optional - just for common interface)
class SortStrategy:
    """Base class that all sorting strategies should follow"""

    def sort(self, tasks):
        """Sort the tasks and return sorted list - override this!"""
        raise NotImplementedError("Each strategy must implement sort()")

    def get_name(self):
        """Return the name of this strategy - override this!"""
        raise NotImplementedError("Each strategy must implement get_name()")
class PrioritySortStrategy(SortStrategy):
    """Sort tasks by priority (high to low)"""

    def sort(self, tasks):
        priority_order = {"low": 1, "medium": 2, "high": 3, "critical": 4}
        return sorted(tasks, key=lambda t: priority_order.get(t.priority, 0), reverse=True)

    def get_name(self):
        return "Priority (High → Low)"


class DateSortStrategy(SortStrategy):
    """Sort tasks by due date (earliest first)"""

    def sort(self, tasks):
        def date_key(task):
            if task.due_date:
                return datetime.strptime(task.due_date, "%Y-%m-%d")
            return datetime.max
        return sorted(tasks, key=date_key)

    def get_name(self):
        return "Due Date (Earliest First)"


class AlphabeticalSortStrategy(SortStrategy):
    """Sort tasks alphabetically by name"""

    def sort(self, tasks):
        return sorted(tasks, key=lambda t: t.name.lower())
    def get_name(self):
        return "Alphabetical (A → Z)"


class CategorySortStrategy(SortStrategy):
    """Sort tasks by category"""

    def sort(self, tasks):
        return sorted(tasks, key=lambda t: t.category)

    def get_name(self):
        return "Category"


class CategoryThenPrioritySortStrategy(SortStrategy):
    """Sort by category, then by priority within each category"""

    def sort(self, tasks):
        priority_order = {"low": 1, "medium": 2, "high": 3, "critical": 4}
        return sorted(tasks,
                     key=lambda t: (t.category, -priority_order.get(t.priority, 0)))

    def get_name(self):
        return "Category → Priority"


# Step 3: Context class that uses strategies
class TaskList:
    """
    TaskList doesn't care HOW tasks are sorted.
    It just delegates to the current strategy!
    """

    def __init__(self, sort_strategy=None):
        self.tasks = []
        # Default strategy
        self.sort_strategy = sort_strategy or PrioritySortStrategy()

    def add_task(self, task):
        self.tasks.append(task)

    def set_sort_strategy(self, strategy):
        """Change the sorting strategy at runtime!"""
        self.sort_strategy = strategy
        print(f"📊 Switched to: {strategy.get_name()}")

    def get_sorted_tasks(self):
        """Get sorted tasks using current strategy"""
        return self.sort_strategy.sort(self.tasks)

    def display(self, title=None):
        """Display tasks using current sorting strategy"""
        if title is None:
            title = f"Tasks (Sorted by: {self.sort_strategy.get_name()})"

        print(f"\n{title}")
        sorted_tasks = self.get_sorted_tasks()
        for i, task in enumerate(sorted_tasks, 1):
            print(f"  {i}. {task.name} [{task.priority}] - {task.due_date} ({task.category})")


# ✅ USAGE: Clean and flexible!
print("=== With Strategy Pattern ===\n")

# Create tasks
task_list = TaskList()  # Uses default PrioritySortStrategy
task_list.add_task(Task("Write report", "high", "2026-03-20", "work"))
task_list.add_task(Task("Buy groceries", "low", "2026-03-16", "personal"))
task_list.add_task(Task("Team meeting", "medium", "2026-03-18", "work"))
task_list.add_task(Task("Call dentist", "medium", "2026-03-17", "personal"))
task_list.add_task(Task("Code review", "critical", "2026-03-15", "work"))

print("--- Default Sorting (Priority) ---")
task_list.display()

# ✅ Easily switch strategies at runtime!
print("\n--- Switching Strategies ---")
task_list.set_sort_strategy(DateSortStrategy())
task_list.display()

task_list.set_sort_strategy(AlphabeticalSortStrategy())
task_list.display()

task_list.set_sort_strategy(CategorySortStrategy())
task_list.display()

task_list.set_sort_strategy(CategoryThenPrioritySortStrategy())
task_list.display()

# ✅ Easy to add new strategies without modifying TaskList!
print("\n--- Adding New Strategy: Reverse Alphabetical ---")

class ReverseAlphabeticalSortStrategy(SortStrategy):
    """Sort tasks in reverse alphabetical order"""

    def sort(self, tasks):
        return sorted(tasks, key=lambda t: t.name.lower(), reverse=True)

    def get_name(self):
        return "Reverse Alphabetical (Z → A)"

task_list.set_sort_strategy(ReverseAlphabeticalSortStrategy())
task_list.display()

print("\n✅ Benefits:")
print("  • Each sorting algorithm is independent")
print("  • Easy to add new strategies without modifying TaskList")
print("  • Can switch strategies at runtime")
print("  • Each strategy can be tested independently")
print("  • Strategies can be reused in other contexts")
```

```python

ꓰꓚQ2 = "What is your go-to spot in CAS?"

```

**Output:**

```
=== With Strategy Pattern ===

--- Default Sorting (Priority) ---

Tasks (Sorted by: Priority (High → Low))
  1. Code review [critical] - 2026-03-15 (work)
  2. Write report [high] - 2026-03-20 (work)
  3. Team meeting [medium] - 2026-03-18 (work)
  4. Call dentist [medium] - 2026-03-17 (personal)
  5. Buy groceries [low] - 2026-03-16 (personal)

--- Switching Strategies ---
📊 Switched to: Due Date (Earliest First)

Tasks (Sorted by: Due Date (Earliest First))
  1. Code review [critical] - 2026-03-15 (work)
  2. Buy groceries [low] - 2026-03-16 (personal)
  3. Call dentist [medium] - 2026-03-17 (personal)
  4. Team meeting [medium] - 2026-03-18 (work)
  5. Write report [high] - 2026-03-20 (work)
📊 Switched to: Alphabetical (A → Z)

Tasks (Sorted by: Alphabetical (A → Z))
  1. Buy groceries [low] - 2026-03-16 (personal)
  2. Call dentist [medium] - 2026-03-17 (personal)
  3. Code review [critical] - 2026-03-15 (work)
  4. Team meeting [medium] - 2026-03-18 (work)
  5. Write report [high] - 2026-03-20 (work)
📊 Switched to: Category

Tasks (Sorted by: Category)
  1. Buy groceries [low] - 2026-03-16 (personal)
  2. Call dentist [medium] - 2026-03-17 (personal)
  3. Write report [high] - 2026-03-20 (work)
  4. Team meeting [medium] - 2026-03-18 (work)
  5. Code review [critical] - 2026-03-15 (work)
📊 Switched to: Category → Priority

Tasks (Sorted by: Category → Priority)
  1. Call dentist [medium] - 2026-03-17 (personal)
  2. Buy groceries [low] - 2026-03-16 (personal)
  3. Code review [critical] - 2026-03-15 (work)
  4. Write report [high] - 2026-03-20 (work)
  5. Team meeting [medium] - 2026-03-18 (work)

--- Adding New Strategy: Reverse Alphabetical ---
📊 Switched to: Reverse Alphabetical (Z → A)

Tasks (Sorted by: Reverse Alphabetical (Z → A))
  1. Write report [high] - 2026-03-20 (work)
  2. Team meeting [medium] - 2026-03-18 (work)
  3. Code review [critical] - 2026-03-15 (work)
  4. Call dentist [medium] - 2026-03-17 (personal)
  5. Buy groceries [low] - 2026-03-16 (personal)

✅ Benefits:
  • Each sorting algorithm is independent
  • Easy to add new strategies without modifying TaskList
  • Can switch strategies at runtime
  • Each strategy can be tested independently
  • Strategies can be reused in other contexts
```

### 🔧 **TL;DR: How to Implement Strategy Pattern**

**In 4 Simple Steps:**

1.  **Define Strategy base class** (optional, but helpful):

    ```python
    class Strategy:
        def execute(self, data):
            raise NotImplementedError("Implement this!")
    ```

2.  **Create concrete strategies** with different algorithms:

    ```python
    class ConcreteStrategyA:
        def execute(self, data):
            # Algorithm A implementation
            return sorted(data)
    ```

3.  **Add strategy to Context** class:

        def set_strategy(self, strategy):
            self.strategy = strategy

4.  **Use the strategy** instead of if-else:
    ```python
    result = context.strategy.execute(data)
    ```

**That's it!** Now you can swap algorithms at runtime!

### 🔑 **Key Takeaways**

- ✅ **Eliminates Conditionals**: Replaces if-else with object composition
- ✅ **Runtime Flexibility**: Change behavior while program is running
- ✅ **Easy to Extend**: Add new strategies without modifying existing code
- ✅ **Testable**: Each strategy can be tested independently
- ✅ **Reusable**: Strategies can be used in different contexts
- ⚠️ **Trade-off**: More classes (but better organization!)

### 💳 **Additional Example: Payments (Anti vs With Strategy)**

Sometimes Strategy clicks better with payments because the checkout flow stays the same, but the payment algorithm changes.

### 🚫 **Anti-Example: Without Strategy Pattern**

```python
# ❌ PROBLEMATIC: One method handles all payment logic with if-else

class Checkout:
    def pay(self, method, amount):
        if method == "cash":
            print(f"💵 Accepting cash payment of PHP {amount:.2f}")
            print("✅ Payment successful")

        elif method == "card":
            print(f"💳 Charging card for PHP {amount:.2f}")
            print("✅ Payment successful")

        elif method == "gcash":
            print(f"📱 Processing GCash payment of PHP {amount:.2f}")
            print("✅ Payment successful")

        elif method == "bank fund transfer":
            print(f"🏦 Validating bank fund transfer for PHP {amount:.2f}")
            print("✅ Payment successful")

        else:
            print(f"❌ Unsupported payment method: {method}")

# so when you need to add a new payment method, you have to modify the Checkout class, which violates the Open/Closed Principle and can lead to bugs if you accidentally break existing payment logic while adding new code

# to checkout with a specific method, you have to call the same pay() method with different parameters, which can lead to confusion and errors if you pass the wrong method or amount
checkout = Checkout()
checkout.pay("cash", 250.00)
checkout.pay("card", 399.50)
checkout.pay("gcash", 120.75)
checkout.pay("bank fund transfer", 1000.00)

print("\n❌ Problems:")
print("  • Adding a new payment method requires editing Checkout.pay()")
print("  • Giant if-else grows over time")
print("  • Harder to test each payment behavior independently")
```

### ✅ **With Strategy Pattern**

```python
# ✅ SOLUTION: Each payment method is a separate strategy

# Step 1: Define base Strategy class
class PaymentStrategy:
    def pay(self, amount):
        raise NotImplementedError("Each payment strategy must implement pay()")

# Step 2: Create concrete payment strategies (Cash, Card, GCash, Bank Fund Transfer)
class CashPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"💵 Cash payment received: PHP {amount:.2f}")


class CardPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"💳 Card charged: PHP {amount:.2f}")


class GCashPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"📱 GCash payment completed: PHP {amount:.2f}")


class BankFundTransferPayment(PaymentStrategy):
    def pay(self, amount):
        print(f"🏦 Bank fund transfer confirmed: PHP {amount:.2f}")

# Step 3: Context class that uses payment strategies (Checkout)
# this class doesn't care HOW payment is processed, it just calls pay() on the current strategy
# how it works in runtime is that we can create a Checkout instance with any payment strategy, and we can switch strategies at runtime without modifying the Checkout class
class Checkout:
    def __init__(self, payment_strategy):
        self.payment_strategy = payment_strategy

    def set_payment_strategy(self, payment_strategy):
        self.payment_strategy = payment_strategy

    def checkout(self, amount):
        self.payment_strategy.pay(amount)
        print("✅ Payment successful")

# these are the checkout flows, notice how the checkout process is the same regardless of payment method, we just swap out the payment strategy
# so to checkout using cash, you just need to call checkout with CashPayment strategy, and if you want to switch to card payment, you just call set_payment_strategy() with CardPayment without modifying the Checkout class at all

# WITH STRATEGY
checkout = Checkout(CashPayment())
checkout.checkout(250.00)

checkout.set_payment_strategy(CardPayment())
checkout.checkout(399.50)

checkout.set_payment_strategy(GCashPayment())
checkout.checkout(120.75)

checkout.set_payment_strategy(BankFundTransferPayment())
checkout.checkout(1000.00)

print("\n✅ Benefits:")
print("  • Checkout flow stays the same")
print("  • Payment methods are interchangeable at runtime")
print("  • New methods can be added without modifying Checkout")
print("  • Each payment strategy can be tested in isolation")
```

---

## Pattern 3: Command Pattern

### 🎯 **Purpose**

Encapsulate a request as an object, allowing you to parameterize clients with different requests, queue or log requests, and support undoable operations.

### 📖 **Concept in Plain English**

Think about a TV remote. Each button (power, volume up, channel change) is a command. The remote doesn't know HOW to change the channel - it just knows to execute the "change channel" command. The TV knows how to actually do it.

This separation is powerful because:

- You can log all button presses
- You can implement undo (press "back" to go to previous channel)
- You can create macros (one button does multiple things)
- You can queue commands to execute later

In our to-do app: Actions like "add task", "delete task", "complete task" become command objects. This gives us undo/redo, action history, and more!

### 🎨 **Visual Diagram**

```
WITHOUT Command:
════════════════
Actions are just method calls, no way to undo:

┌──────────┐
│   User   │
└────┬─────┘
     │ calls directly
     ▼
┌──────────────────┐
│  TaskManager     │
│  .add_task()     │  ❌ Can't undo!
│  .delete_task()  │  ❌ Can't log history!
│  .complete_task()│  ❌ Can't queue for later!
└──────────────────┘
Note: Technically, you can still implement undo by either (a) making a dedicated undo method for each action or (b) by keeping track of previous states --- which is literally what many of you did in CMSC128's "Undo" expanded feature. But it becomes messy and tightly coupled (i.e. TaskManager now has to know about undo logic for every action). You'd have to add undo logic inside TaskManager, which violates SRP (Single Responsibility Principle) and makes the code harder to maintain.

WITH Command:
═════════════
Each action is a command object with execute() and undo():

                                ┌─────────────────────┐
                                │  AddTaskCommand     │
                                │  - execute()        │
                                │  - undo()           │
                                └─────────────────────┘
                                           │
┌──────────┐                    ┌─────────────────────┐
│   User   │───creates─────────>│ DeleteTaskCommand   │
└────┬─────┘                    │  - execute()        │
     │                          │  - undo()           │
     │ gives to                 └─────────────────────┘
     ▼                                   │
┌──────────────┐                ┌─────────────────────┐
│  Invoker     │                │ CompleteTaskCommand │
│  (CommandMgr)│                │  - execute()        │
│  .execute()  │                │  - undo()           │
│  .undo()     │                └──────────┬──────────┘
└──────┬───────┘                           │
       │                                   │ all operate on
       │                                   ▼
       │                           ┌───────────────┐
       └──────────────────────────>│ TaskManager   │
       │                           │ (Receiver)    │
       │                           └───────────────┘
       │
       │ keeps history:
       ▼
┌──────────────────────────┐
│ History:                 │
│  1. AddTaskCommand       │  ✅ Can undo any action!
│  2. CompleteTaskCommand  │  ✅ Can see history!
│  3. DeleteTaskCommand    │  ✅ Can replay actions!
└──────────────────────────┘

In our TV Remote analogy, the TV remote is the Invoker, the TV is the TaskManager (Receiver), and each button press creates a Command object that knows how to execute and undo that action on the TV. The remote can keep a history of commands to allow undoing or redoing actions, and the TV just executes whatever command it receives without needing to know where it came from or why.

```

### 📝 **Pseudocode**

```
PSEUDOCODE:
-----------
INTERFACE Command:
    FUNCTION execute()
    FUNCTION undo()

CLASS AddTaskCommand IMPLEMENTS Command:
    CONSTRUCTOR(receiver, task_data):
        self.receiver = receiver
        self.task_data = task_data

    FUNCTION execute():
        self.task = receiver.add_task(task_data)

    FUNCTION undo():
        receiver.remove_task(self.task)

CLASS Invoker:
    LIST history = []

    FUNCTION execute_command(command):
        command.execute()
        ADD command to history

    FUNCTION undo():
        command = POP last command from history
        command.undo()

USAGE:
    manager = TaskManager()
    invoker = Invoker()

    cmd = AddTaskCommand(manager, "Buy milk")
    invoker.execute_command(cmd)  // Task added

    invoker.undo()  // Task removed!
```

### 🚫 **Anti-Example: Without Command Pattern**

```python
# ❌ PROBLEMATIC: Direct method calls, no undo capability

class TaskManager:
    def __init__(self):
        self.tasks = []

    def add_task(self, name):
        """Add a task"""
        task = {"id": len(self.tasks) + 1, "name": name, "completed": False}
        self.tasks.append(task)
        print(f"✅ Added: {name}")
        return task

        def delete_task(self, task_id):
            """Delete a task with undo capability"""
            task = self._find_task(task_id)
            # this is basically how you would implement undo without the Command pattern - you have to keep track of the deleted task data so you can restore it if needed, but this logic is now mixed in with your TaskManager which should ideally only be responsible for managing tasks only, not undo logic (so the TaskManager  now violates SRP - Single Responsibility Principle).
            # And If you want to add undo for completing a task, you'd have to add more undo logic inside TaskManager, and so on.
            # and this is literally what many of you did in CMSC128 for the "Undo" expanded feature
            if task:
                # Save task data before deletion
                deleted_task = deepcopy(task)
                self.tasks.remove(task)
                print(f"🗑️ Deleted: {task['name']}")
                print(f"⏱️  Undo available for 5 seconds...")

                def undo_delete():
                    """Nested function to restore deleted task"""
                    if deleted_task not in self.tasks:
                        self.tasks.append(deleted_task)
                        print(f"⏪ Restored: {deleted_task['name']}")
                        return True
                    return False
                return {"deleted": True, "task": deleted_task, "undo": undo_delete}
            return False

    def complete_task(self, task_id):
        """Complete a task"""
        task = self._find_task(task_id)
        if task:
            task["completed"] = True
            print(f"✔️ Completed: {task['name']}")
            # if you want to implement undo, you'd have to add a specialized undo function here as well.
            return True
        return False

    # _find_task helper method to find task by ID
    def _find_task(self, task_id):
        for task in self.tasks:
            if task["id"] == task_id:
                return task
        return None

    def show_tasks(self):
        print("\n📋 Current Tasks:")
        if not self.tasks:
            print("  (No tasks)")
        for task in self.tasks:
            status = "✔️" if task["completed"] else "⭕"
            print(f"  {status} {task['id']}. {task['name']}")


print("=== Without Command Pattern ===\n")

manager = TaskManager()

# User performs actions
print("--- Performing Actions ---")
manager.add_task("Write report")
manager.add_task("Review code")
manager.add_task("Team meeting")

manager.show_tasks()

print("\n--- More Actions ---")
manager.complete_task(2)
manager.delete_task(1)

manager.show_tasks()

print("\n❌ Problems:")
print("  • No way to undo actions!")
print("  • Can't see history of what was done")
print("  • Can't replay or queue actions")
print("  • Can't implement redo functionality")
print("  • Can't create macro commands (multiple actions at once)")

print("\n😢 User wants to undo deleting task... but can't!")
```

**Output:**

```
=== Without Command Pattern ===

--- Performing Actions ---
✅ Added: Write report
✅ Added: Review code
✅ Added: Team meeting

📋 Current Tasks:
  ⭕ 1. Write report
  ⭕ 2. Review code
  ⭕ 3. Team meeting

--- More Actions ---
✔️ Completed: Review code
🗑️ Deleted: Write report

📋 Current Tasks:
  ✔️ 2. Review code
  ⭕ 3. Team meeting

❌ Problems:
  • No way to undo actions! (Unless you implement custom undo logic for each action, which gets messy and violates SRP)
  • Can't see history of what was done (unless you do manual logging, which is extra work and not standardized)
  • Can't replay or queue actions
  • Can't implement redo functionality
  • Can't create macro commands (multiple actions at once)

😢 User wants to undo deleting task... but can't! (or at least needs extra work from the dev)
```

### ✅ **Solution: With Command Pattern**

```python
# ✅ SOLUTION: Command pattern with undo/redo support!

from copy import deepcopy

# Step 1: Define Command base class (for common interface)
# remember, a common interface (even in the other patterns) is optional but recommended as it helps the invoker to treat all commands uniformly without needing to know their specific types or implementations --- hence, promoting loose coupling
class Command:
    """Base class for all commands"""

    def execute(self):
        """Execute the command - override this!"""
        raise NotImplementedError("Each command must implement execute()")

    def undo(self):
        """Undo the command - override this!"""
        raise NotImplementedError("Each command must implement undo()")

    def get_description(self):
        # we'll use this for the Macros example
        """Get a description of this command - override this!"""
        raise NotImplementedError("Each command must implement get_description()")


# Step 2: Define Receiver class (the object that knows how to do the work)
# again, this is the "TV" in the TV remote analogy - it knows how to perform the actual operations, but it doesn't know about the commands or the invoker
class TaskManager:
    """The receiver that knows how to perform task operations"""

    def __init__(self):
        self.tasks = []
        self._next_id = 1

    def add_task(self, name):
        """Add a task and return it"""
        task = {
            "id": self._next_id,
            "name": name,
            "completed": False
        }
        self._next_id += 1
        self.tasks.append(task)
        return task

    def remove_task(self, task):
        """Remove a specific task"""
        if task in self.tasks:
            self.tasks.remove(task)

    def mark_completed(self, task):
        """Mark a task as completed"""
        task["completed"] = True

    def mark_incomplete(self, task):
        """Mark a task as incomplete"""
        task["completed"] = False

    def show_tasks(self):
        """Display all tasks"""
        print("\n📋 Current Tasks:")
        if not self.tasks:
            print("  (No tasks)")
        for task in self.tasks:
            status = "✔️" if task["completed"] else "⭕"
            print(f"  {status} {task['id']}. {task['name']}")


# Step 3: Concrete Commands (each one wraps an action)
# These are the "buttons" on our remote - each command knows how to execute and undo a specific action on the TaskManager (Receiver) --- these call the actual methods on TaskManager (tells the TV what to do)
class AddTaskCommand:
    """Command to add a task"""

    def __init__(self, receiver, name):
        self.receiver = receiver  # The TaskManager
        self.name = name
        self.task = None  # Will store the task after adding

    def execute(self):
        # Do the action: add task
        self.task = self.receiver.add_task(self.name)
        print(f"✅ Added: {self.name}")

    def undo(self):
        # Reverse the action: remove the task we added
        if self.task:
            self.receiver.remove_task(self.task)
            print(f"⏪ Undid add: {self.name}")

    def get_description(self):
        return f"Add task: {self.name}"

class DeleteTaskCommand:
    """Command to delete a task"""

    def __init__(self, receiver, task):
        self.receiver = receiver
        self.task = task
        self.task_backup = deepcopy(task)  # Save for undo

    def execute(self):
        self.receiver.remove_task(self.task)
        print(f"🗑️ Deleted: {self.task['name']}")

    def undo(self):
        # Restore the deleted task
        self.receiver.tasks.append(self.task_backup)
        print(f"⏪ Undid delete: {self.task['name']}")

    def get_description(self):
        return f"Delete task: {self.task['name']}"

class CompleteTaskCommand:
    """Command to complete a task"""

    def __init__(self, receiver, task):
        self.receiver = receiver
        self.task = task

    def execute(self):
        self.receiver.mark_completed(self.task)
        print(f"✔️ Completed: {self.task['name']}")

    def undo(self):
        self.receiver.mark_incomplete(self.task)
        print(f"⏪ Undid complete: {self.task['name']}")

    def get_description(self):
        return f"Complete task: {self.task['name']}"


# Step 4: Invoker (manages command execution and history)
# In the TV remote analogy, this is the remote itself - it knows how to execute commands when buttons are pressed (i.e. when Command objects are given to it), and it keeps track of command history allowing for easier undo/redo
class CommandManager:
    """
    The invoker that executes commands and maintains history.
    Provides undo/redo functionality!
    """

    def __init__(self):
        self.history = []      # Commands that have been executed
        self.redo_stack = []   # Commands that have been undone

    def execute_command(self, command):
        """Execute a command and add to history"""
        command.execute()
        self.history.append(command)
        # Clear redo stack when new command is executed
        self.redo_stack.clear()

    def undo(self):
        """Undo the last command"""
        if not self.history:
            print("❌ Nothing to undo!")
            return

        # Get last command and undo it
        command = self.history.pop()
        command.undo()
        self.redo_stack.append(command)

    def redo(self):
        """Redo the last undone command"""
        if not self.redo_stack:
            print("❌ Nothing to redo!")
            return

        # Get last undone command and redo it
cmd_manager.execute_command(cmd1)

cmd2 = AddTaskCommand(task_manager, "Review code")
cmd_manager.execute_command(cmd2)

cmd3 = AddTaskCommand(task_manager, "Team meeting")
cmd_manager.execute_command(cmd3)

task_manager.show_tasks()

# Complete a task
print("\n--- Completing Task ---")
task_to_complete = task_manager.tasks[1]  # Review code
cmd4 = CompleteTaskCommand(task_manager, task_to_complete)
cmd_manager.execute_command(cmd4)

# Delete a task
print("\n--- Deleting Task ---")
task_to_delete = task_manager.tasks[0]  # Write report
cmd5 = DeleteTaskCommand(task_manager, task_to_delete)
cmd_manager.execute_command(cmd5)

task_manager.show_tasks()
cmd_manager.show_history()

# ✅ Undo operations!
print("\n--- Undoing Last Action (Delete) ---")
cmd_manager.undo()
task_manager.show_tasks()

print("\n--- Undoing Again (Complete) ---")
cmd_manager.undo()
task_manager.show_tasks()

# ✅ Redo operations!
print("\n--- Redoing (Complete) ---")
cmd_manager.redo()
task_manager.show_tasks()

print("\n--- Undoing Twice ---")
cmd_manager.undo()
cmd_manager.undo()
task_manager.show_tasks()

print("\n--- Executing New Command (clears redo stack) ---")
cmd6 = AddTaskCommand(task_manager, "Call client")
cmd_manager.execute_command(cmd6)
task_manager.show_tasks()

print("\n--- Trying to Redo (should fail - redo stack was cleared) ---")
cmd_manager.redo()

cmd_manager.show_history()

print("\n✅ Benefits:")
print("  • Full undo/redo support!")
print("  • Complete action history")
print("  • Commands can be saved/loaded")
print("  • Easy to implement macros")
print("  • Can queue commands for later execution")
```

**Output:**

```
=== With Command Pattern ===

--- Executing Commands ---
✅ Added: Write report
✅ Added: Review code
✅ Added: Team meeting

📋 Current Tasks:
  ⭕ 1. Write report
  ⭕ 2. Review code
  ⭕ 3. Team meeting

--- Completing Task ---
✔️ Completed: Review code

--- Deleting Task ---
🗑️ Deleted: Write report

📋 Current Tasks:
  ✔️ 2. Review code
  ⭕ 3. Team meeting

📜 Command History:
  1. Add task: Write report
  2. Add task: Review code
  3. Add task: Team meeting
  4. Complete task: Review code
  5. Delete task: Write report

--- Undoing Last Action (Delete) ---
⏪ Undid delete: Write report

📋 Current Tasks:
  ⭕ 1. Write report
  ✔️ 2. Review code
  ⭕ 3. Team meeting

--- Undoing Again (Complete) ---
⏪ Undid complete: Review code

📋 Current Tasks:
  ⭕ 1. Write report
  ⭕ 2. Review code
  ⭕ 3. Team meeting

--- Redoing (Complete) ---
✔️ Completed: Review code

📋 Current Tasks:
  ⭕ 1. Write report
  ✔️ 2. Review code
  ⭕ 3. Team meeting

--- Undoing Twice ---
⏪ Undid complete: Review code
⏪ Undid delete: Write report

📋 Current Tasks:
  ⭕ 1. Write report
  ⭕ 2. Review code
  ⭕ 3. Team meeting

--- Executing New Command (clears redo stack) ---
✅ Added: Call client

📋 Current Tasks:
  ⭕ 1. Write report
  ⭕ 2. Review code
  ⭕ 3. Team meeting
  ⭕ 4. Call client

--- Trying to Redo (should fail - redo stack was cleared) ---
❌ Nothing to redo!

📜 Command History:
  1. Add task: Write report
  2. Add task: Review code
  3. Add task: Team meeting
  4. Add task: Call client

✅ Benefits:
  • Full undo/redo support!
  • Complete action history
  • Commands can be saved/loaded
  • Easy to implement macros
  • Can queue commands for later execution
```

### 🎯 **Bonus: Macro Commands**

```python
# BONUS: Create macro commands (multiple commands as one)

class MacroCommand(Command):
    """A command that executes multiple commands"""

    def __init__(self, commands, name):
        self.commands = commands
        self.name = name
        for command in self.commands:
            command.execute()

    def undo(self):
        print(f"⏪ Undoing macro: {self.description}")
        # Undo in reverse order!
        for command in reversed(self.commands):
            command.undo()

    def get_description(self):
        return f"Macro: {self.description}"


print("\n\n=== Bonus: Macro Commands ===\n")

# Create a fresh manager
task_manager2 = TaskManager()
cmd_manager2 = CommandManager()

# Create a macro: "Morning Routine" - add multiple tasks at once
print("--- Creating Morning Routine Macro ---")
morning_routine = MacroCommand([
    AddTaskCommand(task_manager2, "Check emails"),
    AddTaskCommand(task_manager2, "Daily standup"),
    AddTaskCommand(task_manager2, "Review priorities")
], "Morning Routine")

cmd_manager2.execute_command(morning_routine)
task_manager2.show_tasks()

print("\n--- Undoing Entire Macro ---")
cmd_manager2.undo()
task_manager2.show_tasks()

print("\n--- Redoing Macro ---")
cmd_manager2.redo()
task_manager2.show_tasks()
```

**Output:**

```
=== Bonus: Macro Commands ===

--- Creating Morning Routine Macro ---
🎬 Executing macro: Morning Routine
✅ Added: Check emails
✅ Added: Daily standup
✅ Added: Review priorities

📋 Current Tasks:
  ⭕ 1. Check emails
  ⭕ 2. Daily standup
  ⭕ 3. Review priorities

--- Undoing Entire Macro ---
⏪ Undoing macro: Morning Routine
⏪ Undid add: Review priorities
⏪ Undid add: Daily standup
⏪ Undid add: Check emails

📋 Current Tasks:
  (No tasks)

--- Redoing Macro ---
🎬 Executing macro: Morning Routine
✅ Added: Check emails
✅ Added: Daily standup
✅ Added: Review priorities

📋 Current Tasks:
  ⭕ 1. Check emails
  ⭕ 2. Daily standup
  ⭕ 3. Review priorities
```

### 🔧 **TL;DR: How to Implement Command Pattern**

**In 5 Simple Steps:**

1. **Create Command base class** with `execute()` and `undo()`:

   ```python
   class Command:
       def execute(self):
           raise NotImplementedError("Override this!")

       def undo(self):
           raise NotImplementedError("Override this!")
   ```

2. **Create Receiver** (object that does the actual work):

   ```python
   class Receiver:
       def action(self):
           # Do the actual work
           pass
   ```

3. **Create Concrete Commands** that wrap receiver actions:

   ```python
   class ConcreteCommand:
       def __init__(self, receiver, params):
           self.receiver = receiver
           self.params = params

       def execute(self):
           self.receiver.action(self.params)

       def undo(self):
           self.receiver.reverse_action(self.params)
   ```

4. **Create Invoker** to manage command history:

   ```python
   class Invoker:
       def __init__(self):
           self.history = []

       def execute_command(self, command):
           command.execute()
           self.history.append(command)

       def undo(self):
           command = self.history.pop()
           command.undo()
   ```

5. **Use it** instead of direct method calls:
   ```python
   command = ConcreteCommand(receiver, params)
   invoker.execute_command(command)
   invoker.undo()  # Can undo!
   ```

**That's it!** Now you have undo/redo and command history!

### 🔑 **Key Takeaways**

- ✅ **Undo/Redo**: Full support for reversing actions
- ✅ **History**: Keep track of all executed commands
- ✅ **Decoupling**: Invoker doesn't know about receiver implementation
- ✅ **Queueable**: Commands can be stored and executed later
- ✅ **Composable**: Create macro commands from multiple commands
- ✅ **Loggable**: Easy to log all user actions
- ⚠️ **Trade-off**: More classes (one per action type)

---

## Best Practices

### ✅ **When to Use Each Pattern**

| Pattern      | Use When                                     | Example                                    |
| ------------ | -------------------------------------------- | ------------------------------------------ |
| **Observer** | Multiple objects need to react to changes    | UI updates, notifications, event tracking  |
| **Strategy** | Need to swap algorithms/behaviors at runtime | Sorting, filtering, payment methods        |
| **Command**  | Need undo/redo, action logging, or queuing   | Editor operations, transactions, workflows |

### 💡 **General Tips**

1. **Start Simple**: Don't use patterns until you have the problem they solve
2. **Combine Patterns**: Observer + Command = undo-able events!
3. **Think Long-term**: Patterns pay off when requirements change
4. **Document Why**: Explain in comments why you chose each pattern

### ⚠️ **Common Mistakes to Avoid**

- ❌ **Observer**: Too many observers can slow down notifications
- ❌ **Strategy**: Creating strategy for every tiny variation
- ❌ **Command**: Forgetting to implement proper undo logic
- ❌ **All Patterns**: Over-engineering simple problems

### 🔄 **Combining Patterns**

These behavioral patterns work great together!

```python
# Example: Observer + Command
# Commands notify observers when executed

class ObservableCommand(Command):
    def __init__(self):
        self.observers = []

    def attach(self, observer):
        self.observers.append(observer)

    def execute(self):
        # Do work
        self.notify("executed")

    def notify(self, event):
        for observer in self.observers:
            observer.update(self, event)

# Now you have commands that broadcast their execution!
```

---

## Summary

### 🎯 **What We Learned**

We explored three behavioral patterns through our to-do list application:

1. **👀 Observer Pattern**
   - One-to-many dependency between objects
   - Perfect for: Event systems, notifications, UI updates
   - Key benefit: Loose coupling - subjects don't know about observers

2. **🔄 Strategy Pattern**
   - Encapsulate algorithms as interchangeable objects
   - Perfect for: Sorting, filtering, payment methods, validation
   - Key benefit: Swap behaviors at runtime without if-else chains

3. **⏮️ Command Pattern**
   - Encapsulate requests as objects with undo support
   - Perfect for: Undo/redo, action logging, macros, queues
   - Key benefit: Full history and reversibility of actions

### 📊 **Pattern Comparison**

| Aspect                 | Observer                 | Strategy                     | Command                      |
| ---------------------- | ------------------------ | ---------------------------- | ---------------------------- |
| **Main Focus**         | Notification             | Algorithm selection          | Action encapsulation         |
| **Key Benefit**        | Loose coupling           | Runtime flexibility          | Undo/redo support            |
| **Communication**      | One-to-many broadcast    | One-to-one delegation        | Invoker → Command → Receiver |
| **When State Changes** | Automatically notify all | Switch to different strategy | Record for undo/redo         |
| **Use Cases**          | Events, updates          | Sorting, filtering, payment  | Undo, logging, macros        |

---

### 🆚 **Behavioral vs Creational Patterns**

**Creational Patterns** (previous lesson) focus on HOW objects are created:

- Singleton: One instance only
- Factory: Create different types
- Builder: Build complex objects step-by-step

**Behavioral Patterns** (this lesson) focus on HOW objects communicate:

- Observer: Broadcast state changes
- Strategy: Swap algorithms
- Command: Encapsulate and undo actions

**Together, they help you build flexible, maintainable applications!**

---

> **"Behavioral patterns are all about responsibility assignment and communication between objects. They help objects work together while staying loosely coupled."** - Gang of Four

---
