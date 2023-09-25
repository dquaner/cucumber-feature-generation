# Datasets 🤝 Arrow

## Arrow 是什么？

[Arrow](https://arrow.apache.org/) 能够快速处理和移动大量数据。它是一种特定的数据格式，将数据存储在一种列式内存布局中。它提供了几个显著的优势：

* Arrow 的标准格式允许[零拷贝读取](https://en.wikipedia.org/wiki/Zero-copy)，这几乎消除了所有序列化开销。
* Arrow 是语言无关的，所以它支持不同的编程语言。
* Arrow 是面向列的，所以它在查询和处理数据片或列时速度更快。
* Arrow 允许无复制切换到标准机器学习工具，如 NumPy, Pandas, PyTorch 和T ensorFlow。
* Arrow 支持许多列类型，包括嵌套的列类型。

## 内存映射

🤗 Datasets 使用 Arrow 作为其本地缓存系统。它允许数据集由磁盘缓存支持，磁盘缓存是内存映射的，用于快速查找。
这种架构允许在设备内存相对较小的机器上使用大型数据集。

例如，加载完整的英文维基百科数据集只需要几MB的 RAM：

```python
>>> import os; import psutil; import timeit
>>> from datasets import load_dataset

# Process.memory_info is expressed in bytes, so convert to megabytes 
>>> mem_before = psutil.Process(os.getpid()).memory_info().rss / (1024 * 1024)
>>> wiki = load_dataset("wikipedia", "20220301.en", split="train")
>>> mem_after = psutil.Process(os.getpid()).memory_info().rss / (1024 * 1024)

>>> print(f"RAM memory used: {(mem_after - mem_before)} MB")
RAM memory used: 50 MB
```

这是可能的，因为 Arrow 数据实际上是从磁盘映射到内存的，而不是加载到内存中。
内存映射允许访问磁盘上的数据，并利用虚拟内存功能进行快速查找。

## 性能

使用 Arrow 迭代内存映射数据集的速度很快。在笔记本电脑上迭代维基百科数据 ，你可以获得 1-3 Gbit/s 的速度:

```python
>>> s = """batch_size = 1000
... for batch in wiki.iter(batch_size):
...     ...
... """

>>> time = timeit.timeit(stmt=s, number=1, globals=globals())
>>> print(f"Time to iterate over the {wiki.dataset_size >> 30} GB dataset: {time:.1f} sec, "
...       f"ie. {float(wiki.dataset_size >> 27)/time:.1f} Gb/s")
Time to iterate over the 18 GB dataset: 31.8 sec, ie. 4.8 Gb/s
```