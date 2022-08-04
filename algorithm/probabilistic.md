# Probabilistic algorithms

## Bloom Filter
validation of existence, false positive

## Count-Min Sketch
frequency estimation of keys, counter version bloom filter

scenarios:
- compressed sensing
- network
- NLP (natural language process) in machine learning
- stream processing

## Filtered Space Saving
from Majority problem, to Top-K problem
- Misra-Gries
- Lossy Counting
- space saving
- Filtered Space Saving

## HyperLogLog
cardinary estimation of unique elements, high precision, small space

- Bernoulli experiment
- introduce buckets

## T-Digest
percentile estimation of large quantity of data

- split data set into mini groups by value
- mean value and count of each group to estimate the quantile
- the count larger, the precision lower
- the 1% and 99% percentile is more accurate than 50%

implementations:
- buffer and merge
- AVL group tree

## References
- probabilistic algorithms: http://docs.pipelinedb.com/probabilistic.html
- probabilistic algorithms in big data: https://soulmachine.gitbooks.io/system-design/content/cn/bigdata/frequency-estimation.html
- count-min sketch: https://zhuanlan.zhihu.com/p/369981005
- count-min sketch: https://www.youtube.com/watch?v=ibxXO-b14j4&t=7s
- heavy hitters (top-k): https://blog.csdn.net/nazeniwaresakini/article/details/109113529
- filtered space saving: https://www.inesc-id.pt/ficheiros/publicacoes/6364.pdf
- hyperLogLog: https://zhuanlan.zhihu.com/p/58519480
- T-Digest: https://bbs.huaweicloud.com/blogs/259254