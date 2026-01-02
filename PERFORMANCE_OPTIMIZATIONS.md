# Performance Optimizations for Chess Move Recommender

## Problem
The chess move recommendation engine was very slow due to multiple full dataset scans and inefficient data processing.

## Optimizations Implemented

### 1. Query Result Caching (LRU Cache)
**What:** Added a cache for query results using Python's LRU (Least Recently Used) pattern.

**Why:** Identical or similar queries no longer require rescanning the entire Parquet dataset.

**Implementation:**
- `get_cached_result()` checks for cached results before scanning
- `store_cached_result()` stores successful query results
- Configurable `CACHE_SIZE` (default: 1000 queries)
- Automatic cache eviction when size limit is reached

**Expected Impact:** 
- **Near-instant response** for repeated queries
- **50-90% faster** for similar position queries during a game session

### 2. Early Termination in Dataset Scans
**What:** Modified `scan_matches()` to stop scanning once enough rows are found.

**Why:** Previously, the function would scan all Parquet files even when we only needed 200 matches out of millions.

**Implementation:**
```python
def scan_matches(dataset, expr, columns, max_rows=None):
    scanner = dataset.scanner(columns=columns, filter=expr)
    batches = []
    rows_collected = 0
    for batch in scanner.to_batches():
        batches.append(batch)
        rows_collected += len(batch)
        if rows_collected >= max_rows:
            break  # Early termination!
```

**Expected Impact:**
- **10-100x faster** for queries that find matches early in the dataset
- Especially beneficial for common positions

### 3. Vectorized History Key Filtering
**What:** Replaced slow `.apply(lambda ...)` operations with vectorized pandas string operations.

**Why:** The previous implementation used:
```python
df = df[df["history_last6_uci"].apply(lambda h: history_key(h, 4) == key)]
```
This calls a Python function for every row, which is slow.

**New Implementation:**
```python
df = df[df["history_last6_uci"].str.split("|").str[-4:].str.join("|") == key]
```
This uses pandas' vectorized string operations implemented in C.

**Expected Impact:**
- **3-5x faster** history filtering
- Particularly beneficial for `fen_last4` and `fen_last2` query levels

### 4. Smarter Query Level Ordering
**What:** Reordered query levels to try most restrictive (and typically fastest) levels first.

**Why:** More specific queries (like `fen_hist_tc`) are faster because they match fewer rows.

**Expected Impact:**
- Faster average query time by finding good matches earlier

## Performance Gains Summary

| Optimization | Expected Speedup | Scenario |
|--------------|------------------|----------|
| Query Caching | 100-1000x | Repeated identical queries |
| Early Termination | 10-100x | Common positions with many examples |
| Vectorized Operations | 3-5x | History key filtering |
| Combined Effect | 5-50x | Typical usage during gameplay |

## Configuration

Adjust these constants in the notebook to tune performance:

```python
CACHE_SIZE = 1000         # Number of queries to cache
MIN_MATCHES = 200         # Minimum examples needed
MAX_ROWS_TO_RANK = 200_000  # Maximum rows to process
PLY_WINDOW = 20           # Ply filter window
```

## Testing Recommendations

1. **Test cache effectiveness:**
   - Run the same query multiple times
   - First run should be slower, subsequent runs should be near-instant

2. **Test early termination:**
   - Query for a very common opening position
   - Should stop scanning after finding enough matches quickly

3. **Test vectorized operations:**
   - Monitor the performance difference for `fen_last4` and `fen_last2` levels
   - Should be noticeably faster than before

## Future Optimization Opportunities

1. **Pre-compute history keys:** Store `history_last4` and `history_last2` as columns in Parquet
2. **Indexing:** Add secondary indices on frequently queried columns
3. **Parallel scanning:** Use multiple threads to scan different Parquet files
4. **Sampling before filtering:** For very large result sets, sample first then filter
5. **Persistent cache:** Store cache to disk between sessions

## Monitoring

To measure the impact, add timing code:

```python
import time
start = time.time()
result = recommend_next_moves(query)
elapsed = time.time() - start
print(f"Query took {elapsed:.2f} seconds")
```

Compare before and after optimization for typical queries.
