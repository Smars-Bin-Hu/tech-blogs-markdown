# Why is APPROX\_COUNT\_DISTINCT() Faster in SparkSQL?

> When working with large datasets in **SparkSQL**, `APPROX_COUNT_DISTINCT()` is a much faster alternative to `COUNT(DISTINCT col)`. This is because it uses **approximate counting** instead of exact counting. Let’s break it down step by step.

[SparkSQL approx\_count\_distinct aggregate function](https://docs.databricks.com/en/sql/language-manual/functions/approx_count_distinct.html)

# **The Problem with** `COUNT(DISTINCT col)`

By default, `COUNT(DISTINCT col)` **scans all data and removes duplicates**, which causes:

*   **High memory usage** – it stores all unique values.
*   **High computational cost** – requires full deduplication.
*   **Performance bottlenecks** – when the dataset is large, Spark needs to shuffle data between nodes, slowing down the query.

# **Why is** `APPROX_COUNT_DISTINCT(col)` **Faster?**

Instead of **exactly counting distinct values**, `APPROX_COUNT_DISTINCT()` uses an efficient **HyperLogLog (HLL)** algorithm.

### **How HyperLogLog (HLL) Works**

1.  Converts input values into **hash values**.
2.  Tracks **the maximum number of leading zeros** in the hash values.
3.  Uses a mathematical formula to estimate the number of unique values.

**Advantages of HLL:**

*   Requires **very little memory**.
*   Handles **billions of unique values** efficiently.
*   Has **O(1) time complexity** (constant time, very fast).

# **Comparing Performance**

| **Method** | **How It Works** | **Performance** | **Best for** |
| --- | --- | --- | --- |
| `COUNT(DISTINCT col)` | **Full scan + shuffle + deduplication** | **Slow** (high memory & CPU usage) | **Small datasets, where exact values are needed** |
| `APPROX_COUNT_DISTINCT(col)` | **Approximate counting using HyperLogLog** | **Fast** (low memory & CPU usage) | **Large datasets, when a small error (1-2%) is acceptable** |

### **SQL Example**

### **Slow Method (Full Deduplication)**

```
SELECT COUNT(DISTINCT player_id) FROM Activity;
```

### **Optimized Method (Faster with Approximation)**

```
SELECT APPROX_COUNT_DISTINCT(player_id) FROM Activity;
```

### **Example Performance**

Assume the `Activity` table has **10 million rows**:

*   `COUNT(DISTINCT player_id)` → **Takes 10 seconds** (due to full deduplication and shuffle).
*   `APPROX_COUNT_DISTINCT(player_id)` → **Takes 1 second** (with only a ~1% error).

# **When to Use** `APPROX_COUNT_DISTINCT()`

*   When working with **big data (millions or billions of records)**.
*   When **a small error (1-2%) is acceptable**.
*   When **you need faster query performance**.
*   When doing **analytics or trend analysis** (e.g., unique user counts, event tracking).

For small datasets (thousands of rows), `COUNT(DISTINCT col)` is fine. But in **distributed Spark computing**, `APPROX_COUNT_DISTINCT()` significantly **reduces shuffle overhead** and **improves speed**.

# **Conclusion**

*   Uses **HyperLogLog** to estimate distinct counts.
*   **Much faster** than `COUNT(DISTINCT col)`.
*   **Avoids expensive shuffle operations**.
*   **Ideal for large-scale data analysis**.

🚀 **If you're dealing with big data in SparkSQL, use** `**APPROX_COUNT_DISTINCT()**` **to speed up your queries!**
