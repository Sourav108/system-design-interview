# Implementation 10: Payment Idempotency & Double-Entry Ledger

A mission-critical **Payment Processing Engine** with Idempotency Key deduplication via Redis mutex locks and an ACID **Double-Entry Bookkeeping Ledger** in PostgreSQL.

---

## 💻 Idempotency Filter & Double-Entry Transaction

```java
@Service
public class PaymentProcessingService {
    @Transactional
    public PaymentResponse processCharge(String idempotencyKey, ChargeRequest req) {
        // 1. Check Redis Idempotency Lock
        String lockKey = "idem:pay:" + idempotencyKey;
        if (!redis.setnx(lockKey, "PROCESSING", Duration.ofMinutes(2))) {
            return redis.get(lockKey + ":resp", PaymentResponse.class);
        }

        // 2. Execute Bank Charge
        BankResult bankRes = bankGateway.charge(req.getAmount());

        // 3. ACID Double-Entry Bookkeeping in PostgreSQL
        ledgerRepository.insertDebit(req.getCustomerAccount(), req.getAmount());
        ledgerRepository.insertCredit(req.getMerchantAccount(), req.getAmount());

        PaymentResponse resp = new PaymentResponse(bankRes.getTxId(), "SUCCESS");
        redis.setex(lockKey + ":resp", Duration.ofHours(24), resp);
        return resp;
    }
}
```
