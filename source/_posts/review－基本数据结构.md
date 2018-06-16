---
title: review-基本数据结构
date: 2017-08-05 14:23:49
tags:
---

复习基础的数据结构（栈，队列，链表）。
<!--more-->

### basic data structure

* Stack

栈(Stack)，是一种支持后进先出(LIFO)的基本数据结构，可以使用数组或者链表实现；
支持的基本操作包括：`push`, `pop`, `size`, `is_empty`, `peek`;

{% codeblock lang:python %}
class Stack(object):
    def __init__(self):
        self.items = []

    def size(self):
        """返回元素数量"""
        return len(self.items)

    def is_empty(self):
        """判断当前栈中是否为空"""
        return self.size() == 0

    def push(self, item):
        """向栈中压入元素"""
        self.items.append(item)

    def pop(self):
        """从栈中弹出元素，若为空栈则抛出ValueError异常"""
        if self.is_empty():
            raise ValueError('Empty Stack')
        return self.items.pop()

    def peek(self):
        """返回栈顶元素，并不弹出，若为空栈则抛出ValueError异常"""
        if self.is_empty():
            raise ValueError('Empty Stack')
        return self.items[-1]

{% endcodeblock %}

栈结构的后入先出的特性有很多用途，如程序函数调用时就使用栈结构来保存现场，表达式句法分析时也会用到栈。

* Queue

队列(queue)，是支持先进先出(FIFO)的基本数据结构，同样可以使用数组或者链表实现；
支持的基本操作包括：`enqueue`, `dequeue`, `size`, `is_empty`;

{% codeblock lang:python %}
class Queue(object):
    def __init__(self):
        self.items = []

    def size(self):
        """返回队列中元素数量"""
        return len(self.items)

    def is_empty(self):
        """判断队列是否为空"""
        return self.size() == 0

    def enqueue(self, item):
        """向队列添加一个元素"""
        self.items.insert(0, item)

    def dequeue(self):
        """从队列返回一个元素，若为空队列则抛出ValueError异常"""
        if self.is_empty():
            raise ValueError('Empty Queue')
        return self.items.pop()
{% endcodeblock %}

队列结构有很多的衍生结构，如双端队列，优先级队列等。
队列结构可以保证元素按照入队列的顺序出队列，在实际使用中，也多用于实现有顺序要求的任务或消息流等；

* Linked List

链表(Linked List), 是使用的最广泛的数据结构。
链表，数组或Python中的列表都是线性数据结构，但数组或列表元素在内存中位置是连续的，而链表元素间是通过指针或者索引联系起来的，而且只有相邻的元素间才会有联系；
这就意味着，对于数组或者列表，只要知道了某元素的索引和起始的位置可以通过内存偏移量访问到该元素，而对于链表只能通过相邻的元素来访问，对于随机访问只能从头遍历来寻找到指定元素；

对比链表与数组，数组在索引方面更有优势，适合于读操作较多的场景；而链表在创建和删除新元素方面没有存储方面的限制，更适合大量写操作的场景；

{% codeblock lang:python %}
class Node(object):
    """链表节点元素类"""
    def __init__(self, data, next=None):
        self._data = data
        self._next = next

    def get_data(self):
        """获取节点数据"""
        return self._data

    def get_next(self):
        """获取下一个节点"""
        return self._next

    def set_data(self, val):
        """设置节点数据"""
        self._data = val

    def set_next(self, node):
        ”“”设置节点下一个指向元素"""
        self._next = node

class SingleLinkedList(object):
    """单链表"""
    def __init__(self):
        self.head = None

    def is_empty(self):
        ”“”判断是否为空"""
        return self.head == None

    def add(self, item):
        """增加新节点"""
        tmp_node = Node(item)
        tmp_node.set_next(self.head)
        self.head = tmp_node

    def size(self):
        """返回链表节点数"""
        cur = self.head
        cnt = 0
        while cur is not None:
            cur = cur.get_next()
            cnt += 1
        return cnt

    def search(self, item):
        """在链表中搜索指定数据的节点"""
        cur = self.head
        found = False
        while cur is not None:
            if cur.get_data() == item:
                return cur
            else:
                cur = cur.get_next()
        return None

    def remove(self, item):
        """删除链表中指定数据的节点"""
        cnt = 0
        prev = None
        cur = self.head
        while cur is not None:
            if cur.get_data() == item:
                if prev is None:  # head match
                    self.set_head(cur.get_next())
                else:
                    prev.set_next(cur.get_next())
                    prev = cur
                cur = cur.get_next()
                cnt += 1
            else:
                prev = cur
                cur = cur.get_next()
        return cnt
{% endcodeblock %}

链表是使用的最广泛的数据结构之一，相关的话题和衍生的结构甚至不是一两篇文章可以讲得清的。
数据结构和算法是基本功，而链表更是数据结构中最基本的部分，需要投入时间和精力进行足够地了结。
之后专门写下树和图结构，判断链表是否有环的分析；


### the end

基本的数据结构：

栈：后进先出(LIFO)，支出入栈和出栈等基本操作；
队列：先进先出(FIFO)，支持如队列和出队列等基本操作；
链表：元素间以指针或索引关联，头结点，节点元素；
