title: "vue源码学习笔记一cache.js"
date: 2016-05-24 22:49:32
categories: 技术 #文章文类
tags: [vue, js]  #文章标签，多于一项时用这种格式[]
description: LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。
keywords: vue js 缓存 LRU
---
前天给自己定了一个学习任务，看vue的源码。随便翻阅vue源码的时候，看到很多计算的地方用了LRU缓存，顿时对LRU特别好奇。之前对LRU也有一点印象，然而没有深入。这次我就好好的感受下LRU。

### LRU原理
LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”。


### LRU实现
最常见的实现是使用一个链表保存缓存数据，详细算法实现如下：
![](/imgs/1337859321_3597.png)
1. 新数据插入到链表头部；
2. 每当缓存命中（即缓存数据被访问），则将数据移到链表头部；
3. 当链表满的时候，将链表尾部的数据丢弃。

<!--more-->

### LRU编码
~~~javascript
/**
 * Created by harrywan on 2016/5/23.
 * LRU实现
 * LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高
 * 1. 新数据插入到链表头部；
 * 2. 每当缓存命中（即缓存数据被访问），则将数据移到链表头部；
 * 3. 当链表满的时候，将链表尾部的数据丢弃。
 */

let Cache = function (limit) {
    this.limit = limit;
    this._map = [];
};

let p = Cache.prototype;

/**
 * 获取缓存
 * @param key
 * @returns {*}
 */
p.get = function (key) {
    var rst;
    var len = this._map.length;
    for (var i = 0; i < len; i++) {
        if (this._map[i].key == key) {
            rst = this._map[i].value;
            //每当缓存命中，则将数据移动到链表头部
            this._map.splice(i, 1);
            this._map.unshift({
                key: key,
                value: rst
            });
            break;
        }
    }
    return rst
};

/**
 * 放入缓存
 * @param key
 * @param obj
 * @returns {*}  被删除的项
 */
p.put = function (key, obj) {
    var rst;
    // 新数据插入到链表头部
    this._map.unshift({
        key: key,
        value: obj
    });
    // 当链表满的时候，将链表尾部的数据丢弃。
    if (this.limit < this._map.length) {
        rst = this._map[this.limit];
        this._map.splice(this.limit, 1);
    }
    return rst
};

module.exports = Cache;
~~~
行云流水般写完之后越想越不对！这个尼玛查询不在缓存队列的数据的时候，要完整遍历一次数组，得多蠢。

查看vue源码，它的LRU实现用的双链表。然后重写我的LRU实现：
~~~javascript
/**
 * Created by harrywan on 2016/5/23.
 * LRU实现
 * LRU（Least recently used，最近最少使用）算法根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高
 * 1. 新数据插入到链表头部；
 * 2. 每当缓存命中（即缓存数据被访问），则将数据移到链表头部；
 * 3. 当链表满的时候，将链表尾部的数据丢弃。
 */
function Cache(limit) {
    this.size = 0;
    this.limit = limit;
    this.head = this.tail = undefined;
    this._keymap = Object.create(null);
}

var p = Cache.prototype;

p.put = function (key, value) {
    var entry = {
        key: key,
        value: value
    };
    var has = this.get(key, true);
    if (has) {
        has.value = value
    } else {
        //如果已经到达长度，就删除head，把head的下一个变成head
        if (this.size == this.limit) {
            this.shift();
        }
        if (!this.head) {
            this.head = entry;
        } else if (!this.tail) {
            entry.last = this.head;
            this.head.next = this.tail = entry;
        } else {
            entry.last = this.tail;
            this.tail = this.tail.next = entry;
        }
        this._keymap[key] = entry;
        this.size++;
    }
};

p.get = function (key, returnObj) {
    var entry = this._keymap[key];
    if (!entry) {
        return;
    }
    //如果正好是tail，则啥都不用改嘛
    if (this.tail.key == key) {
        return returnObj ? entry : entry.value;
    }
    //如果正好是head，则把head的next设为head
    if (this.head.key == key) {
        this.head = this.head.next;
        this.head.last = undefined;
    } else {
        //如果在中间，相当于把中间人挖走，让前后自己建立关系
        entry.last.next = entry.next;
        entry.next.last = entry.last;
    }
    entry.last = this.tail;
    entry.next = undefined;
    this.tail = this.tail.next = entry;
    return returnObj ? entry : entry.value;
};

p.shift = function () {
    if (this.head) {
        this._keymap[this.head.key] = undefined;
        this.head = this.head.next;
        this.head.last = undefined;
        this.size--;
    }
};

module.exports = Cache;
~~~
代码很简单，主要是思路。比如说巧妙地用双链表实现LRU，比如说哪些场景需要用cache.js。所以呀写代码得多思考。
