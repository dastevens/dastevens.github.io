---
layout: post
title:  "Minimal Scope"
date:   2021-02-07 12:07:25 +0100
categories: coding design software
---
Code complexity can be split in two: the innate complexity of the problem, and the accidental complexity that has been allowed to creep into the solution.

So an obvious question is can we reduce accidental complexity, and will it help us to write more expressive code?

## Reducing scope reduces complexity

So what do I mean exactly by scope? 

Let’s review a simple code example to explore some ideas. Suppose we’re writing a service in a web app that needs to log user sessions. A reasonable implementation, one that would sit comfortably in many codebases, would be something like this:

```csharp
void LogSession(HttpContext httpContext, User user)
{
  var url = httpContext.Request.RawUrl;
  var username = user.UserName;
  this.db.tblSessionLogs.Add(new SessionLog(username, url));
}
```

This code is not bad code: it’s fairly clear in what it does, the method is named appropriately, there are no real surprises.

However I would argue that it has a weakness: the method signature is far too generous. Both arguments are rich objects that contain all kinds of properties, and yet the method only needs two key pieces of information.

And this is really what I mean by scope. Another term could be the size of the execution context. I can imagine a code metric that quantifies it by something like the number of objects that could be accessed. It would be a count of all variables, local or global, and their properties, and their properties’ properties, etc. A count of all the leaves of the object graph that is in scope.

Reducing scope means the object graph that is available is smaller. As a consequence, the developer has fewer concepts to keep track of and can instead concentrate on the job in hand, which is in this case writing session logs. 

With the method signature as it stands, the code could potentially access all kinds of information from the HttpContext and from the outside there is no way of telling, the only way is to read the code. 

As is often the case, the problem is laid bare when we try to write some unit tests around this method. To exercise LogSession, we would need to provide an HttpContext and a User object, and to determine which properties would need to be set, the only way forward would be to read the code.

Now I have nothing against reading code, on the contrary I positively love reading code. But to understand how a method works, it shouldn’t be necessary. It should be obvious what a method is going to do from the outside. We shouldn’t have to fish out the source code and wade through all the ifs and fors.

If UX design is centred around the user experience of an app, then when writing code we should be thinking of DX design: focusing on the developer experience. 

So can we improve this method? Well let’s try by reducing the scope. To log a session, all that is required is a url and a username, so let’s see how that works out.

```csharp
void LogSession(URL url, string username)
{
  this.db.tblSessionLogs.Add(new SessionLog(username, url));
}
```

By requiring two basic types we have refined the expectation of what this method does. We shouldn’t need access to the source code and so the developer experience is improved. 

As a welcome side effect, writing a test becomes much easier; no need for partially initialized HttpContext and User objects.

Now you could say that I have just displaced the problem, and I think to some extent that is true. The calling code now needs to break down the HttpContext and User objects whereas before it just passed those in without any fuss. I would argue that the calling method is better placed to do that job. It will probably know all about the HttpContext and User objects since they are already in its scope.

But reducing scope brings another welcome benefit. The method is now more reusable. It doesn’t just log http user sessions but it could be used to log ftp sessions. Although the example is a little contrived, this generalisation comes about precisely because the scope has been reduced.

We may be able to apply the same reduction to the calling code too. It might be that it has no direct use of the HttpContext apart from when logging sessions. The code could be simplified several levels through the call stack as we work up through the calling methods.

So to wrap up, reducing the size of scope of a function is a simple technique, but it can bring several benefits: reasoning about the code becomes easier; writing unit tests is simplified; and the code may be more reusable.