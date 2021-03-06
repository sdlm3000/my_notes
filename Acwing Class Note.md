<h1 style="text-align:center">Acwing Class Note</h1>

[TOC]

## 一、基础算法

### 1. 排序

#### 快速排序

​	基本思想为分而治之。确认分界点x；然后调整区间，小于等于x放在左边，大于等于x放在x右边；递归处理左右两段。

```c++
 // 注意边界问题，这里x = q[r]会有边界问题
// 这种方式 q[l + (r - l >> 2)] 可以避免 l+r 过大超范围问题。
void quick_sort(int *q, int l, int r)
{
    if(l>=r) return;
    int i = l - 1, j = r + 1, x = q[l + (r - l >> 1)]; // 或者 x = q[l]
    while(i<j){
        while(q[++i]<x);
        while(q[--j]>x);
        if(i<j) swap(q[i], q[j]);
    }
    quick_sort(q, l, j);
    quick_sort(q, j+1, r);
}

// 注意边界问题，这里x = q[l]会有边界问题
// 同时主要取中间值时x的取指方法，建议使用上者
void quick_sort(int* q, int l ,int r)
{
    if(l>=r) return;
    int i = l - 1, j = r + 1, x = q[l + (r - l + 1 >> 1)]; // 或者 x = q[r]
    // int x = q[r], i = l - 1, j = r + 1;
    while(i<j){
        while(q[++i]<x);
        while(q[--j]>x);
        if(i<j) swap(q[i], q[j]);
    }
    quick_sort(q, l, i-1);
    quick_sort(q, i, r);
}
```

​	记住第一种情况就好，直接避免考虑边界问题，比较简单。

#### 归并排序

​	基本思路是将两个有序的数组合并成一个有序的数组；先将数组不断二分，最后对长度为一的两个数组进行合并，不断向上递归合并，最终完成排序。

```C++
int temp[MAX_SIZE];
void merge_sort(int *q, int l, int r)
{
    if(l>=r) return;
    int mid = l + (r - l >> 1);
    merge_sort(q, l, mid);
    merge_sort(q, mid+1, r);
    
    int k = 0, i = l, j = mid  + 1;
    while(i<=mid && j<=r)
        temp[k++] = q[i] < q[j] ? q[i++] : q[j++];	
    
    while(i<=mid) temp[k++] = q[i++];
    while(j<=r) temp[k++] = q[j++];
    for(i=l, j=0; i<=r; i++, j++) q[i] = temp[j];
}
```

### 2. 二分查找

#### 整数二分

​	主要注意是寻找左边界还是右边界的问题。

```C++
bool check(int x) {/* ... */} // 检查x是否满足某种性质，为 >= 或者 <=

// 方便记忆，叫做 右边界问题
// 区间[l, r]被划分成[l, mid]和[mid + 1, r]时使用：
int bsearch_1(int l, int r)
{
    int mid;
    while(l < r){
        mid = l + r >> 1;
        if(check(mid)) r = mid;		// check()判断mid是否满足性质
        else l = mid + 1; 
    }
    return l;
}
// 方便记忆，叫做 左边界问题
// 区间[l, r]被划分成[l, mid - 1]和[mid, r]时使用
int bsearch_2(int l, int r)
{
    int mid;
    while(l < r){
        mid = l + r + 1 >> 1;
        if(check(mid)) l = mid;		// 记忆 l = mid 则前面为 mid = l + r + 1 >> 1
        else r = mid - 1; 
    }
    return l;
}

// 如: 1 2 2 3 3 4 6 数组q，分别找 k = 3、4、5的边界的索引
// bsearch_1 的 check(mid) 为 q[mid] >= k
// bsearch_2 的 check(mid) 为 q[mid] <= k
// 3 的bsearch_1 为 3，bsearch_2 为 4
// 4 的bsearch_1 为 5，bsearch_2 为 5
// 5 的bsearch_1 为 6，bsearch_2 为 5
```

#### 浮点二分

```c++
bool check(double x) {/* ... */} // 检查x是否满足某种性质

double bsearch_3(double l, double r)
{
    const double eps = 1e-6;   // eps表示精度，取决于题目对精度的要求
    while (r-l>eps)
    {
        double mid = (l + r) / 2;
        if(check(mid)) r = mid;
        else l = mid;
    }
    return l;
}
```

### 3. 高精度

#### 高精度加法 

```c++
// 向量中低位存放数字对应的低位，即a[0]中存放个位数，后面也是一样的
vector<int> add(vector<int> &a, vector<int> &b)
{
    if(a.size()<b.size()) return add(b, a);
    
    vector<int> c;
    int t = 0;
    for(int i=0; i<a.size(); i++){
        t += a[i];
        if(i<b.size()) t += b[i];
        c.push_back(t%10);
        t /= 10;
    }
    
    if(t) c.push_back(t);
    return c;
}
```

#### 高精度减法

```c++
// 输入的数需要满足：a > b > 0
vector<int> sub(vector<int> &a, vector<int> &b)
{
    vector<int> c;
    int t = 0;
    for(int i=0; i<a.size(); i++){
        t = a[i] - t;
        if(i<b.size()) t -= b[i];
        c.push_back((t+10)%10);		// (t+10)%10 包括不借位情况和借位情况的结果
        if(t<0) t = 1;
        else t = 0;
    }
    while(c.size()>1 && c.back()==0) c.pop_back();	// 去除前导零
    return c;
}
```

#### 高精度乘法

```c++
vector<int> mul(vector<int> &a, int b)
{
    vector<int> c;
    int t = 0;
    // 注意里面 t 的处理方式
    for(int i=0; i<a.size() || t; i++){
        if(i<a.size()) t += a[i] * b;
        c.push_back(t%10);
        t /= 10;
    }
    while (c.size()>1 && c.back()==0) c.pop_back();
    return c;
}
```

#### 高精度除法

```c++
vector<int> div(vector<int> &a, int b, int &r)
{
    vector<int> c;
    r = 0;
    // 除法从高位开始
    for(int i=a.size()-1; i>=0; i--){
        r = r * 10 + a[i];
        c.push_back(r/b);
        r %= b;
    }
    reverse(c.begin(), c.end());
    while(c.size()>1 && c.back()==0) c.pop_back();
    return c;
}
```

### 4. 前缀和与差分

#### 一维前缀和

$$
S[i] = a[1] + a[2] + ... a[i] \\
a[l] + ... + a[r] = S[r] - S[l - 1]
$$

```c++
for(int i=1; i<=n; i++) scanf("%d", &q[i]);		// q[0]系统会默认初始化为0
for(int i=1; i<=n; i++) S[i] = S[i-1] + q[i];	// S[0]系统会默认初始化为0

scanf("%d%d", &l, &r);
printf("%d\n", S[r] - S[l-1]);
```

#### 二维前缀和

```c++
for(int i=1; i<=x; i++)
    for(int j=1; j<=y; j++)
        S[i][j] = S[i-1][j] + S[i][j-1] - S[i-1][j-1] + q[i][j];

scanf("%d%d%d%d", &x1, &y1, &x2, &y2);
printf("%d\n", S[x2][y2] - S[x2][y1-1] - S[x1-1][y2] + S[x1-1][y1-1]);
```

#### 一维差分

​	给区间[l, r]中的每个数加上c，B[l] += c, B[r + 1] -= c

```C++
// 差分函数
void insert(int l, int r, int c)
{
    b[l] += c;
    b[r+1] -= c;
}
// 获得差分序列
for(int i=1; i<=n; i++) scanf("%d", &q[i]);
for(int i=1; i<=n; i++) insert(i, i, q[i]);
// 给区间加c
insert(l, r, c);
// 获得最终的序列
for(int i=1; i<=n; i++) b[i] += b[i-1];
```

#### 二维差分

​	给以(x1, y1)为左上角，(x2, y2)为右下角的子矩阵中的所有元素加上c：
$$
S[x1, y1] += c, S[x2 + 1, y1] -= c, S[x1, y2 + 1] -= c, S[x2 + 1, y2 + 1] += c
$$

```c++
// 差分函数
void insert(int x1, int y1, int x2, int y2, int c)
{
    b[x1][y1] += c;
    b[x2+1][y1] -= c;
    b[x1][y2+1] -= c;
    b[x2+1][y2+1] += c;
}
// 获取差分序列
for(int i=1; i<=n; i++)
    for(int j=1; j<=m; j++)
        insert(i, j, i, j, q[i][j]);
// 给区间加c
insert(x1, y1, x2, y2, c);
// 获得最终序列
for(int i=1; i<=n; i++)
    for(int j=1; j<=m; j++)
        b[i][j] += b[i-1][j] + b[i][j-1] - b[i-1][j-1];
```

### 5. 双指针算法

​	双指针算法为一种思想，在快速排序和归并排序中都有用到，可以理解使用两个指针，再利用数据本身的一些特性，如单调性，直接避免一些没有必要的过程，来优化单纯暴力搜索$O(n^2)$的时间复杂度。个人感觉没有很固定的模板，具体问题具体分析，这种思想很重要，一般对于无序的序列，可以先使用快排排列成有序的序列，然后再使用双指针等等。[最长连续不重复子序列](https://www.acwing.com/activity/content/code/content/1203679/)

```C++
for(int i=0, j=0; i<n; i++){
    while(j<i && check(i,j)) j++;
    // 之后为具体问题的逻辑
    
}
/****常见问题分类：
* (1) 对于一个序列，使用两个指针维护一段区间
* (2) 对于两个序列，维护某种次序，比如归并排序中合并两个有序序列的操作
*/
```

### 6. 位运算

​	求n的第k位数字: `n >> k & 1`
​	返回n的最后一位1及其后面的0：`lowbit(n) = n & -n `，如：100110 -> 10

### 7. 离散化

```c++
vector<int> alls; // 存储所有待离散化的值
sort(alls.begin(), alls.end()); // 将所有值排序
alls.erase(unique(alls.begin(), alls.end()), alls.end());   // 去掉重复元素

// 二分求出x对应的离散化的值
int find(int x) // 找到第一个大于等于x的位置
{
    int l = 0, r = alls.size() - 1;
    while (l < r){
        int mid = l + r >> 1;
        if (alls[mid] >= x) r = mid;
        else l = mid + 1;
    }
    return r + 1; // 映射到1, 2, ...n
}
```

### 8. 区间合并

```c++
// 将所有存在交集的区间合并
typedef pair<int, int> PII;   

void merge(vector<PII> &segs)
{
    vector<PII> res;
    sort(segs.begin(), segs.end());
    int st = -2e9, ed = -2e9;
    for (auto seg : segs)
        if (ed < seg.first){
            if (st != -2e9) res.push_back({st, ed});
            st = seg.first, ed = seg.second;
        }
        else ed = max(ed, seg.second);
    if (st != -2e9) res.push_back({st, ed});
    segs = res;
}
```

## 二、数据结构

### 1. 链表

#### 单链表

​	在算法题中使用数组来模拟链表会比较简单，快速。

​	不要被数组本身的连接关系搞混而误以为索引为0时为链表头，要注意head的变化。

```C++
// head存储链表头，e[]存储节点的值，ne[]储存节点的next指针，idx表示当前用到了哪个节点
// 对于第i个插入的数，e[i]存放了这个节点对应的值，ne[i]存放了该节点指向的下一个节点的数组索引
// idx可以理解为下一个可以使用的数组的下标，和链表本身没有什么关系
int head, e[N], ne[N], idx;
// 初始化
void init()
{
    head = -1, idx = 0;
}
// 在链表头插入一个数a
void insert_to_head(int a)
{
    e[idx] = a, ne[idx] = head, head = idx++;
}
// 在第k个插入的数后面插入一个数x
void insert_to_k(int k, int x)
{
    e[idx] = x, ne[idx] = ne[k], en[k] = idx++;
}
// 将头结点删除，需保证头结点存在
void remove()
{
    head = ne[head];
}
// 删除第k个插入的数后面的数
void remove_k(int k)
{
    ne[k] = ne[ne[k]];
}

// 遍历链表
int temp = head;
while(temp != -1){
    printf("%d ", e[temp]);
    temp = ne[temp];
}
```

#### 双链表

```C++
// e[]表示节点的值，l[]表示节点的左指针，r[]表示节点的右指针，idx表示当前用到了哪个节点
int e[N], l[N], r[N], idx;
// 初始化
void init()
{
    // 默认0为头结点，1为尾结点，它们不记录实际值
    r[0] = 1, l[0] = 0, idx = 2;
}
// 在节点a的右边插入一个数x
// 在节点a的左边插入直接调用 insert(l[a])
void insert(int a, int x)
{
    e[idx] = x;
    l[idx] = a, r[idx] = r[a];
    l[r[a]] = idx, r[a] = idx++;
}
// 删除a节点
void remove(int a)
{
    l[r[a]] = l[a];
    r[l[a]] = r[a];
}
```

### 2. 栈

```c++
// tt表示栈顶
int stk[N], tt = 0;
// 向栈顶插入一个数，从索引1开始存储，tt=0时代表空栈
stk[++tt] = x;
// 从栈顶弹出一个数
tt--;
// 栈顶的值
stk[tt];
```

#### 单调栈

​	[单调栈](https://www.acwing.com/problem/content/submission/code_detail/5411839/)还蛮有意思的，可以看看。

​	应用场景：给定一个数组，给出每个数左边离它最近且小于它的数。或者是其他的类似场景。

```C++
    for(int i=0; i<n; i++){
        while(tt>0 && a[i]<=stk[tt]) tt--;
        
        if(tt==0) printf("-1 ");
        else printf("%d ", stk[tt]);
        stk[++tt] = a[i];

    }
```



### 3. 队列

#### 普通队列

```c++
// hh表示队头，tt表示队尾
int q[N], hh = 0, tt = -1;
// 向队尾插入一个数
q[++tt] = x;
// 从队头弹出一个数
hh++;
// 队头的值
q[hh];
// 判断队列是否为空
if(hh<=tt) not empty;
else empty;
```

#### 循环队列

```C++
// hh 表示队头，tt表示队尾的后一个位置
int q[N], hh = 0, tt = 0;
// 向队尾插入一个数
q[tt++] = x;
if(tt==N) tt = 0;
// 从队头弹出一个数
hh++;
if(hh==N) hh = 0;
// 队头的值
q[hh];
// 判断队列是否为空
if(hh!=tt) not empty;	// 这个地方有点问题，如果队列满的情况，hh==tt，感觉应该加一个变量记录队列长度
else empty;
```

#### 单调队列

​	[滑动窗口](https://www.acwing.com/problem/content/submission/code_detail/5412402/)，大致懂了，但是没有很理解，这个应该可以解决QT上位机界面的动态图标中自动更新坐标轴的问题，有时间看看。

```c++
// 若为求最大值
int hh = 0, tt = -1;
for (int i = 0; i < n; i ++ )
{
    while (hh <= tt && check_out(q[hh])) hh ++ ;	// 判断队头是否滑出窗口
    // 队尾不单调
    // 判断新加入的数是否大于此时滑动窗口中的值
    // 如果大于则原来滑动窗口中的值就没有用处了，让他们出队
    // 因为新入队的是时间周期比他们长，则最大值至少是新入队的值
    while (hh <= tt && check(q[tt], i)) tt -- ;		// 注意这个tt--，实际意义是删掉队尾的元素，相当于双端队列的操作
    q[ ++ tt] = i;
}
```



### 4. KMP

​	KMP算法常用于字符串匹配，

```c++
// s[N]是长文本，p[M]是模式串，ne[M]存放next指针，n是s的长度，m是p的长度
// N>n，M>m，m<=n，从下标1开始存储，不考虑浪费的空间
ne[0] = -1;
// 求模式串的next数组
for(int i=2, j=0; i<=m; i++){
    while(j && p[i]!=p[j+1]) j = ne[j];
    if(p[i]==p[j+1]) j++;
    ne[i] = j;
}
// 字符串匹配
for(int i=1, j=0; i<=n; i++){
    while(j && s[i]!=p[j+1]) j = ne[j];
    if(s[i]==p[j+1]) j++;
    if(j==m){
        j = ne[j];
        // 匹配成功后的操作
        
    }
}

// 从下标0开始存储
int strStr(string haystack, string needle) {
    int ne[needle.size()];
    ne[0] = 0;	// 很关键
    for(int i = 1, j = 0; i < needle.size(); i++)
    {
        while(j > 0 && needle[i] != needle[j]) j = ne[j - 1];
        if(needle[i] == needle[j]) j++;
        ne[i] = j;
    }

    for(int i = 0, j = 0; i < haystack.size(); i++)
    {
        while(j > 0 && haystack[i] != needle[j]) j = ne[j - 1];
        if(haystack[i] == needle[j]) j++;
        if(j == needle.size())
        {
            return i - needle.size() + 1;
        }
    }
    return -1;
}
```

​	字符匹配算法的暴力解法是完全遍历，即需要判断n*m次，而KMP算法的核心是首先提取模式串本身的规矩，使用ne[M]数组存放P[M]中每个值对应的next指针，其中索引i对应的next指针需要满足p[1, next]对应的字符串与p[i-next+1,i]对应的字符串相等。以字符串abcabe为例：

![image-20210508160828923](images\Acwing Class Note\image-20210508160828923.png)

其中索引5对应的next指针应为2，即ne[5]=2，则ne[1...6] = {0, 0, 0, 1, 2, 0}。

​	然后比较时，如果模式串的第j+1个和长文本串的第i个不匹配就将ne[j]+1与i进行比较，如此迭代循环，可以避免从头开始比较。

![image-20210508162435024](images\Acwing Class Note\image-20210508162435024.png)

### 5. Trie树

​	Trie树又叫做前缀树或者字典树。假设Trie数中依次存入inn、int、tea和ton，如下图所示。

<img src="images\Acwing Class Note\image-20210509154144006.png" style="zoom:80%">

```c++
// son[][]存储树中每个节点的子节点
// cnt[]存储以每个节点结尾的单词数量
int son[N][26], cnt[N];
int idx;	// 0号点既是根节点，又是空节点
// 插入新字符串
void insert(char *str)
{
    int p = 0;
    for(int i=0; str[i]; i++){
        int u = str[i] - 'a';
        if(!son[p][u]) son[p][u] = ++idx;
        p = son[p][u];
    }
    cnt[p]++;
}
// 查询字符串个数
int query(char *str)
{
    int p = 0;
    for(int i=0; str[i]; i++){
        int u = str[i] - 'a';
        if(!son[p][u]) return 0;
        p = son[p][u];
    }
    return cnt[p];
}
```

### 6. 并查集

​	并查集的主要操作为：将两个集合合并，询问两个元素是否在一 个集合当中。

​	基本原理:每个集合用一颗树来表示。树根的编号就是整个集合的编号。每个节点存储它的父节点，p[x]表示x的父节点。
​	问题1:如何判断树根: if(p[x] == x)
​	问题2:如何求x的集合编号: while (p[x]!= x)x = p[x];
​	问题3:如何合并两个集合: px是x的集合编号，py是y的集合编号。 p[x] = y

```c++
int p[N];			//存储每个点的祖宗节点
// 返回x的祖宗节点
int find(int x)
{
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}
// 初始化，假定节点编号是1~n
for (int i = 1; i <= n; i ++ ) p[i] = i;
// 合并a和b所在的两个集合：
p[find(a)] = find(b);
```

​	对于并查集还有一些其他的扩展，比如[维护集合大小](https://www.acwing.com/video/262/)和[维护到祖宗节点距离](https://www.acwing.com/video/251/)等。

### 7. 堆

​	堆就是一种基于完全二叉树的结构，分为最大堆和最小堆。最大堆的父节点一定大于它的左右子节点，最小堆则反之。

```c++
// h[N]存储堆中的值, h[1]是堆顶，x的左儿子是2x, 右儿子是2x + 1
int h[N], hsize;
// 最大堆
void down(int u)
{
    int t = u;
    if(u*2<=hsize && h[u*2]>h[t]) t = u * 2;
    if(u*2+1<=hsize && h[u*2+1]>h[t]) t = u * 2 + 1;
    if(t != u){
        swap(h[t], h[u]);	// C++中需要引入iostream和using namespace std才可以直接调用
        down(t);
    }
}
void up(int u)
{
    while(u/2 && h[u/2]<h[u]){
        swap(h[u/2], h[u]);
        u >> 1;
    }
}
// 建堆
for(int i=size/2; i; i--) down(i);
```

​	[模拟堆](https://www.acwing.com/problem/content/841/)还没有看。

### 8. 哈希表

#### 拉链法

```c++

```

#### 开放寻址法

```c++
int h[N];

int find(int x)
{
    int t = (x % N + N) % N;
    while(h[t] != null && h[t] != x)
   {
        t++;
        if(t == N) t = 0;
    }
    return t;
}

// 插入数字
h[find(x)] = x;
// 查找数字
if (h[find(x)] == null)
```

### 9. STL简介

```C++
vector, 变长数组，倍增的思想
    size()  返回元素个数
    empty()  返回是否为空
    clear()  清空
    front()/back()
    push_back()/pop_back()
    begin()/end()
    []
    支持比较运算，按字典序

pair<int, int>
    first, 第一个元素
    second, 第二个元素
    支持比较运算，以first为第一关键字，以second为第二关键字（字典序）

string，字符串
    size()/length()  返回字符串长度
    empty()
    clear()
    substr(起始下标，(子串长度))  返回子串
    c_str()  返回字符串所在字符数组的起始地址
    reverse(str.begin(), str.end());	// 字符串翻转
	strstr(char *str1, char *str2);		// 从str1中查找是否有str2字符串，如果有，从str1中str2出现的位置起返回str1的指针，如果没有，返回nullptr

queue, 队列
    size()
    empty()
    push()  向队尾插入一个元素
    front()  返回队头元素
    back()  返回队尾元素
    pop()  弹出队头元素

priority_queue, 优先队列，默认是大根堆
    size()
    empty()
    push()  插入一个元素
    top()  返回堆顶元素
    pop()  弹出堆顶元素
    定义成小根堆的方式：priority_queue<int, vector<int>, greater<int>> q;

stack, 栈
    size()
    empty()
    push()  向栈顶插入一个元素
    top()  返回栈顶元素
    pop()  弹出栈顶元素

deque, 双端队列
    size()
    empty()
    clear()
    front()/back()
    push_back()/pop_back()
    push_front()/pop_front()
    begin()/end()
    []

set, map, multiset, multimap, 基于平衡二叉树（红黑树），动态维护有序序列
    size()
    empty()
    clear()
    begin()/end()
    ++, -- 返回前驱和后继，时间复杂度 O(logn)

    set/multiset
        insert()  插入一个数
        find()  查找一个数，返回迭代器
        count()  返回某一个数的个数
        erase()
            (1) 输入是一个数x，删除所有x   O(k + logn)
            (2) 输入一个迭代器，删除这个迭代器
        lower_bound()/upper_bound()
            lower_bound(x)  返回大于等于x的最小的数的迭代器
            upper_bound(x)  返回大于x的最小的数的迭代器
    map/multimap
        insert()  插入的数是一个pair
        erase()  输入的参数是pair或者迭代器
        find()
        []  注意multimap不支持此操作。 时间复杂度是 O(logn)
        lower_bound()/upper_bound()

unordered_set, unordered_map, unordered_multiset, unordered_multimap, 哈希表
    和上面类似，增删改查的时间复杂度是 O(1)
    不支持 lower_bound()/upper_bound()， 迭代器的++，--

bitset, 圧位
    bitset<10000> s;
    ~, &, |, ^
    >>, <<
    ==, !=
    []

    count()  返回有多少个1

    any()  判断是否至少有一个1
    none()  判断是否全为0

    set()  把所有位置成1
    set(k, v)  将第k位变成v
    reset()  把所有位变成0
    flip()  等价于~
    flip(k) 把第k位取反

作者：yxc
链接：https://www.acwing.com/blog/content/404/
来源：AcWing
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```





## 三、搜索与图论

### 1.DFS与BFS

​	深度优先搜索（DFS）一种用于遍历或搜索树或图的算法。 沿着树的深度遍历树的节点，尽可能深的搜索树的分支。搜索到最深的就回溯。

​	回溯与剪枝，回溯时一般需要注意恢复现场。

​	没有特定的模板，可以理解为递归

![image-20211126203440101](images\Acwing Class Note\image-20211126203440101.png)

​	广度优先搜索（BFS），最短路特性。其代码实现需要用到队列。常见显示如下：

```C++
const int N = 102;
int q[N * N];

// 普通的队列
int hh = 0, tt = -1;
// 队列初始化
q[++tt] = 0;
// 队列不为空
while(hh <= tt)
{
    // 取出队头元素
    auto t = q[hh++];
    // 从队头元素开始向外扩展
    
    // 对于符合条件的点可以插入队列
    q[++tt] = x;
}
```

### 2.树与图的存储

​	图分为无向图和有向图，无向图可以看做一种特殊的有向图。树是一种无环连通图。

​	因此我们可以只考虑有向图的存储。

​	图的存储可以使用邻接矩阵和邻接表。

(1) 邻接矩阵：g\[a][b] 存储边a->b

(2) 邻接表：

```C++
M = 2 * N;
// 对于每个点k，开一个单链表，存储k所有可以走到的点。h[k]存储这个单链表的头结点
int h[N], e[M], ne[M], idx;

// 添加一条边a->b
// 头插法
void add(int a, int b)
{
    e[idx] = b;		// 创建结点将b存入
    ne[idx] = h[a];	// 让结点b指向a的头结点
    h[a] = idx ++;	// 让a的头结点指向b
}

// 初始化
idx = 0;
memset(h, -1, sizeof h);
```

### 3.树与图的遍历

​	深度优先遍历：

```C++
bool st[N];
int dfs(int u)
{
    st[u] = true; // st[u] 表示点u已经被遍历过

    for (int i = h[u]; i != -1; i = ne[i])
    {
        int j = e[i];
        if (!st[j]) dfs(j);
    }
}
```

​	广度优先遍历：

```C++
int q[N], d[N];

int bfs()
{
    int hh = 0, tt = -1;
    // 从点1开始搜，在队头放入点1
    q[++tt] = 1;
    memset(d, -1, sizeof(d));
    // 点1 到 点1 的距离为 0
    d[1] = 0;
    // 如果队列不为空
    while(hh <= tt)
    {
        // 取出队头的点
        int t = q[hh++];
        // 从队头进行延伸
        for(int i = h[t]; i != -1; i = ne[i])
        {
            // 取出该点的临点
            int j = e[i];
            // 如果j没有搜到
            if(d[j] == -1){
                // d[j]存储j节点离起点的距离，并标记为访问过
                d[j] = d[t] + 1;
                q[++tt] = j;
            }
        }
    }
    return d[n];
}
```

### 4.最短路

![image-20211203095823551](images\Acwing Class Note\image-20211203095823551.png)



### 4.最小生成树

![image-20211203101211989](images\Acwing Class Note\image-20211203101211989-16393635494501.png)



### 5.二分图

![image-20211203101240578](images\Acwing Class Note\image-20211203101240578.png)







## 五、动态规划

### 1.01背包问题

![image-20211126083449246](images\Acwing Class Note\image-20211126083449246.png)

​	状态转移方程：`f[i][j] = Max(f[i - 1][j], f[i - 1][j - v[i]] + w[i]) `

​	参考代码：

```C++
// 朴素写法
for(int i = 1; i <= N; i++)		// 从1个物品到N个物品
{
    for(int j = 0; j <= V; j++)	// 从小背包到大背包
    {
        f[i][j] = f[i - 1][j];
        if(j >= v[i]) f[i][j] = max(f[i][j], f[i - 1][j - v[i]] + w[i]);
    }
}
int res = 0;
for(int i = 1; i <= N; i++) res = max(res, f[i][V]);

// 空间优化    
for(int i = 1; i <= N; i++)
{ 
    // 这个地方做了优化，是从上一个状态的小背包到当前状态的大背包
    for(int j = V; j >= v[i]; j--)
    {
        f[j] = max(f[j], f[j - v[i]] + w[i]);
    }
}
```

​	对于背包问题的理解，可以看这个[动态规划博客](https://www.cnblogs.com/kkbill/p/12081172.html)。

### 2.完全背包问题

​	

<img src="images/Acwing Class Note/image-20220609160148256.png" alt="image-20220609160148256" style="zoom: 33%;" />

<img src="images/Acwing Class Note/image-20220609160249022.png" alt="image-20220609160249022" style="zoom:33%;" />

​	最终化简后的状态状态转移方程：`f[i][j] = Max(f[i - 1][j], f[i][j - v[i]] + w[i]) `

​	参考代码：

```C++
for(int i = 1; i <= N; i++)
{ 
    // 这个地方做了优化，是从上一个状态的小背包到当前状态的大背包
    for(int j = v[i]; j <= V; j++)
    {
        f[j] = max(f[j], f[j - v[i]] + w[i]);
    }
}
```



### 3.多重背包问题

​	朴素解法和完全背包问题的朴素解法类似。

​	参考代码：

```C++
for(int i = 1; i <= N; i++)
{
    cin >> v >> w >> s;
    for(int j = 1; j <= V; j++)
    {
        for(int k = 0; k <= s && k * v <= j; k++)
        {
            f[i][j] = max(f[i][j], f[i - 1][j - k * v] + k * w);
        }
    }
}
```

​	使用二进制优化的多重背包问题：

```C++
// 将每种物品根据物件个数进行打包
// 最后相当于有n * log(s)个背包了，即cnt ≈ n * log(s)
int cnt = 0;
for(int i = 1; i <= n; i ++){
    int a, b, s;
    cin >> a >> b >> s;

    int k = 1;
    while(k <= s){
        cnt ++;
        v[cnt] = k * a;
        w[cnt] = k * b;
        s -= k;
        k *= 2;
    }
    if(s > 0){
        cnt ++;
        v[cnt] = s * a;
        w[cnt] = s * b;
    }

}

//多重背包转化为01背包问题
for(int i = 1; i <= cnt; i ++){
    for(int j = m; j >= v[i]; j --){
        f[j] = max(f[j], f[j - v[i]] + w[i]);
    }
}


```



### 4.分组背包问题

<img src="images/Acwing Class Note/image-20220609164458486.png" alt="image-20220609164458486" style="zoom:33%;" />





```C++
for(int i = 1; i <= N; i++)
{
    int a;
    cin >> a;
    for(int j = 1; j <= a; j++)
    {
        cin >> v[j] >> w[j];

    }
    for(int j = 1; j <= V; j++)
    {
        f[i][j] = f[i - 1][j];
        for(int k = 1; k <= a; k++)
        {
            if(j >= v[k]) f[i][j] = max(f[i][j], f[i - 1][j - v[k]] + w[k]);
        }
    }

}
int res = 0;
for(int i = 1; i <= N; i++) res = max(res, f[i][V]);


// 优化后
for(int i = 1; i <= N; i++)
{
    int a;
    cin >> a;
    for(int j = 1; j <= a; j++)
    {
        cin >> v[j] >> w[j];

    }
    for(int j = V; j >= 1; j--)
    {
        for(int k = 1; k <= a; k++)
        {
            if(j >= v[k]) f[j] = max(f[j], f[j - v[k]] + w[k]);
        }
    }
}
```









## 注意事项

1. 关于scanf和printf的使用

   **scanf的效率高于cin，printf的效率高于cout，一般就使用scanf和printf**；但是注意对于double型的输入和输出为**“%lf”**，对于longlong型的输入输出为**“%lld”**，对于unsigned int型的输出为**“%u”**；**注意对于string类型，C++中就直接使用cin和cout了**（如果要使用scanf需要提前给string定义长度，类似于这样`a.resize(100); scanf("%s", &a[0]);`比较麻烦，或者使用char[N]，而且printf没办法直接输出string），而C语言中一般使用字符串数组来弄（注意末尾的'\0'）。

2. **C++中的引用&是什么意思**

