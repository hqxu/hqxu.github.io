title: 读入父子结点关系对，构建二叉树并输出 (百度面试题)
date: 2016-03-28 23:30:10
tags: 面试题
categories: 面试题
---
题目大意就是读取输入文件中的数据(父结点与子结点的关系对)，构建一颗二叉树，并按照一定的形式输出这颗树。
<!--more-->

具体描述如下:  读取TreeInput.txt文件，其每一行描述的是二叉树中父结点与子结点的关系。 如下:

|  父  | &nbsp;|  子  |
| ---  | ---    |  --- |
|  A   | &nbsp; |   B  |
|  B   | &nbsp; |   D  |
|  A   | &nbsp; |   C  |
|  B   | &nbsp; |   E  |
|  D   | &nbsp; |   F  |


根据文件中的数据，构建二叉树、并按下面的格式将该二叉树输出:

|    |       |     |     |   
| --- |  ---   |  --- |  --- |
|  A  |        |      |      |
|     |   B    |      |      |
|     |        |   D  |      |
|     |        |      |   F  |
|     |        |   E  |      |
|     |   C    |      | &nbsp;|

问题分析
------------------------
如果该二叉树构建完成的话，直接dfs先序遍历, 按照**每个结点所在的层次数**乘以一定的空格数，即可按照要求输出。
 ```c++
    void dfs(TreeNode *root, int level)
    {
        if (NULL != root)
        {
            const int spaceNum = 10 * level;
            for (int i=0; i<spaceNum; i++)
                cout << " ";
            cout << root->val << endl;

            dfs(root->left, level+1);
            dfs(root->right, level+1);
        }
    }
```

从而该问题的本质在于，如何利用<父，子>pair信息构建出二叉树。这可以分成两步:

Step1: 读取文件中的<父结点, 子结点>关系对，构建一个记录map(对应字母ID和二叉树的结点), 然后生成出二叉树;

这一步，与经典的面试题**深拷贝二叉树**和LeetCode上的[Clone Graph](https://leetcode.com/problems/clone-graph/)很相似，`核心都是利用一个map, 记录原结点与新结点的对应关系`，以此避免生成重复结点。

Step2: 找出该二叉树的根结点root --- 其入度为0。同样可以利用一个map, 记录每个结点的入度次数。

解决方案
------------------------
### Standard Solution

 #### 算法复杂度分析
  利用了两个map，其中一个nodeMap记录的是字母ID与二叉树的结点对应关系，另外一个记录的是每个字母(结点)的入度数目。所以 **空间复杂度** 为`O( N )`。
  假设目标二叉树有N个结点，如果输入合法，那么说明应该有N-1条边，即对应的输入文件为N-1行pair数据。dfs前序遍历一遍二叉树时间复杂度是O(N), 但由于用的是C++中的map(底层是红黑树)，而不是unordered_map(hash), 在构建二叉树的时候每次需要根据字母ID去find对应的结点，从而**时间复杂度** 是`O( NlogN)`。

<br/>
 #### 代码实现
  ```c++
#include <iostream>
#include <fstream>
#include <vector>
#include <map>
using namespace std;

#define IDType char

struct TreeNode 
{
    IDType val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(IDType x) : val(x), left(NULL), right(NULL) {}
};

// 方案1: step1: 读取文件中的<父结点, 子结点>关系对，构建一个记录map(对应字母ID和二叉树的结点), 然后生成出二叉树(保证输入合法);
//        step2: 然后找出入度为0的结点为根结点root -- 也是利用一个map(对应字母ID和inDegree的数量);
//        step3: 最后先序遍历这颗树并按照要求的格式输出。
class Solution 
{
public:
    TreeNode * buildBinaryTree(string fileName) 
    {
        TreeNode * root = NULL;
        ifstream infile(fileName);

        //step1.
        IDType parentID;
        IDType childID;
        map<IDType, TreeNode * > nodeMap;
        map<IDType, int> inDegreeMap;

        while ( infile >> parentID >> childID)
        {
            //cout << parentID << "   " << childID << endl;

            map<IDType, TreeNode *>::iterator it;

            it = nodeMap.find(parentID);
            if (it == nodeMap.end())        // first time appear -parentID
            {
                TreeNode * p = new TreeNode(parentID);
                nodeMap[parentID] = p;

                inDegreeMap[parentID] = 0;  // init in-degree
            }

            it = nodeMap.find(childID);
            if (it == nodeMap.end())        // first time appear -childID
            {
                TreeNode * c = new TreeNode(childID);
                nodeMap[childID] = c;

                inDegreeMap[childID] = 1;   // init in-degree
            }
            else
            {
                inDegreeMap[childID]++;     // count in-degree
            }

            // make relations for the nodes of the new tree by the input pair: <parentID, childID>
            if (NULL != nodeMap[parentID]->left)
            {
                nodeMap[parentID]->right = nodeMap[childID];
            }
            else
            {
                nodeMap[parentID]->left = nodeMap[childID];
            }
        }

        //step2. find root node -- the indegree of the root node is zero!
        for (auto it = inDegreeMap.begin(); it != inDegreeMap.end(); it++)
            if ( 0 == it->second)
            {
                root = nodeMap[it->first];
                break;
            }

        //
        infile.close();

        return root;
    }

    void printBinaryTree(TreeNode *root)
    {
        if ( NULL == root ) return ;

        dfs(root, 0);
    }

    void dfs(TreeNode *root, int level)
    {
        if (NULL != root)
        {
            const int spaceNum = 10 * level;
            for (int i=0; i<spaceNum; i++)
                cout << " ";
            cout << root->val << endl;

            dfs(root->left, level+1);
            dfs(root->right, level+1);
        }
    }
};

int main(int argc, char *argv[])
{
    Solution solution;
    string fileName = "./TreeInput.txt";
    TreeNode * root = NULL;

    root = solution.buildBinaryTree(fileName);

    solution.printBinaryTree(root);

    system("pause");

    return 0;
}

  ```

<br/>

### 面试期间在纸上写的代码
   ```c++
   /*
方案2: 面试在纸上写的时候，虽然大体思路跟上面类似, 但由于许多重复代码，在纸上不好重构，以及思路没有上面那么清晰，所以写了半天~~搞复杂了。
面试期间当时的思路:
step1: 读取文件中的<父结点, 子结点>关系对，构建一个记录relationMap(对应父节点ID和两个孩子的信息)
step2: 利用relationMap, 然后找出入度为0的结点为根结点root -- 也是利用一个map(对应字母ID和inDegree的数量);
step3: 利用relationMap, 创建出一个nodeMap(对应字母ID和二叉树的结点),从而创建出二叉树
step4: 最后先序遍历这颗树并按照要求的格式输出。

嗯，对第4步毫无疑问；但可以看到，面试期间的思路中多了第一步，首先构造出relationMap,其实是没有必要的；
其次找root以及创建二叉树分成两步做，代码上很多冗余，不像方案1)将写法合并在了一起。 当发现有许多冗余的地方时……在纸上又不好重构。。
故而写的很慢，让面试官等待过久-.-  惭愧。

另外对C++的文件太久没写,用法生疏了……导致当时没有直接使用infile >> A_ID >> B_ID这样的语句，而是用python风格的伪码
geline(infile, str);   (A_ID, B_ID) = str.split(" ");
*/
class Solution2
{
public:
    typedef struct Child
    {
        int left;
        int right;
        Child(int l=0, int r=0):left(l), right(r) {}
    }Child;

    TreeNode * buildBinaryTree(string fileName) 
    {
        map<int, Child> relation;

        // step1: get data and build relationMap;
        ifstream infile(fileName);
        char parentID;
        char childID;

        while (infile >> parentID >> childID)
        {
            map<int, Child>::iterator it;
            it = relation.find(parentID);
            if ( it != relation.end() )
            {
                it->second.right = childID;
            }
            else
            {
                Child *cp = new Child(0, 0);
                cp->left = childID;

                relation[parentID] = *cp;    // first time
            }
        }
        infile.close();

        // relation Map
        //  A --> B, C       
        //  B --> D, E


        // step2. find the root node.Its degree equals zero
        map<int, int> inDegreeMap;
        for (auto it=relation.begin(); it!=relation.end(); it++)
        {
            int pID = it->first;
            int clID = it->second.left;
            int crID = it->second.right;

            auto pIt = inDegreeMap.find(pID);
            if (pIt == inDegreeMap.end()) // first time
            {
                inDegreeMap[pID] = 0;
            }

            auto clIt = inDegreeMap.find(clID);
            if (clIt == inDegreeMap.end()) // first time
            {
                inDegreeMap[clID] = 1;
            }
            else
            {
                inDegreeMap[clID]++;
            }

            // -. -冗余代码，手写纸上的时候就很郁闷. 重构的话专门写成一个函数的话，又觉得麻烦……
            auto crIt = inDegreeMap.find(crID);
            if (crIt == inDegreeMap.end()) // first time
            {
                inDegreeMap[crID] = 1;
            }
            else
            {
                inDegreeMap[crID]++;
            }
        }

        int rootID = 0;
        for (auto it = inDegreeMap.begin(); it!=inDegreeMap.end(); it++)
            if ( 0 == it->second)
            {
                rootID = it->first;
                break;
            }

        //step3. build the binary tree by using relationMap and building a nodeMap, return root Node by using rootID
        map<int, TreeNode *> nodeMap;
        for (auto it=relation.begin(); it!=relation.end(); it++)
        {
            // -.- 又见冗余代码, 再纸上写着很郁闷 (嗯，说明这构建Tree和找root结点，都需要遍历同一个信息。可以合并的)
            int pID = it->first;
            int clID = it->second.left;
            int crID = it->second.right;

            TreeNode *p = NULL;
            TreeNode *cl = NULL;
            TreeNode *cr = NULL;

            auto pIt = nodeMap.find(pID);
            if (pIt == nodeMap.end()) // first time
            {
                nodeMap[pID] = new TreeNode(pID);
            }

            if ( 0 != clID )
            {
                auto clIt = nodeMap.find(clID);
                if (clIt == nodeMap.end()) // first time
                {
                    nodeMap[clID] = new TreeNode(clID);
                }

                nodeMap[pID]->left = nodeMap[clID];
            }


            if ( 0 != crID )
            {
                auto crIt = nodeMap.find(crID);
                if (crIt == nodeMap.end()) // first time
                {
                    nodeMap[crID] = new TreeNode(crID);
                }
                nodeMap[pID]->right = nodeMap[crID];
            }
        }// end of for

        return nodeMap[rootID];
    }

    void printBinaryTree(TreeNode *root)
    {
        if ( NULL == root ) return ;

        dfs(root, 0);
    }

    void dfs(TreeNode *root, int level)
    {
        if (NULL != root)
        {
            const int spaceNum = 10 * level;
            for (int i=0; i<spaceNum; i++)
                cout << " ";
            cout << root->val << endl;

            dfs(root->left, level+1);
            dfs(root->right, level+1);
        }
    }
};
   ```

<br/>

面试小结
------------------------
面试官给出这道题目后，我立即想到了dfs前序输出，以及利用记录map进行二叉树的深度拷贝。总体思路是对的。可惜的是，当时在构建二叉树的时候想复杂了一些，引入了一个relationMap(具体参见代码), 并且在纸上撰写代码时，发现许多冗余部分……如果在计算机上编写，可以很快的合并冗余或模块化，但是纸上就不好重构了……所以硬着头皮写了冗余代码。。。导致撰写代码的时间过长…… -.- 

以后再遇到这种情况，还是应该等思路更清晰的时候再撰写，**事先预估可能出现的状况**。 另外就是，**面试的这个阶段，着实每天应该写题，保持手感，不至于在纸上写代码的时候出现生疏感, 对于常见的API, 用法陌生。**

下面为两个方案的源码。
第1个是面试后总结，在电脑上写的解决方案(思路清晰, 将可能导致冗余代码的操作合并在了一起)；
第2个是面试期间, 在纸上撰写的代码。 对比方案1), 可以看到面试时想复杂了，并且很多冗余代码。  
两者输出一致。