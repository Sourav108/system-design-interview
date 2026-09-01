# Implementation 04: Consistent Hashing Ring with Virtual Nodes

A thread-safe **Consistent Hash Ring** in Java using MurmurHash3 and `TreeMap` (Red-Black Tree) with configurable Virtual Nodes (100–300 per physical node) to ensure uniform load distribution.

---

## 💻 Core Implementation

```java
public class ConsistentHashRing<T> {
    private final int virtualNodes;
    private final NavigableMap<Integer, T> ring = new ConcurrentSkipListMap<>();

    public ConsistentHashRing(int virtualNodes, Collection<T> nodes) {
        this.virtualNodes = virtualNodes;
        for (T node : nodes) addNode(node);
    }

    public void addNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            int hash = hash(node.toString() + "#VN#" + i);
            ring.put(hash, node);
        }
    }

    public T getNode(String key) {
        if (ring.isEmpty()) return null;
        int hash = hash(key);
        Map.Entry<Integer, T> entry = ring.ceilingEntry(hash);
        return (entry != null) ? entry.getValue() : ring.firstEntry().getValue();
    }

    private int hash(String key) {
        return Hashing.murmur3_32_fixed().hashString(key, StandardCharsets.UTF_8).asInt();
    }
}
```
