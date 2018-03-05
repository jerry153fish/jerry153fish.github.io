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

### User search Bounded Context

To begin with, user search bounded context is the cornerstone of this application. The figure below displays the main process when user try to search jobs by keywords and location.

![User search Bounded Context](/assets/img/jobseek/user-query-bounded-context.jpg)

1. The UI send command to search for eg: 'developer' in 'hobart'. This command will create a query aggregate with it id as hash of keywords, location and time interval (time round down??).

2. The query aggregate will be persisted into database

3. Event query created will be raised and sent to message bus? -- global? local?

4. Message bus will send command Scrapy jobs according to query aggregate and route to Job aggregate in scrapy context
    * scrapy services will be discussed later

5. Job aggregate check if there is is a cache hit

6. If not cached scrapy jobs then raise jobs fetched event if cached directly raised
    * There are two types of search here: list search and detailed search discussed later
    * If cached steps 7 - 9 should be ignored
    * If not cached process to detailed Job search which all raise sequence of job fetched events

7. Message bus the send command Tag Job 

8. After Tag service tagged job job aggregate will raise job tagged event

9. Job aggregate and query aggregate will be updated and persisted


### Aggregate








