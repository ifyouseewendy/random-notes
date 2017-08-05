# Pluming Event Machine

Reference

* [Advanced Eventmachine by Jonathan Weiss](https://www.youtube.com/watch?v=mPDs-xQhPb0)
* [Defeating blocking I/O with eventmachine](https://www.slideshare.net/noverloop/defeating-blocking-io-with-eventmachine)
* [EventMachine internals and the Reactor pattern](https://medium.com/@neektza/eventmachine-internals-and-the-reactor-pattern-55ae0f491d8e)

## Basic

* Process multiple I/O “at once” without the need for threads.
* Register callbacks to respond to events

![](/assets/1D9AECDE-4E6F-4637-A487-B448B472E3FD.png)

This is the event loop:

```ruby
EM.run do
  …
  …
  EM.stop
end
```

**What happened in the event loop?**

![](/assets/event-loop.png)

EM run timers, check FDs and networking stuff, then run the callbacks you registered. So, there should no synchronous call in the event loop. \(Note, even if you are reading a lot of messages on the sockets can somehow slow down the event loop\)

## Advanced

### [EM.next\_tick](http://www.rubydoc.info/github/eventmachine/eventmachine/EventMachine.next_tick)

It runs the block you passed in the next iteration of the event loop. If you are doing a bunch of operations, you can use it to reschedule your work. It can bring you back from a background thread. It’s also useful to move I/O from deferred thread to main thread.

```ruby
work = proc do
  do_something()
  EM.next_tick &work
end


EM.next_tick &work
```

### [EM.defer](http://www.rubydoc.info/github/eventmachine/eventmachine/EventMachine.defer)

It runs the block you passed in a background thread. If you have long running computations / processes, if you do something that’s potentially blocking the main loop, wrap it in a `EM.defer`. If you are doing I/O, it should be rescheduled to the main loop using `EM.next_tick`.

`EM.defer` is used to push a otherwise blocking operation on an EM internal queue whose jobs are then processed by a thread pool of 20 threads \(by default\). You can use `EM.defer` with operations which aren’t made to work asynchronously. `EM.defer` is one of EventMachine’s mechanisms for lightweight concurrency. `EM.defer` makes it easy and graceful to initiate a long-running \(“deferrable”\) operation \(such as a call to an external web site\), and attach any number of code blocks to be executed when the deferrable operation completes.

“I/O in the main reactor thread, and anything that’s potentially slow in the deferred thread. If you are not sure, wrap it in a `EM.next_tick` or a `EM.defer` block”.

![](/assets/eventmachine-in-practice.png)

Demo 1: Running operations in deferred threads and callbacks in the main loop, which means IO can block deferred threads.

```ruby
require "eventmachine"

operation = proc {
  puts "defer: #{Thread.current}"
  # perform a long-running operation here, such as a database query.
  "result"
}
callback = proc { |_| puts "callback: #{Thread.current}" }
errback = proc {}

EM.run do
  puts "main: #{Thread.current}"

  EM.defer(operation, callback, errback)

  puts "running..."
end

main: #<Thread:0x007fa2a207f390>
running...
defer: #<Thread:0x007fa2a404fb98>
callback: #<Thread:0x007fa2a207f390>
```

Demo 2: Use event based gem to enable the deferred threads to register a callback for a HTTP request, which leverages the power of event loop.

```ruby
require "eventmachine"
require "em-http-request"
require "httparty"

blocking_operation = proc {
  puts "defer: #{Thread.current}"
  time = Time.now.to_f
  10.times { HTTParty.get("https://google.com").response } # blocking IO
  puts "finish operation: #{Time.now.to_f - time} s"
}

non_blocking_operation = proc {
  puts "defer: #{Thread.current}"
  time = Time.now.to_f
  10.times { EM::HttpRequest.new("https://google.com").get } # non-blocking IO
  puts "finish operation: #{Time.now.to_f - time} s"
}

callback = proc do |_|
  puts "callback: #{Thread.current}"
  EM.stop
end
errback = proc {}

EM.run do
  puts "main: #{Thread.current}"

  EM.defer(blocking_operation, callback, errback)
  # main: #<Thread:0x007fd68c87f3b0>
  # running...
  # defer: #<Thread:0x007fd68cdf0c90>
  # finish operation: 3.247986078262329 s
  # callback: #<Thread:0x007fd68c87f3b0>

  # EM.defer(non_blocking_operation, callback, errback)
  # main: #<Thread:0x007fb18707f3b0>
  # running...
  # defer: #<Thread:0x007fb189a1ed88>
  # finish operation: 0.013278961181640625 s
  # callback: #<Thread:0x007fb18707f3b0>

  puts "running..."
end
```

### [Deferrables](http://www.rubydoc.info/github/eventmachine/eventmachine/EventMachine/Deferrable)

Basically, it’s a state machine with callbacks for building evented libraries. Status: nil, succeed, failed. You can register the callback at anytime.

* callback, executed when transitioned to :succeed or already :succeed
* errback, executed when transitioned to :failed or already :failed

Allows for EventMachine style concurrency in your code.

### [EM::Queue](http://www.rubydoc.info/github/eventmachine/eventmachine/EventMachine/Queue)

A cross thread, reactor scheduled, linear queue.

### [EM::Channel](http://www.rubydoc.info/github/eventmachine/eventmachine/EventMachine/Channel)

Provides a simple thread-safe way to transfer data between \(typically\) long running tasks in defer and event loop thread.

### [EM::Iterator](http://www.rubydoc.info/github/eventmachine/eventmachine/EventMachine/Iterator)

A simple iterator for concurrent asynchronous work.

* Process the given items in parallel
* Callback when all are ready
* You need to manually signal the iteration end

For example, to fetch 10 urls at a time, simply pass in a concurrency of 10:

```ruby
responses = urls.map{ |url| sync_http_get(url) } ... puts 'all done!’
EM::Iterator.new(urls, 10).map(
  proc{ |url,iter| async_http_get(url){ |res| iter.return(res) } },
  proc{ |responses| ... puts 'all done!’ }
)
```

### [EM::Schedule](http://www.rubydoc.info/github/eventmachine/eventmachine/EventMachine#schedule-class_method)

Runs the given callback on the reactor thread, or immediately if called from the reactor thread. Accepts the same arguments as Callback

### [Fibers](http://ruby-doc.org/core-2.4.1/Fiber.html) & [EM-Synchrony](https://github.com/igrigorik/em-synchrony)

Write synchronous code with asynchronous underneath.

### [Promise.rb](https://github.com/lgierth/promise.rb)

Promise.rb is a thin wrapper on `Em.defer`. It helps us avoid passing callbacks all the time, instead features us thenable \(chainable\) operations. Better handle I/O in the main loop and everything potentially slow in the deferred threads.

~~If blocking I/O happens in deferred thread, we can use Em.next\_tick to transfer the control to the main thread. That’s the reason why to use Promise.rb, at least we have to define a defer method yielding operations in Em.next\_tick. ~~

No, the `EM.next_tick` method Promise.rb uses is called to transfer the control to main loop to run the callback.

### Concerns

1. Exception handling
2. Test

## Wrap up

1. Do not block the event loop \(Even though you are not using synchronous code, you still should not do anything that takes too long.\)
2. No blocking I/Os in the main loop. Put everything potentially slow to deferred threads
3. Handle all I/O in the main loop
4. Avoid non-evented libraries
5. Callback runs until finished. \(So split workload over multiple iterations if needed: `EM.next_tick`\)
6. Use `EM.next_tick` and `EM.defer` to make sure you are in the right context
7. Use Deferrables and Fibers to make the code nicer

Here, I got confused when I reviewed the notes here. We say “handle I/O in the main loop”, but why no blocking I/Os in the main loop. Isn’t it possible that you put a DB query in the main loop and EM will run it asynchronously and automatically? NO! EM features `select` underneath only means that it will help you detect the IO changes and run your callbacks asynchronously and automatically. So, you have to register your blocking operation \(DB query\) with a callback to transform it into a non-blocking operation.

In some way, event based program is all about registering callbacks. Keep a loop running without blocking operations. We can transform synchronous code into event with callbacks, thus to enable asynchronism. To schedule the callbacks, one is to use `EM.next_tick` to make it ran later in the main loop and the other is to use `EM.defer` to make it ran in threads. When I/O happens in a deferred thread, we can use `EM.next_tick` to kick the control back to the mail loop.

