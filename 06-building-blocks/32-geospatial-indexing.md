# Building Block 32: Geospatial Indexing (Geohash, Quadtree & Google S2)

> **Module**: `06-building-blocks`
> **Reusability Category**: Ingress / Storage / Messaging / Coordination / Reliability / Telemetry / AI Infrastructure

---

## 1. Problem
Traditional database B-Tree indexes operate on 1-dimensional scalar values (numbers/strings). Querying 2-dimensional spatial coordinates (`WHERE lat BETWEEN x AND y AND lon BETWEEN a AND b`) causes slow multi-column bounding box scans across millions of geographic records.

## 2. Requirements
- **Functional Requirements**: Provide robust, highly available, low-latency primitives for client and microservice workloads.
- **Non-Functional Requirements**:
  - **Availability**: $99.999\%$ uptime SLA (Five Nines).
  - **Latency**: $\text{p}99 < 5\text{ms}$ in-memory / $\text{p}99 < 50\text{ms}$ network hop.
  - **Scalability**: Linearly scale to millions of concurrent requests.

## 3. Why It Exists
Geospatial Indexing transforms 2D latitude/longitude coordinates into 1D hierarchical string/integer tokens (Geohash / Google S2 Hilbert Curves) or spatial tree hierarchies (Quadtrees), enabling sub-millisecond proximity radius searches and K-Nearest Neighbors (KNN) queries.

## 4. Mental Model
Dividing a global map into a grid of labeled square boxes, where zooming in subdivides each box into 4 smaller sub-boxes with hierarchical prefixes.

## 5. Core Concepts
Geohash (Base32 spatial string encoding), Quadtree (Hierarchical 4-way spatial tree), Google S2 Geometry (Hilbert Space-Filling Curve on 6-cube faces), K-Nearest Neighbors (KNN), Spatial Bounding Box, Proximity Search.

## 6. Architecture
```mermaid
flowchart TD
    Location["Driver GPS: (37.7749, -122.4194)"] --> S2Engine[Google S2 / Geohash Engine]
    S2Engine --> S2Token["S2 Cell ID / Geohash: '9q8yyk' (Level 13: ~500m x 500m)"]

    subgraph SpatialIndex["Redis In-Memory Spatial Index"]
        Cell1["Cell '9q8yyk' -> [Driver 101, Driver 104]"]
        Cell2["Cell '9q8yym' (Neighbor) -> [Driver 109]"]
        Cell3["Cell '9q8yyj' (Neighbor) -> [Driver 112]"]
    end
    S2Token --> Cell1

    RiderQuery["Rider Search (Radius = 1km)"] --> QueryEngine[Query S2 Cell + 8 Neighbors]
    QueryEngine --> SpatialIndex
    SpatialIndex --> ReturnDrivers[Return Closest Drivers: [101, 104, 109] ✅]
```

## 7. Request/Data Flow
1. Driver app broadcasts `(lat, lon)`. 2. Server converts coordinates to Geohash / S2 Cell ID. 3. Inserts driver ID into Redis Geospatial Index (`GEOADD`). 4. Rider searches within 1km radius. 5. Server calculates target S2 cell + 8 neighboring adjacent cells. 6. Fetches driver IDs from those 9 cells. 7. Computes exact Haversine distance and returns top-K drivers.

## 8. Data Model
Geospatial Record: `entity_id (STRING)`, `latitude (FLOAT64)`, `longitude (FLOAT64)`, `geohash (STRING)`, `s2_cell_id (UINT64)`, `updated_at (INT64)`.

## 9. API Design
`GEOADD drivers_key lon lat driver_id`, `GEORADIUS drivers_key lon lat 1 km WITHDIST WITHCOORD ASC`.

## 10. Algorithms
Interleaving Latitude/Longitude bits (Geohash), Hilbert Space-Filling Curve projection (Google S2), Haversine Great-Circle Distance formula.

## 11. Scaling
Scale out horizontally by partitioning spatial cells across Redis shards based on geographic region/city.

## 12. Partitioning
Partitioned by City / S2 Level 8 Region Cell (e.g. `geo:drivers:san_francisco`).

## 13. Replication
In-memory Redis Cluster with multi-AZ replication; historical trip logs archived in PostgreSQL PostGIS.

## 14. Consistency
Eventual consistency; driver locations update every 3–5 seconds.

## 15. Failure Scenarios
High driver density in small area (concert/downtown causes hot cell), Driver network disconnection.

## 16. Recovery
Dynamic quadtree splitting or S2 cell level down-scaling to maintain bounded driver counts per query.

## 17. Observability
Spatial Query Latency (p99 < 5ms), Active Driver Updates/sec, Geohash calculation rate, Cache hit ratio.

## 18. Security
Sanitizing driver exact coordinates (fuzzing driver location until ride is accepted to protect driver privacy).

## 19. Performance
S2 64-bit integer cell lookups are $10\times$ faster than string-based Geohash string prefix lookups.

## 20. Trade-offs
Google S2 / Geohash (In-memory, massive write throughput, ride hailing) vs PostGIS / R-Tree (Complex spatial polygon intersection, GIS mapping).

## 21. When to Use
Uber/Lyft driver matching, Yelp/Google Maps proximity search, food delivery tracking, geofencing alerts.

## 22. When NOT to Use
Static non-spatial relational data.

## 23. Implementation Strategy
Implement a Java geospatial matching service using Google S2 Geometry library and Redis Geospatial commands (`GEOSEARCH`).

## 24. Practical Exercise
Write a Java test converting 10,000 global GPS coordinates to S2 Cell IDs, execute a 2km radius KNN search, and verify Haversine distance accuracy.

## 25. Interview Questions
1. Explain how Geohash converts 2D coordinates into a 1D string. 2. Why does a proximity radius query require checking the target cell plus its 8 neighboring cells? 3. Why is Google S2 (Hilbert curve) superior to standard Geohash?

## 26. Common Mistakes
Only searching the single central cell without checking the 8 neighboring boundary cells (causes drivers right across the boundary line to be missed!).

## 27. Quick Revision
Geospatial = Lat/Lon -> 1D S2 Cell ID / Geohash -> Redis in-memory grid -> Query target cell + 8 neighbor cells for sub-5ms proximity search.

## 28. Related Building Blocks
`BB-04` (KV Store), `BB-10` (Distributed Cache), `BB-37` (WebSocket Gateway)

## 29. Related Case Studies
`CS-03` (Google Maps), `CS-04` (Yelp Proximity), `CS-05` (Uber Ride Sharing)
