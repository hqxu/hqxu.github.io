title: LeetCode143 | Reorder List (小米面试题)
date: 2016-03-17 23:12:10
tags: 
- 面试题
- LeetCode
categories: 面试题
---
小米2面, 聊完项目和机器学习后面试官出的编程题。将链表按照一定的方式改变顺序，题目是**假设输入的单链表为L1 -> L2 -> .... -> Ln-1 -> Ln, 改变其顺序，输出为L1 -> Ln -> L2 -> Ln-1 -> .....**
面试结束后回去总结时，得知这是LeetCode上面一道关于链表的题，挺经典的，适合考察基本功，故记录在此。

面试时，我想到了两个方案。
<!--more-->

  >1. 引入一个指针数组记录每个Node, 然后遍历这个指针数组改变每个Node的next域, 该法时空复杂度都是O(N).

  >2. 另一个方案，就是两重循环，每次遍历都找到剩余链表中的最后两个结点，然后改变它们的next区域。 新链表L1->Ln不断的扩大，剩余链表L2->...Ln-1不断的被切掉首尾.时间复杂度为O(N^2), 空间复杂度为O(1).

嗯，两种不同的方案，典型的时空trade-off. 当时面试官让我手写第1种方案，事后提交到LeetCode上面快速AC. 后看到别人总结，才得知`这题标准的解法`是(面试官期望的答案?):
  >3、利用快慢指针找到链表的中点，然后将链表一分为二，将后半段逆序，最后往前半段挨个插入。时间复杂度是O(N), 空间复杂度为O(1).


`快慢指针`: 第一次遇见这个概念，是在学习数据结构实现`链式归并排序`时~其中一步就是需要找到中点，从而将整个链表一分为二，然后前半段做归并，后半段做归并，如此递归下去到一个结点。后面再见就是做LeetCode上面的题目了，如判断一个链表里面是否存在环等等.
  ```c++
  ListNode *getMiddleNode(ListNode *head)
  {
      ListNode *fast, *slow;
      fast = slow = head;
      while (NULL!=fast && NULL!=fast->next)
      {
        slow = slow->next;
        fast = fast->next->next;
      }
      return slow;
  }
  ```
快慢指针实现时唯一需要注意的就是考虑链表长度为奇数或偶时，是否需要保证前半段的结点个数大于等于后半段，以及循环结束条件。


这次的面试收获
------------------------
1) 对于一个编程题，如果面试官没有过多的要求说明，那你就往简单的方面考虑! 实现后然后再看情况。别自己加难度! 毕竟时间短。 所以我立即想到了**利用额外数组** `短平快`

2) 在动手写之前，最好跟面试官交流互动一下。把自己的想法跟他说。确定OK后再动手写。因为没有时间可浪费的! 就10分钟

3) 面试之前，着实需要做下LeetCode，刷手感! 尽管基础不错，但太久没写题，熟练度、思维各方面都不行，面试时就郁闷了……明明很简单，却会出错。

问题描述
------------------------
Given a singly linked list L: L0→L1→…→Ln-1→Ln,
reorder it to: L0→Ln→L1→Ln-1→L2→Ln-2→…

You must do this in-place without altering the nodes' values.
For example,
Given {1,2,3,4}, reorder it to {1,4,2,3}.


  > C++ Code Interface
  ```c++
  /**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */

  class Solution {
    public:
      void reorderList(ListNode* head) {
      }
  };
  ```

<br/>

解决方案
------------------------
### Naive Solution
  #### 算法复杂度分析
  方案2), 面试时最开始想到的两重循环。嗯，每次遍历都找到剩余链表中的最后两个结点，然后改变它们的next区域。 新链表L1->Ln不断的扩大，剩余链表L2->...Ln-1不断的被切掉首尾。**时间复杂度** 为`O( N^2 )`，**空间复杂度** 为`O( 1 )`, **运行情况**为 `TLE(超时)`。

  #### 代码实现
  ```c++
  class Solution 
  {
  public:
    // 方法2.暴力 时间复杂度O(N^2)
    void reorderList(ListNode* head) 
    {
      // special case
      if (NULL==head || NULL==head->next || NULL==head->next->next) return ;

      ListNode *first, *second;
      ListNode *pre_rear, *rear;

      first = head;
      second = head->next;
      
      // at least having three node
      while (NULL!=second && NULL!=second->next)
      {
        pre_rear = first;
        rear = second;          // point last node
        while (rear->next != NULL)
        {
          pre_rear = pre_rear->next;
          rear = rear->next;
        }

        // separate the last node and link it to new list
        pre_rear->next = NULL;
        first->next = rear;
        rear->next = second;

        // update the first node and second node for old list
        first = second;
        second = first->next;
      }
    }
  };
  ```

<br/>

### Better Solution
  #### 算法复杂度分析
  方案1)，面试时写在纸上的解决方案。引入一个指针数组记录每个Node, 然后遍历这个指针数组改变每个Node的next域。**时间复杂度** 是`O( N )`, **空间复杂度** 为`O( N )`, **运行情况**为 `69ms`。

  #### 代码实现
  ```c++
  class Solution 
  {
  public:
    void reorderList(ListNode* head) 
    {
      // special case
      if (NULL==head || NULL==head->next || NULL==head->next->next) return ;

      // 1. get the length of the list
      int n = 0;
      ListNode *p = head;
      while ( NULL != p )
      {
        n++;
        p = p->next;
      }

      // 2. create a pointer array for access each node easily
      ListNode **pA;
      pA = new ListNode *[n];

      int i;
      p = head;
      for (i=0; i<n; i++)
      {
        pA[i] = p;
        p = p->next;
      }

      // 3. change each node's next value
      int j;
      for (i=0,j=n-1; i<j; i++,j--)
      {
        pA[i]->next = pA[j];
        pA[j]->next = pA[i+1];
      }
      pA[i]->next = NULL;

      delete [] pA;
    }
  };
  ```

<br/>

### Best Solution

  #### 算法复杂度分析
  方案3) 首先利用快慢指针找到链表的中点， 然后将链表后半段逆序，最后将前半段的指针与后半段的指针连接起来。该方法得到的**时间复杂度** 是`O( N )`, **空间复杂度** 为`O( 1 )`, **运行情况**为 `80ms`。

  #### 代码实现
  ```c++
  class Solution 
  {
  public:
    ListNode *getMiddleNode(ListNode *head)
    {
      ListNode *fast, *slow;
      
      fast = slow = head;
      while (NULL!=fast && NULL!=fast->next)
      {
        slow = slow->next;
        fast = fast->next->next;
      }

      return slow;
    }

    ListNode* reverseList(ListNode *head)
    {
      ListNode *p = head;
      ListNode *q = p->next;
      p->next = NULL;

      ListNode *r;
      while ( NULL != q )
      {
        r = q->next;
        q->next = p;

        p = q;
        q = r;
      }

      return p;
    }

    void reorderList(ListNode* head) 
    {
      // special case
      if (NULL==head || NULL==head->next || NULL==head->next->next) return ;

      // step1. get the middle node. if the lenght of the list is even, the middle choose right side. eg: [1 2 3 4], middle=3
      ListNode *middle;
      middle = getMiddleNode(head);

      ListNode *firstList = head;
      ListNode *secondList = middle->next;
      middle->next = NULL;

      // step2. reverse the second list( All after middle node in the origin list)
      secondList = reverseList(secondList);

      // step3. insert the second list's node to the first list
      ListNode *f_p, *f_q;
      ListNode *s_p, *s_q;

      f_p = firstList;
      f_q = firstList->next;

      s_p = secondList;
      s_q = secondList->next;
      while ( NULL != s_p )
      {
        f_p->next = s_p;
        s_p->next = f_q;

        f_p = f_q;
        if ( NULL != f_q ) f_q = f_q->next;

        s_p = s_q;
        if ( NULL != s_q ) s_q = s_q->next;
      }
    }
  };
  ```
<br/>


测试用例
------------------------
> Input: 1 2 3 4 5
  Expect: 1 5 2 4 3

> Input: 1 2 3 4
  Expect: 1 4 2 3

> Input: 1
  Expect: 1

参考
------------------------
  1. [LeetCode题目链接][1]
  2. [源代码传送门][2]
  3. [他人对该题的分析、总结][3]

[1]: https://leetcode.com/problems/reorder-list/
[2]: 
[3]: http://www.acmerblog.com/reorder-list-leetcode-6088.html
