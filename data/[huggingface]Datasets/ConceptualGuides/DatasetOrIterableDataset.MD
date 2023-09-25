# Dataset 和 IterableDataset 的区别


有两种类型的数据集对象：[`Dataset`] 和 [`IterableDataset`]。
选择使用或创建哪种类型的数据集取决于数据集的大小，
一般来说，由于 [`IterableDataset`] 的懒惰行为和速度优势，它是理想的大数据集（想想数百GBs!）；而 [`Dataset`] 足以处理所有其他的数据。

本页将比较 [`Dataset`] 和 [`IterableDataset`] 之间的差异，以帮助您选择合适的数据集对象。

## Downloading and streaming

当你有一个常规的 [`Dataset`]，你可以使用 `my_dataset[0]` 访问它，这提供了对行的随机访问。
这样的数据集也被称为“映射式（map-style）”数据集。
例如，你可以像这样下载 `ImageNet-1k` 并访问任意一行:

```python
from datasets import load_dataset

imagenet = load_dataset("imagenet-1k", split="train")  # downloads the full dataset
print(imagenet[0])
```

但需要注意的是，您必须将整个数据集存储在磁盘或内存中，这将导致你不能访问比磁盘大的数据集。因为 [`Dataset`] 对于大数据集来说不方便，所以存在另一种类型的数据集，即 `IterableDataset`。
当你有一个 `IterableDataset` 时，你可以使用 `for` 循环来访问它，在你迭代数据集时逐步加载数据。
通过这种方式，只有一小部分示例被加载到内存中，并且您不会在磁盘上写入任何内容。

例如，您可以流式传输 `ImageNet-1k` 数据集，而无需将其下载到磁盘上：


```python
from datasets import load_dataset

imagenet = load_dataset("imagenet-1k", split="train", streaming=True)  # will start loading the data when iterated over
for example in imagenet:
    print(example)
    break
```

流可以在不向磁盘写入任何文件的情况下读取在线数据。
例如，你可以流式传输由多个分片组成的数据集，每个分片都是数百GB，如 [C4](https://huggingface.co/datasets/c4)， [OSCAR](https://huggingface.co/datasets/oscar) 或 [LAION-2B](https://huggingface.co/datasets/laion/laion2B-en)。
你可以在 [Dataset Streaming Guide](https://huggingface.co/docs/datasets/stream) 中了解更多关于如何流式访问数据集的信息。

这并不是唯一的区别，`IterableDataset` 在创建和处理数据集时还会出现“懒惰（lazy）”行为。

## 创建映射式数据集和可迭代数据集

你可以使用列表或字典创建一个 [`Dataset`]，数据会完全转换为 Arrow，所以你可以很容易地访问任何一行：

```python
my_dataset = Dataset.from_dict({"col_1": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]})
print(my_dataset[0])
```

另一方面，要创建一个 `IterableDataset`，你必须提供一种“惰性”的方式来加载数据。
在Python中，我们通常使用生成器函数。这些函数一次 `yield` 一个例子，这意味着你不能像普通的 `Dataset` 一样通过切片来访问一行：

```python
def my_generator(n):
    for i in range(n):
        yield {"col_1": i}

my_iterable_dataset = IterableDataset.from_generator(my_generator, gen_kwargs={"n": 10})
for example in my_iterable_dataset:
    print(example)
    break
```

## 整个加载和逐步加载本地文件

可以使用 [`load_dataset()`] 将本地或远程数据文件转换为Arrow [`Dataset`]:

```python
data_files = {"train": ["path/to/data.csv"]}
my_dataset = load_dataset("csv", data_files=data_files, split="train")
print(my_dataset[0])
```

但是，这需要从 CSV 到 Arrow 格式的转换步骤，如果数据集很大，则需要花费时间和磁盘空间。

为了节省磁盘空间并跳过转换步骤，你可以通过直接从本地文件流式传输来定义 `IterableDataset`。
这样，当你遍历数据集时，数据就会从本地文件中逐步读取:

```python
data_files = {"train": ["path/to/data.csv"]}
my_iterable_dataset = load_dataset("csv", data_files=data_files, split="train", streaming=True)
for example in my_iterable_dataset:  # this reads the CSV file progressively as you iterate over the dataset
    print(example)
    break
```

支持多种文件格式，如 CSV、JSONL 和 Parquet，以及图像和音频文件。
你可以在加载 [tabular](https://huggingface.co/docs/datasets/tabular_load)、[text](https://huggingface.co/docs/datasets/nlp_load)、[vision](https://huggingface.co/docs/datasets/image_load) 和 [audio](https://huggingface.co/docs/datasets/audio_load) 数据集的相应指南中找到更多信息。

## 主动数据处理和延迟数据处理

当你使用 [`Dataset.map()`] 处理一个 [`Dataset`] 对象时，整个数据集会立即处理并返回。
这与 `pandas` 的工作模式类似。

```python
my_dataset = my_dataset.map(process_fn)  # process_fn is applied on all the examples of the dataset
print(my_dataset[0])
```

另一方面，由于 `IterableDataset` 的“懒惰”性质，调用 [`IterableDataset.map()`] 不会对整个数据集应用 `map` 函数。相反，你的 `map` 功能是动态应用的。

正因为如此，你可以链接多个处理步骤，当你开始迭代数据集时，它们将同时运行:

```python
my_iterable_dataset = my_iterable_dataset.map(process_fn_1)
my_iterable_dataset = my_iterable_dataset.filter(filter_fn)
my_iterable_dataset = my_iterable_dataset.map(process_fn_2)

# process_fn_1, filter_fn and process_fn_2 are applied on-the-fly when iterating over the dataset
for example in my_iterable_dataset:  
    print(example)
    break
```

## 精确 shuffling 和快速近似 shuffling

When you shuffle a [`Dataset`] using [`Dataset.shuffle()`], you apply an exact shuffling of the dataset.
It works by taking a list of indices `[0, 1, 2, ... len(my_dataset) - 1]` and shuffling this list.
Then, accessing `my_dataset[0]` returns the row and index defined by the first element of the indices mapping that has been shuffled:
```python
my_dataset = my_dataset.shuffle(seed=42)
print(my_dataset[0])
```

Since we don't have random access to the rows in the case of an `IterableDataset`, we can't use a shuffled list of indices and access a row at an arbitrary position.
This prevents the use of exact shuffling.
Instead, a fast approximate shuffling is used in [`IterableDataset.shuffle`].
It uses a shuffle buffer to sample random examples iteratively from the dataset.
Since the dataset is still read iteratively, it provides excellent speed performance:
```python
my_iterable_dataset = my_iterable_dataset.shuffle(seed=42, buffer_size=100)
for example in my_iterable_dataset:
    print(example)
    break
```

But using a shuffle buffer is not enough to provide a satisfactory shuffling for machine learning model training. So [`IterableDataset.shuffle`] also shuffles the dataset shards if your dataset is made of multiple files or sources:

```python
# Stream from the internet
my_iterable_dataset = load_dataset("c4", "en", split="train", streaming=True)
my_iterable_dataset.n_shards  # 1024

# Stream from local files
data_files = {"train": [f"path/to/data_{i}.csv" for i in range(1024)]}
my_iterable_dataset = load_dataset("csv", data_files=data_files, split="train", streaming=True)
my_iterable_dataset.n_shards  # 1024

# From a generator function
def my_generator(n, sources):
    for source in sources:
        for example_id_for_current_source in range(n):
            yield {"example_id": f"{source}_{example_id_for_current_source}"}

gen_kwargs = {"n": 10, "sources": [f"path/to/data_{i}" for i in range(1024)]}
my_iterable_dataset = IterableDataset.from_generator(my_generator, gen_kwargs=gen_kwargs)
my_iterable_dataset.n_shards  # 1024
```

## Speed differences

Regular [`Dataset`] objects are based on Arrow which provides fast random access to the rows.
Thanks to memory mapping and the fact that Arrow is an in-memory format, reading data from disk doesn't do expensive system calls and deserialization.
It provides even faster data loading when iterating using a `for` loop by iterating on contiguous Arrow record batches.

However as soon as your [`Dataset`] has an indices mapping (via [`Dataset.shuffle`] for example), the speed can become 10x slower.
This is because there is an extra step to get the row index to read using the indices mapping, and most importantly, you aren't reading contiguous chunks of data anymore.
To restore the speed, you'd need to rewrite the entire dataset on your disk again using [`Dataset.flatten_indices`], which removes the indices mapping.
This may take a lot of time depending of the size of your dataset though:

```python
my_dataset[0]  # fast
my_dataset = my_dataset.shuffle(seed=42)
my_dataset[0]  # up to 10x slower
my_dataset = my_dataset.flatten_indices()  # rewrite the shuffled dataset on disk as contiguous chunks of data
my_dataset[0]  # fast again
```


In this case, we recommend switching to an [`IterableDataset`] and leveraging its fast approximate shuffling method [`IterableDataset.shuffle`].
It only shuffles the shards order and adds a shuffle buffer to your dataset, which keeps the speed of your dataset optimal.
You can also reshuffle the dataset easily:

```python
for example in enumerate(my_iterable_dataset):  # fast
    pass

shuffled_iterable_dataset = my_iterable_dataset.shuffle(seed=42, buffer_size=100)

for example in enumerate(shuffled_iterable_dataset):  # as fast as before
    pass

shuffled_iterable_dataset = my_iterable_dataset.shuffle(seed=1337, buffer_size=100)  # reshuffling using another seed is instantaneous

for example in enumerate(shuffled_iterable_dataset):  # still as fast as before
    pass
```

If you're using your dataset on multiple epochs, the effective seed to shuffle the shards order in the shuffle buffer is `seed + epoch`.
It makes it easy to reshuffle a dataset between epochs:
```python
for epoch in range(n_epochs):
    my_iterable_dataset.set_epoch(epoch)
    for example in my_iterable_dataset:  # fast + reshuffled at each epoch using `effective_seed = seed + epoch`
        pass
```

## Switch from map-style to iterable

If you want to benefit from the "lazy" behavior of an [`IterableDataset`] or their speed advantages, you can switch your map-style [`Dataset`] to an [`IterableDataset`]:
```python
my_iterable_dataset = my_dataset.to_iterable_dataset()
```

If you want to shuffle your dataset or [use it with a PyTorch DataLoader](./use_with_pytorch#stream-data), we recommend generating a sharded [`IterableDataset`]:
```python
my_iterable_dataset = my_dataset.to_iterable_dataset(num_shards=1024)
my_iterable_dataset.n_shards  # 1024
```