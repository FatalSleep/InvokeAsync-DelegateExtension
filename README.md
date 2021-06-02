# InvokeAsync-DelegateExtension
Better implementation for invokable async events by Aepot on StackExchange.

Invokable asynchronous event handling via a delegate extension method. Name is irrelevant, must be static and public to work. Requires the following include: `System`, `System.Linq` and `System.Threading.Tasks`.
```C#
public static class DelegateExtensions {
    public static Task InvokeAsync<TArgs>(this Func<object, TArgs, CancellationToken, Task> func, object sender, TArgs e, CancellationToken t) =>
        func == null ? Task.CompletedTask : Task.WhenAll(func.GetInvocationList().Cast<Func<object, TArgs, CancellationToken, Task>>().Select(f => f(sender, e, t)));
}
```
Example Usage:
```C#
using System;
using System.Threading.Tasks;
using System.Linq;

namespace ConsoleApp1 {
    public static class DelegateExtensions {
        public static Task InvokeAsync<TArgs>(this Func<object, TArgs, Task> func, object sender, TArgs e) =>
            func == null ? Task.CompletedTask : Task.WhenAll(func.GetInvocationList().Cast<Func<object, TArgs, Task>>().Select(f => f(sender, e)));
    }

    class Program {
        public static event Func<object, EventArgs, Task> MyAsyncEvent;

        static async Task SomeAsyncMethodA(object src, EventArgs args, CancellationToken tkn) {
            Console.WriteLine("Task A...");
            await Task.Delay(2000);
            Console.WriteLine("Task A finished");
        }

        static async Task SomeAsyncMethodB(object src, EventArgs args, CancellationToken tkn) {
            Console.WriteLine("Task B...");
            await Task.Delay(1000);
            Console.WriteLine("Task B finished");
        }

        static async Task Main(string[] args) {
            MyAsyncEvent += SomeAsyncMethodA;
            MyAsyncEvent += SomeAsyncMethodB;

            await MyAsyncEvent.InvokeAsync(null, EventArgs.Empty, null);

            Console.WriteLine("Invoked");
            Console.ReadKey();
        }
    }
}
```
