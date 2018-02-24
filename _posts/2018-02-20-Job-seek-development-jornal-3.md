---
layout: post
title: Job Seek Development Journal - 3
key: 20180128
tags: C# Entity Aggregate Repository DDD Journal
---

In last journal, I mainly focused on setting up TDD framework and exploring EF Core. Now, I need to start think in DDD and try to form a simple domain.

### Message

Before exploring user search bounded context, some constrains need to be added.

1. Message: Messages passed inside Job Seek such as commands and events

```cs
// IMessage.cs
public interface IMessage
{ }

```

2. Command: Message that trigger a procedure in Job Seek.

```cs
// ICommand.cs
public interface IMessage
{ }
```

3. Event: Message that show things happened in Job Seek.

```cs
// IEvent.cs
public interface IEvent
{ }
```

4. Message Bus: Message bus manage all message in Job Seek, such as pub / sub to message, state control and so on.

```cs
// IBus.cs

public interface IBus
{ }
```

### User Management Bounded Context

