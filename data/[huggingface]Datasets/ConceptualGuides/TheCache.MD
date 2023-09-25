# 缓存

缓存是 🤗 Datasets 如此高效的原因之一。它存储以前下载和处理过的数据集，所以当你需要再次使用它们时，它们直接从缓存中重新加载。这避免了重新下载数据集或重新应用处理函数。即使在关闭并启动另一个 Python 会话之后，🤗 Datasets 也会直接从缓存中重新加载数据集!

## 指纹

缓存如何跟踪应用于数据集的转换（transforms）? 🤗 Datasets 将指纹分配给缓存文件，指纹跟踪数据集的当前状态。初始指纹是使用 Arrow 表中的散列计算的，如果数据集在磁盘上，则使用 Arrow 文件的散列计算。后续的指纹是通过结合前一个状态的指纹和应用在数据集上的最新转换的散列来计算的。

> 转换（transforms）是指来自 [How-to Process](https://huggingface.co/docs/datasets/process) 指南中的任何处理方法，例如 [`Dataset.map`] 或 [`Dataset.shuffle`]。

Here are what the actual fingerprints look like:

```py
>>> from datasets import Dataset
>>> dataset1 = Dataset.from_dict({"a": [0, 1, 2]})
>>> dataset2 = dataset1.map(lambda x: {"a": x["a"] + 1})
>>> print(dataset1._fingerprint, dataset2._fingerprint)
d19493523d95e2dc 5b86abacd4b42434
```

为了使转换可以计算哈希，它需要通过 [dill](https://dill.readthedocs.io/en/latest/) 或 [pickle](https://docs.python.org/3/library/pickle) 进行 picklable。

当你使用一个非哈希的转换时，🤗 Datasets 将使用随机指纹并发出警告。非哈希的转换被认为与前面的转换不同。因此，🤗 Datasets 将重新计算所有转换。确保你的转换是可以用 pickle 或 dill 序列化的，以避免这种情况!

🤗 Datasets 重新计算所有内容的一个示例是禁用缓存时。发生这种情况时，每次都会生成缓存文件，并将它们写入临时目录。一旦 Python 会话结束，临时目录中的缓存文件将被删除。随机散列被分配给这些缓存文件，而不是指纹。

> 当缓存被禁用时，记得使用 [`Dataset.save_to_disk()`](https://huggingface.co/docs/datasets/v2.14.5/en/package_reference/main_classes#datasets.Dataset.save_to_disk) 保存转换后的数据集，否则会话结束后数据集将被删除。

## 哈希

哈希被传递给 `map` ，或者 `map` 的参数（`batch_size`，`remove_columns` 等）的方法来更新一个数据集的指纹。

你可以使用 [`fingerprint.Hasher`](https://huggingface.co/docs/datasets/v2.14.5/en/package_reference/main_classes#datasets.fingerprint.Hasher) 来检查任何 Python 对象的哈希值：

```py
>>> from datasets.fingerprint import Hasher
>>> my_func = lambda example: {"length": len(example["text"])}
>>> print(Hasher.hash(my_func))
'3d35e2b3e94c81d6'
```

哈希是通过使用 `dill` pickler 转储对象并对转储的字节进行哈希来计算的。Pickler 递归地转储函数中使用的所有变量，因此对函数中使用的对象所做的任何更改都会导致哈希值发生变化。

如果你的某个函数在会话之间似乎没有相同的哈希值，这意味着它的至少一个变量包含一个不确定的 Python 对象。
发生这种情况时，请哈希任何你认为可疑的对象，以尝试找到导致哈希更改的对象。
例如，如果您使用一个列表，其元素的顺序在会话之间不是确定的，那么哈希值在会话之间也不会相同。