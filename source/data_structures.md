数据结构
========

DATE: 2018-03-20

该文档涵盖了计算机基础的数据结构设计.

阅读完该文档后，您将会了解到:

* 基础的数据结构设计如 List, Queue, Stack.
* 基于基础数据结构之上的复杂结构如 Graph, Tree
* 数据结构的基本组成部分

--------------------------------------------------------------------------------

TL;DR
-----
### Java Collections
![Collection_interfaces](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/Collection_interfaces.png)

INFO: 推荐阅读[这篇文章](https://www.ntu.edu.sg/home/ehchua/programming/java/J5c_Collection.html), 了解Java的Collections框架

List
----
1. 基础操作

|     操作  |  释义    |
|    ----   | ------   |
|    get    | 获取数据 |
|    set    | 设置数据 |
|    add    | 添加数据 |
|    remove | 移除数据 |
|    size   | 获取长度 |

2. Java中List的接口关系图

    ![Collection_interfacesList](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/Collection_ListImplementation.png)

NOTE: 为什么接口中要提供一个`iterator`? 我的理解包括下面几点: iterator 要求在获取数据的时候, List没有被修改, 否则就报错(`ConcurrentModificationException`), 这样相对更安全, 更多讨论请查看StackOverflow上的[讨论](https://stackoverflow.com/a/27984817/8186609)

### ArrayList
#### ArrayList implements Iterable
在 ArrayList 里面, 需要有一个 `iterator` 方法, 这里用到了 inner class

```java
public class MyArrayList<AnyType> implements Iterable<AnyType> {
    private AnyType[] theItems;

    public java.util.Iterator<AnyType> iterator() {
        return new ArrayListIterator<AnyType>();
    }

    private static class ArrayListIterator<AnyType> implements java.util.Iterator<AnyType> {
        private int current = 0;

        public AnyType next() {
            return theItems[current++];
        }

        // ...
    }
}
```

使用 inner class 的原因是: 我们在 inner class 里面可以获取当前对象的fields. 在这里是: `theItems`

#### ensureCapacity
List和数组最大的一个特点是不用设置长度

但是ArrayList是用 Array 来实现的, 每一个数组需要在初始化的时候就将长度设置好, 那 ArrayList 是如何做到的呢?

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    private static final int DEFAULT_CAPACITY = 10;

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // 在这里判断容量不够用了, 就对容量进行增长
        if (minCapacity - elementData.length > 0) {
            grow(minCapacity);
        }
    }

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```

在上面看到了几个关键的方法

- ensureExplicitCapacity
- grow

在 grow 方法中, 可以看到关键的一行

```java
newCapacity = oldCapacity + (oldCapacity >> 1)
```

NOTE: >> 符号为[位移](http://www.cnblogs.com/hongten/p/hongten_java_yiweiyunsuangfu.html)操作, 可以看下[测试code](https://github.com/dengqinghua/my_examples/blob/master/java/src/test/java/com/dengqinghua/algorithms/QuickSortTest.java#L72)

新增的容量为原有容量的1.5倍

INFO: ArrayList能动态地增长长度, 当容量不够的时候, 会进行`grow`操作, 将所有的数据拷贝(Arrays.copyOf)到一个新的数组中, 并进行数组的扩容处理.

#### modCount
在Iterator的实现中, 有一个非常奇怪的变量: modCount, 源码中对她的解释为:

> The number of times this list has been <i>structurally modified</i>.
Structural modifications are those that change the size of the
list, or otherwise perturb it in such a fashion that iterations in
progress may yield incorrect results.

我们在源码中可以看到, 只要对该ArrayList进行任何操作, 都会修改这个值. 我的理解是 modCount 为 modifications count 的缩写, 即修改的次数. 它用在哪儿呢?

在 Iterator 中, 定义了一个 `expectedModCount`

```java
public class MyArrayList<AnyType> implements Iterable<AnyType> {
    private AnyType[] theItems;

    public java.util.Iterator<AnyType> iterator() {
        return new ArrayListIterator<AnyType>();
    }

    private static class ArrayListIterator<AnyType> implements java.util.Iterator<AnyType> {
        int expectedModCount = modCount;

        // ...
    }
}
```

expectedModCount 初始值为 modCount 的值, 如果发现 Iterator 对象在使用的时候, 发现两个值不相等, 则会抛出`ConcurrentModificationException`异常

INFO: modCount确保了在使用Iterator的过程中, 这个List没有被修改过. 在LinkedList也有相同的变量.

```java
final void checkForComodification() {
    if (modCount != expectedModCount) {
        throw new ConcurrentModificationException();
    }
}
```

#### 支持RandomAccess
她支持`RandomAccess`, 在 [二分法查找](https://github.com/dengqinghua/my_examples/blob/master/java/src/main/java/com/dengqinghua/algorithms/BinarySearch.java#L30) 的时候比 LinkedList 更加有优势

### LinkedList
![linkedList](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/linkedList.png)

#### Doubly Linked
在Java中, LinkedList是双向的, 他还实现了 [Deque](https://docs.oracle.com/javase/7/docs/api/java/util/Deque.html), double ended queue

在 数据结构中, 有一个 inner class: `Node`

```java
public class LinkedList<E> {
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
}
```

其中包含`next`和`prev`两个field

#### Sentinel Nodes
- head node
- tail node

一些使用场景如下:

1. clearNode

    当清空该 LinkedList 的时候, 只需要设置

    ```java
    head.next = tail
    ```

2. addAll

    将两个LinkedList连接, 只需要直接处理 head 和 tail 即可

3. 查找某个位置的节点数据
    在源码中可以看到, 有了size, head 和 tail node, 如果要找位置index的node, 则可以判断 index 和 size 之间的关系:

    如果 index < size / 2, 则从head开始找, 否则从tail开始找

    ```java
        Node<E> node(int index) {
          // 如果 index < size / 2, 则从head开始找, 否则从tail开始找
          if (index < (size >> 1)) {
              Node<E> x = head;
              for (int i = 0; i < index; i++)
                  x = x.next;
              return x;
          } else {
              Node<E> x = tail;
              for (int i = size - 1; i > index; i--)
                  x = x.prev;
              return x;
          }
      }
    ```

#### 不支持RandomAccess
二分查找法的时候, 总的来说平均时间比 ArrayList 要慢. 在原生的Java的二分查找的时候, 对非RandomAccess的List做了不同的处理

```java
public class Collections {
    public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c) {
        if (c==null)
            return binarySearch((List<? extends Comparable<? super T>>) list, key);

        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key, c);
        else
            return Collections.iteratorBinarySearch(list, key, c);
    }
}
```

### LinkedList和ArrayList的使用场景
见 StackOverflow: [When to use LinkedList over ArrayList?](https://stackoverflow.com/a/322742/8186609)

INFO: 所经历的项目中没有使用过LinkedList.

Stack
------
![stack](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/stack.png)

Java8中用`Vector`来实现栈的数据结构

```java
class Stack extends Vector {}
class Vector extends AbstractList implements List, RandomAccess {}
```

|  操作  |  释义    |
| ----   | ------   |
| pop    | 出栈 |
| push   | 入栈 |
| size   | 栈的高度 |
| peek   | top data |
| size   | 获取长度 |

### Vector
#### elementData
Vector使用数组来存储数据

```java
class Vector {
    protected Object[] elementData;
}
```

和 ArrayList 类似, 她也有一个初始的容量 10:

```java
class Vector {
    /**
     * Constructs an empty vector so that its internal data array
     * has size {@code 10} and its standard capacity increment is
     * zero.
     */
    public Vector() {
        this(10);
    }
}
```

支持在容量不够的时候, 自动地`grow`. 实现方式和ArrayList类似, 这里不再重复

#### synchronized
Vector 有很多方法是添加了 `synchronized` 关键词.

```
class Vector {
    public synchronized void trimToSize() {}
    public synchronized void ensureCapacity(int minCapacity) {}
}
```

#### 官方不建议使用Vector
>  As of the Java 2 platform v1.2, this class was retrofitted to
implement the {@link List} interface, making it a member of the
Java Collections Framework.  Unlike the new collection
implementations, Vector is synchronized.  If a thread-safe
implementation is not needed, it is recommended to use ArrayList
in place of Vector.

建议用 ArrayList 替换 Vector

### 一些使用栈的场景
#### Balancing Symbols
检查一些符号, 如`(\['"` 是否封闭

算法思路: 以`()`的检测举例子, 遇到 `(` 的时候, 入栈, 遇到 `)` 的出栈. 最终在所有的符号都结束之后, 看下栈里面是否有数据, 如果有, 则说明`()`是未封闭的

```
(a == 1) && (b == 2)  // 检测通过
(a == 1) && (b == 2   // 检测不通过, 栈内还存在元素: (
```

Queue
-----
![queue](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/queue.png)

|  操作  |  释义    |
| ----   | ------   |
| pop    | 出队列 |
| push   | 入队列 |
| size   | 队列长度 |

核心fields:

- theItems 数组, 记录队列的值
- front    队列头的位置
- back     队列尾的位置
- size     队列长度

### Circular Array
可以使用环状的数组来存储Queue的数据. 初始化的时候, 而 back 的位置为 `N - 1`, front 的位置为 `N - k - 1`. 其中 N 为数组的长度, k 为初始队列的数据的个数

入队列, 则从数组的头开始. 入队列和出队列就是改变 front 和 back 的过程

空Queue的条件为

```
back = front - 1
```

另外, 还需要注意Queue满的情况,此时需要考虑扩容.

### Queue的Java实现
Java中, LinkedList实现了 Deque, Deque继承自Queue

```java
class LinkedList implements Deque {}
interface Deque extends Queue {}
```

NOTE: Deque: A linear collection that supports element insertion and removal at
both ends.  The name <i>deque</i> is short for "double ended queue"
and is usually pronounced "deck".  Most {@code Deque}
implementations place no fixed limits on the number of elements
they may contain, but this interface supports capacity-restricted
deques as well as those with no fixed size limit.

Tree
----
![tree](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/tree.png)

1. 基础概念

    |   名称    | 释义                                                              |
    |  ----     | ------                                                            |
    |  node     | 节点                                                              |
    |  edge     | 连接线                                                            |
    |  path     | 节点到节点之间的路径                                              |
    |  length   | path所经过的edge的个数                                            |
    |  root     | 根节点                                                            |
    |  depth    | 深度,是指从root节点到该节点经过某一个path的length.节点J的depth为2 |
    |  hieght   | 高度,是指节点到最远的一个leave的length,节点E的hieght为2           |
    |  leaves   | 叶子节点                                                          |
    |  siblings | 兄弟节点                                                          |
    |  child    | 子树                                                              |
    |  preorderTraversal   | 前序遍历 ![pre_order_traversal](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/preorder_traversal.jpeg) |
    |  inorderTraversal    | 中序遍历 ![in_order_traversal](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/inorder_traversal.jpeg) |
    |  postorderTraversal  | 后序遍历 ![post_order_traversal](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/postorder_traversal.jpeg) |

### Binary Tree
![binary_tree](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/binary_tree.png)

#### 基本结构

- element
- leftNode
- rightNode

#### 使用场景
##### 表达式
将表达式转化为postfix形式, 再用栈创建一颗可以中序遍历的树

```
(a + b) * (c * (d + e))
```

利用栈, 转变为postfix形式:

```
a b + c d e + * *
```

利用栈, 转化为树.

NOTE: `a b + c d e + * *` 依次入栈, 遇到 a, b 创建树, 分别入栈, 遇到 `+` 将 `a, b` 出栈, `a, b, +` 组成一个新的树, 最终的树为下图所示. 最终可以通过中序遍历进行运算. 另外, 可以后序遍历恢复为 `a b + c d e + * *``

![tree_example](https://raw.githubusercontent.com/dengqinghua/roses/master/assets/images/tree_example.png)

INFO: `Operand` 操作数, 如 a, b, c, d, e; `Operator` 操作符, 如 `+ *`

##### BST BinarySearchTree
- leftNode <= parentNode <= rightNode

References
----------
- [Grokking Algorithms: An illustrated guide for programmers and other curious people](https://www.amazon.com/Grokking-Algorithms-illustrated-programmers-curious/dp/1617292230/ref=sr_1_1?ie=UTF8&qid=1519440970&sr=8-1&keywords=Grokking+Algorithms)
- [Data Structures and Algorithm Analysis in Java](https://www.amazon.com/Data-Structures-Algorithm-Analysis-Java/dp/0132576279/ref=sr_1_1?s=books&ie=UTF8&qid=1519441056&sr=1-1&keywords=Data+Structures+Algorithm+Analysis+java)