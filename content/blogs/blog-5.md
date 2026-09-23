+++
date = '2026-09-23T12:13:56Z'
draft = false
title = 'Finding the root cause of high latency in your Backend Application'
+++

---
One of the more frustrating backend problems is an API that is slow without an obvious reason.

You might have an endpoint that normally responds in a few hundred milliseconds, but suddenly starts taking two or three seconds. You check the CPU and memory usage, and everything looks normal. The database doesn't appear to be under heavy load either.

So where is all that time going?

A typical backend request rarely does just one thing. It might authenticate a user, query a database, call another service, fetch something from a cache, and perform some application logic before finally returning a response.

If the entire request takes three seconds, looking at the total response time doesn't tell you which of those operations caused the delay.

You could add a few log statements:

```go
log.Println("starting database query")

user, err := findUser(ctx, userID)

log.Println("finished database query")

if err != nil {
    return err
}
```

That can help, but as an application grows, adding and searching through logs for every operation becomes tedious. You also need some way to connect those individual events back to the same request.

A better approach is to collect information about the request as it moves through your application.

That's the problem distributed tracing tries to solve, and OpenTelemetry is one of the tools you can use to implement it.

{{< tableofcontents >}}

---
## What is OpenTelemetry?

OpenTelemetry, or OTel, is an open-source observability framework for collecting telemetry from your applications.

It supports three common types of telemetry:

- **Traces** tell you what happened during a request.
- **Metrics** tell you how your system is behaving over time.
- **Logs** record individual events.

For our latency problem, traces are the most interesting part.

A trace represents a single request, while a span represents an individual operation performed during that request.

For example, instead of only knowing that this request took three seconds, you might be able to see that:

```
GET /checkout     3.0s
database query    40ms
payment API       2.8s
response          10ms
```

Now you have a much better idea of what to investigate.

The application itself might be perfectly fine. The majority of the latency could be coming from a downstream service.

---
## Adding tracing to a Go application

Let's say we have a simple Go handler:

```go
func checkout(w http.ResponseWriter, r *http.Request) {
    user, err := findUser(r.Context(), "123")
    if err != nil {
        http.Error(w, "internal server error", http.StatusInternalServerError)
        return
    }

    // Process checkout...

    json.NewEncoder(w).Encode(user)
}
```

At the moment, we know that "checkout" was called, but we don't know how long "findUser" took.

We can create an OpenTelemetry span around the operation:

```go
func findUser(ctx context.Context, userID string) (User, error) {
    ctx, span := tracer.Start(ctx, "findUser")
    defer span.End()

    // Database query...

    return user, nil
}
```

The important part is the "Start" and "End" calls.

```go
ctx, span := tracer.Start(ctx, "findUser")
defer span.End()
```

OpenTelemetry can now measure how long the operation took and associate it with the request that called it.

You can do the same thing around other operations:

```go
func chargePayment(ctx context.Context) error {
    ctx, span := tracer.Start(ctx, "chargePayment")
    defer span.End()

    // Call payment service...

    return nil
}
```

If the payment service starts taking two seconds, the trace can make that visible.

---
## Context connects the operations

You may have noticed that the "context.Context" is passed into "tracer.Start".

That's important because requests usually contain multiple operations.

For example:

```go
func checkout(ctx context.Context) error {
    ctx, span := tracer.Start(ctx, "checkout")
    defer span.End()

    if err := findUser(ctx); err != nil {
        return err
    }

    if err := chargePayment(ctx); err != nil {
        return err
    }

    return nil
}
```

Each function receives the context from the previous operation.

OpenTelemetry uses that context to associate the spans with the same request.

This becomes particularly useful when your application communicates with other services.

Imagine your Go API calls a payment service, which then calls a database. With distributed tracing and proper context propagation, a trace can follow that request across service boundaries.

You don't have to manually correlate unrelated log files and timestamps to figure out what happened.

---
## Recording useful information

A span can also contain additional information about an operation.

For example:

```go
ctx, span := tracer.Start(ctx, "findUser")
defer span.End()

span.SetAttributes(
    attribute.String("user.id", userID),
)
```

You can use attributes to provide additional context when investigating a trace.

You can also record errors:

```go
if err != nil {
    span.RecordError(err)
    return err
}
```

This is useful when the slow operation is also failing.

You should be careful about the data you attach to telemetry, though. Authentication tokens, passwords, and other sensitive information shouldn't be casually added to spans.

---
## What about the OpenTelemetry Collector?

Your application also needs somewhere to send its telemetry.

For a small experiment, you can export traces directly from your Go application. In a production environment, you'll often see an OpenTelemetry Collector involved.

The Collector receives telemetry from your applications and can process and export it to an observability backend.

That means your application can be instrumented with OpenTelemetry without being tightly coupled to a particular monitoring product.

You can change where the telemetry goes without having to rewrite all of your instrumentation.

---
## Finding the root cause

Let's return to the original problem.

Your users report that "/checkout" is slow.

You look at a trace for one of the slow requests and find that the request took 2.7 seconds.

Most of that time came from:

```
POST /payment    2.5s
```

The database query took 30ms.

Your own application code took another 20ms.

The problem isn't your database or the Go handler. The request is spending most of its time waiting for the payment service.

That distinction is important.

Without tracing, you might spend an hour investigating your database configuration, connection pool, indexes, or application code when the actual delay is somewhere else.

OpenTelemetry doesn't make the payment service faster. It gives you the information needed to identify where the time is going.

And that's the part I find most useful about observability.

When an application becomes slow, you don't want to guess which component is responsible. You want to be able to look at an actual request and see what happened.

OpenTelemetry gives you the building blocks to do exactly that.

{{< nextprev >}}
