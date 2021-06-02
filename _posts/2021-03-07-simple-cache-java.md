---
layout: post
title: 'Implementing a Simple Cache in Java'
date: 2021-03-08 00:00:00
featured_image: '/assets/img/blog/2021-03-08/floppy.jpg'
excerpt: Every programmer keeps a set of reusable utility programs, patterns, and algorithms that they have accumulated over the years. One program in my tool belt that I've made use of multiple times in the past is a simple in-memory cache - let's see how it works!
---

![](/assets/img/blog/2021-03-08/floppy.jpg)

## The need for fast data access

We all know about databases, they're a great solution for storing large amounts of data with facilities for managing data access from a multitude of consuming systems. However, there is a cost that comes with the security and efficiency that a database offers - processing overhead and added system complexity are a part of that cost. Sometimes we need to store a small amount of data that we need quick access to, especially when it comes to real-time applications such as system locking or session management. An in-memory "database" would do the trick here, otherwise known as a cache.

<img class="center inline-image" src="/assets/img/blog/2021-03-08/cash-money.jpg" />

Cash, cache... clever yeah? Bad jokes and old memes aside, let's see what the implementation of a simple cache in the Java programming language can look like.

First, we need to think about the type of data structure we want to use for storing our cacheable items. The data structure must provide fast access to our data set, and facilitate concurrent access as multiple threads could be accessing the cache simulatenously. For the first requirement, fast data access, the [performance of a hash table implementation](https://en.wikipedia.org/wiki/Hash_table#Performance) would have a complexity of **O(1)** (constant time) in the best case scenario (no collisions of hash codes), or a complexity of **O(n)** (linear time) in the worst case scenario (a collision occurs for every hash code). For the second requirement, concurrent data access, Java provides an implementation of a hash table that is thread-safe and fast, see [ConcurrentHashMap](https://devdocs.io/openjdk~8/java/util/concurrent/concurrenthashmap) for details. For those who aren't familiar with how hash tables work, they're basically a lookup table that maps an index (i.e. key) to a value, like how a phone book maps a name to a phone number. Knowing all of this, let's decide to use ConcurrentHashMap for the backing data structure of our cache.

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

Looks good so far yeah? We have our `get`, `put`, and `remove` methods to retrieve, insert, and remove items from our cache respectively. But wait, what's with that `CacheableObject` class and why are we using `UUID` objects as the key to accessing items in our backing data strucuture? Remember that the performance of a hash table is dependent on the uniqueness of the keys that are used to identify items in the table; the Java `UUID` class provides a way to generate unique ID values fairly easily (see [UUID.randomUUID](https://devdocs.io/openjdk~8/java/util/uuid#randomUUID--)) and even provides a way to customize that ID generation if needed (see [UUID.nameUUIDFromBytes](https://devdocs.io/openjdk~8/java/util/uuid#nameUUIDFromBytes-byte:A-)), thus we can use these unique values as keys for items in our cache. The usage of the `CacheableObject` class will become more apparent later, but at a high level it provides us with an expectation of the kinds of attributes and functionality the objects stored in the cache will have.

Now we need to think about keeping the cache clean. We can certainly keep every object we put in the cache forever (or until the cache is removed from memory) - but that approach would make the memory footprint of the cache larger than needed as well as reduce performance when operating on the objects stored in the cache. Let's add code for a scheduled task that will check each entry in the cache and remove those that haven't been used within a specified time frame - like removing expired products from a shelf at the grocery store.
```java
import java.util.Iterator;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;
import java.util.logging.Level;
import java.util.logging.Logger;

public class Cache {

    private static final Logger logger = Logger.getLogger(Cache.class.getName());
    
    private final ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
    private final ConcurrentMap<UUID, CacheableObject> cacheableObjects = new ConcurrentHashMap<>();

    public Cache() {
        // Check for unused items every few seconds
        executor.scheduleAtFixedRate(new RemoveUnusedEntriesTask(15000L), 10, 10, TimeUnit.SECONDS);
    }

    public CacheableObject get(UUID uuid) {
        CacheableObject cacheableObject = cacheableObjects.get(uuid);

        if (cacheableObject != null) {
            // Update the time the entry was last used
            cacheableObject.access();
        }

        return cacheableObject;
    }

    public void put(CacheableObject cacheableObject) {
        cacheableObjects.put(cacheableObject.getUuid(), cacheableObject);
    }

    public CacheableObject remove(UUID uuid) {
        return cacheableObjects.remove(uuid);
    }

    public void shutdown() {
        this.executor.shutdownNow();
    }

    public abstract class CacheableObject {

        private final UUID uuid;
        // Time last accessed in milliseconds
        private long lastAccessedTime;

        public CacheableObject(UUID uuid) {
            this.uuid = uuid;
        }

        public UUID getUuid() {
            return uuid;
        }

        public void access() {
            this.lastAccessedTime = System.currentTimeMillis();
        }

        public long getLastAccessedTime() {
            return lastAccessedTime;
        }

        // Optional method
        abstract void cleanup();
    }

    private class RemoveUnusedEntriesTask implements Runnable {

        // Timeout threshold in milliseconds
        private final long timeout;

        public RemoveUnusedEntriesTask(long timeout) {
            this.timeout = timeout;
        }

        public void run() {
            long now = System.currentTimeMillis();
            Iterator<Map.Entry<UUID, CacheableObject>> entries = cacheableObjects.entrySet().iterator();

            while (entries.hasNext()) {
                Map.Entry<UUID, CacheableObject> entry = entries.next();
                CacheableObject cacheableObject = entry.getValue();
                long age = now - cacheableObject.getLastAccessedTime();
                // Check if the last time it was accessed is older than our expiration threshold
                if (age >= timeout) {
                    logger.log(Level.FINE, "Item [{0}] has expired. Removing item.", entry.getKey());
                    entries.remove();

                    try {
                        cacheableObject.cleanup();
                    } catch (Exception e) {
                        logger.log(Level.SEVERE, 
                                   "Unable to perform cleanup for item ["+ entry.getKey() +"].", e);
                    }
                }
            }
        }
    }
}
```

A lot of things were added here, let's go over them separately starting with our additions to `CacheableObject`. A `lastAccessedTime` property was added to the class, its value will be the timestamp of when the object instance was last used, represented in milliseconds (similar to [unix time](https://en.wikipedia.org/wiki/Unix_time)). We can use `lastAccessedTime` to know if an object in our cache isn't being used and can be removed. An abstract `cleanup` method was also added that will be executed right after the object is removed from the cache. Since `cleanup` is abstract it allows subclasses of `CacheableObject` to provide their own implementation if desired. For example, `cleanup` can be used to close resources that are no longer needed by the `CacheableObject` instance when it's removed from the cache, like network connections, file handles, or system locks.

Now let's look at the `RemoveUnusedEntriesTask` class, our code that runs on a scheduled basis that removes old objects from the cache. Notice that `RemoveUnusedEntriesTask` implements the `Runnable` interface, this basically means that this code can be run on a different thread multiple times, which is useful since we need to run this code every so often to continue to keep the cache clean. Our cache program runs `RemoveUnusedEntriesTask` on a separate thread using an [Executor](https://devdocs.io/openjdk~8/java/util/concurrent/executor) instance, which manages a pool of threads that can be used to run asynchrous code. The `Executor` instance used in our cache was created with this line of code (notice our executor has access to only a single thread):
```java
private final ScheduledExecutorService executor = Executors.newScheduledThreadPool(1);
```

Our cache is set up to run the code in `RemoveUnusedEntriesTask` every 10 seconds with this line of code in the `Cache` constructor:
```java
executor.scheduleAtFixedRate(new RemoveUnusedEntriesTask(15000L), 10, 10, TimeUnit.SECONDS);
```

This code works by checking every object in the cache against the time when `RemoveUnusedEntriesTask` started running, and if an object hasn't been used within the last 15 seconds (see `new RemoveUnusedEntriesTask(15000L)` in the code above to see we configured it to remove objects older than 15 seconds), then that object is removed from the cache and its `cleanup` method is executed. Cool stuff yeah?

And there you have it, a fast cache that keeps itself clean. You can find the latest version of my cache [here](https://gist.github.com/crmiller64/ac29231c3d542b3391b291972548724c). This is one of my favorite programs in my toolbelt and it's helped me solve some tough problems, hopefully you can find use for it in your programming too!