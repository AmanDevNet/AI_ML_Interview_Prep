# Chapter 6: NumPy & Pandas Internals — "High-Performance Data Engineering"

---

## Topic Overview

| Aspect | Details |
|--------|---------|
| **Definition** | **NumPy** provides high-performance multidimensional arrays (`ndarrays`) written in C. **Pandas** builds on NumPy to provide structured DataFrame objects for data manipulation. |
| **Why It Exists** | Python lists are arrays of pointers to scattered objects in memory (bad cache locality). NumPy arrays store raw values in contiguous blocks of memory, allowing lightning-fast operations. |
| **Where It Is Used** | Machine learning (data preprocessing, neural network weights), data analysis, scientific computing, and computer vision. |
| **Real-World Analogy** | **Python List:** A treasure map where every clue points to a box located in a different city. You have to travel (pointer lookup) to find each value. **NumPy Array:** An egg carton where all eggs (values) are placed side-by-side in a single container. You can grab them instantly. |

---

## Think of It This Way

### 1. NumPy: Strides & Zero-Copy Slicing
A 2D NumPy array might look like a grid to you, but in memory, it is a single, flat, one-dimensional block of bytes.
To navigate this flat block as a 2D grid, NumPy uses **Strides**. 
Strides are the number of bytes Python needs to step in memory to move to the next element along each dimension.

Because of strides, when you slice an array (e.g. `sub_array = array[1:3, 0:2]`), NumPy does **not** copy the values in memory. It simply creates a new **view** with different strides pointing to the *same* raw data buffer. This is a zero-copy operation and is near-instant.

### 2. Pandas: The BlockManager Overhead
Under the hood, a Pandas DataFrame is not just a collection of rows. It uses a **BlockManager** to group columns of the same datatype together into 2D NumPy arrays. 
If you have 10 float columns and 5 int columns, the BlockManager groups them into one 2D float array and one 2D int array.

This makes column-wise operations fast, but **row-wise operations (`iterrows`) extremely slow** because Pandas has to compile a new Series object for each row dynamically.

---

## Step 1: NumPy Strides & Memory Layout

```python
import numpy as np

# Create a 2D array of 32-bit integers (4 bytes each)
arr = np.array([[10, 20, 30],
                [40, 50, 60]], dtype=np.int32)

# Strides returns bytes to step along (dimension 0, dimension 1)
# To go to the next row (dim 0), we must step 12 bytes (3 ints * 4 bytes).
# To go to the next column (dim 1), we must step 4 bytes (1 int * 4 bytes).
print(arr.strides) # (12, 4)
```

### Broadcasting Rules
When performing operations on two arrays of different shapes, NumPy compares their dimensions from **right to left**:
1. The dimensions must be equal, OR
2. One of the dimensions must be `1`.

```python
a = np.array([[1, 2, 3],
              [4, 5, 6]]) # Shape (2, 3)
b = np.array([10, 20, 30]) # Shape (3,) -> treated as (1, 3)

# Broadcasting works because columns match (3 == 3) and rows match (1 fits into 2)
print(a + b)
# Output:
# [[11, 22, 33]
#  [14, 25, 36]]
```

---

## Step 2: Optimizing Pandas Performance

Avoid `iterrows()` at all costs in production data codebases:

```python
import pandas as pd
import time

df = pd.DataFrame({"val": range(100_000)})

# ❌ SLOW: Row-by-row iteration
start = time.perf_counter()
res = []
for index, row in df.iterrows():
    res.append(row["val"] * 2)
print(f"Iterrows: {time.perf_counter() - start:.4f} seconds")

# ✅ FAST: Vectorized operation (runs in compiled C/C++)
start = time.perf_counter()
res_fast = df["val"] * 2
print(f"Vectorized: {time.perf_counter() - start:.4f} seconds")
```
*Vectorization runs up to **100 times faster** because it pushes loops out of Python into C-level execution blocks.*

---

## Code Examples

### Easy Examples
```python
# 1. NumPy memory check
x = np.ones((1000, 1000), dtype=np.float64)
# 1,000,000 floats * 8 bytes = 8,000,000 bytes
print(x.nbytes) # 8000000

# 2. View vs Copy check
parent = np.array([1, 2, 3])
view = parent[0:2]
view[0] = 99
print(parent) # [99,  2,  3] (parent was modified because 'view' shared its memory!)
```

### Medium Examples
```python
# 3. Optimizing Pandas Memory with Categoricals
# Ideal for columns with low cardinality (e.g. state names, categories)
df_cat = pd.DataFrame({"state": ["CA", "NY", "TX"] * 100000})

print(df_cat.memory_usage(deep=True)) # Large string overhead

# Convert to category
df_cat["state"] = df_cat["state"].astype("category")
print(df_cat.memory_usage(deep=True)) # Significantly reduced!
```

### Advanced Examples
```python
# 4. Sharing memory between PyTorch tensors and NumPy arrays
# PyTorch allows wrapping NumPy arrays without duplicating the underlying data buffer
import torch

np_arr = np.array([1, 2, 3], dtype=np.float32)
torch_tensor = torch.from_numpy(np_arr)

# Modifying NumPy array changes the PyTorch tensor!
np_arr[0] = 99.0
print(torch_tensor) # tensor([99.,  2.,  3.])

# 5. pd.eval() for massive formula calculations
# Avoids allocating intermediate copies of massive DataFrames in memory
df1 = pd.DataFrame(np.random.randn(1_000_000, 3), columns=['A', 'B', 'C'])
df2 = pd.DataFrame(np.random.randn(1_000_000, 3), columns=['A', 'B', 'C'])

# Under the hood, this compiles the equation, reducing cache allocation overhead
result = pd.eval("df1 + df2")
```

---

## Common Mistakes & Edge Cases

### 1. Modifying a slice without calling `.copy()`
If you slice a slice of a DataFrame (e.g. `df2 = df[df["val"] > 10]`) and then assign values (`df2["new_col"] = 1`), Python will trigger a `SettingWithCopyWarning`. This happens because Pandas cannot guarantee if the assignment will modify the original DataFrame or only the sliced view.
* **Fix:** Always explicitly declare a copy if you plan to modify a slice: `df2 = df[df["val"] > 10].copy()`.

---

## Interview Questions (Top 5)

**Q1: How do strides work in NumPy arrays, and how do they enable zero-copy operations?**
> A NumPy array is a flat block of memory. To navigate it, strides store the number of bytes that must be skipped in memory to move to the next element in each dimension. 
> Slicing returns a new array object (a **view**) that points to the same underlying data buffer but has modified offsets, shapes, and strides. Because no data is copied, slicing is `O(1)` in both time and memory.

**Q2: What is broadcasting in NumPy, and what are the rules governing it?**
> Broadcasting is a mechanism that allows arithmetic operations between arrays of different shapes. Starting from the trailing (rightmost) dimension, NumPy compares shape lengths. The dimensions are compatible if they are equal, or if one of them is `1`. If compatible, NumPy conceptually repeats the smaller array along the missing dimension to match the larger array without allocating memory.

**Q3: Why is `pd.DataFrame.iterrows()` slow, and what should you use instead?**
> `iterrows()` is slow because it runs a loop in Python rather than in C. For each row, Pandas has to instantiate a new `pd.Series` object, copy the row elements, and map the datatypes, which adds massive object-allocation overhead.
> You should use **vectorized operations** (which run compiled C/C++ loops directly), or `.apply()` with lambda functions as a fallback, or package the logic using **Numba** for raw speed.

**Q4: Explain the BlockManager architecture in Pandas and its performance impact.**
> Pandas DataFrames group columns of the same datatype into single 2D NumPy arrays using an internal structure called the `BlockManager`. 
> * **Benefit:** Fast column-wise computations (like calculating the mean of a numeric column) because the column data is contiguous in memory.
> * **Drawback:** High copy overhead when inserting, deleting, or altering datatypes because the BlockManager has to reconstruct the 2D arrays.

**Q5: What is the difference between a view and a copy in NumPy, and how can you check which one you have?**
> A **view** shares the memory buffer of the original array; modifying the view modifies the parent. A **copy** allocates a new memory buffer, leaving the original array unaffected. 
> You can check if an array is a view by reading its `.base` attribute. If it returns another NumPy array, it is a view; if it returns `None`, it owns its own memory (it is a copy).

---

*← [Previous: Chapter 5 (FastAPI & Serialization)](05_FastAPI_Serialization.md) | [Next: Chapter 7 (Python in ML Pipelines & AI Agents) →](07_Python_ML_Pipelines_Agents.md)*
