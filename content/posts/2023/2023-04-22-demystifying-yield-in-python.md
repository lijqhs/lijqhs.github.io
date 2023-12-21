---
title: Demystifying Yield in Python
slug: demystifying-yield-in-python
date: 2023-04-22
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/1/ales-krivec-okzxVsJNxXc-unsplash.jpg
thumbnailImagePosition: left
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/1/ales-krivec-okzxVsJNxXc-unsplash.jpg
categories:
- python
tags:
- python
- yield
- generator
- coroutine
- asyncio
- async
- asynchronous programming
---

Python's `yield` statement is a powerful tool that can be used in a variety of contexts, including generator functions, coroutines, and asynchronous programming.

<!--more-->

{{< toc >}}

## Generators

`yield` was introduced in Python 2.2 as a simple way to create generators ([PEP 255](https://peps.python.org/pep-0255/)). Generators are functions that can pause their execution and return a value, then resume when `next` is called on them again. This "pausing function" functionality of `yield` enables some very powerful programming concepts in Python.

When a generator function is called, it returns a generator object that can be used to generate the values one at a time. Each time the `yield` statement is encountered in the generator function, the function is paused and the value following the `yield` keyword is returned. The next time the generator is called, execution resumes after the last `yield` statement.

Here's an example of a simple generator function:

```python
def double_gen(n):
    for x in range(n):
        a = 2 * x
        yield a
        print(f"last value: {a}", end=', ')

gen = double_gen(3)

print(f"curr value: {next(gen)}") 
# curr value: 0
print(f"curr value: {next(gen)}") 
# last value: 0, curr value: 2
print(f"curr value: {next(gen)}") 
# last value: 2, curr value: 4
print(f"curr value: {next(gen)}") 
# Traceback (most recent call last):
#   File "<stdin>", line 1, in <module>
# StopIteration
# last value: 4, 
```


- In this example, we define a generator function called `double_gen` that generates the doubles of the numbers from `0` to `n` using the `yield` keyword. 
- We then create a generator object called `gen` by calling the function. When `next()` is first invoked, it sets `a` to `0` and yields `a` back to its caller, where we print `curr value: 0`. 
- When `double_gen` is resumed, `double_gen` continues after the `yield` with local state intact, where we print `last value: 0, `, and `double_gen` loops back to the `yield`, yielding `2` (`a = 2 * 1`) to its invoker. 
- And so on, until the generator function runs out, at this point a `StopIteration` exception is raised.

### Generator Expressions

In addition to using generator functions, generators can also be created using generator expressions ([PEP 289](https://peps.python.org/pep-0289/)), which are similar to list comprehensions and return an generator instead of a list.

```python
gen = (x**2 for x in range(10))
# print(gen)
# <generator object <genexpr> at 0x7fbd75d43ae0>
```

Generator expressions are essentially syntactic sugar for generators. They provide a concise way to create simple generators, but ultimately they compile down to the same generator objects that can be created using `def`. The above example is equivalent to:

```python
def gen():
    for x in range(10):
        yield x**2
```

### Context Managers

[Context managers](https://www.aaronnotes.com/blog/posts/context-managers-in-python/) are objects that define a context in which a set of operations can be performed. Context managers can be defined by **generators** using the `@contextmanager` decorator, in which case, `yield` keyword used in generator acts more like a separator between code entering and exiting the context manager.

```python
from contextlib import contextmanager

@contextmanager
def file_opener(filename, mode):
    try:
        # entering context manager
        f = open(filename, mode)
        yield f
        # exiting context manager
    finally:
        f.close()
```

- In this example, we have defined a `file_opener` context manager using the `@contextmanager` decorator. The `file_opener` function is a generator function that yields a file object, which can be used by the calling code to read or write data to the file.
- The `try` block in the `file_opener` function opens the file with the specified filename and mode. The `yield` statement produces the file object and temporarily suspends the generator function, allowing the calling code to resume execution. The calling code can use the file object to read or write data to the file.
- The `finally` block in the `file_opener` function ensures that the file is closed properly when the context is exited, even if an exception occurs during execution.

Here's an example of how you can use the `file_opener` context manager:

```python
with file_opener("hello.txt", "w") as f:
    f.write("Hello, world!")
```

## `yield` expressions

Python 2.5 ([PEP 342](https://peps.python.org/pep-0342/)) introduced methods and adjustments to **use generators to implement coroutines** and other forms of co-operative multitasking. The most interesting features were `yield` expression and `send()` method. 

For example:

```python
def guess(n):
    hint = None
    while True:
        x = yield hint
        if x > n:
            hint = 'too big'
        elif x < n:
            hint = 'too small'
        else:
            hint = 'bingo'
```

```python
>>> game = guess(5)
>>> game.send(None)
>>> game.send(1)
'too small'
>>> game.send(10)
'too big'
>>> game.send(5)
'bingo'
```

- In this example, first we call the generator function to define a generator `game = guess(5)` to play a guess number game with target number 5. 
- And then we call `game.send(None)` just like `next()` to advance the generator `guess` to the `yield` which will pause the execution and yield a `None` value. 
- After we call `game.send(1)`, the generator `guess` receives a value via `x = yield` (think it as `x = f()`, where the return value of `f()` is `1` which is sent by `game.send(1)`) and resumes its execution (executing `if-elif-else` block) until `yield hint` and returns local state `hint` to its caller `game.send(1)` (like `next()`), where `game.send(1)` returns `'too small'`.

The `send()` method allows you to resume the execution of the generator or coroutine at the point where it left off, with a new value that is sent back into the generator or coroutine.

## Generator-based Coroutines

`yield` later gained additional power with the introduction of `yield from` (introduced in Python 3.3, [PEP 380](https://peps.python.org/pep-0380/)). `yield from` allowed a generator to delegate to another generator or coroutine. 

For example:

```python
def double_gen(n):
    for x in range(n):
        yield 2 * x

def triple_gen(n):
    for x in range(n):
        yield 3 * x

def delegate_gen(n):
    double_gen(n)
    triple_gen(n)

for x in delegate_gen(3):
    print(x)
# TypeError: 'NoneType' object is not iterable
```

- When we run the above code, we will get `TypeError` because the `delegate_gen` function is just a regular function in which only two generator objects are created.
- Because when we call the generator functions, we simply get the generator object, not exactly call the functions in the normal way and we need to explicitly iterate generator again and again to re-yield values that it produces. 
- In this example `delegate_gen` doesn't have an `yield` statements so it cannot be used as generator in the `for` loop. And this regular function will return `None` as default, that's why we get `TypeError: 'NoneType' object is not iterable`.

With `yield from` expression (the "delegating generator"), we can make `delegate_gen` a generator function.

```python
def double_gen(n):
    for x in range(n):
        yield 2 * x

def triple_gen(n):
    for x in range(n):
        yield 3 * x

def delegate_gen(n):
    yield from double_gen(n)
    yield from triple_gen(n)

for x in delegate_gen(3):
    print(x, end=' ')
# prints: 0 2 4 0 3 6 
```


`yield from` simply provides a clean syntax to delegate control between coroutines, **passing values back and forth**. This enables powerful patterns like coroutine pipelines, trees, and more complex asynchronous algorithms built from simpler components.


With `async def` keyword we can define coroutines from generators:

```python
import asyncio

async def main():
    for x in delegate_gen(3):
        print(x, end=' ')

asyncio.run(main())
# 0 2 4 0 3 6
# main() is a coroutine
```

**Caveats**. According to the documentation [Generator-based Coroutines](https://docs.python.org/3.8/library/asyncio-task.html#generator-based-coroutines): Support for generator-based coroutines is deprecated and is scheduled for removal in Python 3.10. Use `async def` instead.

{{< alert info >}}
Generator-based coroutines predate `async/await` syntax. They are Python generators that use `yield from` expressions to await on Futures and other coroutines.

Generator-based coroutines should be decorated with `@asyncio.coroutine`, although this is not enforced. This decorator enables legacy generator-based coroutines to be compatible with `async/await` code. This decorator should not be used for `async def` coroutines.
{{< /alert >}}

```python
@asyncio.coroutine
def old_style_coroutine():
    yield from asyncio.sleep(1)

async def main():
    await old_style_coroutine()
```


## Native Coroutines

There are a number of shortcomings to implement coroutines via generators because with `yield` or `yield from`. For example, it is sometimes confusing to distinguish coroutines from regular generators. In Python 3.5 ([PEP 492](https://peps.python.org/pep-0492/)) native coroutines and the associated new syntax features such as `async` and `await` were introduced. 

### async def

We can define a native coroutine with `async def`:

```python
async def coro(n):
    pass
```

Some key properties:
- Functions defined with `async def` are always coroutines, even if they do not contain `await` expressions. 
- It is a `SyntaxError` to have `yield` or `yield from` expressions in an `async` function. (`yield` can be used in `async` function, see [PEP 525](https://peps.python.org/pep-0525/))
- Regular generators, when called, return a generator object; similarly, coroutines return a coroutine object.
- `StopIteration` exceptions are not propagated out of coroutines, and are replaced with a `RuntimeError`. 
- `CO_COROUTINE` is used to mark native coroutines and `CO_ITERABLE_COROUTINE` is used to make generator-based coroutines compatible with native coroutines.


### await

`await` uses the `yield from` implementation with an extra step of validating its argument. `await` only accepts an `awaitable`. There are three main types of [awaitable](https://docs.python.org/3/library/asyncio-task.html#awaitables) objects: **coroutines**, **Tasks**, and **Futures**.

## Asynchronous Generators

In Python 3.6 ([PEP 525](https://peps.python.org/pep-0525/)), the concept of asynchronous generators were introduced. 

The asynchronous generator object is modeled after the standard Python generator object. Essentially, the behaviour of asynchronous generators is designed to replicate the behaviour of synchronous generators, with the only difference in that the API is asynchronous. 

Asynchronous generators can be defined with `async def` and `yield`. 

```python
import asyncio

# an asynchronous generator function
async def double_coro(n):
    for x in range(n):
        yield 2 * x
        await asyncio.sleep(1)
```

Asynchronous generators require an event loop to run and finalize them because they are meant to be used from coroutines and whereas an event loop or a scheduler is required to run coroutines. We can wrap it in a coroutine and run it with `asyncio.run`.

```python
async def main():
    async for x in double_coro(3):
        print(x, end=' ')

asyncio.run(main())
# prints: 0 2 4 
```

For an asynchronous generator object `agen`, the following methods and properties are defined:
- `agen.__aiter__()`: Returns `agen`.
- `agen.__anext__()`: Returns an *awaitable*, that performs one asynchronous generator iteration when awaited.
- `agen.asend(val)`: Returns an *awaitable*, that pushes the `val` object in the agen generator. When the `agen` has not yet been iterated, `val` must be `None`.


## Asyncio

`asyncio` is a Python library for asynchronous programming, introduced in Python 3.4 ([PEP 3156](https://peps.python.org/pep-3156/)). It provides a way to write asynchronous, non-blocking code using coroutines, tasks, and event loops.

When the `asyncio` module was first released, it did not support the `async` and `await` syntax. Instead, it used the `yield from` syntax to define coroutines. Later, when the `async` and `await` syntax were introduced in Python 3.5, the `asyncio` module was updated to support this new syntax.

However, to ensure backwards compatibility with legacy code that was using the old `yield from` syntax, the `asyncio.coroutine` decorator function was introduced. Any legacy code that had a function that needed to be run concurrently (i.e. awaited) had to use this decorator function to make it compatible with the new `async` and `await` syntax.

In short, the `asyncio.coroutine` decorator function was a temporary solution to ensure that existing code that used the old `yield from` syntax could still be used with the new `async` and `await` syntax. It allowed developers to gradually transition their code to the new syntax. This feature has been deprecated since Python 3.8.

### Key features

`asyncio` provides a number of key features, including:

- [Coroutines](https://peps.python.org/pep-3156/#coroutines): Coroutines can be defined using the `async def` syntax, and can use the `await` keyword to pause their execution until an asynchronous operation completes. Coroutines (and tasks) can only run when the event loop is running.
- [Tasks](https://peps.python.org/pep-3156/#tasks): A Task is an object that manages an independently running coroutine. The Task interface is the same as the Future interface, and in fact Task is a subclass of Future. Tasks are also useful for interoperating between coroutines and callback-based frameworks like Twisted. After converting a coroutine into a Task, callbacks can be added to the Task. To convert a coroutine into a task, call the coroutine function and pass the resulting coroutine object to the `loop.create_task()` method. You may also use `asyncio.ensure_future()` for this purpose.
- [Event Loop](https://docs.python.org/3/library/asyncio-eventloop.html): The event loop is the core of every asyncio application. Event loops run asynchronous tasks and callbacks, perform network IO operations, and run subprocesses. To get the event loop for current context, use `get_event_loop()`.
- [Futures](https://peps.python.org/pep-3156/#futures): `asyncio` uses futures to represent the result of an asynchronous operation that has not yet completed. Futures can be awaited like coroutines, and can be used to retrieve the result of an asynchronous operation once it completes.

### Alternatives

Asyncio is just one of several Python libraries for asynchronous programming, each with its own strengths and weaknesses. Here's some of the other popular libraries for asynchronous programming in Python:

- [Twisted](https://twisted.org/): Twisted is a Python-based open-source network framework that enables the creation of SMTP, HTTP, proxy, and SSH servers (among other things) with ease. It operates asynchronously and in an event-driven manner, enabling applications to react to various network connections without relying on conventional threading models.
- [Tornado](https://www.tornadoweb.org/en/stable/): Tornado is an asynchronous networking library and web framework for Python. Unlike many other Python web frameworks, Tornado doesn't rely on WSGI and typically only requires one thread per process to run.
- [Trio](https://stackoverflow.com/questions/49482969/what-is-the-core-difference-between-asyncio-and-trio): Trio is a contemporary Python library that facilitates the creation of asynchronous applications. These are programs that perform multiple tasks simultaneously by parallelizing I/O operations, such as a web spider that retrieves numerous pages concurrently or a web server managing several simultaneous downloads.
- [Curio](https://github.com/dabeaz/curio): Curio is a library for concurrent system programming in Python that operates using coroutines. It offers common programming abstractions like tasks, sockets, files, locks, and queues. It is a small, fast, and enjoyable library that should feel familiar to users.

## Related References

- [PEP 255 – Simple Generators](https://peps.python.org/pep-0255/)
- [PEP 289 – Generator Expressions](https://peps.python.org/pep-0289/)
- [PEP 342 – Coroutines via Enhanced Generators](https://peps.python.org/pep-0342/)
- [PEP 380 – Syntax for Delegating to a Subgenerator](https://peps.python.org/pep-0380/)
- [PEP 3156 – Asynchronous IO Support Rebooted: the “asyncio” Module](https://peps.python.org/pep-3156/)
- [PEP 492 – Coroutines with async and await syntax](https://peps.python.org/pep-0492/)
- [PEP 525 – Asynchronous Generators](https://peps.python.org/pep-0525/)
- [Coroutines and Tasks](https://docs.python.org/3/library/asyncio-task.html)
- [From yield to async/await](https://mleue.com/posts/yield-to-async-await/)
- [Python 101: iterators, generators, coroutines](https://www.integralist.co.uk/posts/python-generators/)
- [Asynchronous Context Managers and Asynchronous Iterators](https://bbc.github.io/cloudfit-public-docs/asyncio/asyncio-part-3.html)