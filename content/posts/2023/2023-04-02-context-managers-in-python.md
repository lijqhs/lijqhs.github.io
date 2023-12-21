---
title: What are Context Managers in Python
slug: context-managers-in-python
date: 2023-04-02
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/arno-smit-qY_yTu7YBT4-unsplash.jpg
thumbnailImagePosition: left
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/arno-smit-qY_yTu7YBT4-unsplash.jpg
categories:
- python
tags:
- python
- context manager
- generator
- decorator
---


In Python, context managers are a powerful tool for managing resources such as files, sockets, and locks. While Python provides built-in context managers such as `with open() as f:` for working with files, you can also define your own context managers using the `contextlib` module. In this post, I'll explore how to define your own context managers in Python.

<!--more-->

{{< toc >}}


## What are Context Managers?

A context manager is simply a Python object that defines two methods: `__enter__` and `__exit__`. The `__enter__` method is called when a block of code is entered, and the `__exit__` method is called when the block of code is exited. The purpose of a context manager is to provide a clean and safe way to manage resources that need to be acquired and released.

For example, let's say you want to open a file and read its contents. Normally, you would open the file using the `open()` function, read its contents, and then close the file using the `close()` method. However, if an exception occurs while you're reading the file, the `close()` method may not be called, leaving the file open and potentially causing problems. Using a context manager, you can ensure that the file is always closed, even if an exception occurs.

## How to Use Context Managers

### Using the `with` Statement

The most common way to use a context manager is by using the `with` statement. The `with` statement enables us to use context managers. The general format is:

```python
with context_manager as target:
    with_block
```

This will call `__enter__()` on context_manager, binding the result to target. The `with_block` is then executed. Finally, `__exit__()` is called on context_manager.

### Example: File I/O

A common example is opening a file. We can do:

```python
with open('file.txt', 'r') as f:
    content = f.read()
# f is automatically closed here    
```

In this example, the `open()` function returns a file object that acts as a context manager. 
- When the `with` statement is executed, the `__enter__` method is called, which opens the file. 
- Then, the code inside the `with` block is executed, which reads the file contents into the `content` variable. Finally, the `__exit__` method is called, which closes the file. 
- If an exception occurs inside the `with` block, the `__exit__` method is still called, ensuring that the file is closed. 

This is much cleaner than:

```python
f = open('file.txt')
try: 
    read_data = f.read() 
finally:
    f.close()
```

### Other Use Cases

Except managing file resources, other examples of context managers include: 
- locking resources, 
- managing socket resources, 
- timing code blocks, 
- temporarily modifying environment variables, 
- connecting database, 
- managing thread resources, 
- managing resources that require initialization and cleanup, such as hardware devices,third-party libraries, and more.

Context managers provide a simple way to setup and teardown resources precisely and robustly, which can be used with a wide range of resources to improve the reliability and maintainability of your code.

## How to Implement Context Managers in Python

### Defining context manager as a class

A context manager is any object that implements the context manager protocol by defining `__enter__()` and `__exit__()` methods. The `__enter__()` method is called when entering the context, and should return an object bound to the target of the as clause in the with statement. The `__exit__()` method is called when leaving the context, and handles any exception that occurred in the block.

The most common way to define a context manager in Python is by defining a class with `__enter__` and `__exit__` methods. Here's an example:

```python
class MyContextManager:
    def __init__(self):
        # Constructor code here
        pass

    def __enter__(self):
        # Enter code here
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        # Exit code here
        pass

# Usage
with MyContextManager() as cm:
    # Code to use context manager here
```

In this example, we define a class called `MyContextManager` with `__enter__` and `__exit__` methods. 
- The `__init__` method is optional and can be used to initialize the context manager. 
- The `__enter__` method is called when the `with` statement is executed, and should return the context manager object. 
- The `__exit__` method is called when the `with` block is exited, and should clean up any resources that were acquired by the context manager.

#### Example: Timer

For example, here's a context manager that times a block of code:

```python
class Timer:
    def __enter__(self):
        self.start = time.time()

    def __exit__(self, exc_type, exc_val, exc_tb):
        print(time.time() - self.start)
```

You can use this context manager like this:

```python 
with Timer() as timer:
    do_something()  # Runs for 1 second
# Prints 1.0
```

#### Example: Suppress Exceptions

Here is another simple example to define context manager as a class. The `__exit__()` method handles exceptions raised in the block. We can choose to suppress exceptions by returning `True`, or re-raise them by returning `False` or any other non-True value. For example:

```python
class SuppressException:
    def __exit__(self, exc_type, exc_val, exc_tb):
        print('Suppressing exception!')
        return True
        
with SuppressException():
    raise ValueError('Error!')  # Exception raised but suppressed! 
```

#### Example: Managing database connection using context manager

Here's an example of implementing a context manager to connect to a database using a class:

```python
import mysql.connector

class DBConnection:
    def __init__(self, user, password, host, database):
        self.user = user
        self.password = password
        self.host = host
        self.database = database

    def __enter__(self):
        self.conn = mysql.connector.connect(user=self.user, password=self.password, 
                                           host=self.host, database=self.database)
        self.cursor = self.conn.cursor()
        return self.cursor

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.commit()
        self.cursor.close()
        self.conn.close()

# Usage: 
with DBConnection(user='john', password='pwd', host='localhost', database='mydb') as cursor:
    cursor.execute('SELECT * FROM employees')
    for row in cursor:
        print(row) 
```

This context manager, `DBConnection`, handles connecting to a MySQL database, creating a cursor, and then committing changes, closing the cursor and connection upon exiting the with block.

We can use it as a context manager by instantiating the class and using it in a `with` statement. The `__enter__` method returns the database cursor, allowing us to use it to query the database. Then `__exit__` handles committing changes and properly closing connections even if an exception occurs.

Without a context manager, we'd have to do this logic explicitly:

```python
conn = mysql.connector.connect(user=user, password=password, host=host, database=database) 
cursor = conn.cursor()

try: 
    cursor.execute('SELECT * FROM employees')
    # ... 
except Exception:
    conn.rollback()  
finally:
    cursor.close()
    conn.close()
```

### Defining a Context Manager Function

In addition to defining a context manager class, we can decorate a generator function to turn it into a context manager with the `@contextmanager` decorator from [`contextlib`](https://docs.python.org/3/library/contextlib.html), which is a module provides utilities for common tasks involving the `with` statement.

```python
from contextlib import contextmanager

@contextmanager
def my_context_manager():
    # Enter code here
    try:
        yield
    finally:
        # Exit code here
        pass

# Usage
with my_context_manager():
    # Code to use context manager here
```

In this example, we define a context manager function called `my_context_manager` using the `@contextmanager` decorator. 
- The `@contextmanager` decorator "magically" turns the generator function into a context manager by handling the entry and exit in accordance with the context manager protocol. 
- Inside the function, we define the code to execute when the `with` block is entered, and use the `yield` statement to indicate where the code inside the `with` block should be executed. 
- Finally, we define the code to execute when the `with` block is exited inside a `finally` block.

#### Example: Managing threads using context manager

Below is an example of defining a context manager to manage threads using a generator function and `@contextmanager`:

```python
from contextlib import contextmanager
import threading

@contextmanager
def thread_manager():
    thread = threading.current_thread()
    try:
        yield thread
    finally:
        del thread
```

We define a context manager using the `@contextmanager` decorator. The `thread_manager` function is a generator that yields the current thread object using the `threading.current_thread()` method. When the `with` block is exited, the `finally` block is executed to delete the thread object.

To use this context manager, we can wrap our thread operations in a `with` block:

```python
def print_thread_name():
    with thread_manager() as thread:
        print(f"Thread ID: {thread.ident}, Name: {thread.name}")

thread1 = threading.Thread(target=print_thread_name, name="Thread 1")
thread2 = threading.Thread(target=print_thread_name, name="Thread 2")

thread1.start()
thread2.start()

thread1.join()
thread2.join()
```

- We define a function `print_thread_name` that prints the ID and name of the current thread. 
- Within the function, we use the `with` block to create a context using the `thread_manager` context manager. We then start two threads `thread1` and `thread2` that call the `print_thread_name` function. 
- When the threads are joined, the context is exited and the thread objects are deleted.

When we run this code, we should see output similar to the following:

```
Thread ID: 123145380864000, Name: Thread 1
Thread ID: 123145380864000, Name: Thread 2
```

In this output, we can see that both threads have the same ID and are using the names "Thread 1" and "Thread 2" respectively. This demonstrates that the context manager is successfully managing the threads.


#### Example: Managing threads using context manager 2

```python
from contextlib import contextmanager
import threading

@contextmanager
def thread_manager():
    thread = threading.Thread(target=do_something)
    thread.start()

    try:
        yield
    finally:
        thread.join()

def do_something():
    print('Doing something')

# Usage:
with thread_manager():
    print('Doing other things')
```

This context manager starts a new thread that runs the `do_something()` function. Then, it yields control to the body of the `with` statement. Finally, after exiting the `with` block, it joins the thread to wait for it to complete.

So the full logic looks like this:
1. Start a new thread to run `do_something()`
2. Yield control to the with block
3. Run logic in the `with` block
4. Upon exiting `with` block, join the thread
5. Thread completes

Without the context manager, we'd have to remember to properly join the thread to avoid issues:

```python
thread = threading.Thread(target=do_something)
thread.start()

# Do other things...
# Make sure to join the thread! 
thread.join() 
```

If we forgot to join the thread, it would continue running in the background even after our main program exits. So the context manager encapsulates this logic to ensure the thread is always properly cleaned up. Even if an exception occurs in the `with` block, the thread will still be joined.

## Conclusion

Context managers are a great way to ensure setup/teardown logic is executed correctly, especially in more complex cases. For something as potentially tricky as thread management, a context manager helps make the usage very clean while handling edge cases behind the scenes.