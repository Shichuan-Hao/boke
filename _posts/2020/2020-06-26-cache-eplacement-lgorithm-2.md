---
title: 缓存置换算法
author:
  name: superhsc
  link: https://github.com/happymaya
date: 2020-06-26 22:33:35 +0800
categories: [计算机基础, 操作系统]
tags: [缓存置换算法]
math: true
mermaid: true
---

# 在 TLB 多路组相联缓存设计中（比如 8-way），如何实现 LRU 缓存

TLB 是 CPU 的一个“零件”，在 TLB 的设计当中不可能再去内存中创建数据结构。因此在 8 路组相联缓存设计中，每次只需要从 8 个缓存条目中选择 Least Recently Used 缓存。

## 增加累计值

用硬件同时比较 8 个缓存中记录的缓存使用次数。这种方案需要做到 2 点：

1. 缓存条目中需要额外的空间记录条目的使用次数（累计位）。类似在页表设计中讨论的基于计时器的读位操作——每过一段时间就自动将读位累计到一个累计位上；
2. 硬件能够实现一个快速查询最小值的算法。

这种方案会产生额外空间开销，还需要定时器配合，成本较高。 注意缓存是很贵的，对于缓存空间利用自然能省则省。而第 2 种方法也需要额外的硬件设计。那么，有没有更好的方案呢？

## 1bit 模拟 LRU

一个更好的方案就是模拟 LRU，可以考虑继续采用上面的方式，但是每个缓存条目只拿出一个 LRU 位（bit）来描述缓存近期有没有被使用过。 缓存置换时只是查找 LRU 位等于 0 的条目置换。

还有一个基于这种设计更好的方案，可以考虑在所有 LRU 位都被置 1 的时候，清除 8 个条目中的 LRU 位（置零），这样可以节省一个计时器。 相当于发生内存操作，LRU 位置 1；8 个位置都被使用，LRU 都置 0。

## 搜索树模拟 LRU

另一个一个巧妙的方法——用搜索树模拟 LRU。

对于一个 8 路组相联缓存，这个方法需要 $8 -1 = 7bit$ 去构造一个树。如下图所示：

![This is a image](https://maxpixelton.github.io/images/assert/os/thinking-2601.png)

8 个缓存条目用 7 个节点控制，每个节点是 1 位。0 代表节点指向左边，1 代表节点指向右边。

初始化的时候，所有节点都指向左边，如下图所示：

![This is a image](https://maxpixelton.github.io/images/assert/os/thinking-2602.png)

接下来每次写入，会从根节点开始寻找，顺着箭头方向（0 向左，1 向右），找到下一个更新方向。比如现在图中下一个要更新的位置是 0。更新完成后，所有路径上的节点箭头都会反转，也就是 0 变成 1，1 变成 0。

![This is a image](https://maxpixelton.github.io/images/assert/os/thinking-2603.png)

上图是read a后的结果，之前路径上所有的箭头都被反转，现在看到下一个位置是 4，我用橘黄色进行了标记。

![This is a image](https://maxpixelton.github.io/images/assert/os/thinking-2604.png)

上图是发生操作read b之后的结果，现在橘黄色可以更新的位置是 2。

![This is a image](https://maxpixelton.github.io/images/assert/os/thinking-2605.png)

上图是读取 c 后的情况。后面我不一一绘出，假设后面的读取顺序是 d,e,f,g,h，那么缓存会变成如下图所示的结果：

![This is a image](https://maxpixelton.github.io/images/assert/os/thinking-2606.png)

这个时候用户如果读取了已经存在的值，比如说c，那么指向c那路箭头会被翻转，下图是read c的结果：

![This is a image](https://maxpixelton.github.io/images/assert/os/thinking-2607.png)

这个结果并没有改变下一个更新的位置，但是翻转了指向 c 的路径。 如果要读取 x，那么这个时候就会覆盖橘黄色的位置。

因此，本质上这种树状的方式，其实是在构造一种先入先出的顺序。任何一个节点箭头指向的子节点，应该被先淘汰（最早被使用）。

这是一个我个人觉得非常棒的设计，因为如果在这个地方构造一个队列，然后每次都把命中的元素的当前位置移动到队列尾部。就至少需要构造一个链表，而链表的每个节点都至少要有当前的值和 next 指针，这就需要创建复杂的数据结构。在内存中创建复杂的数据结构轻而易举，但是在 CPU 中就非常困难。 所以这种基于 bit-tree，就轻松地解决了这个问题。当然，这是一个模拟 LRU 的情况，你还是可以构造出违反 LRU 缓存的顺序。