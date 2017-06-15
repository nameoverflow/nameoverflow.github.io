---
title: 基于 Hash 与 Winnowing 方法的文档查重
date: 2017-05-11 23:20:08
tags:
---
> 这只是一篇实验报告

## 设计思路

基于现实计算能力考虑，许多大段文档之间的相似性比较不可能使用传统的文本 diff 算法等耗时长的方法。Hash 值可以在一定程度上反应数据的特征；但是一般的 Hash 方法强调避免碰撞，源数据的少许改变就可以引起 Hash 值的较大变化。对于查重来说，需要提取出文档的特征，这个特征在源数据相似时也应具有相似性。

Winnowing 方法是 Saul Schleimer 等提出的提取文档特征（文档指纹）的方法。通过对文档的 K-gram 序列进行 hash ，提取出能够反应文档相似性的特征值序列，再对这个特征值序列进行比较[1]，得到的相同特征值的比例即可反映出文档之间的相似性。

<!--more-->

## Winnowing 方法

[K 元语法（K-gram）](https://en.wikipedia.org/wiki/N-gram)是指文本中连续出现的 K 个语词（字母、单词等），是一种概率语言模型，一定程度上能反应语句的结构并且顺序不敏感。

如文本序列

```
我可以吞下玻璃而不伤身体
```

的 3 元语法序列是

```
我可以
可以吞
以吞下
吞下玻
下玻璃
玻璃而
璃而不
而不伤
不伤身
伤身体
```

显然对于长度为 N 的序列其 K 元语法序列长为 N - K + 1。

K 元语法单元可以反应语义，可以对其进行 Hash 来方便存储和比较。

Winnowing 方法中采用的 Hash 方式为

{% math %}
H(c_1 ...c_{k})=\Sigma^{k}_{i =1}c_i*b^{k-i}
{% endmath %}

其中 {% math %} c_i {% endmath %} 表示源文本中第 i 个语词， {% math %} c_1...c_k {% endmath %} 表示从第 1 个语词到第 k 个语词的 K 元语法单元。b 为参数，一般取质数。

这种方法下，计算第 n 个 K 元语法单元的 Hash 值可以简化为

{% math %}
H(c_n...c_{n+k-1})=(H(c_{n-1}...c_{(n+k-1)-1})-c_{n-1}*b^{(n+k-1)-2})*b+c_{n+k-1}
{% endmath %}

计算得到 Hash 序列之后，摘取长度为 S 的滑动窗口中的最小值作为特征值。如果窗口中的最小值不变，则直接往后滑动，不改变特征值。

如对于上述 3 元语法序列，取 {% math %} b=3 {% endmath %}，对中文取 unicode 编码，有

```
我可以 = 310603
可以吞 = 275508
以吞下 = 266354
吞下玻 = 283370
下玻璃 = 298519
玻璃而 = 388904
璃而不 = 386764
而不伤 = 375223
不伤身 = 277132
伤身体 = 312216
```
得到 Hash 值序列

```
[310603, 275508, 266354, 283370, 298519, 388904, 386764, 375223, 277132, 312216]
```

取窗口长 4 ，得到以下的滑动窗口。其中第 0、2、3、4 个窗口中最小值发生变化，取这些窗口中的最小值作为特征值

```
([275508, 266354, 283370, 298519]) => 266354
([266354, 283370, 298519, 388904])
([283370, 298519, 388904, 386764]) => 283370
([298519, 388904, 386764, 375223]) => 298519
([388904, 386764, 375223, 277132]) => 277132
([386764, 375223, 277132, 312216])
```

得到以下特征值序列作为这段文本的指纹

```
266354, 283370, 298519, 277132
```

## 程序实现

使用 Python 实现，运行环境为 Python 3.6+，依赖 Pandoc 进行 docx/doc 文件的读取。

程序的主要功能是读取指定文件夹下所有文档，计算出每个文档的特征值并两两比较，得到相同的特征值的比例，并可以将结果存储为 csv 文件。

命令行使用方法为：

```
python similarity.py <path> <dest_file_name> <mode>
```

`<path>` 为待检测的文档集所在目录；不可省略。

`<dest_file_name>` 为检测结果存放的 csv 文件文件名；不可省略。

`<mode>` 为结果模式选择，可省略。如果为 `sus` ，只记录相似程度高（> 0.2）的文件名和相似度，否则记录所有比较结果。

### 程序主体

根据命令行参数，计算给出的文件夹下的所有文档的特征值序列并进行比较，储存结果。

```python
"""
Plagiarism detection based on winnowing
"""

from typing import Iterator, Tuple, Dict, List
from operator import itemgetter
from itertools import islice
from collections import deque

import csv
import sys
import os

import pypandoc

# ...

if __name__ == '__main__':
    fingerprints = {}
    similarities = {}
    arglen = len(sys.argv)
    if arglen < 2:
        print('Expect command-line argument <path>')
        exit(1)
    elif arglen < 3:
        print('Expect command-line argument <dest_file_name>')
        exit(1)
    target = sys.argv[1]
    dump = write_suspect if arglen > 3 and sys.argv[3] == 'sus' else write_csv

    files = [fname for fname in os.listdir(target)]
    for fname in files:
        fullpath = os.path.join(target, fname)
        fingerprints[fname] = compute_file(fullpath)
        similarities[fname] = []


    for fsrc in files:
        fpsrc = fingerprints[fsrc]
        keylen = len(fpsrc)
        for fdest in files:
            fpdest = fingerprints[fdest]
            coll = [fpdest[h] for h in fpsrc if h in fpdest]

            similarities[fsrc].append(round(len(coll) / keylen, 3))

    print("Compared " + str(len(files)) + " documents")
    dump(files, similarities, sys.argv[2])
```

### 取得 K-gram

简单的从字符串中逐个摘取长为 K 的子串。

```python
def gram(k: int, text: str) -> Iterator[str]:
    """
    Generate K-gram
    """
    length = len(text)
    for i in range(length):
        if i + k > length:
            break
        yield text[i : i + k]
```

### 计算 Hash

使用上述公式计算每个 K-gram 的 Hash 值

```python
def hash_gram(gram_gen: Iterator[str], base: int) -> Iterator[int]:
    """
    Compute hash of K-gram
    """
    prev_gram = next(gram_gen)
    K = len(prev_gram)
    prev_hash = sum([ord(prev_gram[i]) * base ** (K - i - 1) for i in range(K)])
    yield prev_hash
    for e in gram_gen:
        # print(e)
        prev_hash = ((prev_hash - ord(prev_gram[0]) * base ** (K - 1)) * base + ord(e[-1]))
        prev_gram = e
        yield prev_hash
```



### 摘取特征值

使用 deque 作为滑动窗口。这里除了摘取窗口中的最小值外，还记录了作为特征值的 Hash 值在原 Hash 序列中的位置。

```python
def min_index(it) -> Tuple[int, int]:
    """
    Return the minimal element together with its index
    """
    return min(enumerate(it), key=itemgetter(1))

def winnow(hashs: Iterator[int], size: int) -> Iterator[Tuple[int, int]]:
    """
    Winnowing
    """

    window = deque(islice(hashs, size))
    pos_base = 0

    min_pos, min_val = min_index(window)
    yield (min_val, min_pos)

    for n in hashs:
        pos_base = pos_base + 1
        left = window.popleft()
        window.append(n)
        print(window)
        if left == min_val:
            min_pos, min_val = min_index(window)
            yield (min_val, min_pos + pos_base)
        elif n < min_val:
            min_val = n
            min_pos = size - 1
            yield (min_val, min_pos + pos_base)
```



### 文件处理

读取文件，将文件内容的空格和换行去除，计算出特征值序列。

将特征值与其位置保存为一个 k-v 映射，方便比较文件时进行查询。

```python
BASE = 4
K = 4

def compute_file(fname):
    """
    Compute the fingerprint of a document using winnowing
    """
    text = ''.join(str(pypandoc.convert_file(fname, to='plain')).split())
    # print(text)
    fingerprint = {}

    for hashed, pos in winnow(hash_gram(gram(K, text), BASE), 4):
        fingerprint[hashed] = pos

    return fingerprint
```

### 储存结果

将比较的结果（相同的特征值个数与总的特征值个数的比例）储存在 csv 文件中。

```python
def write_csv(file_names: List[str], data_rows: Dict[str, List[float]], dest: str):
    """
    Write results to csv file
    """

    with open(dest, mode='w', encoding='gbk', newline='') as file:
        # 因为 office 默认 GBK 编码所以不得已用 gbk……
        writer = csv.writer(file, dialect='excel')
        writer.writerow([''] + file_names)
        for f in file_names:
            writer.writerow([f] + data_rows[f])

def write_suspect(file_names: List[str], data_rows: Dict[str, List[float]], dest: str):
    """
    List all document pairs suspected to be plagiarism
    """
    with open(dest, mode='w', encoding='gbk', newline='') as file:
        writer = csv.writer(file, dialect='excel')
        for f in file_names:
            for d, dname in enumerate(file_names):
                similarity = data_rows[f][d]
                if f != dname and similarity > 0.2:
                    writer.writerow([f, dname, similarity])
```

## 测试

从维基百科[百度百科对维基百科的侵权/列表](https://zh.wikipedia.org/wiki/Wikipedia:%E7%99%BE%E5%BA%A6%E7%99%BE%E7%A7%91%E5%B0%8D%E7%B6%AD%E5%9F%BA%E7%99%BE%E7%A7%91%E7%9A%84%E4%BE%B5%E6%AC%8A/%E5%88%97%E8%A1%A8#.E5.84.AA.E8.89.AF.E6.A2.9D.E7.9B.AE)中摘取部分条目（见附录）的正文部分写入 docx 文件进行比较测试。

| -    | 百度1   | 维基1   | 百度2   | 维基2   | 百度3   | 维基3   | 百度4   | 维基4   |
| ---- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| 百度1  | 1.0   | 0.595 | 0.033 | 0.041 | 0.074 | 0.055 | 0.083 | 0.061 |
| 维基1  | 0.477 | 1.0   | 0.026 | 0.091 | 0.066 | 0.139 | 0.042 | 0.174 |
| 百度2  | 0.029 | 0.029 | 1.0   | 0.744 | 0.117 | 0.079 | 0.062 | 0.086 |
| 维基2  | 0.034 | 0.094 | 0.71  | 1.0   | 0.126 | 0.142 | 0.066 | 0.18  |
| 百度3  | 0.006 | 0.007 | 0.011 | 0.012 | 1.0   | 0.091 | 0.011 | 0.023 |
| 维基3  | 0.01  | 0.032 | 0.017 | 0.031 | 0.21  | 1.0   | 0.025 | 0.07  |
| 百度4  | 0.054 | 0.034 | 0.047 | 0.052 | 0.088 | 0.09  | 1.0   | 0.266 |
| 维基4  | 0.017 | 0.062 | 0.028 | 0.062 | 0.083 | 0.11  | 0.117 | 1.0   |

其中百度1-维基1、百度2-维基2的结果值非常高，通过人工辨别发现明显相似。

使用 sus 模式运行程序，结果为

```
百度1.docx,维基1.docx,0.595
百度2.docx,维基2.docx,0.744
百度4.docx,维基4.docx,0.266
维基1.docx,百度1.docx,0.477
维基2.docx,百度2.docx,0.71
维基3.docx,百度3.docx,0.21
```

列出了所有相似程度较高的文件对。



## 附录

### 测试文档列表

测试文档为：

百度1：[北美负鼠_百度百科](http://baike.baidu.com/item/%E5%8C%97%E7%BE%8E%E8%B4%9F%E9%BC%A0)

维基1：[北美负鼠 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%8C%97%E7%BE%8E%E8%B2%A0%E9%BC%A0)

百度2：[喀布尔大学_百度百科](http://baike.baidu.com/item/%E5%96%80%E5%B8%83%E5%B0%94%E5%A4%A7%E5%AD%A6)

维基2：[喀布尔大学 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%96%80%E5%B8%83%E5%B0%94%E5%A4%A7%E5%AD%A6)

百度3：[拉丁美洲文学爆炸_百度百科](http://baike.baidu.com/item/%E6%8B%89%E4%B8%81%E7%BE%8E%E6%B4%B2%E6%96%87%E5%AD%A6%E7%88%86%E7%82%B8)

维基3：[拉丁美洲文学爆炸 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E6%8B%89%E4%B8%81%E7%BE%8E%E6%B4%B2%E6%96%87%E5%AD%A6%E7%88%86%E7%82%B8)

百度4：[台北世界贸易中心_百度百科](http://baike.baidu.com/item/%E5%8F%B0%E5%8C%97%E4%B8%96%E7%95%8C%E8%B4%B8%E6%98%93%E4%B8%AD%E5%BF%83)

维基4：[台北世界贸易中心 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%8F%B0%E5%8C%97%E4%B8%96%E7%95%8C%E8%B2%BF%E6%98%93%E4%B8%AD%E5%BF%83)

## 参考文献

[1] Schleimer, Saul, Daniel S. Wilkerson, and Alex Aiken. "Winnowing: local algorithms for document fingerprinting." *Proceedings of the 2003 ACM SIGMOD international conference on Management of data*. ACM, 2003.