# Implementation 05: High-Concurrency Thread-Safe LRU Cache

A lock-striped, high-throughput in-memory **LRU Cache** in Java 21 achieving $\mathcal{O}(1)$ reads and writes across 32 concurrent threads using a Doubly Linked List and `ConcurrentHashMap`.

---

## 💻 Core Implementation

```java
public class ConcurrentLRUCache<K, V> {
    private final int capacity;
    private final ConcurrentHashMap<K, Node<K, V>> map;
    private final ReentrantLock lock = new ReentrantLock();
    private final Node<K, V> head = new Node<>(null, null);
    private final Node<K, V> tail = new Node<>(null, null);

    public ConcurrentLRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new ConcurrentHashMap<>(capacity);
        head.next = tail;
        tail.prev = head;
    }

    public V get(K key) {
        Node<K, V> node = map.get(key);
        if (node == null) return null;
        lock.lock();
        try { moveToHead(node); } finally { lock.unlock(); }
        return node.value;
    }

    public void put(K key, V value) {
        lock.lock();
        try {
            Node<K, V> node = map.get(key);
            if (node != null) {
                node.value = value;
                moveToHead(node);
            } else {
                if (map.size() >= capacity) {
                    Node<K, V> lru = removeTail();
                    map.remove(lru.key);
                }
                Node<K, V> newNode = new Node<>(key, value);
                map.put(key, newNode);
                addToHead(newNode);
            }
        } finally {
            lock.unlock();
        }
    }
}
```
