+++
date = '2026-09-22T05:36:53+08:00'
draft = false
title = 'Why you must explicitly handle Signals in your Backend Application?'
+++

---
If you've built a backend application, you've probably written something like this:

```go
func main() {
    server := http.Server{
        Addr: ":6969",
    }

    server.ListenAndServe()
}
```

It works. The server starts, accepts requests, and does its job.

But what happens when you stop it?

You press "Ctrl+C", your container gets stopped, Kubernetes terminates your pod, or your process receives a termination signal.

The process can simply exit.

And that's where things can get messy.

This is why backend applications should explicitly handle operating system signals.

**Spoiler alert:** this is what graceful shutdown is for.

{{< tableofcontents >}}

---
## What is a signal?

A signal is basically a message sent to a running process by the operating system or another process.

For example, when you press "Ctrl+C" in your terminal, your application receives "SIGINT".

There are a few signals you'll commonly encounter when running backend applications:

- **SIGINT** — usually sent when you press "Ctrl+C"
- **SIGTERM** — asks the process to terminate
- **SIGKILL** — immediately kills the process and cannot be handled by the application

For backend applications, **SIGTERM** is particularly important.

For example, when your application is running inside a container, the container runtime can send "SIGTERM" when the container is being stopped.

Your application then has a chance to clean itself up before it exits.

---
## What happens if you don't handle the signal?

Imagine your server is currently processing a request.

Maybe the request is:
```POST /checkout```

The application receives the request and starts doing some work.

Then the process receives **SIGTERM**.

If your application immediately exits, that request can be interrupted.

Depending on what your application was doing, you could end up with things like:

- incomplete HTTP requests
- interrupted database transactions
- abruptly closed connections
- unacknowledged messages
- unflushed files
- interrupted background jobs

Not every application will experience all of these problems, but the underlying issue is the same:

**the application doesn't get a chance to clean up before it dies.**

---
## This is where graceful shutdown comes in

Graceful shutdown means giving your application a chance to finish what it's currently doing before completely shutting down.

Instead of immediately terminating the process, the application can stop accepting new work, finish the work that's already in progress, close its resources, and then exit.

In Go, you can listen for signals using the **os/signal** package.

For example:
{{< highlight go "hl_lines=7-18 22-31" >}}
ctx, stop := signal.NotifyContext(
    context.Background(),
    os.Interrupt,
    syscall.SIGTERM,
)
defer stop()

server := http.Server{
    Addr: ":6969",
}

go func() {
    if err := server.ListenAndServe(); err != nil &&
        !errors.Is(err, http.ErrServerClosed) {
        log.Fatal(err)
    }
}()

<-ctx.Done()

log.Println("shutting down server...")

shutdownCtx, cancel := context.WithTimeout(
    context.Background(),
    10*time.Second,
)
defer cancel()

if err := server.Shutdown(shutdownCtx); err != nil {
    log.Printf("server shutdown: %v", err)
}
{{< /highlight >}}

The important part is:
```go
<-ctx.Done()
```

The application waits until one of the signals we're interested in arrives.

Once that happens, we can start shutting the server down.

---
## Why use "Shutdown()" instead of just closing the server?

This distinction is important.

If you simply close the server, you're essentially saying:

"We're done. Stop everything."

"http.Server.Shutdown()" gives active connections an opportunity to finish while preventing the server from accepting new connections.

That's exactly what we want during a graceful shutdown.

The timeout is important too:

```go
context.WithTimeout(context.Background(), 10*time.Second)
```

You don't want graceful shutdown to mean waiting forever for one request that got stuck.

If something takes too long, the context expires and the shutdown can continue.

---
## Graceful shutdown isn't just about HTTP

This pattern becomes even more useful when your application has other resources.

For example, your backend might have:

- an HTTP server
- database connections
- Redis connections
- message queue consumers
- background workers
- file handles

When shutting down, you might need to stop accepting new HTTP requests, stop background workers, finish active work, and close your connections.

The exact order depends on your application.

The important part is that shutdown becomes an intentional part of your application's lifecycle.

---
## Signals become even more important in containers

This becomes much more obvious once your backend runs in Docker, Kubernetes, or another containerized environment.

Containers are frequently started and stopped.

Deployments happen.

Pods get replaced.

Machines restart.

Instances scale down.

Your application shouldn't assume that its process will live forever.

A production application needs to be prepared for its process to be asked to terminate.

That's why graceful shutdown isn't just something you add because it looks good in a backend tutorial.

It's part of running a server reliably.

---
## There's one signal you can't gracefully handle

There's an important exception:

**SIGKILL**

You can't catch or handle "SIGKILL".

When the operating system sends it, the process is terminated immediately.

This is why graceful shutdown should also be relatively fast and predictable.

You don't want your application to take several minutes to shut down.

---
## The bigger lesson

Handling signals is really about acknowledging that your application has a lifecycle.

It doesn't just start and run forever.

Eventually, something is going to ask it to stop.

When that happens, your application should have a plan.

It should know when to stop accepting new work, what work needs to finish, which resources need to be closed, and when it's safe to exit.

Once you start thinking about your backend this way, graceful shutdown becomes much less mysterious.

You're simply giving your application a chance to clean up before the process disappears.

And that's a much better way to stop a backend application than simply pulling the plug.

{{< nextprev >}}
