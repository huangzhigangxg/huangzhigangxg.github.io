---
layout: post

title: 数据结构与算法笔记

date: 2017-03-09 15:32:24.000000000 +09:00

---

## 前言
学习数据结构的目的就是为了更高效的管理数据，对数据的操作无非就是增删改查，数组和链表简单的结构都可以管理数据。但是碰到需要管理大量的数据时，他们的增删改查效率就很难符合生产要求，需要更高效的数据结构来提高增删改查的效率。比如 哈希表，树，图。这些高效的结构体比数组和链表的结构也更复杂，能表达的数据逻辑更多。


## 数组
数组经常用来做数据的排序。先看看关于数组的几种常见的排序算法，以及他们的优缺点。

[稳定性:假定在待排序的记录序列中，存在多个具有相同的关键字的记录，若经过排序，这些记录的相对次序保持不变，即在原序列中，r[i]=r[j]，且r[i]在r[j]之前，而在排序后的序列中，r[i]仍在r[j]之前，则称这种排序算法是稳定的；否则称为不稳定的。](https://baike.baidu.com/item/排序算法稳定性)

[若算法的T(n) = O(log n)，则称其具有对数时间。由于计算机使用二进制的记数系统，对数常常以2为底（即log2 n，有时写作lg n）。然而，由对数的换底公式，loga n和logb n只有一个常数因子不同，这个因子在大O记法中被丢弃。因此记作O（log n），而不论对数的底是多少，是对数时间算法的标准记法。](https://zh.wikipedia.org/wiki/时间复杂度)

O(1) < O(logn) < (n) < O(nlogn) < O(n^2) < O(n^3) < O(2^n) < O(n!) < O(n^n)



### 插入排序：直接插入，希尔排序

```C
//每次都从后面无序那出一个像前边有序部分插入。
void insertSort(int a[], int count ){
    shell_insert(a, count, 1);
}

void shellSort(int a[], int count){
    int gap = count/2;
    while (gap > 0) {
        shell_insert(a,count,gap);
        gap = gap / 2;
    }
}

void shell_insert(int a[], int count,int gap){
    int temp,j ;
    for (int i = gap ; i < count ; i = i + gap){
        temp = a[i];
        for (j = i - gap; j > 0 && a[j] > temp; j = j - gap) {
            a[j + gap] = a[j];
        }
        a[j + gap] = temp;
    }
}

```

排序类型  | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 | 是否稳定排序
----|--------------|--------------|----------|-----------
直接插入 | O(n^2)  | O(n^2) | O(1) | 是
希尔排序 | 已知最好步长O(nlogn) | O(nlogn) |O(1) |否


备注：

1. 希尔排序是改进版的插入排序，根据递减的步长，将相邻较远的元素做比较，一次交换可以性解决多个逆序。所以突破的插入排序的O(n^2)的界限。
2. 由于是通过远距离比较提高效率的。所以不能保证稳定性。
3. 还有希尔排序的步长不同，时间复杂度不同。

### 交换排序：冒泡排序，快速排序 

```C
//起泡排序：每次把最小的比出来。
void bubbleSort(int a[] , int size)
{
  for (int i = 0; i < size - 1; i++) {
    for (int j = i + 1; j < size; j++) {
      if (a[j] < a[i]) {
        swap(&a[j],&a[i]);
      }
    }
  }
}
//分治的思想：
void quickSort(int a[] , int left , int right)
{
  if (left >= right) {
    return;
  }
  int i = left, j = right, key = a[left];
  while (i<j) {
    while (a[j] > key && i < j) {
      j--;
    }
    a[i] = a[j];
    while (a[i] < key && i < j) {
      i++;
    }
    a[j] = a[i];
  }
  a[i] = key;
  quickSort(a, left, i-1);
  quickSort(a, i+1,right);
}

```

排序类型  | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 | 是否稳定排序
----|--------------|--------------|----------|-----------
冒泡排序 | O(n^2)  | O(n^2) | O(1) | 是
快速排序 | O(nlogn) | O(n^2)  | O(1) |否

备注：

1. 快速排序相比起泡排序，区别是远距离比较和交换，提高效率的。所以也不能保证稳定性。


### 选择排序：直接选择，堆排序

```C
void selectSort(int a[] , int size)
{
  int min;
  for (int i = 0; i < size - 1; i++) {
    min = i;
    for (int j = i + 1; j < size; j++) {
      if (a[j] < a[min]) {
        min = j;
      }
    }
    swap(&a[min],&a[i]);
  }
}
//构建大顶堆
/*
 现实情况index从1开始，满足 rChild = 2*root , lChild = 2*root + 1
 数组实现时index从0开始，满足 rChild = 2*root + 1, lChild = 2*root + 2
 */
void sift(int a[] ,int index , int size)
{
    int temp,child;
    for (temp = a[index]; 2*index + 1 < size; index = child) {
        child = 2*index + 1;
        if (child + 1 < size && a[child + 1] > a[child]) {
            child = child + 1;
        }
        if ( a[child] > temp ){
            a[index] = a[child];
        }else{
            break;
        }
    }
    a[index] = temp;
}

void heapSort(int a[] , int size)
{
    for (int i = size/2; i >= 0 ; i --) { //因为是向下移动，必须逆序从低向上才能构建正确的大顶堆
        sift(a,i,size);
    }

    for (int i = size - 1; i > 0; i -- ) {
        swap(&a[i], &a[0]);
        sift(a, 0, i);
    }
}

```

排序类型  | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 | 是否稳定排序
----|--------------|--------------|----------|-----------
直接选择 | O(n^2)  | O(n^2) | O(1) | 是
堆排序 	| O(nlogn) | O(nlogn) |O(1)  |否


备注：

1. 堆排序和选择排序 都是每次从无序区遍历选择最大或者最小的一项，放在有序的区域。但是堆排序在无序区对比的次数少于直接选择排序，选择排序需要对比n次，堆排序对比的logn次。所以效率比选择排序高。
2. 堆排序由于也是通过远距离比较提高效率的，所以不能保证稳定性。
3. 但堆排序的好处是最坏情况要比快速排序好很多 O(nlogn) < O(n^2)
4. 在堆中找到最大值或者最小值，只需要logn的复杂度！可以用来海量数据搜素

### 合并排序 

```C
void mergeSort(int a[] , int beign , int end)
{
    if (beign < end)
    {
        int mid = ( beign + end ) /2;
        mergeSort(a, beign, mid);
        mergeSort(a, mid + 1, end);
        merge(a,beign,mid,end);
    }
}
void merge(int a[] , int begin , int mid , int end)
{
    int leftSize = mid - begin + 1;
    int rightSize = end - mid;
    int left[leftSize],right[rightSize];
    int leftIndex = 0 ,rightIndex = 0 ;
    for (int i = 0 ; i < leftSize; i ++) {
        left[i] = a[begin + i];
    }
    for (int i = 0 ; i < rightSize; i ++) {
        right[i] = a[mid + 1 + i];
    }
    for (int i = begin; i <= end; i ++) {
        if (leftIndex >= leftSize){
            a[i] = right[rightIndex++];
        }else if (rightIndex >= rightSize){
            a[i] = left[leftIndex++];
        }else if (left[leftIndex] > right[rightIndex]){
            a[i] = right[rightIndex ++];
        }else{
            a[i] = left[leftIndex ++];
        }
    }
}

```

排序类型 | 平均时间复杂度 | 最差时间复杂度 | 空间复杂度 | 是否稳定排序
----|--------------|--------------|----------|-----------
合并排序 | O(nlogn) | O(nlogn) |O(n)  |是

备注 ：

1. 合并排序是唯一最差时间复杂度为O(nlogn)的稳定排序。
2. 缺点就是需要额外n的空间。

## 数组和链表区别
1. 数组空间连续，靠偏移量定位元素，查找时间复杂度O(1),以为移位，插入和删除是 O(n). 
2. 链表空间不连续，靠遍历节点定位元素，查找时间复杂度O(n),不用移位，插入和删除是O(1).

数组比链表更擅长查找，链表比数组更擅长修改。

## 数组和链表的查找

数据结构中有个叫`符号表`的东西，和字典比较像，是Key-Value方式记录数据的容器。操作`符号表`时，如果想更新某个值，先检索Key，有的话更新值，没有的插入容器中，可以通过有序数组和无序链表来实现。

```C
//数组的二分法查找
int binSearch(int a[],int size,int data)
{
    int left = 0,right = size -1;
    int mid;
    while (left <= right) {
        mid = (left + right)/2;
        if (a[mid] == data){
            return mid;
        }else if(a[mid] > data){
            right = mid - 1;
        }else{
            left = mid + 1;
        }
    }
    return -1;
}
```

结构 | 查找 | 插入 
----|-----|-----
链表 | O(n) | O(n) 
数组 | O(logn) | O(n) 

无序链表的实现方法中，查找是 O(n)，虽然插入新值可以在表头插入，但是插入新的值也要先遍历O(n)一遍。
有序数组虽然可以利用二分法将查找速度降低为O(logn) 但是插入一个新的值除了要找到位置还要讲后面的元素后移，所以还是O(n)的效率并不高。

`下面介绍查找和插入速度都在O(logn)范围的结构体 ：树 (二叉树)`

## 栈和队列

栈和队列都可以用一个数组来实现

栈的实现比较简单，记录一个top指针，push时 top++ ，pop时top--。

队列稍微复杂些，记录两个指针 head 和 tail.
enqueue入队时 取 array[tail] 并且 tail = (tail + 1)%arrayLength
dequeue出队时 取 array[head] 并且 head = (head + 1)%arrayLength
仔细想一下，这样做法可以循环利用数组，每次一出一进，前面会有用过空出的空间，但是 每次tail和head都是+1 模size，这样就能重复利用前面的空间了。注意判断size的大小，如果size大于arrayLength了就要及时扩充。
扩充的办法可以采用简单的数据拷贝，开辟双倍的空间realloc，再便利数组拷贝进去。

```C
//数组实现的Stack
#define stack_size 128
typedef struct Stack{
  BiTree *data[stack_size];
  int top;
}Stack;

void stack_init(Stack *pStack)
{
  memset(pStack,0,sizeof(Stack));
  pStack->top = -1;
}
BiTree *stack_pop(Stack *pStack)
{
  if (stack_is_empty(pStack)) {
    return NULL;
  }
  return pStack->data[pStack->top--];
}
int stack_push(Stack *pStack,BiTree *data)
{
  if (pStack->top == stack_size-1) {
    return 0;
  }
  pStack->data[++pStack->top] = data;
  return 1;
}
int stack_is_empty(Stack *pStack)
{
  return pStack->top == -1;
}
BiTree * stack_get_top(Stack *pStack)
{
  if (pStack ->top == -1) {
    return NULL;
  }else{
    return pStack->data[pStack->top];
  }
}
void stack_destroy(Stack *pStack)
{
    pStack ->top = -1;
}

//数组实现的Queue
#define queue_size 4
typedef struct Queue {
    BiTree *data[queue_size];
    int head;
    int tail;
    int size;
}Queue;

void queue_init(Queue *queue)
{
    memset(queue,0,sizeof(queue));
    queue->head = -1;
    queue->tail = -1;
    queue->size = 0;
}
void queue_destroy(Queue *queue)
{
    queue->head = -1;
    queue->tail = -1;
    queue->size = 0;
}
int queue_enqueue(Queue *queue,BiTree *node)
{
    if (queue_is_full(queue)) {
        printf("queue_is_full");
        return 0;
    }
    queue->head = (queue->head+1)%queue_size;
    queue->size++;
    queue->data[queue->head] = node;
    return 1;
}
BiTree * queue_dequeue(Queue *queue)
{
    if (queue_is_empty(queue)){
        printf("queue_is_empty");
        return NULL;
    }
    queue->tail = (queue->tail+1)%queue_size;
    queue->size--;
    return queue->data[queue->tail];
}
int queue_is_empty(Queue *queue)
{
    return queue->size == 0;
}
int queue_is_full(Queue *queue)
{
    return queue->size == queue_size;
}
void queue_print(Queue *queue)
{
    printf("\n~~~~~~~~~~~~~~~~~~~~~~~~~~~~\n");
    int index = queue->tail;
    for (int i = 0 ; i < queue->size ; i++ ){
        index = (index + 1)%queue_size;
        printf("%d\n",queue->data[index]->data);
    }
    printf("~~~~~~~~~~~~~~~~~~~~~~~~~~~~\n");
}

```
时间复杂度的备忘

1. 查找：二分法查找就是log(n)，就像二叉树的高度一样 log(n) + 1
2. 排序：效率高的快排和归并都是nlog(n),效率一般的插入或选择就是n^2
3. 访问：数组O(1),链表O(n),哈希表O(1)

一般实现字典这种数据结构有两种方式：

1. 哈希表
2. 二叉搜索树(红黑树)

对于链表，删除和查找，效率都是O(n),所以链表并没有字典使用广泛，字典能把维护数据的复杂度都控制在O(1)~O(logn).


## 树

树相比于链表，再处理大量数据时，插入删除性能，查找性能，都有明显的优势。
因为将数据整合进有宽度有长度的平面图形中，数据可以依靠数节点的多条边，可以表达更多的逻辑。

### 二叉树的基本性质

1. 在二叉树的第i层上最多有2^(i-1)个节点 
2. 二叉树中如果深度为k,那么最多有2^(k)-1个节点
3. 在完全二叉树中，具有n个节点的完全二叉树的深度为(log2n)+1

### 基本含义和种类
抽象地说，基本上有序列的地方就可以应用树，因为树结构即是一种序列索引结构
传统的文件夹，就是很好的应用实例。

1. 无序树：树中任意节点的子节点之间没有顺序关系，这种树称为无序树，也称为自由树；
2. 有序树：树中任意节点的子节点之间有顺序关系，这种树称为有序树；
3. 二叉树：每个节点最多含有两个子树的树称为二叉树；
4. 完全二叉树：对于一颗二叉树，假设其深度为d（d>1）。除了第d层外，其它各层的节点数目均已达最大值，且第d层所有节点从左向右连续地紧密排列，这样的二叉树被称为完全二叉树；
5. 满二叉树：所有叶节点都在最底层的完全二叉树。
6. 排序二叉树(二叉查找树（英语：Binary Search Tree），也称二叉搜索树、有序二叉树)
7. 平衡二叉树（AVL树）：当且仅当任何节点的两棵子树的高度差不大于1的二叉树。
8. 2-3树 : 一个节点可能有两到三个孩子，一或两个值，也是自平衡二叉树。
9. 红黑树：自平衡二叉树，每次插入或删除，修复自身保持左右平衡，保证检索效率
10. 霍夫曼树：带权路径最短的二叉树称为哈夫曼树或最优二叉树；
11. B树：一种对读写操作进行优化的自平衡的二叉查找树，能够保持数据有序，拥有多余两个子树


从1 到 10 是树的进化，逐步有序，有规则，查找和更新的效率更高，意义更大。

人借助计算机实现对数据的管理，其中利用树这种数据结构优势很大。既在概念上模仿了现实生活中的一些数据关系，使人更能方便理解，又在树型结构的特点是能更灵活的保证对数据的增删改查的效率。

从6开始，二叉树可以实现排序的功能，但插入和删除的效率不稳定，最坏情况退化树变成链表O(n)。因为这些操作的时间复杂度跟树的高度相关

到7的话，有了自平衡的概念，就是每次修改树都要重新恢复树的平衡，保证树的高度都差不大与1.从而保证了查找插入删除的效率最坏的情况O(logn)

到8的话，节点可以保存一或两个值，二到三个孩子，每次增删改查可以借助2节点变3节点，或者是3节点分裂成2节点，将分裂后的三个孩子中的中间值挤到上面一层的节点里，如果上面是2节点，自然变成三节点，如果上面是三节点，就再次向上面分裂。 将变化推演到上层，使其影响局部或者全局分布，这样能保证到每一个空节点到根节点的高度都是一样的。

### 遍历算法 DFS 和  BFS

#### DFS   (Depth-First-Search) 广度优先

规律 ：一层一层遍历，从距离近的节点到距离远的。

借助队列先入先出的原则，在一次递归函数中 放入这一层的所有节点，保证他们优先于子节点被遍历。

作用 ： DFS多用于连通性问题

#### BFS（Breadth-First-Search） 深度优先

规律 ： 尽可能深得搜索树的分支，当V节点所在边都被搜索后，才回溯到发现V节点的那条边的起始节点。反复这一过程，达到所有子节点都被遍历。

借助栈的先入后出的原则

作用 ：BFS多用于解决最短路问题

![](/assets/images/9F909745-9890-4F89-B474-5CBDE00374B6.png)

### 优缺点

抽象地说，基本上有序列的地方就可以应用树，因为树结构即是一种序列索引结构。

序列的核心接口就是三个：插、查、删。

这个索引可以把原本O(n)的查找操作变为O(logn)，可以简单地理解为在数据结构的层面上构造了一个二分查找算法。

Hash查找(散列表) 时间复杂度 O(1)         需要开辟大量空间 
二叉树查找       时间复杂度 O(logn)      灵活控制内存
链表查找         时间复杂度 O(n)         灵活控制内存


现在很多大型高效的数据库（如mysql大多用B+树）都是利用树，因为内存控制更灵活，相比Hash表的搜索，往往适用于小范围可控的数据，因为内存上开辟开辟巨大空间，即使有扩容和重建算法效率也比较一般。
7，8，9 ，11 都是对自平衡方法的优化。


## 哈希表

这是一种查找效率更高的数据结构。键-值(key-indexed) 存储数据的结构，我们只要输入待查找的值即key，即可查找到其对应的值。可以使用一个简单的无序数组来实现：将键作为索引，值即为其对应的值，这样就可以快速访问任意键的值。

使用哈希查找有两个步骤:

1. 使用哈希函数将被查找的键转换为数组的索引。在理想的情况下，不同的键会被转换为不同的索引值，但是在有些情况下我们需要处理多个键被哈希到同一个索引值的情况。所以哈希查找的第二个步骤就是处理冲突
2. 处理哈希碰撞冲突。有很多处理哈希碰撞冲突的方法，本文后面会介绍拉链法和线性探测法。

数组的索引：

1. 可以用正整数来做KEY。模数组的SIZE。
2. 可以用字符串KEY。字符串就相当于一个大值的数字。

解决冲突的办法：

1. 链表法，将key相同的元素都用链表链接起来。
2. 线性探测法，加入100的位置被占领了，将以此向后101，102找位置。

### 涉及到的问题

当数组不够大时需要 涉及到更复杂的操作，比如哈希表扩充，哈希表 重建，等算法

当出现地址冲突的时候，涉及到的操作还有，链表扩展，开放地址法

Jave 8 HashMap 当用链表处理冲突时，列表长度超过8 ，就会将列表转换为红黑树，已增加查找效率，因为列表要一个个线性遍历。效率很慢，同时插入和删除的效率也有增加

Redis 链表法解决冲突，将新的数据放在列表头部，插入效率高，检索效率高

### 哈希表来做缓存池
+ LRU ： 利用hashmap 存贮的Value是 双向链表的节点，用两个指针指向列表的head和tail。查找时通过hashmap找到节点位置，然后通过找到的节点的前后指针和head指针，使这个节点插入到head前，并将head移动到这个上面。和使查找和移动都是O(1)
+ FIFO:  结构还是上面的。只不过每次插入新的都从head，删除都从tail。找到节点后不用插入到head前。
+ LFU： 利用hashmap 存贮的Value是 单项链表中有一个用来记录访问次数的字段，每次找到Value都+1 ，删除的时候只能遍历整个列表。O(n)



+ [浅谈算法和数据结构](http://www.cnblogs.com/yangecnu/p/Introduction-Stack-and-Queue.html)

+ [《编程之法：面试和算法心得》](https://www.gitbook.com/book/wizardforcel/the-art-of-programming-by-july/details)



