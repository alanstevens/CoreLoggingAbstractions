# Core Logging
This project is a set of abstractons over the dotnet core logging framework.

It is exciting that logging is a first class service in the dotnet core framework now. Here you will find some wrappers and extensions to make logging in dotnet core even easier to work with. There are three levels of abstraction.

## Testable Interfaces
While Microsoft provides an `ILogger` interface, the frequently called logging methods are all extension methods. This makes it difficult to determine if your logging message was called. Steve Smith has an excellent [article](https://ardalis.com/testing-logging-in-aspnet-core) laying out these difficulties.

I have followed his recommendation to use an adapter to wrap the framework `Logger`. The result is `ICoreLogger` and `CoreLogger`. `ICoreLogger` is easy to mock and test and has the most used logging methods directly on the interface rather than using extension methods.

## Static Logger
While injecting an `ICoreLogger` is neat and testable, putting the interface on every constructor gets old fast. I believe Logging should be available everywhere. My first solution is a static logger class called `ApplicationLogger`. Static classes can be tricky to test. `Application Logger` is safe to call at test time. 

Internally it uses a set of delgates.  If  `Initialize` is never called, the logging methods are a no-op. If you want to verify logging with `ApplicationLogger`, mock an `ICoreLoggerFactory` that returns a mocked `ICoreLogger`. Call `ApplicationLogger.Initialize()` with your mocked factory and verify calls to the `ICoreLogger`.

## Extension Methods
This may be one step too far down the rabbit hole for some people. Because I want logging to be ambiently available everywhere, I have created a set of extension methods on `object` for the most common logging methods.  This means you can simply call `this.LogDebug("My message.)`. 

I put the extension methods in a seperate namespace so that they will not pollute your intellisense unless you explicitly import the namespace. The extension methods call `ApplicationLogger` internally, so they can be tested the same as above.

## Testing
See the unit tests for examples of the test approaches I describe above.

## Startup
There is a `.AddCoreLogging()` extension method on `ServiceCollection` to configure Core Logging. Simply chain `.AddCoreLogging()` after `.AddMVC()` in `Startup.cs`.