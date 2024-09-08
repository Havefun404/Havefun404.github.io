---
title: faiss向量检索
date: 2024-09-07 17:18:17
tags:
---
faiss向量检索
<!--more-->
# Faiss介绍
> 为什么使用faiss？
- Faiss提供相似度搜索和聚类算法，可以用于大规模的向量检索库，传统的向量相似度计算效率低下、不适合大规模检索以及不太好支持gpu
> 原理是什么？
- 实现了多种索引结构，利用ANN(**近似最近邻搜索**)提高搜索效率；可以使用向量量化技术。将高维向量映射到紧凑表示；对高维向量进行聚类操作；还有各种高效的搜索算法。
- 采用的相似度计算方法主要是两种：欧式距离和点积
- 关键技术包括
	- OpenMP ，并行计算中的理论
	- 堆排序，一种高效的排序算法，Faiss中用在TopK个近邻的获取上
	- PQ算法，一种矢量量化方法
	- 倒排索引
	- Kmeans
	- PCA，主成分分析
> 如何使用？
- 构建训练数据集，通常是embedding的向量表示
- 选择合适的index，将训练数据add到index当中
- 使用query进行向量检索，得到最终结果

# Faiss索引类型
Faiss索引类型，会分为暴力精准检索和近似近邻索引，下图是官方的分类表：
[官方链接](https://github.com/facebookresearch/faiss/wiki/Faiss-indexes)
![[Pasted image 20240908222154.png]]
在选择index类型的时候可以按照以下思路来思考选择：
- 数据量有多大？比较小的话可以选择"Flat" index，如果你的数据没有办法全部放入RAM，那么可以构建一个个小的index，然后再把搜索结构拼接起来([参考](https://github.com/facebookresearch/faiss/wiki/Brute-force-search-without-an-index#combining-the-results-from-several-searches))
- 是否需要精确检索呢？
	- 需要则选择Flat index(IndexFlatL2)
- 是否需要考虑内存问题？
	- 不考虑：选择HNSW
	- 考虑：可以选择FPQ

# 常用索引方式实现
### 暴力 IndexFlatL2
> 暴力索引，会返回全局最优，不需要训练，遍历全部数据集，返回准确的相似向量。适用于数据量在十万以内的检索，可以秒级返回
- 创建索引
```
d = vector.shape   # 向量dims
index = faiss.IndexFlatL2(d)   # build the index
print(index.is_trained)   # 不会训练
```
- 添加数据集
```
index.add(xb)                  # add vectors to the index
print(index.ntotal)
```
- 寻找相似向量
```
k = 4                          # we want to see 4 nearest neighbors
D, I = index.search(xq, k)     # actual search
print(I[:5])                   # neighbors of the 5 first queries
print(D[-5:])                  # neighbors of the 5 last queries
```

### 更快速度IndexIVFFlat
> IndexIVFFlat使用倒排，先聚类再搜索，可以加快检索速度，先将xb中的数据进行聚类，nlist: 聚类的数目；nprobe: 在多少个聚类中进行搜索，默认为1, nprobe越大，结果越精确，但是速度越慢；在内存允许的条件下，适用于亿级别的向量检索
- 创建索引
```
nlist = 100                       #聚类中心的个数
k = 4
quantizer = faiss.IndexFlatL2(d)  # the other index
index = faiss.IndexIVFFlat(quantizer, d, nlist, faiss.METRIC_L2)
   # here we specify METRIC_L2, by default it performs inner-product search, another one is METRIC_INNER_PRODUCT
```
其中quantizer是定义的另一个index，用于查询query与哪个簇心比较
- 训练
```
index.train(xb)
assert index.is_trained # 需要训练

index.add(xb)
```
- 寻找相似向量
```
D, I = index.search(xq, k)     # actual search
print(I[-5:])                  # neighbors of the 5 last queries
index.nprobe = 10              # default nprobe is 1, try a few more
D, I = index.search(xq, k)
print(I[-5:])                  # neighbors of the 5 last queries
```
nprobe是查找多少个簇心，默认是1个

### 内存需要IndexIVFPQ
> 索引IndexFlatL2和IndexIVFFlat都会全量存储所有的向量在内存中；IndexIVFPQ 基于Product Quantizer(乘积量化)
- 创建索引
	- m：乘积量化，将原来的维度分成多少份，d必须是m的整数倍
	- bits：每个子向量用多少个bits表示，默认是8
	- 定义的维度为d = 64，向量的数据类型为float32。这里压缩成了8个字节。所以压缩比率为 (64*32/8) / 8 = 32

```
nlist = 100
m = 8                             # number of bytes per vector
k = 4
quantizer = faiss.IndexFlatL2(d)  # this remains the same
index = faiss.IndexIVFPQ(quantizer, d, nlist, m, 8)
									# 8 specifies that each sub-vector is encoded a
```
- 训练，同上
- 寻找相似向量，同上

# GPU使用
> 有些index支持gpu使用

- 单gpu
```
res = faiss.StandardGpuResources()  # use a single GPU, 这个命令需要安装Faiss GPU 版本
# build a flat (CPU) index
index_flat = faiss.IndexFlatL2(d)
# make it into a gpu index
gpu_index_flat = faiss.index_cpu_to_gpu(res, 0, index_flat)
gpu_index_flat.add(xb)         # add vectors to the index
print(gpu_index_flat.ntotal)

k = 4                          # we want to see 4 nearest neighbors
D, I = gpu_index_flat.search(xq, k)  # actual search
print(I[:5])                   # neighbors of the 5 first queries
print(I[-5:])                  # neighbors of the 5 last queries
```
- 多gpu
```
ngpus = faiss.get_num_gpus()
print("number of GPUs:", ngpus)
cpu_index = faiss.IndexFlatL2(d)
gpu_index = faiss.index_cpu_to_all_gpus(cpu_index)   # build the index
gpu_index.add(xb)              # add vectors to the index
print(gpu_index.ntotal)
k = 4                          # we want to see 4 nearest neighbors
D, I = gpu_index.search(xq, k) # actual search
print(I[:5])                   # neighbors of the 5 first queries
print(I[-5:])                  # neighbors of the 5 last queries
```

相关参考：
- https://github.com/facebookresearch/faiss/wiki
- https://github.com/daiyizheng/NLP-Interview-Notes/blob/main/NLPinterview/QA/Faiss/readme.md
- https://www.cnblogs.com/yhzhou/p/10569311.html
