---
layout: post
title: STL里的容器们
categories: blog
tags:
- STL
- Container
description: STL提供了一般化的数据结构，减少了重复制造轮子的成本，其中容器是很重要的一部分。
---

整篇还缺乏实践的检验，都是书本上的理论，有待进一步体验。

## 共通能力和共通操作

三个核心能力：

- 都是“value语意”而不是“reference语意”。容器在进行元素添加操作的时候，是通过拷贝操作将元素的副本添加到容器中的。所以也要求容器的元素都可以被拷贝。
- 所有元素有一个次序（order）。可以利用迭代器遍历容器元素。
- 各项操作非安全。传入参数必须符合相应操作的参数要求，非法参数会造成未知错误。通常STL不会自己抛出异常。

容器的通用操作包括三个方面的操作，注意下述操作都是**容器间的**：

- 与大小相关的操作:size(), empty(), max_size()
- 比较操作：操作符==, !=, <, <=, >, >=，定义依据如下：
	- 比较的两个操作数必须为统一型别
	- 如果两个容器的所有元素依序相等，则两个容器相等，其中采用operator==检查元素是否相等
	- 采用字典式顺序比较原则来判断容器的大小关系
- 赋值和swap操作，swap的性能较优

## Vector

	namespace std {
		template <class T, class Allocator = allocator<T> >
		class vector;
	}

vector模塑了一个动态数组。要求模板类型T必须assignable和copyable。vector是一个有序群集（ordered collection），支持常数时间的随机存取。末尾添加删除元素的性能很好，但是中间插入需要移动插入点之后的所有元素。

vector是一个以空间换时间的例子，它的随机存取特性是因为**配置了比其所容纳的元素所需更多的内存**。除了共通操作中的大小函数之外，vector还有另一个函数capacity()，用来获取当前实际能够容纳的元素数量。当一旦需要的容量大于了现在所配置的容量，则需要重新配置存储，但是原vector中的reference、pointer、iterator都会失效。两个方法避免重新配置内存：

- 使用reserve()函数设置预留空间，则只要空间够设置的数量，就不会重新配置
- 在初始化的时候制定需要的空间大小，即元素个数

vector在没有指定空间大小的情况下，各个不同版本预留的空间大小也是不同的。

### 元素访问

vector在获取访问元素的时候可以使用类似数组访问的方式进行，也可以利用函数at()，或者利用front()和back()获取第一个元素和最后一个元素。另外，begin()和end()函数可以返回指向第一个元素和最后元素**下一个位置**的随机存取迭代器。

常用push_back()和pop_back()来向尾部添加和删除元素；erase()函数删除元素，返回的是最后一个有效元素的下一个位置；clear()可以移除所有元素，清空容器。

### 特殊的vector

vector<bool\>是一个经过优化的vector，其每一个元素仅仅消耗1bit的存储空间，但是要在访问的时候要注意，其通过下标和at()函数返回的都是一个reference，而且是经过proxy的（具体实现是怎样的还不清楚），所以不能想当然的觉得返回的一定是bool或者bool*。另外，相对于普通的vector，由于是通过proxy实现bit操作，所以会影响性能效率。

## Deque

	namespace std {
		template <class T, class Allocator = allocator<T> >
		class deque;
	}

deque也采用动态数组来管理元素，提供随机存取，而且接口几乎和vector一样，不同是的deque在头尾都提供快速插入删除。deque在实现的时候是以**一组独立区块**的方式实现，其中第一区朝某个方向扩展，而最后一区朝另外一个方向发展。同时，有另外一个索引数组记录着每一个区块的地址，从而可以实现随机访问。也因为这个特殊的结构，使得deque在内存重配置上比vector更为灵活，但在进行中间元素的插入删除时也更加麻烦。

相较于vector，多了相应的头部操作：

- push_front(elem) 在头部插入元素elem的副本
- pop_front() 移除头部元素

同时，deque和vector还有些不同：deque不提供容量操作capacity()和reserve()。

## List

	namespace std {
		template <class T,
					class Allocator = allocator<T> >
		class list;
	}

list是一个双向链表的实现。要求模板类型T是assignable和copyable。

由于是基于链表实现的，所以不支持随机读取，不支持下标操作，所以元素访问只有两个方式：

- front() 返回第一个元素，不检查元素存在于否
- back() 返回最后一个元素，同样不检查元素存在性

但是元素的插入和删除操作会比vector更快速，同时，也更灵活。相较于vector在插入删除操作上多了对于头部的操作（类似于deque）和元素的删除操作：

- remove(val) 移除所有值为val的元素
- remove_if(op) 移除所有使op操作为true的元素

### 特殊操作

由于链表的特殊性，所以list支持Splice（拼接）操作，上述的remove操作是一个体现，但还有一些其他的操作：

- unique() 删除重复元素，只留下一个
- splice() 将一个list的部分或全部拼接到另一个list的某个位置
- sort() 链表排序，复杂度`O(nlogn)`，排序后迭代器都保持有效
- merge() **假设两个list是有序的**，合并两个list
- reverse() 反转一个list，很有链表特色的操作

## Set

	namespace std {
		template <class T, class Compare = less<T>, 
					class Allocator = allocator<T> >
		class set;

		template <class T, class Compare = less<T>,
					class Allocator = allocator<T> >
		class multiset;
	}

set，即数据集合，但是它可以自动将数据排序，同时，multiset允许元素重复，而set不允许。其模板类别T要求是assignable，copyable和comparable。数据的顺序要遵循strict weak ordering，即离散数学中的偏序，要求满足三个条件：

- 反对称，`x<y => y<x`
- 可传递，`x<y && y<z => x<z`
- 非子反，`x<x为假`

set在实现的时候通常以平衡二叉树的结构，而实际上在具体实现中多以红黑树实现（它保证在插入元素的时候只会做最多两个重新连接的动作，同时到达某个元素的最长路径最多是其最短路径的两倍）。所以可以保证在查找时候可以达到`O(logn)`的复杂度。

set的插入删除操作和查找操作与下面的map是一样的，具体可以看map的相关。

## Map

	namespace std {
		tmplate <class Key, class T,
				class Compare = less<Key>,
				class Allocator = allocator<pair<const Key, T> > >
		class map;
		
		tmplate <class Key, class T,
				class Compare = less<Key>,
				class Allocator = allocator<pair<const Key, T> > >
		class multimap;
	}

map和multimap的序别在于后者可以有重复元素存在。其中key和value的类型必须assignable且copyable，同时key的类型还要comparable，以进行排序。

map/multimap通常也是用平衡二差树实现的（实际版本中可能多为红黑树），与set/multiset很类似，只不过set中没有key/value对，而是value既是数据又是key。另外和set类似的是，在map中想要改变key的话只能通过先删除再添加的办法实现，但是value可以直接修改。

### 插入删除操作

map的添加可以通过insert()其中包括了：

- insert(elem) 插入elem的一个副本，返回新元素的位置
- insert(pos, elem) 插入元素elem，其中pos是一个hint，指出安插操作的搜寻起点，pos设置的恰当会加快速度
- insert(beg, end) 将区间[beg, end]内所有元素的副本安插到c，无返回值

map的元素添加的时候要是pair的形式，注意，这里的key都被视作**常数**，所以要保证key的类型正确。可以通过三种形式生成：

- std::map<KEY, VALUE>::value_type(key, value)
- std::pair<key, value>
- std::make_pair(key, value)

对应地，删除操作的erase()也有三个：

- erase(elem) 删除value与elem相等的所有元素，返回被移除的元素个数
- erase(pos) 移除pos位置上的元素，无返回值
- erase(beg, end) 移除区间[beg, end]之间的所有元素，无返回值

multimap的删除操作时要注意，想要删除重复元素的第一个就需要通过find()查找，然后再通过erase(pos)删除。

### 查找操作

map的查找操作有五个：

- count(key) 返回键值等于key的元素个数
- find(key) 返回键值等于key的第一个元素，找不到就返回end()
- lower_bound() 返回“键值>=key”的第一个元素的位置
- upper_bound() 返回“键值>key”的第一个元素的位置
- equal_range() 返回“键值==key”的元素区间，返回值是一个pair，其中first和second都是迭代器

### 关联数组

另外，map可以被看作是**关联数组（Associated Array）**，可以通过类似python中的dict的方式来进行访问，比如m[key]会返回一个reference指向键值位key的元素，如果该元素不存在，则安插一个新元素。


## 参考文献

1. "The C++ Standard Library: A Tutorial and Reference"
