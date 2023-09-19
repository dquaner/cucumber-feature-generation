# Metric: rouge

> [原文链接](https://huggingface.co/spaces/evaluate-metric/rouge)

## Metric Description

ROUGE (Recall-Oriented Understudy for Gisting Evaluation) 用于评估 NLP 中的自动摘要和机器翻译任务。
这个指标将自动生成的摘要/译文和一系列参考（人工制作的）摘要/译文做比较。

注意，ROUGE 是大小写不敏感的。

这个指标是对 [Google Research 重新实现的 ROUGE](https://github.com/google-research/google-research/tree/master/rouge) 的包装。

如果这是基于文本的指标，请确保将输入用双引号引起来。或者，你可以使用 JSON 格式的列表作为输入。

## How to Use

最简单的情况，该指标将预测列表和参考列表作为输入：
```python
>>> rouge = evaluate.load('rouge')
>>> predictions = ["hello there", "general kenobi"]
>>> references = ["hello there", "general kenobi"]
>>> results = rouge.compute(predictions=predictions,
...                         references=references)
>>> print(results)
{'rouge1': 1.0, 'rouge2': 1.0, 'rougeL': 1.0, 'rougeLsum': 1.0}
```

传入自定义的 tokenizer：
```python
>>> results = rouge.compute(predictions=predictions,
...                         references=references,
                            tokenizer=lambda x: x.split())
>>> print(results)
{'rouge1': 1.0, 'rouge2': 1.0, 'rougeL': 1.0, 'rougeLsum': 1.0}
```

对每个预测提供一个参考列表：
```python
>>> rouge = evaluate.load('rouge')
>>> predictions = ["hello there", "general kenobi"]
>>> references = [["hello", "there"], ["general kenobi", "general yoda"]]
>>> results = rouge.compute(predictions=predictions,
...                         references=references)
>>> print(results)
{'rouge1': 0.8333, 'rouge2': 0.5, 'rougeL': 0.8333, 'rougeLsum': 0.8333}
```

### Inputs
- **predictions** (`list`): 要评分的预测列表。
        每个预测应该是一个字符串，其中的词元以空格分隔。
- **references** (`list` or `list[list]`): 预测的参考列表或每个预测分别的参考列表。 
        每个参考应该是一个字符串，词元以空格分隔。
- **rouge_types** (`list`): 要计算的 rouge 类型。 默认为 `['rouge1', 'rouge2', 'rougeL', 'rougeLsum']`。
    - Valid rouge types:
        - `"rouge1"`: unigram (1-gram) based scoring
        - `"rouge2"`: bigram (2-gram) based scoring
        - `"rougeL"`: Longest common subsequence based scoring.
        - `"rougeLSum"`: splits text using `"\n"`
        - See [here](https://github.com/huggingface/datasets/issues/617) for more information
- **use_aggregator** (`boolean`): 如果为 True，则返回聚合结果。默认为 `True`。
- **use_stemmer** (`boolean`): 如果为 True，则使用 Porter 词干分析器去除单词后缀。默认为 `False`。

### Output Values
The output is a dictionary with one entry for each rouge type in the input list `rouge_types`. If `use_aggregator=False`, each dictionary entry is a list of scores, with one score for each sentence. E.g. if `rouge_types=['rouge1', 'rouge2']` and `use_aggregator=False`, the output is:

```python
{'rouge1': [0.6666666666666666, 1.0], 'rouge2': [0.0, 1.0]}
```

If `rouge_types=['rouge1', 'rouge2']` and `use_aggregator=True`, the output is of the following format:
```python
{'rouge1': 1.0, 'rouge2': 1.0}
```

ROUGE 值的范围是 0 到 1。


### Examples
An example without aggregation:
```python
>>> rouge = evaluate.load('rouge')
>>> predictions = ["hello goodbye", "ankh morpork"]
>>> references = ["goodbye", "general kenobi"]
>>> results = rouge.compute(predictions=predictions,
...                         references=references,
...                         use_aggregator=False)
>>> print(list(results.keys()))
['rouge1', 'rouge2', 'rougeL', 'rougeLsum']
>>> print(results["rouge1"])
[0.5, 0.0]
```

The same example, but with aggregation:
```python
>>> rouge = evaluate.load('rouge')
>>> predictions = ["hello goodbye", "ankh morpork"]
>>> references = ["goodbye", "general kenobi"]
>>> results = rouge.compute(predictions=predictions,
...                         references=references,
...                         use_aggregator=True)
>>> print(list(results.keys()))
['rouge1', 'rouge2', 'rougeL', 'rougeLsum']
>>> print(results["rouge1"])
0.25
```

The same example, but only calculating `rouge_1`:
```python
>>> rouge = evaluate.load('rouge')
>>> predictions = ["hello goodbye", "ankh morpork"]
>>> references = ["goodbye", "general kenobi"]
>>> results = rouge.compute(predictions=predictions,
...                         references=references,
...                         rouge_types=['rouge_1'],
...                         use_aggregator=True)
>>> print(list(results.keys()))
['rouge1']
>>> print(results["rouge1"])
0.25
```

## Limitations and Bias
See [Schluter (2017)](https://aclanthology.org/E17-2007/) for an in-depth discussion of many of ROUGE's limits.

## Citation
```bibtex
@inproceedings{lin-2004-rouge,
    title = "{ROUGE}: A Package for Automatic Evaluation of Summaries",
    author = "Lin, Chin-Yew",
    booktitle = "Text Summarization Branches Out",
    month = jul,
    year = "2004",
    address = "Barcelona, Spain",
    publisher = "Association for Computational Linguistics",
    url = "https://www.aclweb.org/anthology/W04-1013",
    pages = "74--81",
}
```

## Further References
- This metrics is a wrapper around the [Google Research reimplementation of ROUGE](https://github.com/google-research/google-research/tree/master/rouge)