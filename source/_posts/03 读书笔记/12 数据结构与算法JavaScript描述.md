---
title: 《数据结构与算法JavaScript描述》读书笔记
top: false
date: 2020-12-31 11:07:45
updated: 2020-12-31 11:07:50
tags:
- 算法
- 数据结构
categories: 读书笔记
---

[《数据结构与算法JavaScript描述》](https://book.douban.com/subject/25945449/)读书笔记。

总的来说，书的质量不太高，如果作为前端工程师算法入门的书的话，不如去读《算法图解》，相比之下更容易理解，讲解也更加准确。这本JavaScript描述只是用了JavaScript去实现一些基础算法，而且实现的也不是很令人满意。

豆瓣评分：6.5 个人评分：☆☆☆

<!-- more -->

# 第1章 JavaScript的编程环境与模型

略

# 第2章 数组

略

# 第3章 列表

我的疑惑是，在实现列表的`remove`方法时，使用了数组的`splice`方法，那么为什么列表的其他方法不可以直接使用数组的方法来实现呢？

实际上列表完全可以用元素时对象的数组来代替。

# 第4章 栈

## 栈的实现

> 对于栈，我同样有上一章的疑问，入栈和出栈能够使用数组的`push`和`pop`方法来实现，为什么我们不可以直接使用呢？或者说，什么情况下可以使用数组的方法、什么时候不可以使用数组的方法来实现对应的数据结构呢？

栈的元素只能通过列表的一端访问，这一端成为栈顶。一摞盘子就是一个典型的栈，只能从最上面取盘子，盘子洗净后也只能放置在一摞盘子的最上面。

栈是一种先入后出（LIFO，last-in-first-out）的数据结构

栈包含的属性和方法有：

- `length`，栈内存储了多少元素
- `push`，入栈
- `pop`，出栈
- `peak`，返回栈顶元素
- `clear`，清空站内元素

栈的底层数据结构可以使用数组实现，其中声明了一个内部变量`top`，记录栈顶位置，当栈内没有元素是，`top`为零

栈的完整实现：

```JS
function Stack() {
  this.data = [];
  this.top = 0;
}

Stack.prototype.push = function (val) {
  this.data[this.top++] = val;
};

Stack.prototype.pop = function () {
  return this.data[this.top <= 0 ? 0 : --this.top];
};

Stack.prototype.peak = function () {
  return this.data[this.top - 1];
};

Stack.prototype.length = function () {
  return this.top;
};

Stack.prototype.clear = function () {
  this.top = 0;
  this.data = [];
};
```

相比于书中的实现，我进行了三点改动

1. 方法由实例方法改为了原型方法
2. 在`pop`方法时判断了`top`的正负，因为`top`不能为负数
3. 在`clear`时，同时将数据结构清空了

## 利用栈实现数制间的相互转换

利用栈将一个数字从一种数制转换为另一种，算法如下：

1. 最高位为`n%b`，将此位压入栈中
2. 使用`n/b`代替`n`
3. 重复步骤1和2，直到`n`等于`0`，且没有余数
4. 将栈内元素逐个弹出，直到栈为空

> 只适用于基数为2-9的情况


```JS
function mulBase(num, base) {
  let temp = num;
  const s = new Stack();
  let result = '';
  while (temp > 0) {
    s.push(temp % base);
    temp = Math.floor(temp / base);
  }
  while (s.length()) {
    result += s.pop();
  }
  return result;
}

console.log(mulBase(32, 2)); // 100000
```

## 判断回文字符串

回文指的是从前到后书写与从后到前书写都相同的单词、短语和数字，可以利用栈来判断回文字符串，方法就是将一个字符串按顺序压入栈中，然后再依次弹出，这样得到的字符串时一个与原来的字符串顺序相反的字符串，判断新字符串与旧字符串是否相同即可


```JS
function isPalindrome(word) {
  let s = new Stack();
  for (let i = 0; i < word.length; i++) {
    s.push(word[i]);
  }

  let newWord = '';
  while (s.length() > 0) {
    newWord += s.pop();
  }

  return newWord === word;
}

console.log(isPalindrome('hello')); // false
console.log(isPalindrome('racecar')); // true
```

# 第5章 队列

队列是一种先进先出（First-In-First-Out，FIFO）的数据结构，用在很多地方例如提交操作系统执行的一系列进程等。

可以使用数组模拟队列。

# 第6章 链表

## 数组的缺点

其他编程语言中的数组长度是固定的，当数组被数据填满后，要再加入新的元素会很困难。另外加入、删除元素，都需要移动其他元素，很麻烦。

JavaScript中的数组不存在上述问题，因为JavaScript中的数组实际上被实现成了对象，但是与其他语言的数组相比效率很低。

如果发现在使用时数组很慢，可以考虑使用链表来代替，除了对数据的随机访问外，链表几乎可以用在任何可以使用一维数组的情况。

## 定义链表

链表是一组节点组成的集合，每个节点都使用一个对象的引用指向它的后继。指向另一个节点的引用叫做链

数组元素靠位置进行引用，链表元素靠相互间的关系进行引用，下图是一个有头节点的链表

![](http://image.oldzhou.cn/FoweXZiUB7W3tSYN-ll_KdJ3DyUl)

## 设计一个基于对象的链表

包含两个类：

1. Node类表示节点
2. LinkedList类提供插入节点、删除节点、显示列表元素的方法，以及一些辅助方法


```JS
function Node(element) {
  this.element = element;
  this.next = null;
}

function LList() {
  this.head = new Node('head');
}

LList.prototype.insert = function (newElement, item) {
  const currNode = this.find(item);
  const newNode = new Node(newElement);
  newNode.next = currNode.next;
  currNode.next = newNode;
};

LList.prototype.find = function (item) {
  let currNode = this.head;
  while (currNode.element !== item) {
    if (currNode.next === null) {
      return currNode;
    }
    currNode = currNode.next;
  }
  return currNode;
};

LList.prototype.remove = function () {};

LList.prototype.display = function () {
  let currNode = this.head;
  while (currNode.next) {
    console.log(currNode.next.element);
    currNode = currNode.next;
  }
};

const cities = new LList();
cities.insert('Beijing', 'head');
cities.insert('Tianjin', 'Beijing');
cities.insert('Shanghai', 'Tianjin');
cities.insert('Guangzhou', 'Beijing');

cities.display();

// Beijing
// Guangzhou
// Tianjin
// Shanghai
```

实现`find`时候稍有点不一样，判断了找不到的当前节点的情况，如果找不到的话就插入到最后

实现`remove`方法时，需要先实现`findPrevious`方法：

```JS
LList.prototype.findPrevious = function (item) {
  let currNode = this.head;
  while (currNode.next !== null && currNode.next.element !== item) {
    currNode = currNode.next;
  }
  return currNode;
};

LList.prototype.remove = function (item) {
  const prevNode = this.findPrevious(item);
  if (prevNode && prevNode.next) {
    prevNode.next = prevNode.next.next;
  }
};

const cities = new LList();
cities.insert('Beijing', 'head');
cities.insert('Tianjin', 'Beijing');
cities.insert('Shanghai', 'Tianjin');
cities.insert('Guangzhou', 'Beijing');

cities.remove('Guangzhou');

cities.display();
// Beijing
// Tianjin
// Shanghai
```

## 双向链表

普通链表从后向前遍历很麻烦，可以为`Node`类添加一个属性，该属性存储指向前驱节点的链接，这样反向遍历就容易多了

![](http://image.oldzhou.cn/FilogJ26OosyBtuZZlj0l3xhSpcw)

```JS
function Node(element) {
  this.element = element;
  this.next = null;
  this.previous = null;
}
```

在`insert`时为新添加的节点的`previous`属性设置值

## 循环链表

将链表的尾节点指向头结点，就形成了一个循环链表

# 第7章 字典

字典是一种以键值对形式存储数据的结构，JavaScript的`Object`就是以字典的形式来设计，定义一个`Dictonary`会更方便

# 第8章 散列表

散列表也叫做哈希表，思想主要是基于数组支持按照下标随机访问数据时间复杂度为`O(1)`的特性，可以认为是数组的扩展。

例如为了记录学生信息，要求可以按照学号（学号格式为入学时间+年级+专业+专业内自增号）快速找到某个学生的信息，这个时候可以取学号的自增序号部分，即后四位作为数组的索引下标，把学生相应的信息存储到对应的空间内

![](http://image.oldzhou.cn/Fh9EYeVUyJukAhkU1cBw9ZwJADk-)

当按照键值（学号）查找时，只需要再次计算出索引下标，取出相应数据即可。

## 散列函数

上面例子中截取学号后四位的函数就是一个简单的散列函数。散列函数的作用就是将`key`值映射成为数组的下标，散列函数的设计方法有很多，比如直接寻址法、数字分析法、随机数法等，但是再优秀的设计也无法避免散列冲突

## 散列冲突

![](http://image.oldzhou.cn/FgLNcD6ZSAl_RqgXWKMdIOrB5PnR)

不同的原始输入，有可能通过散列函数计算出的`key`相同，这就是散列冲突。另外如果`key`长度为`100`，而数组的索引数量只有`50`，那么散列冲突是不可避免的。解决散列冲突办法有很多：

（1）开放寻址法

开放寻址法的核心思想是，如果出现了散列冲突，就重新探测一个新的空闲位置，例如线性探测法，如果当前位置被占用，那么从当前位置开始向后寻找空闲位置

![](http://image.oldzhou.cn/Fuo34-39YqCYmfvE67BG9PIn5Mh1)

（2）链表法（拉链法）

如果遇到冲突，在原地址新建一个空间，然后以链表节点的形式插入到该空间

![](http://image.oldzhou.cn/FjbRwPe71RRpDKZDgCyVNO3Rho9o)

## 负载因子

可以使用负载因子来衡量散列表的健康状况，负载因子就是填入表中元素个数与散列表长度的的长度。负载因子过大会导致散列表性能下降，负载因子过小会导致内存不能合理利用，造成内存浪费

#  第9章 集合

集合有两个重要特性：

1. 集合中的成员是无序的
2. 集合中不允许相同成员存在

对集合的基本操作有：

- 并集
- 交集
- 补集

# 第10章 二叉树和二叉树查找

树是一种非线性的数据结构，常用来存储具有层级关系的数据，比如文件系统中的文件。二叉树是一种特殊的树，在二叉树上进行查找非常快（在链表上查找则不是这样），为二叉树添加或删除元素也非常快（对数组执行添加或删除操作则不是这样）

树的术语：

- 一棵树最上面的节点成为根节点
- 如果一个节点下面连接多个节点，称为父节点
- 父节点下面的节点称为子节点，一个节点可以有0个、1个或多个子节点
- 没有任何子节点的节点称为叶子节点

## 二叉树和二叉查找树

二叉树的子节点个数不超过两个，二叉查找树是以一种特殊的二叉树，相对较小的值保存在左节点，较大的值保存在右节点。这个特性使得查找的效率很高

### 创建二叉查找树

首先需要声明一个`Node`类，及保存了数据，又保存了和其他节点的链接（`left`和`right`）

```JS
function Node(data, left, right) {
  this.data = data;
  this.left = left;
  this.right = right;
}

Node.prototype.show = function show() {
  return this.data;
};
```

然后创建二叉树的类，它只包含一个数据成员，一个表示二叉查找树根节点的Node对象，初始化时根节点为`null`，以此创建空节点。然后实现插入数据的`insert`方法

```JS
function BAS() {
  this.root = null;
}

BAS.prototype.insert = function insert(data) {
  // 数据以 Node 对象的形式插入
  const n = new Node(data, null, null);

  // 如果根节点为 null，那么这是一颗新树
  if (this.root === null) {
    this.root = n;
  } else {
    // 声明当前节点，并保存父节点
    let parent = (current = this.root);

    // 开始遍历，不断更新当期节点和父节点
    while (true) {
      parent = current;

      if (data < current.data) {
        current = current.left;
        // 如果节点为 null，说明可以插入数据了，否则需要继续向下遍历
        if (current === null) {
          parent.left = n;
          break;
        }
      } else {
        current = current.right;
        if (current === null) {
          parent.right = n;
          break;
        }
      }
    }
  }
};
```

### 遍历二叉查找树

有三种遍历BST的方式：中序、前序和后序:

- 前序遍历（dlr）：根-左-右
- 中序遍历（ldr）：左-根-右
- 后序遍历（lrd）：左-右-根

先实现中序遍历：

```JS
BST.prototype.inOrder = function inOrder(node, result = []) {
  if (node) {
    inOrder(node.left, result);
    result.push(node.show());
    inOrder(node.right, result);
  }
  return result;
};

const tree = new BST();
tree.insert(23);
tree.insert(45);
tree.insert(16);
tree.insert(37);
tree.insert(3);
tree.insert(99);
tree.insert(22);

console.log(tree.inOrder(tree.root));
// [3, 16, 22, 23, 37, 45, 99]
```

前序遍历：

```JS
function preOrder(node, result = []) {
  if (node) {
    result.push(node.show());
    preOrder(node.left, result);
    preOrder(node.right, result);
  }
  return result;
};
```

后序遍历：

```JS
function postOrder(node, result = []) {
  if (node) {
    postOrder(node.left, result);
    postOrder(node.right, result);
    result.push(node.show());
  }
  return result;
};
```

### 在二叉查找树上进行查找

有三种类型的查找：

- 查找给定值
- 查找最小值
- 查找最大值

#### 查找最小值和最大值

查找最小值只需要遍历左子树，直到找到最后一个节点

```JS
BST.prototype.getMin = function getMin() {
  let current = this.root;
  while (current.left) {
    current = current.left;
  }
  return current.data;
};
```

查找最大值，只需要遍历右子树，找到最后一个节点

```JS
BST.prototype.getMax = function getMin() {
  let current = this.root;
  while (current.right) {
    current = current.right;
  }
  return current.data;
};
```

#### 查找给定值

查找给定值需要比较改值与当前节点上值的大小，来确定向左还是向右进行遍历：

```JS
BST.prototype.find = function find(data) {
  let current = this.root;
  while (current) {
    if (current.data === data) {
      return current;
    }
    current = current.data < data ? current.right : current.left;
  }
  return current;
};
```

### 从二叉查找树上删除节点

删除节点需要分为三种情况考虑：

（1）待删除的节点是叶子节点，那么只需要将父节点指向它的链接指向`null`，例如下图删除节点9时

![](http://image.oldzhou.cn/FhYhpB0SbHX9OAKuZa0IdLbJ85QI)

（2）待删除节点只包含一个子节点，那么需要将原本指向它的节点，改为指向它的子节点，例如下图删除节点6时

![](http://image.oldzhou.cn/FiAvbSr3Oqr3ZFvK-_XKKM2T8jG_)

（3）待删除的节点包含两个子节点时，例如杉树下图的节点5时：

![](http://image.oldzhou.cn/FvC56JEr2VqV-Eub08jita5LXKkA)

为了保证删除后的二叉树仍然是『二叉树』，即遵循二叉树每个节点最多有两个子节点、且左节点相对较小的规则，需要找到一个节点的值的大小介于3和7之间，然后由找到的节点代替被删除的节点5

实现方法有两种：查找待删除节点的左子树的最大值，或者查找右子树上的最小值，如果采用前者，那么会将节点4找到，用它来替换被删除的节点5

![](http://image.oldzhou.cn/FsaW8OwGAhY8P5rNOYwG-YIVp98J)

```JS
function getSmallest(node) {
  if (node.left == null) {
    return node;
  }
  return getSmallest(node.left);
}

BST.prototype.removeNode = function removeNode(node, data) {
  if (node === null) {
    return null;
  }
  if (data === node.data) {
    // 待删除节点没有子节点
    if (node.left === null && node.right === null) {
      return null;
    }
    // 没有左子节点的节点
    if (node.left === null) {
      return node.right;
    }
    // 没有右子节点的节点
    if (node.right === null) {
      return node.left;
    }
    // 有两个子节点的节点
    // 找到右子树的最小节点
    const tempNode = getSmallest(node.right);
    // 用 tempNode 替换待删除的 Node
    node.data = tempNode.data;
    // 把用来替换待删除的节点，在原右节点树中删除
    node.right = removeNode(node.right, tempNode.data);
    return node;
  } else if (data < node.data) {
    node.left = removeNode(node.left, data);
  } else {
    node.right = removeNode(node.right, data);
  }
};
```

# 第11章 图和图算法

## 图的定义

图由边的几何及顶点的集合组成，分为有向图和无向图

## 图类

### 表示顶点

需要创建一个`Vertex`类来保存顶点，这个类有两个数据成员，一用于标识顶点，另一个是表明这个顶点是否被访问过的布尔值

```JS
function Vertex(label, wasVisited) {
  this.label = label;
  this.wasVisited = wasVisited;
}
```

### 表示边

表示图的边的方法称为邻接表或者邻接表数组，这种方法将边存储为由顶点的相邻顶点构成的数组，并以此顶点作为索引。这样在程序中引用一个顶点时，可以高效的访问由这个顶点相连的所有顶点的列表

![](http://image.oldzhou.cn/Fkw_YnsLG0hz0mG_CDfkgpZqWaek)

### 构建图

```JS
function Graph(v) {
  this.vertics = v;
  this.edgs = 0;
  this.adj = [];
  for (let i = 0; i < this.vertics; i++) {
    this.adj[i] = [];
  }
}

Graph.prototype.addEdge = function addEdge(v, w) {
  this.adj[v].push(w);
  this.adj[w].push(v);
  this.edgs++;
};
```

## 搜索图

### 深度优先搜索

深度优先搜索包括从一条路劲给的起始顶点开始追溯，直到到达最后一个顶点，然后回溯，继续追溯下一条路径，直到到达最后的顶点，如此往复，直到没有路径为止。

这不是在搜索特定的路径，而是通过搜索来查看图中有哪些路径可以选择。

![](http://image.oldzhou.cn/FtXdkvwIPGh27so1juyOhGSUghTC)

深度优先搜索的原理是，访问一个没有访问过的顶点，将它标记为已访问，再递归的去访问在初始顶点的邻接表中其他没有访问过的顶点

在`Graph`类中添加一个数组，用来存储已访问过的顶点，将其元素全部初始化为`false`

```JS
function Graph(v) {
  this.vertics = v;
  this.edgs = 0;
  this.adj = [];
  for (let i = 0; i < this.vertics; i++) {
    this.adj[i] = [];
  }
  this.marked = [];
  for (let i = 0; i < this.vertics; i++) {
    this.marked[i] = false;
  }
}

Graph.prototype.addEdge = function addEdge(v, w) {
  this.adj[v].push(w);
  this.adj[w].push(v);
  this.edgs++;
};

Graph.prototype.dfs = function dfs(v, result = []) {
  this.marked[v] = true;
  if (this.adj[v] !== undefined) {
    result.push(v);
  }
  for (let i = 0; i < this.adj[v].length; i++) {
    if (!this.marked[this.adj[v][i]]) {
      this.dfs(this.adj[v][i], result);
    }
  }
  return result;
};

const g = new Graph(5);
g.addEdge(0, 1);
g.addEdge(0, 2);
g.addEdge(1, 3);
g.addEdge(2, 4);

console.log(g.dfs(0));
// [0, 1, 3, 2, 4]
```

### 广度优先搜索

广度优先搜索从第一个顶点开始，尝试访问尽可能靠近它的顶点，这种搜索是逐层移动的

![](http://image.oldzhou.cn/FpWzUt4bST2YR3D-TJPw0Ki2B4gI)

思路：

1. 查找与当前顶点相邻的未访问顶点
2. 从图中取出下一个顶点`v`，添加到已访问的顶点列表
3. 将所有与`v`相连的的未访问顶点添加到队列


```JS
Graph.prototype.bfs = function bfs(s) {
  let queue = [];
  let result = [];
  this.marked[s] = true;
  queue.push(s);
  while (queue.length > 0) {
    const v = queue.shift();
    if (this.adj[v] !== undefined) {
      result.push(v);
    }
    for (let i = 0; i < this.adj[v].length; i++) {
      if (!this.marked[this.adj[v][i]]) {
        this.marked[this.adj[v][i]] = true;
        queue.push(this.adj[v][i]);
      }
    }
  }
  return result;
};

const g = new Graph(5);
g.addEdge(0, 1);
g.addEdge(0, 2);
g.addEdge(1, 3);
g.addEdge(2, 4);

console.log(g.bfs(0));
// [0, 1, 2, 3, 4]
```

## 查找最短路径

查找最短路径可以按照广度优先搜索的思路来实现，在广度优先搜索时，深度每增加一层，都会比较是否达到了目的地，如果到了的话，当前路径就是的最短路径。

具体实现我没有使用书中的方法，而是使用了之前《算法图解》中的思路，在存入队列时，没有直接将顶点存进去，而是存进去了一个对象，其中增加的属性就是当前节点的路径

![](http://image.oldzhou.cn/FtCwlDaKZB6NXnWezrIeGcyD0T73)

```JS
Graph.prototype.bfs = function bfs(s, e) {
  let queue = [];

  this.marked[s] = true;
  queue.push({node: s, path: [s]});

  while (queue.length > 0) {
    const {node, path} = queue.shift();
    for (let i = 0; i < this.adj[node].length; i++) {
      const newPath = [...path, this.adj[node][i]];
      if (this.adj[node][i] === e) {
        return newPath;
      }
      if (!this.marked[this.adj[node][i]]) {
        this.marked[this.adj[node][i]] = true;
        queue.push({node: this.adj[node][i], path: newPath});
      }
    }
  }
  return [];
};

const g = new Graph(7);
g.addEdge(0, 1);
g.addEdge(0, 2);
g.addEdge(0, 3);
g.addEdge(1, 4);
g.addEdge(4, 5);
g.addEdge(2, 5);
g.addEdge(3, 6);
g.addEdge(6, 4);

console.log(g.bfs(0, 5)); // [0, 2, 5]
```

# 第12章 排序算法

## 基本排序算法

### 冒泡排序

```JS
function bubbleSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length - i; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
      }
    }
  }
  return arr;
}
```

### 选择排序

```JS
function chooseSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    let minIndex = i;
    for (let j = i; j < arr.length; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
  }
  return arr;
}
```

### 插入排序

```JS
function insertSort(arr) {
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j >= 1; j--) {
      if (arr[j] < arr[j - 1]) {
        [arr[j], arr[j - 1]] = [arr[j - 1], arr[j]];
      }
    }
  }
  return arr;
}
```

## 高级排序算法

### 希尔排序

希尔排序在插入排序的基础上做了很大的改善，会首先比较校验的元素，而非相邻的元素。这样可以使离正确位置很远的元素更快地回到合适的位置。当开始用这个算法遍历数据集时，所有元素之间的距离会不断减小，直到处理到数据集的末尾，这时算法比较的就是相邻元素了。

### 归并排序

归并排序的原理：把一系列排好序的子序列合并成为一个大的完整的有序序列

```JS
function mergeSort(arr, start = 0, end = arr.length - 1) {
  if (start > end) {
    return [];
  }

  if (start === end) {
    return [arr[start]];
  }

  const middleIndex = Math.floor((start + end) / 2);
  const left = mergeSort(arr, start, middleIndex);
  const right = mergeSort(arr, middleIndex + 1, end);

  let result = [];
  while (left.length > 0 && right.length > 0) {
    result.push(left[0] < right[0] ? left.shift() : right.shift());

    if (left.length === 0) {
      result = [...result, ...right];
    } else if (right.length === 0) {
      result = [...result, ...left];
    }
  }
  return result;
}
```

### 快速排序

快速排序是处理大数据集最快的排序算法之一，它的思想是分治法，通过递归的方式将数据依次分解为包含较小元素和较大元素的不同子序列，该算法不断重复这个步骤直到所有数据都是有序的

```JS
function quickSort(arr) {
  if (arr.length <= 1) {
    return arr;
  }

  const index = Math.floor(arr.length / 2);
  const base = arr.splice(index, 1)[0];

  let left = [],
    right = [];

  for (let i = 0; i < arr.length; i++) {
    (arr[i] < base ? left : right).push(arr[i]);
  }

  return [...quickSort(left), base, ...quickSort(right)];
}
```

# 第13章 检索算法

在列表中查找数据有两种方式：顺序查找和二分查找。顺序查找适用于元素随机排列的列表，二分查找适用于元素已排序的列表。二分查找效率更高，但是需要在查找之前进行排序。

## 顺序查找

### 使用自组织数据

对于未排序的数据集来说，当查找的数据位于数据集的起始位置时，查找时最快、最成功的。通过将成功找到的元素置于数据集的起始位置，可以保证在以后的操作中该元素能被更快的查找到。

该策略背后的理论是，通过将频繁查找到的元素置于数据集的起始位置来最小化查找次数。

对数据的查找遵循『80-20原则』，『80-20原则』是指对某一数据集执行的80%查找操作都是对其中20%的数据进行查找，自组织的方式最终会把这20%数据置于数据集的起始位置。

类似这种『80-20原则』的概率被称为帕累托分布。

## 二分查找

查找数据是有序的话，二分查找比顺序查找更高效。


```JS
function binarySearch(arr, target) {
  let start = 0,
    end = arr.length - 1;

  while (start < end) {
    let middle = ~~((start + end) / 2);
    if (arr[middle] === target) {
      return middle;
    }
    if (arr[middle] > target) {
      end = middle - 1;
    } else {
      start = middle + 1;
    }
  }

  return -1;
}
```

# 第14章 高级算法

## 动态规划

使用递归解决问题虽然解决，但是效率不高。许多使用递归去解决的问题编程问题，可以重写为使用动态规划的技巧去解决。

动态规范方案通常会使用一个数组建立一张表，用于存放被分解为众多子问题的解。当算法执行完毕，最终的解将会在这个表中很明显的地方被找到。

### 动态规划实例：计算斐波那契额数列

使用普通递归实现的斐波那契额数列的执行效率很低，因为有太多的值在递归调用中被重复计算

![](http://image.oldzhou.cn/Fp7AbHEyrtrGOCT5Cqe6AKOgm3cb)

可以使用动态规划的技巧来设计一个效率更高的算法。使用动态规划设计的算法从它能解决的最简单的子问题开始，继而通过得到的解，去解决更复杂的子问题，直到整个问题都被解决。所有子问题的解通常被存储在一个数组中以便于访问

实现方法：

```JS
function fibonacci(n) {
  let temp = [];
  if (n === 0 || n === 1) {
    return 1;
  }

  temp[0] = 1;
  temp[1] = 1;

  for (let i = 2; i <= n; i++) {
    temp[i] = temp[i - 1] + temp[i - 2];
  }

  return temp[n];
}
```

实际上在计算过程中，不需要将所有的数据都缓存起来，只需要缓存最近的两个计算结果就可以

```JS
function fibonacci(n) {
  if (n === 0 || n === 1) {
    return 1;
  }

  let last = 1;
  let nextLast = 1;
  let result = 0;

  for (let i = 2; i <= n; i++) {
    result = last + nextLast;
    nextLast = last;
    last = result;
  }

  return result;
}
```

### 寻找最长公共子串

> 这本书关于这部分写的太难以理解了，还是《算法图解》中 描述的更容易理解一些，通过一个表格的形式来推导出算法

在填充表格时，没有找出计算公式的简单办法，必须通过尝试才能找出有效的公式。**有些算法并非精确的解决步骤，而是帮助你理清思路的框架**。

在计算`FISH`和`HISH`的最大公共子串时，填写出来的表格和计算公式是：

![](http://image.oldzhou.cn/FvFXMVoXNM9W8GcvZsU1Avfj7a79)

实现的代码：

```JS
const maxSubstring = (string1, string2) => {
  let result = {
    value: 0,
    string: ''
  };
  let cell = [];
  for (let i = 0; i < string1.length; i++) {
    cell[i] = [];
    for (let j = 0; j < string2.length; j++) {
      if (string1[i] === string2[j]) {
        const lastCell = cell[i - 1] && cell[i - 1][j - 1];
        cell[i][j] = {
          value: lastCell ? lastCell.value + 1 : 1,
          string: lastCell ? lastCell.string + string1[i] : string1[i]
        }
      } else {
        cell[i][j] = {value: 0, string: ''};
      }
      if (result.value < cell[i][j].value) {
        result.value = cell[i][j].value;
        result.string = cell[i][j].string;
      }
    }
  }
  return result
};
```

### 背包问题

背包问题：有不同体积、不同价格的物品，如何使用体积优先的背包，装走价值最大物品集合

```JS
const targets = [
  {value: 4, size: 3},
  {value: 5, size: 4},
  {value: 10, size: 7},
  {value: 11, size: 8},
  {value: 13, size: 9}
];

const capacity = 16
```

#### 递归解法

```JS
function knapsack(capacity, targets, index = targets.length) {
  if (capacity === 0 || index === 0) {
    return 0;
  }

  const currentTarget = targets[index - 1];

  if (currentTarget.size > capacity) {
    return knapsack(capacity, targets, index - 1);
  }

  const value1 = currentTarget.value + knapsack(capacity - currentTarget.size, targets, index - 1);
  const value2 = knapsack(capacity, targets, index - 1);
  return Math.max(value1, value2);
}
```

#### 动态规划解法

动态规划的解法效率更高，感觉还是《算法图解》关于这部分讲解的更加清楚

**动态规划先解决子问题，在逐步解决大问题**。每个动态规划算法都从一个网格开始，背包问题的网格如下：

![](http://image.oldzhou.cn/Fmr34MFG0USbDmNqmNeNu45ywTW4)

填充的过程是逐行进行的，原则是**保证当前单元格的值为此列的最大值**。如果装入当前单元格的物品后，重量有剩余，则回退一行，找到剩余重量对应的最大值。

使用的公式为：

![](http://image.oldzhou.cn/FmOxsuyNafgyhDN69YCLJ8YC1Msi)

**通过表格求解子问题，可以合并两个子问题的解来得到更大问题的解**。

```JS
const targets = [
  {value: 4, size: 3},
  {value: 5, size: 4},
  {value: 10, size: 7},
  {value: 11, size: 8},
  {value: 13, size: 9}
];

const capacity = 16;

function knapsack(capacity, targets) {
  let cell = [];

  for (let i = 0; i < targets.length; i++) {
    cell[i] = [];
    for (let j = 0; j <= capacity; j++) {
      const remainSize = j - targets[i].size;
      if (remainSize >= 0) {
        const lastRowCellValue = cell[i - 1] && cell[i - 1][j] ? cell[i - 1][j] : 0;

        const remainSizeValue = cell[i - 1] && cell[i - 1][remainSize] ? cell[i - 1][remainSize] : 0;
        const currentMaxValue = targets[i].value + remainSizeValue;

        cell[i][j] = Math.max(lastRowCellValue, currentMaxValue);
      } else {
        cell[i][j] = cell[i - 1] && cell[i - 1][j] ? cell[i - 1][j] : 0;
      }
    }
  }
  return cell[targets.length - 1][capacity];
}
```

## 贪婪算法

贪婪算法是一种比较简单的算法，它总会选择当下的最优解，而不去考虑这一次的选择会不会对未来的选择造成影响。贪婪算法可会得到次优解，但是对于很多问题来说，寻找最优解很麻烦，这么做不值得，使用贪心算法获得次优解就够了。

对于背包问题，贪婪算法显然不能获得最优解。些时只需要找到一个能够大致解决问题的算法，此时贪婪算法刚好可以派上用场，因为它们实现起来很容易，得到的结果由于正确结果相当接近。

```JS
// 贪婪算法求解背包问题近似解
const knapsackProblem = () => {
  const bagSize = 4;
  const items = [
    {name: 'sound', size: 4, value: 3000},
    {name: 'laptop', size: 3, value: 2000},
    {name: 'guitar', size: 1, value: 1500},
    {name: 'iphone', size: 1, value: 2000},
  ];

  let value = 0;
  let names = [];
  let remainSize = bagSize;

  // 从价值最大的开始装，能装多少算多少
  items.sort((a, b) => b.value - a.value).forEach(v => {
    if (v.size <= remainSize) {
      value += v.value;
      names.push(v.name);
      remainSize = remainSize - v.size;
    }
  });

  return {
    value, names
  };
};
```
