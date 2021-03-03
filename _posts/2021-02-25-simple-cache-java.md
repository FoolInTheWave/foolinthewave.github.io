---
title: 'Implementing a Simple Cache in Java'
date: 2021-02-25 00:00:00
featured_image: '/images/blog/2021-02-25/floppy.jpg'
excerpt: Every programmer keeps a set of reusable utility programs, patterns, and algorithms that they have accumulated over the years. One program in my tool belt that I've made use of multiple times in the past is a simple in-memory cache - let's see how it works!
---

![](/images/blog/2021-02-25/floppy.jpg)

## The need for fast data access

We all know about databases, they're a great solution for storing large amounts of data with facilities for managing data access from a multitude of consuming systems. However, there is a cost that comes with the security and efficiency that a database offers - processing overhead and added system complexity are a part of that cost. Sometimes we need to store a small amount of data that we need quick access to, especially when it comes to real-time applications such as system locking or session management. An in-memory "database" would do the trick here, otherwise known as a cache.

![](/images/blog/2021-02-25/cash-money.jpg)

Cash, cache... clever yeah? Bad jokes and old memes aside, let's see what the implementation of a simple cache in the Java programming language can look like.

First, we need to think about the type of data structure we want to use for storing our cacheable items. The data structure must provide fast access to our data set, and facilitate concurrent access as multiple threads could be accessing the cache simulatenously. For the first requirement, fast data access, the [performance of a hash table implementation](https://en.wikipedia.org/wiki/Hash_table#Performance) would have a complexity of **O(1)** (constant time) in the best case scenario (no collisions of hash codes), or a complexity of **O(n)** (linear time) in the worst case scenario (a collision occurs for every hash code). For the second requirement, concurrent data access, Java provides an implementation of a hash table that is thread-safe and fast, see [ConcurrentHashMap](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html) for details. For those who aren't familiar with how hash tables work, they're basically a lookup table that maps an index (i.e. key) to a value, like how a phone book maps a name to a phone number. Knowing all of this, let's decide to use ConcurrentHashMap for the backing data structure of our cache.

Now we can start coding. Let's start with creating the cache class definition with accessor methods to retrieve, insert, and remove items from the cache.
```java
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.logging.Level;
import java.util.logging.Logger;

public class Cache {

    private static final Logger logger = Logger.getLogger(Cache.class.getName());

    private final ConcurrentMap<UUID, CacheableObject> cacheableObjects = new ConcurrentHashMap<>();

    public CacheableObject get(UUID uuid) {
        return cacheableObjects.get(uuid);
    }

    public void put(CacheableObject cacheableObject) {
        cacheableObjects.put(cacheableObject.getUuid(), cacheableObject);
    }

    public CacheableObject remove(UUID uuid) {
        return cacheableObjects.remove(uuid);
    }

    public abstract class CacheableObject {

        private final UUID uuid;

        public CacheableObject(UUID uuid) {
            this.uuid = uuid;
        }

        public UUID getUuid() {
            return uuid;
        }
    }
}
```

Looks good so far yeah? We have our `get`, `put`, and `remove` methods to retrieve, insert, and remove items from our cache respectively. But wait, what's with that `CacheableObject` class and why are we using `UUID` objects as the key to accessing items in our backing data strucuture?