# Implementation 07: High-Performance Counting Bloom Filter

A thread-safe, memory-efficient **Counting Bloom Filter** supporting insertions, membership queries, and deletions with mathematically optimized hash functions and $1\%$ false positive bounds.

---

## 💻 Core Implementation

```java
public class CountingBloomFilter {
    private final int bitSize;
    private final int numHashFunctions;
    private final AtomicIntegerArray counters;

    public CountingBloomFilter(int expectedInsertions, double fpp) {
        this.bitSize = (int) (-expectedInsertions * Math.log(fpp) / (Math.log(2) * Math.log(2)));
        this.numHashFunctions = Math.max(1, (int) Math.round((double) bitSize / expectedInsertions * Math.log(2)));
        this.counters = new AtomicIntegerArray(bitSize);
    }

    public void add(String item) {
        for (int hash : getHashes(item)) counters.incrementAndGet(hash);
    }

    public boolean mightContain(String item) {
        for (int hash : getHashes(item)) {
            if (counters.get(hash) <= 0) return false;
        }
        return true;
    }

    public void remove(String item) {
        if (mightContain(item)) {
            for (int hash : getHashes(item)) counters.decrementAndGet(hash);
        }
    }
}
```
