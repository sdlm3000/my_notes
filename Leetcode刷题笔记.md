# Leetcode刷题笔记

账号：18810202669

密码：leetcode1881

[TOC]

## 数组

### 1.两数之和

#### 排序+双指针

​	[详见C++中的解法](https://leetcode-cn.com/problems/two-sum/)。

​	先使用库中的排序函数（快排）进行排序，然后双指针的方法，这种方法的总的时间复杂度为O(NlogN)，因为要输出原来的数组的下标，所以过程比较复杂，需要用一个数组存原来的数据，然后双指针找到数之后，还需要和原来数组进行匹配找到对应的下标，空间的开销比较大。

​	时间复杂度：O(NlogN)

#### 哈希表

​	详见C中的解法。

​	不考虑空间上的开销，按大于数组最大长度的两倍的最小的质数为长度开两个数组来模拟哈希表，一个存放值，一存放值对应的index，使用开放寻址法来构造哈希表，具体见《Acwing Class Note》中的哈希表的笔记。然后思路就比较简单了，只需要遍历N次，每次判断哈希表中是否有target-nums[i]这个值，没有就把nums[i]和i插入哈希表中，有就输出i和哈希表中target-nums[i]所在位置的对应的index的值。一般每次都需要使用两次find函数。

​	时间复杂度：$O(N)$



### 2.寻找两个正序数组的中位数

​	[详见C中的解法](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)，比较典型的双指针的题目。

​	使用从头开始的双指针把原来两个数组有序拼接起来，因为要求找到中位数，所以最后不需要全部拼接，而是拼接到(nums1Size + nums2Size) / 2 + 1就可以了；同时对于奇数序列和偶数序列的问题，一般使用判断的方法，但是也可以使用如下方法，将奇数和偶数序列的情况都包含；还有就是需要注意整型要强转一下。另外一点如果malloc后的变量为一个中间值的话，在函数末尾free一下吧。

```C
# 奇数序列是两个index一样，偶数序列两个index为对应的两个中间值
mid = (nums[(nums1Size + nums2Size) / 2] + nums[(nums1Size + nums2Size - 1) / 2]) / 2.0;
```



### 3.盛最多水的容器

​	[详见C中的解法](https://leetcode-cn.com/problems/container-with-most-water/)，解法也类似双指针，比较简单。

​	使用分别从头尾开始的双指针，然后计算体积，判断更新最大体积，然后将值较小对应的指针往里移动一位，不断计算更新，直到指针相遇。



### 4.接雨水

​	[详见C中的解法](https://leetcode-cn.com/problems/trapping-rain-water/)，双指针。

​	和上题类似，但是计算体积时需要减去内部的空间，然后指针往里更新（高度较低对应的指针更新），然后再以上次的高度为基准，计算体积，同时也需要减去内部空间。



### 5.[最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)

​	动态规划。

​	比较简单，每次更新之和上一个状态有关系，所以最后可以把空间复杂度优化O(1)。

​	时间复杂度：$O(N)$			空间复杂度：O(1)



### 6.[三数之和](https://leetcode-cn.com/problems/3sum/)

​	详见C++中的解法，排序 + 双指针。

- 首先使用快排将数组排序，因为题目为三数之和，理论上需要三个指针，我一开始的思路是使用两个指针从头尾逐步往中间移动，第三个指针在两者之间动态遍历，但是这种方式不能解决这种情况 [-4 -2 -2 0 2 2 4]，即存在两数相等+零的情况是，头尾的指针不管怎么更新都可能会漏结果。最终的双指针解法为，在左侧固定一个指针 p，然后右侧剩下的数据使用头尾双指针的方法，往中间更新遍历，遍历结束后 p往左移动一位，直到完全遍历。

- leetcode的官方题解中的双指针更好一些，很好的利用了排序后数据的单调性，比我的代码要少很多判断条件，要更加简洁，甚至速度更快。

​	需要注意的几点：

1. 题目要求不可以包含重复的三元组，所以需要去除相等的数，一般使用这样的循环`while(j > i && nums[j] == nums[--j]);`，需要注意边界条件和 ++ / -- 需要放在后面
2. 可以使用一些中间变量，较少一些运算操作，如：target。



直接看[题解](https://leetcode.cn/problems/3sum/solution/pai-xu-shuang-zhi-zhen-zhu-xing-jie-shi-python3-by/)

### 7.[最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest/)

​	详见C++解法中网页的笔记，排序 + 双指针。

- 个人解法：首先使用排序，然后使用双指针，pi指向左边界，pj指向右边界，pk在两者之间从左开始移动，因为pk移动的过程中数据和是递增的，只要和与target的偏差变大，就可以跳出这层循环；然后pj左移一位，pk重新开始类似的循环，直到pj也遍历完了；pi右移一位，pj和pk重新开始遍历循环。

  这种方法本质上还是类似于纯粹的暴力解法，只是利用了排序后数据的单调性，排除了很多情况，实际的运行时间不尽如人意。思路上更类似于[三数之和](https://leetcode-cn.com/problems/3sum/)中的官方题解的方法，**但是这两个题目的差别在哪里？**

- leetcode官方解法思路类似于[三数之和](https://leetcode-cn.com/problems/3sum/)中我的个人思路，

​	现在在看没动自己这两题的思路，直接看官方题解吧，基本思路都是i指针从左往右遍历，然后设置两个动态双指针`j = i + 1,k = n - 1` ，每次都遍历这个数组，遍历完一次i++。然后需要注意去重问题



### 8.[买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

​	详见C++解法。

- 暴力解法会超时，**感觉我的解法有点类似于单调栈的更新思路？**；官方给的题解的思路和我的一样，但是代码似乎更好一些。

- **可以在思考一下这道题动态规划怎么去解。**

​	时间复杂度：$O(N)$			空间复杂度：O(1)



### 9.[除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self/)

​	详见C++解法。

- 暴力解法会超时。

- 参考官方解法的左右乘积后感觉这道题很简单吧，而且似乎这种方法也可以叫做动态规划，L和R数组就类似于在存储一个一维的状态量，它与它上一个状态相关。

​	时间复杂度：$O(N)$			空间复杂度：O(1)



### 10.[柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

​	详见C++解法。

- 我的暴力解法为找到每种情况的长和宽，时间复杂度为$O(N^2)$
- 官方给出的暴力解法思路为以每个数进行遍历，以它自己为高，找到这个数左边离它最近且比它小的数，和右边离它最近且比它小的数，然后就可以得到长，然后就可以计算得到以它为高的最大的矩形的面积。如果寻找左右离它最近而且小于它的数使用暴力解法的话，总的时间复杂度也是$O(N^2)$。
- 单调栈优化寻找左右离它最近而且小于它的数。这本身也是单调栈的常见应用场景，总的时间复杂度就降到$O(N)$。



### 11.[ 多数元素](https://leetcode-cn.com/problems/majority-element/)

​	详见C++解法笔记。

- 哈希表解法，好像使用手撸的开放寻址法和STL库中的unordered_map其实差别并不大，甚至手撸的开放寻址法要更快一点。注意开放寻址法只需要找数应该在哪里，而不用实际把书存放到h[N]中，同时需要开辟一个index[N]来计数。时间复杂度为$O(N)$，空间复杂度为$O(N)$。

- 摩尔投票法，很巧妙，思路为一换一，最后多数元素肯定会至少剩下一个。时间复杂度为$O(N)$，空间复杂度为$O(1)$。

  

### 12.[加一](https://leetcode-cn.com/problems/plus-one/)

​	详见C++解法笔记。

- 正确理解题意后就很简单。单个数字就是0~9，就太简单了



### 13.[合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)

​	详见C++解法笔记。

- 典型的双指针题目，但是我的循环思路有点乱，边界条件有点多呀。时间复杂度为$O(N+2M)$，空间复杂度为$O(M)$。
- 官方解法的逆向双指针的解法就很好，因为nums1本身就是N+M的长度，就选大的从nums1的尾部合并，这样不会覆盖掉nums1本身的数值了。也可以学一下他双指针合并的方式，比较简洁好懂。时间复杂度为$O(N+M)$，空间复杂度为$O(1)$。



### 14.[买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

​	详见C++解法。

- 知道这道题应该要用动态规划来做，但是没有想出来应该怎么进行状态表示和状态转移，自己做没做出来，，，
- 参考官方解法后，醍醐灌顶，原来可以设置两个状态量一个表示之前没有买入股票的情况，一个表示之前已经买入股票的情况，他们的状态转移方程看代码后就比较好理解。而且都是只和上一个状态有关的，所以最终的可以将空间复杂度优化为$O(1)$，时间复杂度为$O(N)$。
- 同理买股票问题1也可以是类似的动态规划做（我自己第二遍做还是没有此想出来，，），区别就在于 lf0 和 lf1 的状态更新上。



### 15.[合并区间](https://leetcode-cn.com/problems/merge-intervals/)

​	详见C++解法。

- 合并区间模板题，详见《算法基础课》，时间复杂度为$O(N)$，空间复杂度为$O(1)$。



### 16.[柱状图中最大的矩形](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

​	详见C++解法。

- 纯暴力解法，两层循环，计算每次的长，寻找区间内的最小值作为宽，计算矩形面积，然后更新最大值，时间复杂度为$O(N^2)$，空间复杂度为$O(1)$。
- 官方的暴力解法，默认以数组中的每个元素为高，遍历寻找数组中每个数左边和右边的边界，来计算长，最终遍历计算矩形面积，更新最大值。时间复杂度为$O(N^2)$，空间复杂度为$O(N)$。
- 使用单调栈优化上述暴力解法，上述暴力解法的关键在于求每个元素左边和右边离它最近且小于它的值的位置，很显然可以使用单调栈来求解，只需要从左至右和从右至左用单调栈走一遍就可以得到，则每个高对应的长就可以求到了。时间复杂度为$O(N)$，空间复杂度为$O(N)$。



### 17.[最大矩形](https://leetcode-cn.com/problems/maximal-rectangle/)

​	详见C++解法。

- 纯粹的暴力解法必然超时。
- 本题为16题的扩展题，难度较大，基本思路是对每一行进行遍历，每一列看成类似于16题中的柱子，对每一行进行遍历时，遇到1则柱长加1，遇到0就直接将柱长清零，然后对每一行调用一次16题的选择柱状图中的最大矩形。



### 18.[移动零](https://leetcode-cn.com/problems/move-zeroes/)

​	详见C++解法。

- 比较简单，可以使用类似双指针的算算法实现一次遍历。



### 19.[数组拆分 I](https://leetcode-cn.com/problems/array-partition-i/)

​	详见C++解法。

- 很简单，就是排序过后的数组对应的值最小。



### 20.[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

​	详见C++解法。

- 暴力解法，一次遍历然后匹配目标值。时间复杂度为$O(N)$，空间复杂度为$O(1)$。
- 二分查找，可以在仔细看一下这种**非完全有序的数组**如何进行二分查找。



### 21.[旋转图像](https://leetcode-cn.com/problems/rotate-image/)

​	详见C++解法。

- 我的暴力解法，每对应的四个数，进行三次交换，完成一次旋转，需要注意每个数的坐标表示，这个地方会比较绕，也可以使用四数交换的方式取代目前的两两swap的方式。	



### 22.[下一个排列](https://leetcode-cn.com/problems/next-permutation/)

​	详见C++解法。

- 我的解法思路行不通。
- 见官方解法思路，然后我自己的代码编写还可以在精简一点。



### 23.[生命游戏](https://leetcode-cn.com/problems/game-of-life/)

​	详见C++解法。

- 元胞自动机，纯粹暴力解法，能过。
- 官方说的**原地算法**可以看一看。



### 24.[缺失的第一个正数](https://leetcode-cn.com/problems/first-missing-positive/)

​	详见C++解法。

- 自己没有想出来解法。
- 官方解法，需要注意到缺失的第一个正数一定在$[0,N+1]$中，然后如果单纯的使用哈希表查找$[0,N+1]$，那么空间复杂度就会超，所以解决方法是使用原数组打标记的方法，首先将负数和零都换成$N+1$，然后遍历数组把每个对应的数字减一作为数组的下标，将该下标对应的数字标记为负数，如果不在数组下标范围内就不用管了，然后最后最前面出现的正数的下标加一就是缺失的第一个正数，如果没有就是缺失的正数就是$N+1$。里面还需要注意要使用绝对值。时间复杂度为$O(N)$，空间复杂度为$O(1)$。
- **需要再理解一下哈希表**



### 25.[最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

​	详见C++解法。

- 自己在构思哈希表的解法是有了初步的雏形，但是开始的想法是往两头都检索判断，思路就乱了。
- 官方解法，使用哈希表将数据存入，然后遍历数组，检索到`nums[i]-1`在哈希表中则`continue`，检索到`nums[i]+1`在哈希表中继续叠加查找，同时记录序列的长度。这种方法就很好的避免了我的问题，同时也保证只会连续序列的最小值开始往上查找。时间复杂度为$O(N)$，空间复杂度为$O(N)$。
- 官方解法中为什么`for(int num : num_set)`会比`for(int num : nums)`和`for(int i = 0; i < n; i++)`的方法快很多，**10倍左右**？



### 26.[数组中重复的数据](https://leetcode-cn.com/problems/find-all-duplicates-in-an-array/)

​	详见C++解法。

- 与24中缺失第一个正数的思路类似，利于它数组的规律，取`flag = n+1`作为标志位，然后遍历数组，以每个数减去一作为索引，将其加上`flag`，即`nums[nums[i] % flag - 1] += flag`，这里需要注意对nums[i]的取余操作，然后再对处理后的数组进行遍历，如果`nums[i] / flag == 2`，则`i + 1`为重复出现过两次的数字。

- 题解中，用负数做标记感觉很简单一些，第二次检索到的数据为负数时，说明它出现过两次。



### 27.[乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/)

​	详见C++解法。

- 我的解法，一开始没有形成思路，总觉得如果中间出现两个负数的情况动态规划没办法解决，就用来纯粹的暴力解法，也能过。之后动态规划想到多开一个数组存放最小值。

  f_max[i]状态表示：包含nums[i]的乘积最大的子数组

  f_min[i]状态表示：包含nums[i]的乘积最小的子数组

  状态转移：

  ```C++
  f_min[i] = min(min(f_max[i - 1] * nums[i], f_min[i - 1] * nums[i]), nums[i]);
  f_max[i] = max(max(f_max[i - 1] * nums[i], f_min[i - 1] * nums[i]), nums[i]);
  ```

  同样这道题可以从优化成空间复杂度为$O(1)$的，详见C++中的解法笔记。

  **对于这种类型的，是子数组的，即连续的，一般使用动态规划时，状态表示时都是类似于f[i]中一定包含nums[i]的**。



### 28.









# 华为108

### 1. 质因子问题















# 问题总结

## C++语法相关

### 1. C++编写相关

```c++
// C++的代码空模板
#include <iostream>
using namespace std;
int main(){
    
    return 0;
}

// 常用库函数
#include <cmath>
	sqrt()
        
#include <algorithm>
        


```

### 2. 输入输出相关

​	C++中使用`cin >> string/char`或者`scanf("%s", char)`（scanf不支持string），都默认以**空格**或者**换行符**作为输入的结尾，对于字符串`hello nowcoder`，它们都只能获取到`hello`。如果想完全获取数据，有以下几种方法：

- 使用`while(cin >> string);`获取每个以空格分开的单词，需要自己拼接。注意`while(scanf("%s", char));`的形式不成立；
- 使用`getline(cin, str);`的方法获得整行数据；
- 使用`a = getchar();`接收每个字符，当遇到`\n`时停止接收输入；
- 使用`scanf("%[^\n]", word);`的方法获取整行数据。其中`^`表示任意字符串，只有在遇到字符`\n`时，输入才会截止。

​	其中第三种和第四种在C语言中也适用

### 3. 引号问题

​	单引号 ‘a’  ‘A’ 用于指代字符a  A ,  一般用法if(word[i] == 'a') 判断是否为字符a ,  

​	双引号”a" “A” 则用于指代字符串、常量, 一般用法 const char * = "abdcd";

### 4. 字符串常用操作





### 5. 常见ASCII码

- '0'~'9' : 48~57
- 'A'~'Z' : 65~90
- 'a'~'z' : 97~122



### 6. 常用函数

1. `tolower(char c)`：将单个字符转化小写，同理有`toupper(char c)`
2. 

​	



### 7. sort函数的使用

​	添加自定义的比较函数，如下所示：

```C++
const int N = 100010;
typedef struct _myPair
{
    int l;
    int r;
}myPair;
// 按结构体中r的数值从小到大排序
bool LLess(const myPair &R1, const myPair &R2)
{
    return R1.r < R2.r;
}
myPair R[N];

sort(R, R + N, LLess);
```



### 8. 优先队列

​	C++中优先队列的使用：

```C++
#include <queue>

// 小根堆
priority_queue<int, vector<int>, greater<int>> heap1; // 升序
// 大根堆
priority_queue<int, vector<int>, less<int>> heap2;	// 降序

top()		// 访问队头元素
empty()		// 队列是否为空
size()		// 返回队列内元素个数
push()		// 插入元素到队尾 (并排序)
pop()		// 弹出队头元素
emplace()	// 原地构造一个元素并插入队列
swap()		// 交换内容
```

​	那么我们可以总结出关于大根堆和小根堆的结论：

（1）堆是一棵完全二叉树；
（2）小根堆的根节点是堆中最小值，大根堆的根节点是堆中最大值；
（3）堆适合采用顺序存储。



### 链表题目相关

​	有时候为了方便处理，可以new一个新的结点作为原链表的头结点：

```c++
ListNode* dummyHead = new ListNode(0);
dummyHead->next = head;
...
return dummyHead->next;
```

