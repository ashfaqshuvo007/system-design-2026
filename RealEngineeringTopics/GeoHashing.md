# Geo-Hashing: System Design Refresher

## 1. What is Geo-Hashing?

Geo-Hash is a method that converts **latitude/longitude coordinates** into a **short alphanumeric string**. Think of it as zooming into a map: each additional character in the string zooms in further, giving you a smaller, more precise cell on Earth's surface.

- **Core idea**: Turn 2D coordinates into a 1D string that can be indexed efficiently.
- **Key property**: Similar geohashes = nearby locations on the map.

---

## 2. How Is It Calculated? (Step-by-Step)

### The Algorithm (Base-32 Encoding)

1. **Start with latitude and longitude ranges**:
   - Latitude: `[-90, 90]`
   - Longitude: `[-180, 180]`

2. **Binary interleaving**:
   - Repeatedly bisect the latitude and longitude intervals
   - Encode which half the coordinate falls into (0 = lower half, 1 = upper half)
   - Interleave the bits: `lon-bit, lat-bit, lon-bit, lat-bit, ...`

3. **Base-32 encoding**:
   - Group bits into 5-bit chunks
   - Map each chunk to a Base-32 character (`0-9`, `b-z` excluding `a`, `i`, `l`, `o`)

**Example**: `u4pruyd` (7 characters) → covers ~1.2km × 0.6km area.

---

## 3. Precision vs. Specificity (Critical Relationship)

| Geohash Length | Approx. Cell Size  | Specificity Level                      |
| -------------- | ------------------ | -------------------------------------- |
| 1 character    | ~5,000km × 5,000km | Continent                              |
| 2 characters   | ~1,250km × 625km   | Country                                |
| 3 characters   | ~156km × 156km     | Large city/region                      |
| 4 characters   | ~39km × 19.6km     | Town/small city                        |
| 5 characters   | ~4.9km × 8.9km     | Neighborhood                           |
| 6 characters   | ~1.2km × 0.6km     | Street block                           |
| 7 characters   | ~1.2km × 0.6km     | Specific location                      |
| 9 characters   | ~4.77m × 4.77m     | Building-level                         |
| 12 characters  | ~0.006m × 0.006m   | Millimeter precision (theoretical max) |

**Precision ↔ Specificity relationship**:

- **More characters = smaller cell = higher precision = higher specificity**
- Precision is the **grid size**; specificity is **how uniquely you identify a location**

---

## 4. Why Geo-Hashing Matters in System Design (Mid-Level Perspective)

### Key Benefits

| Use Case               | How Geo-Hash Helps                                             |
| ---------------------- | -------------------------------------------------------------- |
| **Spatial indexing**   | Converts 2D proximity into 1D string prefix matching           |
| **Proximity searches** | Find nearby POIs by querying same/adjacent geohash prefixes    |
| **Database indexing**  | Works with B-trees, Redis, MongoDB for efficient range queries |
| **Load balancing**     | Shard data by geohash prefix (e.g., taxi dispatch by region)   |
| **Caching strategy**   | Cache geohash cells for frequently queried areas               |

### Real-World Examples

1. **Uber/Lyft**: Partition cities into geohash cells; drivers/riders matched within same cell
2. **Yelp**: "Restaurants near me" → query geohash prefix + 8 neighboring cells
3. **Weather apps**: Aggregate sensor data by geohash region
4. **Social networking**: "Friends nearby" → same geohash cell chat

---

## 5. Tradeoffs & Design Considerations (Senior-Level Nuance)

### Tradeoffs

| Issue                        | Impact                                                                | Mitigation                                                        |
| ---------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Edge cases at boundaries** | Two nearby locations may have different geohashes if on cell boundary | Query 8 neighboring cells + current cell                          |
| **Polar distortion**         | Cell shapes become elongated near poles                               | Use alternative projections for polar regions                     |
| **Precision vs. storage**    | Longer strings = more storage/memory                                  | Choose precision based on use case (e.g., 6 chars for city-level) |
| **Prefix collisions**        | Different locations can share prefixes at low precision               | Use sufficient length (≥6 for urban apps)                         |
| **Non-uniform cell sizes**   | Cells aren't perfectly square (lat/lon bisection differs)             | Acceptable for most apps; account in precision calculations       |

### Design Rules of Thumb

1. **Choose precision based on use case**:
   - City-level: 4–5 characters
   - Neighborhood: 6 characters
   - Street/building: 7–9 characters

2. **For proximity queries**: Always check **current cell + 8 neighbors** (edge case handling)

3. **Indexing**: Store geohash as string in B-tree/Redis sorted set for O(log n) lookups

4. **Maximum length**: 12 characters (beyond that, floating-point precision limits accuracy)

---

## 6. Quick Mental Model

```
Geohash = "u4pruyd"
        ↓
World → 32 cells → pick 1 → subdivide into 32 → pick 1 → ... (7 times)
        ↓
Final cell: ~1.2km × 0.6km area in San Francisco
```

**Rule**: Each character adds ~5 bits of precision (32 = 2⁵), narrowing the cell by ~√32 ≈ 5.6× in each dimension.

---

## Key Takeaway for Your Interviews

Geo-Hashing is your go-to for **location-based services** when you need to:

- Index spatial data efficiently
- Answer "find nearby X" queries
- Avoid expensive haversine/bbox calculations on every request

Remember: **prefix matching = proximity**, but always handle boundary cases by querying neighbors.
