---
layout: post
title:  "Application logging"
date:   2021-09-17
categories: coding software
---

<p align="center">
  <img src="/assets/images/header-640.131.3.png" width="100%">
</p>

Diagnosing unexpected issues in long-running background processes can be hard. The issue may only arise rarely and it may be difficult to reproduce outside of the production environment.

There is quite an array of information on how to implement logging, but less on what to log. This blog entry offers some guidelines to a solid approach to application logging that will help reconstruct the timeline of events that happened before a software failure.

So without further ado, let’s get stuck in. Here are my tips for writing application logs.

## Remember why you are logging

In a nutshell, an application that works perfectly doesn’t need logs. It’s only when something goes wrong that logs are even looked at. So when writing code that emits logs, aim to provide relevant contextual information of what the software is doing.

## Keep in mind who reads logs

This could be a support analyst or another developer on your team and they will not necessarily have the same depth of knowledge of the application as you do. Aim to provide them with the information that they need to figure out what the application was doing before it failed.

## Add detail to each log

Imagine that you have been asked to investigate an issue that affects some users when they login to your system. You recover the logs and start browsing through. Would you prefer to read ‘Logging on’ or ‘Logging on user {userName} at URL {loginUrl}’?

Maybe the issue being investigated is only related to a particular user, or maybe the login URL has been incorrectly configured. You can’t know that ahead of time, so provide relevant contextual information in the log message.

```csharp
logger.Trace($"Logging on user {userName} at URL {loginUrl}...");
```

## Avoid ambiguity

Be clear about whether an event is about to happen or has already happened.

- Prefer the present continuous tense for something that is about to happen: ‘Trying to connect’ or ‘Querying DB’
- Use the past tense for something that just happened: ‘Successfully connected’ or ‘Queried DB’ or ‘Failed while…’

## Add a log for each external call

- Before the call emit a trace.
- If the call succeeds emit a second trace.
- If it fails then emit an error and include the exception message and call stack if available.

```csharp
try
{
    logger.Trace("Querying DB for product list...");
    var products = db.GetProducts();
    logger.Trace($"Got {products.Length} products");
}
catch (Exception e)
{
    logger.Error(e, "Failed to query DB for product list");
    throw new Exception(e, "Failed to query DB for product list");
}
```

## Log all exceptions as they happen

Log exceptions systematically, even if they are handled by the code. This may help if the exception is not being handled correctly, or the exception is a precursor to a later software failure.

```csharp
try
{
    ... // Send an email
    logger.Info($"Sent email to {emailAddress}");
}
catch (Exception e)
{
    // Log the error
    logger.Error(e, $"Failed while sending an email to {emailAddress} using smtp server {smtpServer}");
    // Rethrow with extra contextual information
    throw new Exception(e, $"Failed while sending an email to {emailAddress} using smtp server {smtpServer}");
}
```

## Use a logging framework

Logging frameworks like Microsoft.Extensions.Logging, NLog and Log4Net offer many options. A few common concepts are log targets, levels and rules, which are managed in configuration and can be changed on the fly without any code changes.

- Targets define where the log is sent, from console output, through rotating timestamped files, to advanced network distributed logging.
- Log levels indicate the significance of an event and can be used to filter log entries.
- Log rules allow you to apply filters to log entries and route them to logging targets.

## Use log levels

The logging framework will provide different severity levels. NLog gives this handy list of log levels, with examples of typical use:

| Level	| Typical Use |
| ----- | ----------- |
| Fatal	| Something bad happened; application is going down |
| Error	| Something failed, application may or may not continue |
| Warn  | Something unexpected; application will continue |
| Info  | Normal behaviour like mail sent, user updated profile etc. |
| Debug	| For debugging; executed query, user authenticated, session expired |
| Trace	| For trace debugging; begin method X, end method Y |

## Use rolling logs

If logging to file, make sure you use rolling logs, and if available use a disk partition that is separate from the OS. Without these precautions, over time the log files will grow, and you don’t want to fill up the OS partition with logs.

## Don’t pretty print exception messages

Surface up all the gory details of the message and stack trace. Logging frameworks will provide a way of formatting exception messages and stack traces, let them do their job. You don’t want to hide details that might be useful.

In one company where I worked there were a high number of unhandled exceptions with just the information ‘Null reference exception’ but no call stack or contextual information. It was basically impossible to resolve any of those issues without going back to the code and fixing the exception handling first.

## Don’t leak secrets

Redact passwords and keys. You can’t be certain of the role of the person reading the logs. The log contents may end up in an email chain in a support request. So you should take care when logging sensitive information. A common practice is to replace secrets and passwords with asterisks. Or you might decide to show the start or end of the secret and replace the rest with asterisks.

## In summary

Logs are your friend, they can help you get out of a tight spot. So look after them, log often and log clearly.

I don’t think I have ever complained about having too many logs. However badly written logs, ambiguous logs, too few logs or no logs at all make diagnosing and fixing production failures difficult.