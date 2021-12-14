# DirectMap

1. 结构
2. 插入
3. 删除
4. 迭代
5. 线程安全
6. 堆外内存管理

## 1. 结构

![structure.png](offheapmap/structure.png)

#### 1.1 node 函数
```
int length = key.length + value.length
long addr = malloc(length)
put(addr, key, value)

long node(addr) {
    return addr << 1 | 0
}
```

#### 1.2 path 函数
```
int length = 128 * 8
long addr = malloc(length)

long path(addr) {
    return addr << 1 | 1
}
```

#### 1.3 slot 函数
```
# slot 函数决定图中root即蓝色框的index
init slots = 32
int hash = hash(key.hashcode)
int slot(hash) {
    return hash & 31
}
```

#### 1.4 index 函数
```
# index 函数决定图中path即绿色框的index
init level = 4
int hash = hash(key.hashcode)
int index(hash, level) {
    return (hash >>> ((level - 1) * 8)) & 127;
}
```

```
example:
int hash = 01101001||11010100||10110100||10010011
if level = 4 then index = 01101001 & 127
if level = 3 then index = 11010100 & 127
if level = 2 then index = 10110100 & 127
if level = 1 then index = 10010011 & 127
```

## 2. 插入

#### 2.1 插入到某一位置
![insert0.png](offheapmap/insert0.png)

#### 2.2 插入引发的层扩容

![insert1.png](offheapmap/insert1.png)

![insert2.png](offheapmap/insert2.png)

#### 2.3 层扩容引发的冲突

![insert3.png](offheapmap/insert3.png)

## 3. 删除

#### 3.1 删除某一结点

![delete0.png](offheapmap/delete0.png)

#### 3.2 删除的Path的头节点

![delete1.png](offheapmap/delete1.png)

#### 3.3 删除引发的层缩容

![delete2.png](offheapmap/delete2.png)

## 4. 迭代

#### 3.1 迭代Map

![iterator0.png](offheapmap/iterator0.png)

```
1th iterate [key1, key2, key3, key9]
2th iterate [key4, key5, key6]
3th iterate [key7]
4th iterate [key8]
```

#### 3.2 迭代Map时发生了层扩容

```
1th iterate [key1, key2, key3, key9]
insert key10
2th iterate [key4, key5, key6]
3th iterate [key7]
4th iterate [key8]
```

![iterator1.png](offheapmap/iterator1.png)

```
1th iterate [key3, key9, key2, key1]
insert key10
2th iterate [key4, key5, key6]
3th iterate [key7]
4th iterate [key8]
```

## 5. 线程安全

```java  
DirectMap<Integer, Integer> map = new DirectMap<>(32);
get(key) {
    int hash = hash(key.hashcode)
    int slot = hash & 31;
    lock[slot].readlock.lock
    // find that key's value in the search tree
    value = lookup(key)
    lock[slot].readlock.unlock
    return value
}

put(key, value) {
    int hash = hash(key.hashcode)
    int slot = hash & 31;
    lock[slot].writelock.lock
    // insert key and value to the search tree
    oldvalue = insert(key, value)
    lock[slot].writelock.unlock
    return oldvalue
}

```

## 6. 堆外内存管理
