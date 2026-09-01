# Implementation 08: Enterprise Kafka Consumer with DLQ & Retries

A production-grade **Apache Kafka Producer & Consumer** in Spring Boot with idempotent producers (`enable.idempotence=true`), manual acknowledgment, Dead-Letter Queues (DLQ), and exponential backoff.

---

## 💻 Consumer Configuration

```java
@Component
public class OrderEventConsumer {
    @KafkaListener(topics = "orders.v1", groupId = "order-billing-group")
    public void processOrder(ConsumerRecord<String, OrderEvent> record, Acknowledgment ack) {
        try {
            executeBilling(record.value());
            ack.acknowledge(); // Manual commit on success
        } catch (Exception e) {
            throw new RetriableException("Billing gateway timeout, will retry...", e);
        }
    }
}
```
