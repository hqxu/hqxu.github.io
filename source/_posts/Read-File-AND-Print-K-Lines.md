title: 打印输入文件的最后K行 (循环队列)
date: 2016-03-23 22:05:13
tags: 面试题
categories: 面试题
---
题目要求就是读入一个文件，打印出这个文件的最后K行的内容 (要求: **只能读取文件一次**)
<!--more-->
问题分析
------------------------
最简单的方案，就是先遍历一下文件，统计出行数N。然后打印出N-K+1行到第N行的内容。但是这么一来，就需要读两遍文件。众所周知, 相比内存, 文件IO是比较耗时的。

面试官要求只能读取文件一次，在此想到数据结构中的`循环队列`。嗯， 我们可以使用一个数组，存放从文件读取到的所有K行和最后的K行。因此，这个数组起初包含的是1~K行，然后是2~K+1行，接着是3~K+2行，以此类推。每次读取新的一行，就将数组中最早读入的那一行清掉。用空间换时间（trade-off)

    相关知识点: 循环队列(circular queue); 循环式数组(circular array);

解决方案
------------------------
  ### 算法复杂度分析
  该方案只需读取文件一次，并且任意时刻只会在内存里存放K行内容。**时间复杂度**为`O( N )`,  **空间复杂度**为`O( K )`

  ### 代码实现
  ```c++
  const int K = 10;
  void printLastKLine(string fileName)
  {
    ifstream infile(fileName);
    string lines[K];                 // vector<string> lines;
    int size = 0;

    // 逐行读取文件，并存入循环式数组
    while( getline(infile, lines[size++ % K]) ) ;

    // 计算循环式数组的开头和大小 (考虑总行数size小于K的情况)
    int start = size>K ? (size%K) : 0;
    int count = min(K, size);

    // 根据读取顺序，打印数组元素
    for (int i=0; i<count; i++)
    {
        cout << lines[ (start+i) % K] << endl;
    }
  }

  ```

循环队列
------------------------
队列（queue）是只允许在一端进行插入操作，而在另一端进行删除操作的线性表。是一种先进先出的线性表（FIFO）。 允许插入的一端称为队尾(rear)，允许删除的一端称为队头(front)。相比栈(stack), 栈操作的top指针在push时增大而在pop时减小，栈空间是可以重复利用的，而队列的front、rear指针都在一直增大，虽然前面的元素已经出队了，但它所占的存储空间却不能重复利用(`假溢出问题`)。但大多数程序并不是这样使用队列的，**一般情况下出队的元素就不再有保存价值了，这些元素的存储空间应该回收利用，由此想到把队列改造成环形队列**(`Circular queue`): 把queue数组想像成一个圈，front和 rear指针仍然是一直增大的，当指到数组末尾时就自动回到数组开头，就像两个人围着操场赛跑，沿着它们跑的方向看，从 front到rear之间是队列的有效元素，从rear到front之间是空的存储位置，如果front追上rear就表示队列空了，如果rear 追上front就表示队列的存储空间满了。故一般我们将queue实现为循环队列，当出队列时就不需要全部进行移动，只需要修改 队头指针，从而解决“假溢出”的问题。

注1：为了避免队列空和满两个状态混淆，采用空闲一个位置的方式，即N个元素空间的循环队列最多只能存放N-1个有效元素。
     可参考这个说明：http://www.nowamagic.net/librarys/veda/detail/2351

注2: 循环队列的提出，是为了有效的利用空间，解决了队列的"假溢出"问题。但要注意的是循环队列仍可能面临数组溢出的问题。
    - 循环队列初始化：front=rear=0
    - 入队操作：rear=(rear+1)%size
    - 出队操作：front=(front+1)%size
    - 队列长度: (rear-front+size)%size
    - 判断是否为空队列：front==rear
    - 判断队列是否已满：front==(rear+1)%size


```c++
#include <iostream>  
using namespace std;  

/*循环队列的类型定义*/  
const int Queue_Size=100;  
typedef struct circularQueue  
{  
    char *elem;  
    int front;
    int rear;  
    int size;  
}circularQueue;


// 入队  
void push(circularQueue &Q, char x)  
{  
    if( isFull(Q) )  //判断栈满的情况  
    {
        cout << "Queue OverFlow!" << endl;
        return ; 
    }
               
    Q.elem[Q.rear] = x;  
    Q.rear = (Q.rear+1) % Q.size;  //尾指针应以此种方式加1，才会实现循环队列。  
}  

//出队
char pop(circularQueue &Q)  
{  
    if( isEmpty() )  
    {
        cout << "Queue Empty" << endl;
        return ;
    }
    
    char e = Q.elem[Q.front];  
    Q.front = (Q.front+1) % Q.size; // 头指针应以此种方式加1，才会实现循环队列。  
    
    return e;  
}  

// 初始化  
void init(circularQueue &Q)  
{  
    Q.elem = new char[Queue_Size];  
    Q.front = Q.rear = 0;            
    Q.size = Queue_Size;    // default init size: 100
}  

// 销毁队列  
void destroy(circularQueue &Q)  
{  
    delete [] Q.elem;  

    Q.front = Q.rear = 0;  
    Q.size = 0;  
}  
   
//首尾指针相等说明队列为空
bool isEmpty(const circularQueue &Q)
{
    return Q.front == Q.rear;
}

//为了区别isEmpty, '浪费'一个空间来标识队满的情形
bool isFull(const circularQueue &Q)
{
    return (Q.rear+1)%Q.size == Q.front;
}

// 求队列的长度, 考虑两种情况(rear大于front，以及小于front的情形)
int length(const circularQueue &Q)  
{  
    return (Q.rear-Q.front+Q.size) % Q.size;
}  

void traveres(const circularQueue &Q)
{
    for (int i=Q.front; i!=Q.rear; i=(i+1)%Q.size)
    {
        cout << Q.elem[i] << endl;
    }
}

void main()  
{  
    circularQueue Q;  
    
    init(Q);  
       
    push(Q,'a');  
    push(Q,'b');  
    push(Q,'c');  
       
    cout << pop(Q) << endl;  
    cout << pop(Q) << endl;  
       
    destroy(Q);  
}  
```