XForm was written to provide the best performance C# is capable of. The core model passes data in batches of rows in a zero-copy, zero-allocation, zero-boxing way. Performance critical pieces are written in hard-coded C# loops for each primitive data type. A native acceleration library, XForm.Native, implements accelerated versions of key pieces in Managed C++ using the AVX-256 SIMD instructions.

XForm stores all columns which have fewer than 256 distinct values as one byte per row.

On a mainstream desktop (i5-7500 with Samsung 960 Evo), XForm can:
  - Import tabular data at ~100MB/s.
  - Read and Write binary tables at ~1.5GB/s.
  - Match one byte columns (bool, byte, and all with under 256 distinct values) at 3.3B rows/sec.

These numbers are running single threaded and with no pre-sorting or pre-indexing - just brute force matching on the raw data.