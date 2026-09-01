# Storage Capacity & 5-Year Growth Estimation

Storage estimation calculates disk space required across databases, object storage, and backup systems, accounting for multi-year growth and replication overhead.

---

## 1. The Storage Calculation Formula

$$\text{Daily Ingestion} = \text{Daily Writes} \times \text{Average Record Size}$$

$$\text{5-Year Raw Storage} = \text{Daily Ingestion} \times 365 \times 5$$

$$\text{Total 5-Year Disk Capacity} = \text{Raw Storage} \times (1 + \text{Index Overhead } 0.20) \times \text{Replication Factor } (3\times)$$

---

## 2. Standard Metadata vs Blob Payload Sizing

| Entity Type | Typical Payload Size | Typical Storage Engine |
|---|---|---|
| **User Profile Metadata** | $1\text{ KB}$ | Relational SQL (PostgreSQL) |
| **Short Text Post / Tweet** | $300\text{ bytes}$ | Relational SQL / Distributed NoSQL |
| **E-Commerce Order Record**| $2\text{ KB}$ | Relational SQL (ACID) |
| **Compressed Photo Thumbnail** | $50\text{ KB}$ | Object Store (S3) + CDN |
| **High-Res Photo (JPEG/PNG)** | $2\text{ MB}$ | Object Store (S3) |
| **Video File (1080p, 1 min)** | $50\text{ MB}$ | Object Store (S3) + HLS/DASH Chunks |

---

## 3. Worked Interview Example: Photo Sharing Service (Instagram)

- **Assumptions**:
  - $50\text{M}$ new photos uploaded per day.
  - Average photo size = $2\text{ MB}$.
  - Metadata record per photo (ID, user ID, tags, timestamps) = $500\text{ bytes}$.
- **Storage Calculations**:
  - **Daily Blob Storage**: $50\text{M} \times 2\text{ MB} = \mathbf{100\text{ TB/day}}$.
  - **5-Year Blob Storage**: $100\text{ TB/day} \times 365 \times 5 \approx \mathbf{182.5\text{ PB}}$ (Stored in S3).
  - **Daily Metadata Storage**: $50\text{M} \times 500\text{ bytes} = \mathbf{25\text{ GB/day}}$.
  - **5-Year Metadata Storage**: $25\text{ GB/day} \times 365 \times 5 \approx \mathbf{45.6\text{ TB}}$ (Sharded PostgreSQL / Cassandra).
  - **Accounting for Indexes & 3x Replication on Metadata**: $45.6\text{ TB} \times 1.2 \times 3 \approx \mathbf{164\text{ TB}}$ usable disk.
