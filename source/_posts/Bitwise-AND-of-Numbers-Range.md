title: LeetCode201 | Bitwise AND of Numbers Range
date: 2015-05-28 14:13:31
tags: 
- 面试题
- LeetCode
categories: 面试题
---
Given a range [m, n] where 0 <= m <= n <= 2147483647, return the bitwise AND of all numbers in this range, inclusive.
For example, given the range [5, 7], you should return 4.

  > C++ Code Interface
  ```c++
  class Solution {
    public:
      int rangeBitwiseAnd(int m, int n) {
      }
  };
  ```

问题分析
------------------------
这道题是说，给定一个数字区间[m, n], 比如说[5, 7], 让我们将**该区间的每一个数字**都做`位与运算 &`, 问结果是多少？对于[5, 7], 该区间的全部数字为5,6,7。将它们做位与运算, 5 & 6 & 7 =
<!--more-->

  >&nbsp;&nbsp; 0**1**01 &
  >&nbsp;&nbsp; 1**1**00 &
  >&nbsp;&nbsp; 0**1**11
  > = 0**1**00
  > = 4

因为m,n的最大范围可能是2^31-1, 所以肯定不能简单的遍历这个区间的每个数，将它们一一做与运算( 复杂度O(N), 百分百TLE)。 那么怎么做呢？ 恩，一般碰到这样的题，我都是先举几个例子，以图发现某种规律，发现规律后做下*不严格的证明* (逻辑上想明白，觉得这样是OK的..不重不漏..**数学证明~证明自己的算法是正确的!**)，然后快速的实现测试。
> 4: 0100
  5: 0101
  6: 0110
  7: 0111
  8: 1000
`[4, 8] = 0`


> 16: 0001 0000
> 17: 0001 0001
> 18: 0001 0010
> .............
> 32: 0010 0000
> .............
> 64: 0100 0000
`0    1   2    4    8    16    32    64   128`
`[16, 56] = 0`
`[16, 31] = 16`
`[20, 31] = 16`

<br/>
  ### 规律

  当区间 **包含** 多于一个2的幂方的时候，结果肯定为0。何为_** '包含' **_?  假设我们将整数区间分段

  |  0  |  1  |  2  |  4  |  8  | 16       |    32    | 64  | ... | ... | 2^30=1073741824 |
  | --- | --- | --- | --- | --- | ---      | ---      | --- | --- | --- |      ---        |
  |     |     |     |     |     | &nbsp; * |&nbsp;  * |     |     |     |        |        |


  对于m,  我们能找到一个**k**, 满足`2^k <= m && 2^(k+1)>m`; 比如说[20, 31], m=20, 这个k等于4。即2^4=16 <= 20 而2^5=32 > 20;
  同理，对于n， 我们能也找到一个p, 使得2^p<=n && 2^(p+1)>n.
  该区间包含的2的幂方的个数等于p-k+1. 只有当k和p相同时，结果才可能不为0;    那么当k==p时(即m和n落在了同一段上), 答案是多少呢,  结果是`2^k`么?
  一句话：    包含两个以上2的幂，答案就是0; 否则答案就是2^k?

** 室友给出的反例: **
>  5: 0101
   6: 0110
  `[5, 6] = 4`  OK没问题. 但是! <br/>
   6: 0110
   7: 0111
  `[6, 7] = 6` 所以当[m,n]只包含一个2^k时，答案并不一定是2^k, 还有其它情况。(目前能肯定的是当包含多个2^k，答案一定是0)

** 进一步思考，得到最终方案: **

  恩，刚才的分段的想法是没有问题的。答案是肯定包含2^k，这点确信。 对于剩下的部分, 得到新的 ** '子' ** 区间[m-2^k, n-2^k], 可重复上面的思路。如
> 6: 0110
  7: 0111
  `ans = 4  新区间[6-4, 7-4]=[2,3]`
  2: 0010
  3: 0011
  `ans += 2 新区间[2-2, 3-2]=[0,1]`

上述想法的特点在于，找出了区间里每个数字的二进制表达中都含1的那一列对应的2^k, 将它们累加起来即是结果.   6=0110 7=0111  ( 0110 )
从'高位'开始, 向低位一一判断..累加或停止( eg: [39, 49] --> [7, 17] )。 具体参见 [Naive Solution](#Naive_Solution)
<br/>

### 规律进一步提炼，优化
上述方案，细想一下，本质就是** 找出m与n的二进制表达中，相同的前k位 ** (最高相同的2^k, 次高位相同的2^....)
假设m=391=(11000111), n=395=(11001011). 方案一，就是找出了高位的1100
![](http://7xjbmr.com1.z0.glb.clouddn.com/BitwiseANDofNumbersRange1.png)
既然问题转换成 找到m与n的高位相同的前k位。 那么咱用移位运算就可以轻松得到答案啦……当m与n不相等时，就将m,n左移一位。如此反复，当m==n时，就代表此时他们的'前k位'是相同的咯; 乘以刚才左移次数的2次幂，即是结果。
具体参见 [Better Solution](#Better_Solution)
<br/>

### 从另一个角度入手...
 回顾一下题目:
> 给定一个数字区间[m, n], 比如说[5, 7], 让我们将**该区间的每一个数字**都做`位与运算 &`, 问结果是多少？

结果当然等于： `result = n  &  n-1 & n-2 & n-3 & n-4 & n-5 & ......  x & x-1 & x-2 & .... & m-1  &  m  ` -.- 我靠，暴力法嘛。要你说啊.
看观别急. 咱马上可以看到一些有趣的东东. 先思考两个问题:
> 1) n & n-1有什么特点?
  2) 假设 x = n & n-1,  那么 n & n-1 & n-2 & .... &x 等于什么?

来个例子[33, 58]，咱一起看看:
假设n=58=11 1010, m=33=10 0001, 显然结果等于` 58 & 57 & 56 & 55 & 54 & .... & 48 & 47 ..... & 33`

> &nbsp;&nbsp;   58: 11 10**1**0
  & 57: 11 1001
  **= 56: 11 1000**

>  &nbsp;&nbsp;   56: 11 **1**000
  & 55: 11 0111
  **= 48: 11 0000**

* 问题一: n & n-1有什么特点?
  答案:  n & n-1 能将n的二进制表达中最后一个1消去，当n>=1时. ( 如果想不明白，可参考[求二进制数中1的个数-快速法][2] 和[Number of 1 Bits][4])

* 问题二：假设 x = n & n-1,  那么 n & n-1 & n-2 & .... &x 等于什么?
  答案: n & n-1 & n-2 & ... & x 等价于 n & n-1.   为什么?  假设n=56, 可知n-1=55,  x=n&n-1=56&55=48
  > &nbsp;&nbsp;   56: 11 **1**000
  & 55: 11 0111
  >
  & 54: 11 0110
  & 53: 11 0101
  & 52: 11 0100
  & 51: 11 0011
  & 50: 11 0010
  & 49: 11 0001
  >
  & 48: 11 0000

  >**= 48: 11 0000**
  ![](http://7xjbmr.com1.z0.glb.clouddn.com/BitwiseANDofNumbersRange2.png)

  HOHO,咱用**跳跃**暴力法，将区间中所有的数字都做位于运算，得到结果吧. 具体参见 [Best Solution](#Best_Solution)

<br/>

解决方案
------------------------
### Naive Solution

  #### 算法复杂度分析
  假设[m,n]中包含N个数, 暴力方法是将区间中的每个数进行位于运算，得到结果, 时间复杂度是O(N);
  而该方案是，将正整数集先按照2的幂分段，得到的 **时间复杂度** 是`O( logN * LogN )`。最坏情况m=n=2147483647, 二进制全为1。
  **空间复杂度** 为`O( logN )`

  #### 代码实现
  ```c++
  class Solution
  {
    public:
      int rangeBitwiseAnd(int m, int n)
      {
          if ( 0 == m) return 0;

          vector<int> twoPowerVec;

          twoPowerVec.push_back(0);
          for (int i=0; i<=30; i++)
            twoPowerVec.push_back( 1 << i);
          //printVector(twoPowerVec);

          // 0, 2^0, 2^1, .... 2^30  (Total: 32)
          int ans = 0;
          while(m > 0)
          {
            int k=0, p=0;

            // find the first k to sastify: 2^(k) > m
            while (k<=31 && twoPowerVec[k]<=m) k++;
            k--;

            while (p<=31 && twoPowerVec[p]<=n) p++;
            p--;

            //cout << "k= " << k << " p= " << p << endl;

            // the interval includes the power of 2  more than one
            if (p > k) return ans;

            ans += twoPowerVec[k];

            m -= twoPowerVec[k];
            n -= twoPowerVec[k];
        }

        return ans;
      }
  };
  ```

<br/>

### Better Solution
  #### 算法复杂度分析
  该方法，将不满足要求的低位一位位的移出去，得到的 **时间复杂度** 是`O( logN )`。**空间复杂度** 为`O( 1 )`

  #### 代码实现
  ```c++
  class Solution
  {
  public:
    int rangeBitwiseAnd(int m, int n)
    {
        int offset = 0;
        while (m>0 && m!=n)
        {
            m >>= 1;
            n >>= 1;
            offset++;
        }
        return m << offset;
    }
  };
  ```

<br/>

### Best Solution

  #### 算法复杂度分析
  该方法得到的 **时间复杂度** 是`O( logN )`。**空间复杂度** 为`O( 1 )`
  虽然在最坏情况下，该方法与上面的解法复杂度是一样的。但该方法取决于`n的二进制表达中1的个数`和M的值~平均下来更快一些, 代码更简短。

  #### 代码实现
  ```c++
  class Solution
  {
  public:
    int rangeBitwiseAnd(int m, int n)
    {
        while ( n > m ) n &= n-1;
        return n;
    }
  };
  ```
### 小结( 算法上复杂度!=运行效率)
  虽然从上面3个解决方案来看，第3种方法最巧妙，复杂度一般来说也低一些。但在LeetCode运行时间最少的代码是 **非优化版本的第2种方法**(while循环条件中不添加m>0).为什么呢？   可能是因为 左移>> 右移<<在底层执行的更快一些，而方法3中的减法，大小比较等基本运算可能花的时间更多一些。
  That's All. 不深究了。 此处想说明的是， 算法上复杂度最好的并不保证代码运行就最快；以及，同一个算法，不同的人写的具体实现，效率也可能不同。
<br/>


测试用例
------------------------
> Input: [m, n] = [0, 0]
  Expect: 0

> Input: [m, n] = [2147483647, 2147483647]
  Expect: 2147483647

> Input: [m, n] = [0, 2147483647]
  Expect: 0

> Input: [m, n] = [5, 7]
  Expect: 4

> Input: [m, n] = [6, 7]
  Expect: 6

> Input: [m, n] = [7, 8]
  Expect: 0

> Input: [m, n] = [111, 111]
  Expect: 111

参考
------------------------
  1. [LeetCode题目链接][1]
  2. [算法-求二进制数中1的个数][2]
  3. [他人对该题的分析、总结][3]
  4. [LeetCode相关题目][4]

[problemUrl]: https://leetcode.com/problems/bitwise-and-of-numbers-range/
[1]: https://leetcode.com/problems/bitwise-and-of-numbers-range/
[2]: http://www.cnblogs.com/graphics/archive/2010/06/21/1752421.html
[3]: http://www.meetqun.com/thread-8769-1-1.html
[4]: https://leetcode.com/problems/number-of-1-bits/
