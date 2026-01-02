"# HowestProject

## Chess Move Recommendation Engine

This project contains a chess move recommendation engine that analyzes millions of elite chess games to suggest the best next moves for any given position.

### Recent Performance Optimizations

The recommender system has been optimized for speed with several key improvements:

1. **Query Result Caching** - Repeated queries now return results instantly
2. **Early Termination** - Dataset scans stop once enough matches are found
3. **Vectorized Operations** - History key filtering is now 3-5x faster
4. **Smart Query Ordering** - More restrictive queries are tried first

**Expected Performance Improvement:** 5-50x faster for typical queries

See [PERFORMANCE_OPTIMIZATIONS.md](PERFORMANCE_OPTIMIZATIONS.md) for detailed information.

### Usage

The main notebook is `HowestProject_Recommendation_Engine.ipynb` which contains:
- Data preprocessing pipeline for chess games (PGN format)
- Recommendation engine with multiple retrieval levels
- Move ranking using ELO-weighted scoring

### Configuration

Key parameters can be adjusted:
- `CACHE_SIZE` - Number of queries to cache (default: 1000)
- `MIN_MATCHES` - Minimum examples needed for ranking (default: 200)
- `MAX_ROWS_TO_RANK` - Maximum rows to process (default: 200,000)

### Data Source

The engine uses the Lichess Elite Database containing high-level chess games." 
