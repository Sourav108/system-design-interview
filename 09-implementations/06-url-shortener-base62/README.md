# Implementation 06: TinyURL Microservice (Base62 + Redis + Postgres)

A complete, production-grade URL shortener microservice built with **Spring Boot 3, Redis Cache-Aside, PostgreSQL, and Base62 Encoding**.

---

## 💻 Base62 Encoder & Decoder

```java
public class Base62 {
    private static final String ALPHABET = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";

    public static String encode(long num) {
        StringBuilder sb = new StringBuilder();
        while (num > 0) {
            sb.append(ALPHABET.charAt((int) (num % 62)));
            num /= 62;
        }
        while (sb.length() < 7) sb.append('0');
        return sb.reverse().toString();
    }

    public static long decode(String str) {
        long num = 0;
        for (char c : str.toCharArray()) {
            num = num * 62 + ALPHABET.indexOf(c);
        }
        return num;
    }
}
```
