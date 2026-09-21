+++
date = '2026-09-21T08:21:01Z'
draft = false
title = 'Five features of Go that I like and appreciate'
+++

---
I've been writing more Go lately, and I've found myself
appreciating some of its features more and more.
Here are five features that have made Go particularly enjoyable for me.

## 1. The "defer" keyword

"defer" is probably one of those features that becomes more useful the more Go you write.

For example:

```go
file, err := os.Open("data.txt")
defer file.Close()
if err != nil {
    return err
}
```

I can handle the cleanup immediately after opening the resource instead of having to remember to do it somewhere else in the function.

The same pattern works nicely with database, mutexes, HTTP response bodies, and other resources that need to be cleaned up.

It's a small feature, but it makes code easier to follow.

---
## 2. Interfaces

Go's interfaces are interesting because they're implemented implicitly.

```go
type Writer interface {
    Write([]byte) (int, error)
}
```

A type doesn't need to explicitly declare that it implements "Writer". If it has the required method, it satisfies the interface.

I like this approach because I can define an interface around the behavior I actually need rather than creating an interface hierarchy upfront.

It also makes testing pretty straightforward. If my function only needs something that can write data, I can pass it any type that satisfies that interface.

---
## 3. Goroutines

Concurrency is another area where Go feels particularly nice.

Starting concurrent work can be as simple as:

```go
go doSomething()
```

There's obviously more to concurrency than starting a goroutine. Once shared state gets involved, you still have to think about synchronization, race conditions, channels, and data ownership.

But the language gives you a lightweight primitive for concurrent work without making you deal with threads directly.

For backend applications, where I/O and concurrent requests are common, this is especially useful.

---
## 4. The standard library

Go's standard library is probably one of the biggest reasons I enjoy working with the language.

Need an HTTP server?

```go
http.ListenAndServe(":8080", handler)
```

Need JSON?

```go
json.Marshal(data)
json.Unmarshal(data, &data)
```

There are packages for HTTP, files, cryptography, SQL, testing, logging, command-line arguments, and a lot more.

I don't have to install a package every time I need to do something slightly outside the language itself.

That makes starting a new project feel refreshingly straightforward.

---
## 5. Go Test Package

The Go testing tools are another thing I appreciate.

A basic test looks like this:

```go
func TestAdd(t *testing.T) {
    got := Add(2, 3)

    if got != 5 {
        t.Errorf("got %d, want 5", got)
    }
}
```

Then I can run:

```sh
go test ./...
```

Testing is treated as part of the normal Go development workflow rather than something that requires another framework before I can get started.

I also like that the same tooling is used across Go projects, so moving between repositories doesn't usually mean learning a completely different testing setup.
