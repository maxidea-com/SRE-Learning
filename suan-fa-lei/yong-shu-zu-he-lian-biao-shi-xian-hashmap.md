# 用数组和链表实现HashMap

## 一、python

### 原理说明

python语言中的dict底层是基于hashmap结构实现的，hashmap可以在一堆数据中，很快的根据key，找到value，这个关键点主要是由hash函数实现的。

### 简单实现 

为解决hash冲突问题，使用了“链地址法”的结构。 MyHash内部使用items列表来存储数据，items是一个列表，并且每个元素也是一个列表，元素列表中存储了具体的（key,value）元组，不同的key根据hash函数先算出index，即存储在哪条列表中，插入时则直接append，查找时则根据equals方法将待查找的key与列表中的所有元组的第一个值（key）进行比较，找到相等的则返回元组的第二个值（value），找不到则raise KeyError异常。

```text
#!/usr/bin/python2.7
# coding=utf-8


class MyHash(object):

    def __init__(self, length=10):
        self.length = length
        self.items = [[] for i in range(self.length)]

    def hash(self, key):
        """计算该key在items哪个list中，针对不同类型的key需重新实现"""
        return key % self.length

    def equals(self, key1, key2):
        """比较两个key是否相等，针对不同类型的key需重新实现"""
        return key1 == key2

    def insert(self, key, value):
        index = self.hash(key)
        if self.items[index]:
            for item in self.items[index]:
                if self.equals(key, item[0]):
                    # 添加时若有已存在的key，则先删除再添加（更新value）
                    self.items[index].remove(item)
                    break
        self.items[index].append((key, value))
        return True

    def get(self, key):
        index = self.hash(key)
        if self.items[index]:
            for item in self.items[index]:
                if self.equals(key, item[0]):
                    return item[1]
        # 找不到key，则抛出KeyError异常
        raise KeyError

    def __setitem__(self, key, value):
        """支持以 myhash[1] = 30000 方式添加"""
        return self.insert(key, value)

    def __getitem__(self, key):
        """支持以 myhash[1] 方式读取"""    
        return self.get(key)


myhash = MyHash()
myhash[1] = 30000
myhash.insert(2, 2100)
print myhash.get(1)
myhash.insert(1, 3)
print myhash.get(2)
print myhash.get(1)
print myhash[1]
```

执行结果：

```text
30000
2100
3
3
```

##  二、Java

```text

```

