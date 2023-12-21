---
title: An Introduction to Caching in Python
slug: caching-in-python
date: 2023-04-23
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/1/david-marcu-VfUN94cUy4o-unsplash.jpg
thumbnailImagePosition: left
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.5.2/img/gallery/u/1/david-marcu-VfUN94cUy4o-unsplash.jpg
categories:
- python
tags:
- python
- caching
- lru_cache
- decorator
- memoization
- redis
---

Caching is a technique used to improve the performance of software applications by storing frequently accessed data in memory, rather than repeatedly retrieving it from slower sources like disk or network. <!--more-->In Python, caching is a common strategy used to optimize the performance of applications that frequently access data from slow or remote sources.



{{< toc >}}

## Introduction

### How does Caching work in Python?

Caching works by storing frequently accessed data in memory so that it can be quickly retrieved when needed. In Python, there are several ways to implement caching, including using dictionaries, lists, or specialized caching libraries.

One common approach to caching in Python is to use a dictionary to store the results of function calls. When a function is called with the same arguments, the result is returned from the cache rather than recalculating the result. For example:

```python
import time

def get_sum_slowly(a, b):
    time.sleep(2)           # to mimic expensive calculations
    return a + b

print(get_sum_slowly(1, 2))  # cost 2 seconds to print 3
```

Whenever we call function `get_sum_slowly`, it will cost about 2 seconds to get the number `3`. But if we change a little bit, things will be different:


```python
cache = {}

def get_sum_cached(a, b):
    if (a, b) in cache:
        return cache[(a, b)]
    
    result = get_sum_slowly(a, b)    # perform expensive calculation here
    cache[(a, b)] = result
    return result

print(get_sum_cached(1, 2))  # firstly cost 2 seconds to print 3
print(get_sum_cached(1, 2))  # print 3 immediately
```

In this example, the `get_sum_cached` checks the cache to see if the result for the given arguments has already been calculated. If it has, the cached result is returned. If not, the expensive calculation is performed and the result is stored in the cache for future use.

Another approach to caching in Python is to use a memoization decorator. A memoization decorator is a function that takes another function as an argument and returns a new function that caches the results of the original function. For example:

```python
import time
import functools

@functools.cache
def get_sum_cached2(a, b):
    time.sleep(2)   # to mimic expensive calculations
    return a + b

print(get_sum_cached2(1, 2))  # firstly cost 2 seconds to print 3
print(get_sum_cached2(1, 2))  # print 3 immediately
```

`functools.cache` is a decorator that was added in Python 3.9 as part of the functools module. It provides a simple way to cache the results of a function in memory, without having to write custom caching code. When you apply the `@functools.cache` decorator to a function, Python automatically caches the result of the function for a given set of arguments. The next time the function is called with the same arguments, Python returns the cached result instead of recomputing the result.


## Cache strategies

The above example demonstrates the efficiency of caching, but it also exposes a potential flaw of simple dictionary caching: infinite growth of dictionary content. Because we did not put any limit on the content of the dictionary, as the number of calls increases, the content of the dictionary will expand infinitely, which will eventually lead to a crash of the program. So we need caching strategies to prevent such a thing from happening. 

Since memory is always finite, we must decide that some of the content needs to be released from memory to make room for new content. 

There are a few main caching strategies:

1. **LRU (Least Recently Used)**: The least *recently* used items are evicted from the cache when the cache reaches its maximum size. This strategy ensures that the most recently used items are kept in the cache.
2. **LFU (Least Frequently Used)**: The least *frequently* used items are evicted from the cache when the cache reaches its maximum size. This strategy ensures that the most frequently used items are kept in the cache.
3. **Random**: In a random eviction strategy, items are evicted from the cache at random when the cache reaches its maximum size. This strategy is simple to implement, but may not be optimal for all use cases.
4. **FIFO (First In, First Out)**: The oldest items are evicted from the cache when the cache reaches its maximum size. This strategy ensures that items are evicted in the order in which they were added to the cache.
5. **Size-based**: Items are evicted from the cache based on their size when the cache reaches its maximum size. This strategy ensures that the cache does not exceed its maximum size, but may not take into account the popularity of items.
6. **Time-based**: Discards items based on an expiration time. Useful for caching ephemeral data.

### LRU Cache

The LRU cache eviction strategy is popular one, which is based on the assumption that the items that have been accessed most recently are the items that are most likely to be accessed again in the near future.

The algorithm behind LRU cache is based on maintaining a doubly linked list of items in the cache, with the most recently used item at the front of the list and the least recently used item at the back of the list. Each item in the list has a key-value pair, where the key is the item's identifier and the value is the item's data.

When an item is accessed, it is moved to the front of the list. When a new item is added to the cache, it is added to the front of the list. When the cache reaches its maximum size, the least recently used item at the back of the list is evicted from the cache.

Here's a more detailed description of the algorithm:

1. Initialize an empty cache and a doubly linked list.
2. When a new item is accessed or added to the cache, check if it is already in the cache.
3. If the item is in the cache, remove it from its current position in the linked list and move it to the front of the list.
4. If the item is not in the cache, add it to the front of the linked list and add it to the cache.
5. If the cache size exceeds its maximum capacity, remove the least recently used item from the back of the linked list and from the cache.

The LRU cache algorithm ensures that the most frequently accessed items are always at the front of the linked list and are therefore the least likely to be evicted from the cache. The least recently used items are always at the back of the linked list and are therefore the most likely to be evicted from the cache when the cache reaches its maximum capacity.

In Python, the `functools` module provides a built-in LRU cache implementation through the `functools.lru_cache` decorator. In the previous example, `functools.cache` is actually the same as `functools.lru_cache(maxsize=None)`.

```python
# cache -- simplified access to the infinity cache
def cache(user_function, /):
    'Simple lightweight unbounded cache.  Sometimes called "memoize".'
    return lru_cache(maxsize=None)(user_function)
```

### A simple implementation of LRU Cache

Here's an example of how to implement an LRU cache in Python using a dictionary and a doubly linked list:

```python
class Node:
    def __init__(self, key=None, value=None):
        self.key = key
        self.value = value
        self.prev = None
        self.next = None

class LRUCache:
    def __init__(self, capacity):
        self.capacity = capacity
        self.cache = {}
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key):
        if key in self.cache:
            node = self.cache[key]
            self._move_to_front(node)
            return node.value
        else:
            return None

    def put(self, key, value):
        if key in self.cache:
            node = self.cache[key]
            node.value = value
            self._move_to_front(node)
        else:
            if len(self.cache) >= self.capacity:
                self._evict()
            node = Node(key, value)
            self.cache[key] = node
            self._add_to_front(node)

    def _move_to_front(self, node):
        self._remove_node(node)
        self._add_to_front(node)

    def _add_to_front(self, node):
        node.prev = self.head
        node.next = self.head.next
        self.head.next.prev = node
        self.head.next = node

    def _remove_node(self, node):
        node.prev.next = node.next
        node.next.prev = node.prev

    def _evict(self):
        node = self.tail.prev
        self._remove_node(node)
        del self.cache[node.key]
```

- The `Node` class represents a node in the linked list. Each node has a `key` and a `value`, and pointers to the previous and next nodes in the list.

- The `get` method retrieves a value from the cache given a key. If the key is in the cache, the corresponding node is moved to the front of the list (since it was just accessed), and the value is returned. If the key is not in the cache, `None` is returned.

- The `put` method adds a new key-value pair to the cache. If the key is already in the cache, the corresponding node is updated with the new value and moved to the front of the list (since it was just accessed). If the key is not in the cache, a new node is created and added to the front of the list. If the cache is already at its maximum capacity, the least recently used item (at the end of the list) is evicted from the cache.

- The `_move_to_front`, `_add_to_front`, `_remove_node`, and `_evict` methods are internal helper methods that manipulate the linked list as needed.

We can test `LRUCache` with a simple example:

```python
cache = LRUCache(2)  # Create an LRU cache with capacity 2

cache.put(1, 'hello')
cache.put(2, 'world')

print(cache.get(1))  # Output: 'hello'
print(cache.get(2))  # Output: 'world'

cache.put(3, 'foo')  # Evicts key 1 since it was the least recently used

print(cache.get(1))  # Output: None
print(cache.get(2))  # Output: 'world'
print(cache.get(3))  # Output: 'foo'

cache.put(4, 'bar')  # Evicts key 2 since it was the least recently used

print(cache.get(1))  # Output: None
print(cache.get(2))  # Output: None
print(cache.get(3))  # Output: 'foo'
print(cache.get(4))  # Output: 'bar'
```

### Use `LRUCache` for memoization

We can use `LRUCache` to boost the implementation of a Fibonacci function:

```python
def fib(n):
    if n < 2: return n
    return fib(n-2) + fib(n-1)

print(fib(36))      
# 14930352, time cost: 5.440476059000503
```

Cached version:

```python
def fibonacci(n, cache):
    if n < 2: return n
    elif v:=cache.get(n): return v
    else:
        v = fibonacci(n-2, cache) + fibonacci(n-1, cache)
        cache.put(n, v)
        return v
    
cache = LRUCache(2)
print(fibonacci(36, cache)) 
# 14930352, time cost: 0.00013985300029162318
```

In this example, we create an `LRUCache` object with a maximum capacity of only `2`, and the time cost can prove the cached version of the Fibonacci function is much more efficient than the original recursive one. 

## Benefits of Caching in Python

Caching in Python can provide several benefits, including:

1. Improved performance: By storing frequently accessed data in memory, caching can reduce the amount of time spent accessing slow or remote data sources.

2. Reduced resource consumption: Caching can reduce the amount of resources required to execute a Python application, such as CPU time or network bandwidth.

3. Consistent results: Caching can help ensure that a Python application consistently returns the same results for the same inputs, which can be important for applications that require accuracy and precision.


## Distributed caching

Distributed caching is a technique used to improve the performance and scalability of applications by distributing cached data across multiple servers or nodes. In Python, there are several common use cases for distributed caching, including:

1. Web applications: Distributed caching can be used to store frequently accessed data, such as session data, user preferences, or frequently used database queries, in memory across multiple servers. This can significantly improve the performance and scalability of web applications, especially during periods of high traffic.

2. Machine learning: Distributed caching can be used to store large datasets or trained machine learning models in memory across multiple nodes. This can improve the performance of machine learning algorithms by reducing the amount of time spent accessing data from slow or remote data sources.

3. High-performance computing: Distributed caching can be used to store intermediate results or frequently accessed data in memory across multiple nodes in a high-performance computing cluster. This can reduce the amount of time spent transferring data between nodes and improve the overall performance of the system.

4. Real-time analytics: Distributed caching can be used to store frequently accessed data, such as event streams or user behavior data, in memory across multiple nodes. This can improve the performance of real-time analytics systems by reducing the amount of time spent accessing and processing data.

5. Content delivery networks (CDNs): Distributed caching can be used to store frequently accessed content, such as images, videos, or static assets, across multiple servers in a content delivery network. This can improve the performance and availability of content for users around the world.

Some popular distributed caching solutions for Python include **Redis**, **Memcached**, and **Hazelcast**. These solutions provide features such as replication, sharding, and automatic failover to ensure that cached data is always available and consistent across multiple nodes.

## What's the benefits of Redis 

**Redis** is an in-memory key-value data store that can be used as a distributed caching solution. While Python's built-in caching tools, such as `dict` or `lru_cache`, provide a simple and effective way to implement caching in Python, Redis offers several benefits that make it a popular choice for distributed caching. Here are some of the benefits of Redis compared to built-in cache tools in Python:

1. Scalability: Redis is designed to be highly scalable and can be used to distribute cached data across multiple nodes. This allows Redis to handle large amounts of data and high levels of traffic, making it suitable for large-scale applications.

2. Persistence: Redis can be configured to persist cached data to disk, which means that cached data will not be lost if the server is restarted or crashes. This is important for applications that require high availability or have strict data durability requirements.

3. Advanced data structures: Redis provides advanced data structures, such as sets, sorted sets, and lists, that can be used to implement more complex caching strategies. For example, sorted sets can be used to implement time-based caching, where cached data is automatically expired after a certain period of time.

4. Distributed locks: Redis provides distributed locking mechanisms that can be used to ensure that only one process or thread can access a particular resource at a time. This is important for applications that require concurrency control or have shared resources that need to be protected.

5. Integration with other languages and platforms: Redis has libraries and clients available for many programming languages and platforms, making it easy to integrate with existing systems and applications.

6. Performance: Redis is designed to be fast and efficient, with low latency and high throughput. This makes it suitable for high-performance applications that require low response times and high data throughput.

In summary, Redis provides several benefits compared to built-in cache tools in Python, including scalability, persistence, advanced data structures, distributed locks, integration with other languages and platforms, and high performance. These benefits make Redis a popular choice for distributed caching and a powerful tool for improving the performance and scalability of Python applications.

## Conclusion

Caching is a powerful technique for improving the performance of Python applications. By storing frequently accessed data in memory, caching can reduce the amount of time spent accessing slow or remote data sources, which can lead to faster and more efficient Python applications. Whether you use a simple dictionary-based cache or a more sophisticated memoization decorator, caching is a technique worth considering for any Python application that requires frequent access to the same data.