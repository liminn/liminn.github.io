---
title: 剑指Offer解题记录
date: 2017-12-1 9:19:49
mathjax: true
top: true
categories: 
- 数据结构与算法
tags:
---
注：该记录中解题答案均为在牛客网进行[在线编程测试](https://www.nowcoder.net/ta/coding-interviews)时的答案记录，故未自己写测试代码。

# 一、链表
## 1. 两个链表的第一个公共结点
**题目描述**：
输入两个链表，找出它们的第一个公共结点。
**思路**：
- **关键点**：如果两个单链表有公共的结点，那么这两个链表从某一结点开始，它们的`next`指针都指向同一个结点，又由于是单向链表的结点，每个结点只有一个`next`指针，因此**从第一个公共结点开始，之后它们所有的结点都是重合的，不再出现分叉**。
- **思路1**：
 - 对于对于链表一上的每一个结点，顺序遍历链表二上的每一个结点。
 - 若第一个链表的长度为$m$，第二个链表的长度为$n$，那么时间复杂度为$O(mn)$。
- **思路2**：
 - 分别把两个链表的结点放入栈里，这样两个链表的尾结点就位于两个栈的栈顶，接下来比较两个栈顶的结点是否相同。如果相同，则把栈顶弹出接着比较下一个栈顶，直到找到最后一个相同的结点。
 - 该思路需要用到两个辅助栈，如果链表的长度分别为$m$和$n$，那么空间复杂度是$O(m+n)$，时间复杂度是$O(m+n)$。和思路1相比，时间效率得到了提高，相当于用空间消耗换取了时间效率。
- **思路3**：
 - 首先遍历两个链表得到它们的长度。在第二次遍历时，在较长的链表上先走若干步，接着同时在两个链表上遍历，找到的第一个相同的结点就是它们的第一个公共结点。
 - 与思路2相比，该思路的时间复杂度为$O(m+n)$，但不需辅助栈，因此提高了空间效率。

 <!-- more --> 

**代码**：
```C++
//思路3
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x):val(x), next(NULL) {}
};
*/
class Solution 
{
public:
    ListNode* FindFirstCommonNode( ListNode* pHead1, ListNode* pHead2) 
    {
    	if(pHead1==nullptr || pHead2==nullptr)
            return nullptr;
        //首先遍历两个链表得到它们的长度
        int length1,length2;
        length1=length2= 0;
        ListNode* pTemp1 = pHead1;
        ListNode* pTemp2 = pHead2;
        while(pTemp1!=nullptr)
        {
            length1++;
            pTemp1 = pTemp1->next;
        }
        while(pTemp2!=nullptr)
        {
            length2++;
            pTemp2 = pTemp2->next;
        }
        //在第二次遍历时，在较长的链表上先走若干步
        pTemp1 = pHead1;
        pTemp2 = pHead2;
        if(length1>length2)
        {
            int dif = length1-length2;
            for(int i = 0;i<dif;i++)
            {
                pTemp1 = pTemp1->next;
            }
        }
        else
        {
            int dif = length2-length1;
            for(int i = 0;i<dif;i++)
            {
                pTemp2 = pTemp2->next;
            }
        }
        //接着同时在两个链表上遍历，找到的第一个相同的结点就是它们的第一个公共结点
        ListNode* pResult = nullptr;
        while(pTemp1!=nullptr && pTemp2!=nullptr)
        {
        	if(pTemp1 == pTemp2)
            {
            	pResult = pTemp1;
                break;
            }
            else
            {
                pTemp1 = pTemp1->next;
                pTemp2 = pTemp2->next;
            }
        }
        return pResult;
    }
};
```
## 2. 链表中环的入口结点
**题目描述**：
一个链表中包含环，请找出该链表的环的入口结点。
**思路**：
- **核心思想**：假定有环且已知环的结点个数为$y$，令指针`pTemp1`从首结点开始先走$y$步，再令`pTemp2`从首结点开始同`pTemp1`一起向前走，相遇处，即为环的入口结点。
- **证明**：假定环的结点个数为$y$，环之前的结点个数为$x$。`pTemp1`先走$y$步，然后`pTemp2`从首结点开始同`pTemp1`一起再走$x$步，则`pTemp2`在第$(x+1)$个结点处，`pTemp1`在第$(x+y+1)$个结点处。$(x+1)$结点位置和$(x+y+1)$结点位置都是环的入口结点位置。因此，依上述核心思想的步骤，`pTemp1`与`pTemp2`定会相遇，相遇处恰好为环的入口结点。
- **环的结点个数求法**：首先，令一指针`pFast`一次走两步，一指针`pSlow`一次走一步，若相遇，则有环，且相遇处定在环内。然后，`pFast`不动，令一指针`pSlow`继续走且计数，当`pSlow`与`pFast`再次相遇，即得出环的结点个数。
- 注意，代码中充斥着防止指针为空的情况，繁琐但必不可少。

**代码**：
```C++
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution 
{
public:
    ListNode* EntryNodeOfLoop(ListNode* pHead)
    {
        if(pHead==nullptr)
            return nullptr;
        //通过getNumberOfLoop()获取环的结点个数，0则无环；大于0则有环
        ListNode* pTemp1, *pTemp2;
        pTemp1 = pTemp2 = pHead;
        int num = getNumberOfLoop(pHead);
        //若环结点数等于0，直接返回nullptr
        if(num==0) 
            return nullptr;
        //依照核心思想执行，获取pTemp1与pTemp2相遇处指针
        for(int i = 0;i<num;i++)
        {
            pTemp1 = pTemp1->next;
        }
        while(pTemp1!=pTemp2)
        {
            pTemp1 = pTemp1->next;
            pTemp2 = pTemp2->next;
        }
        return pTemp1;
    }
    int getNumberOfLoop(ListNode* pHead)
    {
        //1.令pFast一次走两步，pSlow一次走一步，若相遇，则有环，且相遇处定在环内
        int num = 0;
        ListNode* pFast, *pSlow;
        pFast = pSlow = pHead;
        pFast=pFast->next;    //为了下个while循环的判断条件，pFast先走一步
        while(pFast!=pSlow)
        {
            if(pFast==nullptr)
                break;
            pFast = pFast->next;
            if(pFast==nullptr)
                break;
            pFast = pFast->next;
            if(pSlow==nullptr)
                break;
            pSlow=pSlow->next;
        }
        if(pFast==nullptr || pSlow==nullptr)
            num=0;
        //2.pFast不动，令pSlow继续走且计数，当pSlow与pFast再次相遇，即得出环的结点个数
        else
        {
            pSlow=pSlow->next; //为了下个while循环的判断条件，pSlow先走一步
            num++;
            while(pSlow!=pFast)
            {
                pSlow=pSlow->next;
                num++;
            }
        }
        return num;
    }
};
```

## 3. 反转链表
**题目描述**：
给出一个链表`1->2->3->nullptr`，这个翻转后的链表为`3->2->1->nullptr`
**思路**：
- **方法1**：迭代实现。维护三个指针`pPrev`,`pCurrent`,`pNext`，通过迭代完成链表的反转（画图使思路清晰）：
 - 起始情况：`pPrev`为`nullptr`，`pCurrent`指向`pHead`
 - 终止情况：`pPrev`指向链表尾元素，`pCurrent`为`nullptr`
- **方法2**：递归实现。利用递归走到链表的末端，然后再更新每一个结点的`next`指针 ，实现链表的反转。
- **方法3**：用栈实现。将链表结点指针全部压入栈中。头指针指向链表末尾结点。将栈中结点指针反串起来，栈中最后结点指针指向`nullptr`。

**代码**：
```C++
//方法1：迭代实现
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution 
{
public:
    ListNode* ReverseList(ListNode* pHead) 
    {
        if(pHead==nullptr)
            return nullptr;
        ListNode* pPre, *pCurrent, *pNext;
        pPre=nullptr;
        pCurrent = pHead;
        while(pCurrent!=nullptr)
        {
            pNext = pCurrent->next;
            pCurrent->next = pPre;
            pPre = pCurrent;
            pCurrent = pNext;
        }
        pHead=pPre;
        return pHead;
    }
};
```
```C++
//方法2：递归实现
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution 
{
public:
    ListNode* ReverseList(ListNode* pHead) 
    {
        //如果链表为空或者链表中只有一个元素
        if(pHead==NULL || pHead->next==NULL) 
            return pHead;
        //先反转后面的链表，走到链表的末端结点，返回末端结点，作为新的头结点，保存起来
        ListNode* pHeadReverse=ReverseList(pHead->next);
        //再将当前节点设置为后面节点的后续节点
        pHead->next->next = pHead;
        //当前节点指向NULL
        pHead->next=NULL;
        //返回新的头结点
        return pHeadReverse;
    }
};
```
```C++
//方法3：用栈实现
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution 
{
public:
    ListNode * reverse(ListNode * head) 
    {
        // write your code here
        //防御性编程
        if(head==NULL) return NULL;
        //1.将链表结点指针全部压入栈中
        stack<ListNode*> stack1;
        ListNode* temp = head;
        while(temp!=NULL) 
        {
            stack1.push(temp);
            temp=temp->next;
        }
        //2.头指针指向链表末尾结点
        temp = stack1.top();
        head=temp;
        //3.将栈中结点指针反串起来，栈中最后结点指针指向NULL
        while(!stack1.empty())
        {
            ListNode* pNode1 = stack1.top();
            stack1.pop();
            if(stack1.empty())
            {
                pNode1->next=NULL;
                break;
            }
            ListNode* pNode2 = stack1.top();
            pNode1->next = pNode2;
        }
        return head;
    }
};

```
## 4. 合并两个排序的链表
**题目**：输入两个递增排序的链表，合并这两个链表并使新链表中的结点仍然是按照递增排序的。
**思路**：
- 依照归并排序中的`merge`函数的思想，`merge`两个链表：
 - 1.比较两个链表的头结点，较小的作为新链表的头结点`pHeadNew`；
 - 2.归并两个链表，迭代比较，较小者作为新的尾结点`pTailNew`。
 - 注意，直接改动每一结点的`next`指针，故不需另外的辅助空间。

**代码**：
```C++
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution 
{
public:
    ListNode* Merge(ListNode* pHead1, ListNode* pHead2)
    {
        //防御性编程：若输入链表有任一为空，返回另一个
        if(pHead1==nullptr)
            return pHead2;
        if(pHead2==nullptr)
            return pHead1;
        //1.找出新链表的头结点pHeadNew
        ListNode* pHeadNew = nullptr;
        ListNode* pTemp1 = pHead1;
        ListNode* pTemp2 = pHead2;
        if(pTemp1->val < pTemp2->val)
        {
            pHeadNew = pTemp1;
            pTemp1 = pTemp1->next;
        }
        else
        {
            pHeadNew = pTemp2;
            pTemp2 = pTemp2->next;
        }
        //2.归并两个链表，迭代比较，较小者作为新的尾结点pTailNew
        //直接改动每一结点的next指针，故不需另外的辅助空间
        ListNode* pTailNew = pHeadNew; 
        while(pTemp1!=nullptr && pTemp2!=nullptr)
        {
            if(pTemp1->val < pTemp2->val)
            {
                pTailNew->next = pTemp1;
                pTailNew = pTemp1;
                pTemp1 = pTemp1->next;
            }
            else
            {
                pTailNew->next = pTemp2;
                pTailNew = pTemp2;
                pTemp2 = pTemp2->next;
            }    
        }
        if(pTemp1!=nullptr) pTailNew->next = pTemp1;
        if(pTemp2!=nullptr) pTailNew->next = pTemp2;
        return pHeadNew;
    }
};
```

## 5. 从尾到头打印链表
**题目描述**：
输入一个链表的头结点，从尾到头反过来打印出每个结点的值。
**思路**：
- 首先，假定不能改变链表结构，故不采用迭代反转链表，然后后再打印链表的方法。
- **方法1**：遍历的顺序是从头到尾，输出的顺序是从尾到头，是典型的“后进先出”，用栈实现这种顺序：
 - 遍历链表，每经过一个结点的时候，把该结点压入辅助栈中。
 - 当遍历完整个链表后，再从栈顶开始逐个输出结点的值。
- **方法2**：用递归方法反向打印链表。既然可以用栈实现，而递归本质上就是一个栈结构，故可用递归来实现：
 - 每访问到一个结点，先递归输出它后面的结点，再输出该结点自身，这样链表的输出结果就反过来了。
- 注意，虽然基于递归的代码看起来很简洁，但有一个问题：当链表非常长的时候，就会导致函数调用的层级很深，从而有可能导致函数调用栈溢出。显然用栈基于循环实现的代码的鲁棒性要好一些（存放于自由存储的堆区）。
 
**代码**：
```C++
//方法1
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution 
{
private:
    vector<int> result;    
    stack<ListNode*> s;
public:
    vector<int> printListFromTailToHead(ListNode* pHead) 
    {
        if(pHead==nullptr)
            return result;
        //遍历链表，每经过一个结点的时候，把该结点压入辅助栈中
        ListNode* pTemp = pHead;
        while(pTemp!=nullptr)
        {
            s.push(pTemp);
            pTemp = pTemp->next;
        }
        //当遍历完整个链表后，再从栈顶开始逐个输出结点的值
        while(!s.empty())
        {
            pTemp = s.top();
            result.push_back(pTemp->val);
            s.pop();
        }
        return result;
    }
};
```
```C++
//方法2
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution 
{
private:
    vector<int> result;    
public:
    vector<int> printListFromTailToHead(ListNode* pHead) 
    {
        if(pHead==nullptr)
            return result;
        printListFromTailToHead(pHead->next);
        result.push_back(pHead->val);
        return result;
    }
};
```
## 6. 复杂链表的复制
**题目描述**
输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点），返回结果为复制后复杂链表的head。（注意，输出结果中请不要返回参数中的节点引用，否则判题程序会直接返回空）
**思路**：
- **步骤1**：复制原始链表的任意结点$N$并创建新结点$N'$，再把$N'$链接到$N$的后面。如原来是`A->B->C`，则变成`A->A'->B->B'->C->C'`。
- **步骤2**：设置复制出来的结点的`random`指针。如果原始链表上的结点$N$的`random`指针指向$S$，则它对应的复制结点$N'$的`ramdon`指针指向$S$的复制结点$S'$。如`A1->random = A->random->next;`
- **步骤3**：把长链表拆分成两个链表：把奇数位置的结点用`next`指针链接起来就是原始链表，把偶数位置的结点用`next`指针链接起来就是复制出来的链表。

**代码**：
```C++
/*
struct RandomListNode 
{
    int label;
    struct RandomListNode *next, *random;
    RandomListNode(int x) :label(x), next(NULL), random(NULL) {}
};
*/
class Solution 
{
public:
    RandomListNode* Clone(RandomListNode* pHead)
    {
        if(pHead==nullptr)
            return nullptr;
        RandomListNode* pCurrent = pHead;
        //1.复制原始链表的任一节点N并创建新节点N'，再把N'链接到N的后边
        while(pCurrent!=nullptr)
        {
            RandomListNode* pClone = new RandomListNode(pCurrent->label);
            pClone->next = pCurrent->next;
            pCurrent->next = pClone;
            pCurrent = pClone->next;
        }
        //2.如果原始链表上的节点N的random指向S，则对应的复制节点N'的random指向S的下一个节点S'
        pCurrent = pHead;
        while(pCurrent!=nullptr)
        {
            RandomListNode* pClone = pCurrent->next;
            if(pCurrent->random!=nullptr)
                pClone->random = pCurrent->random->next;
            pCurrent = pClone->next;
        }
        //3.把得到的链表拆成两个链表，奇数位置上的结点组成原始链表，偶数位置上的结点组成复制出来的链表
        RandomListNode* pCloneHead=pHead->next;
        pCurrent = pHead;
        while(pCurrent!=nullptr)
        { 
            RandomListNode* pClone = pCurrent->next;
            pCurrent->next = pClone->next;
            pCurrent = pCurrent->next;
            //pCurrent为nullptr，即pClone->next为nullptr，不用再继续，拆分结束。
            if(pCurrent==nullptr)
                break;
            pClone->next = pCurrent->next;
        }
        return pCloneHead;
    }
};
```

## 7. 链表中倒数第k个结点
**题目**：
输入一个链表，输出该链表中倒数第$k$个结点。本题从1开始计数，即链表的尾结点是倒数第1个结点。例如一个链表有6个结点，从头结点开始它们的值依次是1、2、3、4、5、6。这个链表的倒数第3个结点是值为4的结点。
**思路**：
- 递归遍历链表，返回时通过`count`计数，当`count==k`时，当前`pHead`即为倒数第$K$个结点。

**代码**：
```C++
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x):val(x), next(NULL) {}
};*/
class Solution 
{
    int count = 0;
    ListNode* pTarget=nullptr;
public:
    ListNode* FindKthToTail(ListNode* pHead, unsigned int k) 
    {
        if(pHead==nullptr)
            return nullptr;
        Find(pHead,k);
        return pTarget;
    }
    void Find(ListNode* pHead, unsigned int k)
    {
        if(pHead==nullptr)
            return;
        Find(pHead->next,k);
        count++;
        if(count==k)
            pTarget = pHead;
    }
};
```
## 8. 删除链表中的重复结点
**题目**：
在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。 例如，链表`1->2->3->3->4->4->5`，处理后为`1->2->5`
**思路**：

**代码**：
```C++
/*
struct ListNode 
{
    int val;
    struct ListNode *next;
    ListNode(int x) :val(x), next(NULL) {}
};
*/
class Solution
 {
public:
    ListNode* deleteDuplication(ListNode* pHead)
    {
        // 只有0个或1个结点，则返回
        if(pHead==nullptr || pHead->next==nullptr) 
            return pHead;
        ListNode* pNext = pHead->next;
        // 当前结点是重复结点
        if(pHead->val == pNext->val)
        {
            // 跳过值与当前结点相同的全部结点,找到第一个与当前结点不同的结点
            while(pNext!=nullptr && pNext->val==pHead->val)
            {
                pNext = pNext->next;
            }
            // 从第一个与当前结点不同的结点开始递归
            return deleteDuplication(pNext);
        }
         // 当前结点不是重复结点
        else
        {
            // 保留当前结点，从下一个结点开始递归
            pHead->next = deleteDuplication(pHead->next);
            return pHead;
        }
    }
};
```
-----------
# 二、栈和队列
## 1. 用两个栈实现队列
**题目**：用两个栈来实现一个队列，完成队列的`push`和`pop`操作。队列中的元素为`int`类型。
**思路**：
- 栈是后进先出，队列是先进先出。
- 利用栈`stack1`负责`push`操作。
- 利用栈`stack2`负责`pop`操作：先将`stack1`中所有元素弹出到`stack2`中，然后`stack2`出栈一个元素，最后`stack2`中所有元素弹出到`stack1`。

**代码**：
```C++
class Solution
{
private:
    stack<int> stack1;
    stack<int> stack2;
public:
    //利用栈stack1负责入栈操作
    void push(int node) 
    {
        stack1.push(node);
    }
    //利用栈stack2负责出栈操作
    int pop() 
    {
        //先将stack1中所有元素弹出到stack2中
        while(!stack1.empty())
        {
            int temp = stack1.top();
            stack2.push(temp);
            stack1.pop();
        }
        //然后stack2出栈一个元素
        int result = stack2.top();
        stack2.pop();
        //最后stack2中所有元素弹出到stack1
        while(!stack2.empty())
        {
            int temp = stack2.top();
            stack1.push(temp);
            stack2.pop();
        }
        return result;
    }
};
```

## 2. 包含min函数的栈
**题目**：定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的`min`函数。在该栈中，调用`min`、`push`及`pop`的时间复杂度都是$O(1)$。
**思路**：
- 题目要求的是栈在进行任何的入栈和出栈操作后，都能调用`min`函数返回最小值。两种思路不可行：
 - 定义一个变量`value`存储最小值。因为栈一旦`pop`后，不知道最小值是什么。
 - 每次进栈出栈都重新对栈内元素进行排序。因为复杂度太高。
- 正确解法：采用一个辅助栈，其栈顶保存主栈当前的最小值。主栈每一次`push`，辅助栈都将主栈当前的最小值压入栈顶。主栈每一次`pop`，辅助栈都`pop`。

**代码**：
```C++
class Solution 
{
private:
    stack<int> s1; //主栈
    stack<int> s2; //辅助栈
public:
    //主栈stack1每一次push，辅助栈stack2都将主栈当前的最小值压入栈顶
    void push(int value) 
    {
        if(s1.empty())
        {
            s1.push(value);
            s2.push(value);
        }
        else if(value<s2.top())
        {
            s1.push(value);
            s2.push(value);
        }
        else
        {
            s1.push(value);
            s2.push(s2.top());
        }
    }
    //主栈stack1每一次pop，辅助栈stack2都pop
    void pop() 
    {
        s1.pop();
        s2.pop();
    }
    int top() 
    {
        return s1.top();
    }
    int min() 
    {
        return s2.top();
    }
};
```
## 3. 栈的压入、弹出序列
**题目描述**
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如序列$1,2,3,4,5$是某栈的压入顺序，序列$4,5,3,2,1$是该压栈序列对应的一个弹出序列，但$4,3,5,1,2$就不可能是该压栈序列的弹出序列。
**思路**：
- 判断一个序列是不是栈的弹出序列的规律：
 - 如果下一个弹出的数字刚好是栈顶数字，那么直接弹出；
 - 如果下一个弹出的数字不在栈顶，则把压栈序列中还没有入栈的数字压入辅助栈，直到把下一个需要弹出的数字压入栈为止；
 - 如果所有数字都压入栈后仍然没有找到下一个弹出的数字，那么该序列不可能是一个弹出序列。

**代码**：
```C++
class Solution 
{
private:
    stack<int> s;
public:
    bool IsPopOrder(vector<int> pushV,vector<int> popV) 
    {
        if(pushV.size()==0 || popV.size()==0)
            return false;
        int i=0; //标记pushV
        int j=0; //标记popV
        while(i<pushV.size())
        {
            s.push(pushV[i++]);
            while(!s.empty() && s.top()==popV[j])
            {
                s.pop();
                j++;
            }
        }
        return s.empty();
    }
};
```
--------
# 三、树
## 1. 二叉树的深度
**题目描述**:
输入一棵二叉树，求该树的深度。从根结点到叶结点依次经过的结点（含根、叶结点）形成树的一条路径，最长路径的长度为树的深度。
**思路**：
**解法1**：
- 前序遍历二叉树，计算每一叶结点的深度，找出最大的叶结点的深度，即树的高度。
- 在两种情况下，当前深度变量`currentDepth`进行减一操作：
 - 当前结点是叶结点，则将`currentDepth`与最大的叶结点深度`maxDepth`比较，更新`maxDepth`。之后，`currentDepth`进行减一操作；
 - 任一结点的左右子树遍历完成，`currentDepth`进行减一操作。
- 注：该题规定空树深度为0，根结点深度为1。

**解法2**：
- 将结点的深度从该角度计算：后序遍历时，结点的左右子树的深度较大值加一。
- 因此，树的深度可以这样计算：后序遍历二叉树，较大子树的深度值加一。

**代码**：
```C++
//解法1
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
private:
    int maxDepth=0;
    int currentDepth=0;
public:
    int TreeDepth(TreeNode* pRoot)
    {
        if(pRoot==nullptr)
            return maxDepth;
        currentDepth++;
        //当前结点是叶结点
        if(pRoot->left==nullptr && pRoot->right==nullptr)
        {
            //更新maxDepth
            if(currentDepth>maxDepth)
            {
                maxDepth = currentDepth;
            }
            //currentDepth进行减一操作
            currentDepth--;
            return maxDepth;
        }
        TreeDepth(pRoot->left);
        TreeDepth(pRoot->right);
        //任一结点的左右子树遍历完成，currentDepth进行减一操作
        currentDepth--;
        return maxDepth;
    }
};
```
```C++
//解法2
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
public:
    //递归后序遍历二叉树
    int TreeDepth(TreeNode* pRoot)
    {
        //递归出口条件：若pRoot为nullptr，则返回0
        if(pRoot==nullptr)
            return 0;
    	int left=TreeDepth(pRoot->left);
        int right=TreeDepth(pRoot->right);
        //较大子树的深度值加一
        if(left>right)
            return left+1;
        else
            return right+1;
    }
};
```
## 2. 二叉树的镜像
**题目描述**:
操作给定的二叉树，将其变换为源二叉树的镜像。
**输入描述**:
二叉树的镜像定义：
```
源二叉树 
    	    8
    	   /  \
    	  6   10
    	 / \  / \
    	5  7 9  11
镜像二叉树
    	    8
    	   /  \
    	  10   6
    	 / \  / \
    	11 9 7   5
```
**思路**：
- 前序遍历二叉树，“访问”操作为：交换每一结点的左右子树（左右子树是`nullptr`也没关系，照样交换）。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
public:
    void Mirror(TreeNode *pRoot) 
    {
        if(pRoot==nullptr)
            return;
        //交换左右子树
        TreeNode* temp = pRoot->left;
        pRoot->left = pRoot->right;
        pRoot->right = temp;
        Mirror(pRoot->left);
        Mirror(pRoot->right);
    }
};
```

## 3. 平衡二叉树
**题目描述**
输入一棵二叉树，判断该二叉树是否是平衡二叉树。注意，空树默认为平衡二叉树。
**思路**：
- 依据#1题<二叉树的深度>中解法2的思想，将平衡二叉树的判断准则定为：每一结点的左右子树的深度值相差不超过1。
- 后序遍历二叉树，计算每一结点的左右子树的深度，比较差值。

**代码**：
```C++
class Solution 
{
private:
    bool result=true;
public:
    bool IsBalanced_Solution(TreeNode* pRoot) 
    {
        if(pRoot==nullptr)
            return true;
        TreeDepth(pRoot);
        return result;
    }
    //后序遍历二叉树，计算树的深度
    int TreeDepth(TreeNode* pRoot)
    {
        if(pRoot==nullptr)
            return 0;
        int left = TreeDepth(pRoot->left);
        int right = TreeDepth(pRoot->right);
        //比较左右子树深度的差值
        if(abs(left-right)>1)
            result = false;
        //计算每一结点的左右子树的深度
        if(left>right)
            return left+1;
        else
            return right+1;
    }
};
```

## 4. 把二叉树打印成多行
**题目描述**：
从上到下按层打印二叉树，同一层结点从左至右输出。每一层输出一行。
**思路**：
- 层序遍历二叉树，在每一层末尾加入一个换行符。
- 通过变量`toBePrinted`记录当前层要打印结点个数,`nextLevel`记录下一层要打印结点个数。
- 对于根结点，`toBePrinted`为1，在将根结点的左右子结点入队时记录`nextLevel`。
- 当`toBePrinted`为0，则打印一个换行，`toBePrinted`置为`nextLevel`，`nextLevel`清零。接着，开始处理下一层。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};
*/
class Solution 
{
private:
    int toBePrinted=1;
    int nextLevel = 0;
    queue<TreeNode*> q; 
    vector<int> level;
    vector<vector<int>> result;
public:
    vector<vector<int>> Print(TreeNode* pRoot) 
    {
        if(pRoot==nullptr)
            return result;
        q.push(pRoot);
        while(!q.empty())
        {
            TreeNode* pCurrent = q.front();
            q.pop();
            toBePrinted--;
            level.push_back(pCurrent->val);
            if(pCurrent->left!=nullptr)
            {
                q.push(pCurrent->left);
                nextLevel++;
            }
            if(pCurrent->right != nullptr)
            {
                q.push(pCurrent->right);
                nextLevel++;
            }
            if(toBePrinted==0)
            {
                toBePrinted=nextLevel;
                nextLevel=0;
                result.push_back(level);
                level.clear();
            }
        }
        return result; //注意，最后要return
    }
};
```

## 5. 对称的二叉树
**题目描述**:
请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。
**思路**：
- 一开始的想法：构建该二叉树的镜像二叉树，然后同时遍历两个树，逐一进行比较。但空间复杂度为$O(n)$，不可行。
- 正确解法：前序遍历是`<root><left><right>`。定义一种新的和前序遍历对称的遍历方式`<root><right><left>`。对于对称的二叉树，前序遍历的结果，和新定义的遍历方式的结果应该是一样的。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};
*/
class Solution 
{
public:
    bool isSymmetrical(TreeNode* pRoot)
    {
        if(pRoot==nullptr)
            return true;
        return treversal(pRoot, pRoot);
    }
    bool treversal(TreeNode* pRoot1, TreeNode* pRoot2)
    {
        if(pRoot1==nullptr && pRoot2==nullptr)
            return true;
        if(pRoot1==nullptr || pRoot2==nullptr)
            return false;
        if(pRoot1->val!=pRoot2->val)
            return false;
        bool result1=treversal(pRoot1->left, pRoot2->right);
        bool result2=treversal(pRoot1->right, pRoot2->left);
        return result1 && result2;
    }
};
```

## 6. 二叉树的下一个结点
**题目描述**:
给定一个二叉树和其中的一个结点，请找出中序遍历顺序的下一个结点并且返回。注意，树中的结点不仅包含左右子结点，**同时包含指向父结点的指针**。
**思路**：
- 在中序遍历顺序`<left，root，right>`下，二叉树某一结点`pNode`的下一个结点规律如下：
 - 如果`pNode`含有右子树，则下一结点为`pNode`的**右子树中最深最左的那个结点**。
 - 如果`pNode`不含右子树，则下一结点为**使得`pNode`在其左子树中的最近的祖先结点**。
 注：最极端情况是`pNode`是二叉树中序遍历的最后一个结点，即在二叉树的最右末端，直到回溯到根结点，`pNode`还是在右子树中，即没有满足题意的下一结点。根结点的父节点为`nullptr`，祖先结点为`nullptr`也是终止循环条件之一。
 
**代码**：
```C++
/*
struct TreeLinkNode 
{
    int val;
    struct TreeLinkNode *left;
    struct TreeLinkNode *right;
    struct TreeLinkNode *next;
    TreeLinkNode(int x) :val(x), left(NULL), right(NULL), next(NULL) {}
};
*/
class Solution 
{
private:
    TreeLinkNode* pNext=nullptr; //记录目标结点
public:
    TreeLinkNode* GetNext(TreeLinkNode* pNode)
    {
        if(pNode==nullptr)
            return nullptr;
        //如果pNode含有右子树，则下一结点为pNode的右子树中最深最左的那个结点
        if(pNode->right!=nullptr)
        {
            pNext=pNode->right;
            while(pNext->left!=nullptr)
            {
                pNext=pNext->left;
            }
        }
        //如果pNode不含右子树，则下一结点为使得pNode在其左子树中的最近的祖先结点
        else if(pNode->right==nullptr)
        {
            TreeLinkNode* pParent = pNode->next;
            //最极端情况：若到根节点还没找到
            while(pParent!=nullptr)
            {
                if(pParent->left==pNode)
                {
                    pNext=pParent;
                    break;
                }
                else
                {
                    pNode = pParent;
                    pParent=pParent->next;
                }
            }
        }
        return pNext;
    }
};
```

## 7. 二叉搜索树与双向链表
**题目描述**：
输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。
**思路**：
- 首先，二叉搜索树的中序遍历结果即为有序序列。
- 然后，在中序遍历的“访问”操作中进行如下操作完成双向链表的转换：
 - 维护双向链表的尾结点`pLastNode`，初始化为`nullptr`。 
 - 当前结点的`left`指针指向`pLastNode`，`pLastNode`的`right`指针指向当前结点，更新`pLastNode`。
 - 最终结果： `nullptr<-1<=>2<=>3<=>4<=>5->nullptr`（自己画图即可知）。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
public:
    TreeNode* Convert(TreeNode* pRoot)
    {
        if(pRoot==nullptr)
            return nullptr;
        //pLastNode表示双向链表中的最后一个结点
        TreeNode* pLastNode=nullptr;
        InorderTraversal(pRoot,&pLastNode);
        TreeNode* pHead=pLastNode;
        //找出双向链表中的头结点，并返回
        while(pHead->left!=nullptr)
        {
            pHead=pHead->left;    
        }
        return pHead;
    }
    //中序遍历二叉搜索树
    void InorderTraversal(TreeNode* pRoot,TreeNode** pLastNode)
    {
        if(pRoot==nullptr)
            return;
        InorderTraversal(pRoot->left,pLastNode);
        //如果pLastNode为nullptr
        if(*pLastNode==nullptr)
        {
            pRoot->left=*pLastNode;     //当前结点的left指针指向链表中最后一个结点
            *pLastNode=pRoot;           //更新链表中最后一个结点
        }
        else
        {
            pRoot->left=*pLastNode;     //当前结点的left指针指向链表中最后一个结点
            (*pLastNode)->right=pRoot;  //链表中最后一个结点的right指针指向当前结点
            *pLastNode=pRoot;           //更新链表中最后一个结点
        }
        InorderTraversal(pRoot->right,pLastNode);
    }
};
```

## 8. 从上往下打印二叉树
**题目描述**:
从上往下打印出二叉树的每个节点，同层节点从左至右打印。
**思路**：
- 层序遍历二叉树。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
private:
    queue<TreeNode*> q;
    vector<int> v;
public:
    vector<int> PrintFromTopToBottom(TreeNode* root) 
    {
        if(root == nullptr) 
            return v;
        q.push(root);
        while(!q.empty())
        {
            TreeNode* pCurrent = q.front();
            q.pop();
            v.push_back(pCurrent->val);
            if(pCurrent->left!=nullptr)
                q.push(pCurrent->left);
            if(pCurrent->right!=nullptr)
                q.push(pCurrent->right);
        }
        return v;
    }
};
```
## 9. 二叉树中和为某一值的路径
**题目描述**:
输入一颗二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。路径定义为从树的根结点开始往下一直到叶结点所经过的结点形成一条路径。
**思路**：
- 前序遍历二叉树，计算每一路径和，与期望值进行比较。
- 在两种情况下，从当前状态存储变量（`currentSum`及 `path`）中移去当前结点：
 - 当前结点是叶结点，则将路径和与期望数值比较之后，从当前状态存储变量中移去当前结点(`path.pop_back()`,`currentSum-=root->val`)；
 - 任一结点的左右子树遍历完成,从当前状态存储变量中移去当前结点(`path.pop_back()`,`currentSum-=root->val`)。
 
**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
private:
    int currentSum=0;
    vector<int> path;
    vector<vector<int>> result;
public:
    vector<vector<int>> FindPath(TreeNode* root,int expectNumber) 
    {
        if(root==nullptr)
            return result;
        preOrder(root,expectNumber);
        return result;
    }
    //前序遍历二叉树，计算每一路径和。
    void preOrder(TreeNode* root, int expectNumber)
    {
        if(root==nullptr)
            return;
        path.push_back(root->val);
        currentSum+=root->val;
        //1.当前结点是叶结点，则将路径和与期望数值比较之后，从当前状态存储变量中移去当前结点；
        if(root->left==nullptr && root->right==nullptr)
        {
            if(currentSum==expectNumber)
            {
                result.push_back(path);
            }
            //下面该段可都不写，都留给情况2去处理，但若写，就需要return，否则会处理两次。
            if(path.size()>0)
                path.pop_back();
            currentSum-=root->val;
            return; //注意，需要return
        }
        preOrder(root->left,expectNumber);
        preOrder(root->right,expectNumber);
        //2.任一结点的左右子树遍历完成,从当前状态存储变量中移去当前结点。
        currentSum-=root->val;
        if(path.size()>0)
            path.pop_back();
    } 
};
```

## 10. 按之字形顺序打印二叉树
**题目描述**：
请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。
**思路**：
- 按之字形顺序打印二叉树需要两个栈。
- 若当前打印的是奇数层（如第1层、第3层），则先保存左子节点再保存右子节点到第一个栈。
- 若当前打印的是偶数层（如第2层、第4层），则先保存右子节点再保存左子节点到第二个栈。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};
*/
class Solution 
{
private:
    stack<TreeNode*> s1;
    stack<TreeNode*> s2;
    vector<int> level;
    vector<vector<int>> result;
public:
    vector<vector<int>> Print(TreeNode* pRoot) 
    {
        if(pRoot==NULL) 
            return result;
        s1.push(pRoot);
        while(!s1.empty()||!s2.empty())
        {
            if(!s1.empty())
            {
                while(!s1.empty())
                {
                    TreeNode* pTemp = s1.top();
                    s1.pop();
                    level.push_back(pTemp->val);
                    if(pTemp->left!=nullptr)
                        s2.push(pTemp->left);
                    if(pTemp->right!=nullptr)
                        s2.push(pTemp->right);
                }
                result.push_back(level);
                level.clear();
            }
            if(!s2.empty())
            {
                while(!s2.empty())
                {
                    TreeNode* pTemp = s2.top();
                    s2.pop();
                    level.push_back(pTemp->val);
                    if(pTemp->right!=nullptr)
                        s1.push(pTemp->right);
                    if(pTemp->left!=nullptr)
                        s1.push(pTemp->left);                 
                }
                result.push_back(level);
                level.clear();
            }
        }
        return result;
    }
};
```

## 11. 二叉搜索树的第k个结点
**题目描述**:
给定一颗二叉搜索树，请找出其中的第$k$大的结点。例如， 5 / \ 3 7 / \ / \ 2 4 6 8 中，按结点数值大小顺序第三个结点的值为4。
**思路**：
- 中序遍历BST可得有序序列。
- 在中序遍历的“访问”操作中计数。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};
*/
class Solution 
{
private:
    int num=0;
    TreeNode* pNode=nullptr;
public:
    TreeNode* KthNode(TreeNode* pRoot, int k)
    {
        if(pRoot==nullptr)
            return pNode;
        KthNode(pRoot->left,k);
        //在“访问”操作中计数
        num++;
        if(num==k)
        {
            pNode=pRoot; 
            return pNode;
        }
        KthNode(pRoot->right,k);
        return pNode;
    }
};
```

## 12. 二叉搜索树的后序遍历序列
**题目描述**：
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则输出Yes,否则输出No。假设输入的数组的任意两个数字都互不相同。
**思路**：
- 后序遍历顺序：`<left，right，root>`。
- 对于二叉搜索树的后续遍历序列，如果去掉最后一个元素x（也就是根）的序列为T，那么T满足：T可以分成两段，前一段（左子树）小于x，后一段（右子树）大于x，且这两段（子树）都是合法的后序序列。
- 故以该规律，递归验证每一左右子树。

**代码**：
```C++
class Solution 
{
public:
    bool VerifySquenceOfBST(vector<int> sequence) 
    {
        if(sequence.empty())
            return false;
        return Check(sequence,0,sequence.size()-1);
    }
    bool Check(vector<int> sequence, int start, int end)
    {
        //递归出口条件1：一个或零个元素，返回true
        if(start>=end)
            return true;
        int rootVal = sequence[end];
        //分出左子树
        int i = start;
        for(;i<end;i++)
        {
            if(sequence[i]>rootVal)
                break;
        }
        //验证右子树中所有元素都大于root值
        for(int j = i;j<end;j++)
        {
            //递归出口条件2：右子树中存在小于root值的元素
            if(sequence[j]<rootVal)
            {
                return false;
            }
        }
        //递归验证左右子树，需都返回true
        bool result1 = Check(sequence,start,i-1);
        bool result2 = Check(sequence,i,end-1);
        return result1 && result2;
    }
};
```

## 13. 树的子结构
**题目描述**:
输入两棵二叉树A，B，判断B是不是A的子结构。此外，约定空树不是任意一个树的子结构。
**思路**：
- 遍历树A，在树A中找出所有与树B的根节点值相等的结点，保存至vector容器`v`中。
- 函数`bool Check(TreeNode* pTest, TreeNode* pRoot2)`：检验以`pTest`为根结点的树是否包含树B。**注意，左右子树都需返回`true`**。
- 对于容器`v`中所有的`pTest`结点，逐一进行检验，**只要有一个成立即可**，即包含。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
private:
    vector<TreeNode*> v;
public:
    bool HasSubtree(TreeNode* pRoot1, TreeNode* pRoot2)
    {
        if(pRoot1==nullptr || pRoot2==nullptr)
            return false;
        //在树A中找出所有与树B的根节点值相等的结点，保存至容器v中
        Find(pRoot1,pRoot2);
        bool result = false;
        //对于容器v中所有的pTest结点，逐一进行检验
        for(int i = 0;i<v.size();i++)
        {
            result = Check(v[i],pRoot2);
            //只要有一个成立即可，即包含
            if(result==true) 
                break;
        }
        return result;
    }
    void Find(TreeNode* pRoot1, TreeNode* pRoot2)
    {
        if(pRoot1==nullptr)
            return;
        if(pRoot1->val==pRoot2->val)
            v.push_back(pRoot1);
        Find(pRoot1->left,pRoot2);
        Find(pRoot1->right,pRoot2);
    }
    bool Check(TreeNode* pTest, TreeNode* pRoot2)
    {
        if(pTest==nullptr && pRoot2==nullptr)
            return true;
        if(pTest==nullptr)
            return false;
        if(pRoot2==nullptr)
            return true;
        if(pTest->val!=pRoot2->val)
            return false;
        bool result1 = Check(pTest->left,pRoot2->left);
        bool result2 = Check(pTest->right,pRoot2->right);
        return result1 && result2;
    }
};
```

## 14. 重建二叉树
**题目描述**:
输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列$$\{1,2,4,7,3,5,6,8\}$$和中序遍历序列$$\{4,7,2,1,5,3,8,6\}$$，则重建二叉树并返回。
**思路**：
- 首先，前序遍历序列的首元素给出根结点；然后，通过中序遍历序列得出左右子树长度；最后，可分割出左右子树各自的前序序列和中序序列。
- 分别递归重建左子树和右子树，递归终止条件：
 - 当序列中只剩一元素(`start==end`)时，返回该根结点指针；
 - 当序列中无元素(`start>end`)时，返回`nullptr`。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};*/
class Solution 
{
public:
    TreeNode* reConstructBinaryTree(vector<int> pre,vector<int> vin) 
    {
        if(pre.empty() || vin.empty() || pre.size()!=vin.size())
            return nullptr;
        return reConstruct(pre, 0, pre.size()-1,vin,0, vin.size()-1);
    }
    TreeNode* reConstruct(vector<int> pre, int preStart, int preEnd,vector<int> vin,int vinStart, int vinEnd)
    {
        //递归出口条件1：当序列中只剩一结点，返回该结点指针
        if(preStart==preEnd)
        {
            TreeNode* pNode = new TreeNode(pre[preStart]);
            return pNode;
        }
        //递归出口条件2：当序列无结点时，返回nullptr
        if(preStart>preEnd)
            return nullptr;
        //前序遍历序列的首元素给出根结点
        int rootVal = pre[preStart];
        TreeNode* pRoot = new TreeNode(rootVal);
        //通过中序遍历序列得出左右子树长度
        int i = vinStart;
        for(;i<vinEnd;i++)
        {
            if(vin[i]==rootVal)
                break;
        }
        int leftLength = i-vinStart; int rightLength = vinEnd-i;
        //分割出左右子树各自的前序序列和中序序列
        int preStartLeft = preStart+1;int preEndLeft=preStartLeft+leftLength-1;
        int vinStartLeft=vinStart;int vinEndLeft=i-1;
        int preStartRight = preEndLeft+1;int preEndRight=preEnd;
        int vinStartRight = i+1;int vinEndRight=vinEnd;
        //分别递归重建左子树和右子树
        pRoot->left = reConstruct(pre,preStartLeft,preEndLeft,vin,vinStartLeft,vinEndLeft);
        pRoot->right =reConstruct(pre,preStartRight,preEndRight,vin,vinStartRight,vinEndRight);
        return pRoot;
    }
};
```

## 15. 序列化二叉树
**题目描述**：
请实现两个函数，分别用来序列化和反序列化二叉树
**思路**：
- 对于序列化：
 - 使用前序遍历，递归的将二叉树的值转化为字符，并且在每次二叉树的结点不为空时，在转化val所得的字符之后添加一个`'，'`作为分割。对于空节点则以 `'#'` 代替。
- 对于反序列化：
 - 按照前序顺序，递归的使用字符串中的字符创建左右子树。注意：在递归时，递归函数的参数一定要是`char **`，这样才能保证每次递归后指向字符串的指针会随着递归的进行而移动。

**代码**：
```C++
/*
struct TreeNode 
{
    int val;
    struct TreeNode *left;
    struct TreeNode *right;
    TreeNode(int x) :val(x), left(NULL), right(NULL) {}
};
*/
class Solution 
{
public:
    char* Serialize(TreeNode *root) 
    {    
        if(root==nullptr)
            return nullptr;
        string str;
        Serialize2(root,str);
        char* result = new char[str.size()+1];
        for(int i = 0;i<str.size();i++)
            result[i] = str[i];
        result[str.size()] = '\0';
        return result;
    }
    void Serialize2(TreeNode* root, string& str)
    {
        if(root==nullptr)
        {
            str+='#';
            return;
        }
        string r = to_string(root->val);
        str+=r;
        str+=',';
        Serialize2(root->left, str);
        Serialize2(root->right, str);
    }
    TreeNode* Deserialize(char *str) 
    {
        if(str==nullptr)
            return nullptr;
        TreeNode* result = Deserialize2(&str);
        return result;
    }
    TreeNode* Deserialize2(char** str) 
    {
        if(**str=='#')
        {
            (*str)++;
            return nullptr;
        }
        int num=0;
        while(**str!= '\0' && **str!=',')
        {
            num=num*10+((**str)-'0');
            (*str)++;
        }
        TreeNode* pRoot = new TreeNode(num);
        if(**str =='\0')
            return pRoot;
        else
            (*str)++;
        pRoot->left = Deserialize2(str);
        pRoot->right= Deserialize2(str);
        return pRoot;
    }
};
```
---------
# 四、字符串
## 1. 替换空格
**题目描述**：
请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为"We Are Happy."则经过替换之后的字符串为"We%20Are%20Happy."
**思路**：
- **注意**：假设在原来的字符串上进行替换，并保证输入的字符串后面有足够多的空余内存。
- **步骤**：从后向前替换：
 - 1.遍历字符串，得到原字符串的长度和空格的个数;
 - 2.计算新字符串的长度，新串长度为原串长度加上两倍的空格个数；
 - 3.从后向前替换：设置两个指针，p1指向原字符串末尾，p2指向新字符串末尾。前移指针p1，逐个把它指向的内容复制到p2指向的位置；
- 所有字符串都只复制一次，时间复杂度：O(n)。

**代码**：
```C++
class Solution 
{
public:
    void replaceSpace(char *str,int length) 
    {
        if(str==nullptr || length<1)
            return;
        //1.遍历字符串，得到空格的个数
        int numSpace = 0;
        for(int i = 0;i<length;i++)
        {
            if(str[i]==' ')
                numSpace++;
        }
        //2.计算新字符串的长度
        int lengthNew = length+2*numSpace;
        //3.从后向前替换：设置两个指针，i指向原字符串末尾，j指向新字符串末尾
        int i = length-1;
        int j = lengthNew-1;
        //终止条件，i<0
        while(i>=0)
        {
            if(str[i]==' ')
            {
                str[j--] = '0';
                str[j--] = '2';
                str[j--] = '%';
                i--;
            }
            else
            {
                str[j--] = str[i--];
            }
        }
    }
};
```
## 2. 整数中1出现的次数（从1到n整数中1出现的次数）
**题目描述**
求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数。
**思路**：
- 对1~n的每一个数：通过对10求余数判断整数的个位数字是不是1，如果这个数字大于10，则除以10之后再判断个位数字是不是1。
- 对每个数字都要做除法和求余运算，以求出该数字中1出现的次数。如果输入数字为$n$，$n$有$O(logn)$位，我们需要判断每一位是不是1，那么它的时间复杂度是$O(nlogn)$。

**代码**：
```C++
class Solution {
public:
    int NumberOf1Between1AndN_Solution(int n)
    {
    	int number=0;
        for(unsigned int i = 1;i<=n;i++)
            number+=NumberOf1(i);
        return number;
    }
    int NumberOf1(unsigned int n)
    {
        int number=0;
        while(n)
        {
            if(n%10 ==1)
                number++;
            n=n/10;
        }
        return number;
    }
};
```
## 3. 翻转单词顺序列
**题目描述**:输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student."，则输出"student. a am I"。 
**思路**：
- 第一步翻转句子中所有的字符。比如翻转"I am a student."中所有的字符得到".tenduts a ma I"，此时不但翻转了句子中单词的顺序，连单词内的字符顺序也被翻转了。
- 第二步再翻转每个单词中字符的顺序，就得到了“studnet. a am I”。

**代码**：
```C++
class Solution 
{
public:
    string ReverseSentence(string str) 
    {
        //1.先整体翻转
        ReverseWord(str,0,str.size()-1);
        int i,start,end;
        i=start=end=0;
        while(i<str.size())
        {
            //空格跳过
            while(i<str.size() && str[i]==' ')
            {
                i++;
            }
            //不是空格，找单词最后一个字符的位置
            start = end = i;
            while(i<str.size() && str[i]!=' ')
            {
                i++;
                end++;
            }
            //2.局部翻转
            ReverseWord(str,start,end-1);
        }
        return str;
    }
    void ReverseWord(string& str, int start, int end)
    {
        while(start<end)
        {
            swap(str[start++],str[end--]);
        }
    }
};
```
## 4. 左旋转字符串
**题目描述**
汇编语言中有一种移位指令叫做循环左移（ROL），现在有个简单的任务，就是用字符串模拟这个指令的运算结果。对于一个给定的字符序列S，请你把其循环左移K位后的序列输出。例如，字符序列S=”abcXYZdef”,要求输出循环左移3位后的结果，即“XYZdefabc”。是不是很简单？OK，搞定它！
**思路**：
-  以"abcdefg"为例，想把它的前两个字符移到后面，得到"cdefgab"。
- 将前两个字符分到第一部分，将后面的所有字符分到第二部分。
- 先分别翻转这两部分，于是得到“bagfedc”。
- 接下来翻转整个字符串，得到“cdefgab”，刚好就是把原始字符串左旋转两位的结果。

**代码**：
```C++
class Solution 
{
public:
    string LeftRotateString(string str, int n) 
    {
        if(str.size()<1 || n>str.size() || n<0)
            return str;
        int startFirst = 0;
        int endFirst = n-1;
        int startSecond = n;
        int endSecond = str.size()-1;
        // 翻转字符串的前面n个字符
        ReverseWord(str,startFirst,endFirst);
        // 翻转字符串的后面部分
        ReverseWord(str,startSecond,endSecond);
        // 翻转整个字符串
        ReverseWord(str,0,str.size()-1);
        return str;
    }
    void ReverseWord(string& str, int start, int end)
    {
        while(start<end)
        {
            char temp = str[start];
            str[start] = str[end];
            str[end] = temp;
            start++;
            end--;
        }
    }
};
```
## 5. 把字符串转换成整数
**题目描述**
将一个字符串转换成一个整数，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0
**输入描述**:
输入一个字符串,包括数字字母符号,可以为空
**输出描述**:
如果是合法的数值表达则返回该数字，否则返回0
**示例**:
输入:
+2147483647
    1a33
输出:
2147483647
    0
**思路**：

**代码**：
```C++
class Solution {
public:
    
    int StrToInt(string str) {
        const char* cstr = str.c_str();
        long long num = 0;
        if((cstr != NULL) && (*cstr!= '\0')) //字符串不为空
        {
            bool minus = false; //检查第一位是否为正负号
            if(*cstr=='+')
                cstr++;
            else if(*cstr=='-')
            {
                cstr++;
                minus = true;
            }
            if(*cstr!='\0')
                num = StrToIntCore(cstr,minus);
        }
        return num;
    }
    long long StrToIntCore(const char* digit, bool minus)
    {
        long long num = 0;
        while(*digit!='\0')
        {   
            //计算数字大小
            if(*digit>='\0' && *digit<='9')
            {
                int flag = minus? -1:1;
                num = num*10+flag*(*digit-'0');
                //溢出判断
                if((!minus && num>0x7FFFFFFF) || (minus && num<(signed int)0x80000000))
                {
                    num=0;
                    break;
                }
         		digit++;
            }
            //非法输入
            else
            {
                num=0;
                break;
            }
        }
		return num;        
    }
};
```


## 6. 正则表达式匹配
**题目描述**:
请实现一个函数用来匹配包括'.'和'\*'的正则表达式。模式中的字符'.'表示任意一个字符，而'\*'表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab\*ac\*a"匹配，但是与"aa.a"和"ab\*a"均不匹配
**思路**：
A：下一个字符是`*`
　　1.若当前字符匹配
　　　　选择1：匹配了，但我当作0匹配(匹配0次)。`s不动，p加2`
　　　　选择2：匹配了，匹配结束(匹配1次)。`s+1,p+2`
　　　　选择3：匹配了，我继续匹配下一个(匹配多次)。`s+1，p不动`
　　2.当前字符不匹配。`s不动，p加2`
B:下一个字符不是`*`，当前字符匹配。`s+1,p+1`
C:下一个字符不是`*`，当前字符不匹配。`返回false`
递归出口条件：
**代码**：
```C++
class Solution {
public:
    bool match(char* str, char* pattern)
    {
    	if(str==NULL || pattern==NULL)
        	return false;
        return matchCore(str,pattern);
    }
    bool matchCore(char* str, char* pattern)
    {
        //递归出口条件:字符串和模式串同时走完
        if(*str=='\0' && *pattern=='\0')
            return true;
        //递归出口条件：字符串没走完，模式串走完，返回false
        if(*str!='\0' && *pattern=='\0')
            return false;
        //A：下一个字符是*
        if(*(pattern+1)=='*') 
        {
            //1.若当前字符匹配
            if(*pattern==*str || (*pattern=='.' && *str!='\0'))
            	return matchCore(str,pattern+2)    //选择1：匹配了，但我当作0匹配(匹配0次)
            		|| matchCore(str+1,pattern+2)  //选择2：匹配了，匹配结束(匹配1次)    	
            		|| matchCore(str+1,pattern);    //选择3：匹配了，我继续匹配下一个(匹配多次)
            //2.当前字符不匹配
            else 
            	return matchCore(str,pattern+2); //匹配结束
        }
        //B:下一个字符不是*，当前字符匹配
        if(*pattern==*str || (*pattern=='.' && *str!='\0'))
             	return matchCore(str+1,pattern+1);
        //C:下一个字符不是*，当前字符不匹配
		return false;           
    }
};
```
## 7. 表示数值的字符串
**题目描述**
请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。 但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。
**思路**：
- 表示数值的字符串: A`.`B`e/E`C 例如：“+123.45e+6”
 - 其中，A、C可能是以+/-开头的0~9的数位串，即整型；B也是0~9的数位串，但前面不能有正负号，即无符号整型。
- 故步骤为：扫描A部分；遇到小数点`.`，扫描B部分；遇到`e/E`，扫描C部分。

**代码**：
```C++
class Solution {
public:
    bool isNumeric(char* str)
    {
        if(str==NULL)
            return false;
        bool numeric = scanInteger(&str);
        if(*str=='.')
        {
            ++str;
            numeric = scanUnsignedInteger(&str) || numeric;
        }
        if(*str=='e' || *str=='E')
        {
            ++str;
            numeric = numeric && scanInteger(&str);
        }
        return numeric && *str=='\0';
    }
	bool scanInteger(char** str)
    {
        if(**str=='+' || **str=='-')
            ++(*str);
        return scanUnsignedInteger(str);
    }
    bool scanUnsignedInteger(char** str)
    {
        const char* before = *str;
        while(**str!='\0' && **str>='0' && **str<='9')
            ++(*str);
        return *str>before;
    }
};
```
## 8. 字符串的排列
**题目描述**
输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。
**输入描述**:
输入一个字符串,长度不超过9(可能有字符重复),字符只包括大小写字母。
**思路**：

**代码**：
```C++
待
```
-----------
# 五、数组
## 1. 构建乘积数组
**题目描述**
给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素`B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]`。不能使用除法。
**思路**：
- 把数组B看成一个矩阵来创建（见书中）。B[i]的值可以看作图中的矩阵中每行的乘积。
- 下三角用自上而下的顺序计算，`C[i] =C[i-1]*A[i-1]`。
- 上三角用自下而上的顺序计算，`D[i]= D[i+1]*A[i+1]`。
- 因此，先算下三角中的连乘，即先算出B[i]中的一部分，然后倒过来按上三角中的分布规律，把另一部分也乘进去。

**代码**：
```C++
class Solution 
{
public:
    vector<int> multiply(const vector<int>& A) 
    {
    	int length = A.size();
        vector<int> B(length); //vector容器的构造函数
        if(length<=0) return B;
        //计算下三角连乘
        B[0]=1;
        for(int i = 1;i<length;i++)
        {
            B[i]=B[i-1]*A[i-1];
        }
        //计算上三角
        int temp = 1;
        for(int j=length-2;j>=0;j--)
        {
            temp *= A[j+1];
            B[j] *= temp;
        }
        return B;
    }
};
```
## 2. 连续子数组的最大和
**题目**：输入一个整型数组，数组里有正数也有负数。数组中一个或连续的多个整数组成一个子数组。求所有子数组的和的最大值。要求时间复杂度为 O(n)。
**说明**：例如输入的数组为{1, -2, 3, 10, -4, 7, 2, -5}，和最大的子数组为｛3, 10, -4, 7, 2}。因此输出为该子数组的和18。
**思路**：
- 从头到尾逐个累加数组中的每个数字。若加上`arr[i]`后的当前和`sumCurrent`小于`arr[i]`，则抛弃当前和`sumCurrent`及之前的所有数字，置当前和`sumCurrent`为`arr[i]`。同时每一步累加都记录下最大和`sumMax`。

**代码**： 
```C++
class Solution 
{
public:
    int FindGreatestSumOfSubArray(vector<int> array)
    {
    	if(array.empty()) return 0;
        int size = array.size();
        int sumCurrent=0;
        int sumMax = 0x80000000;//int的最小值
        //从头到尾逐个累加数组中的每个数字
        for(int i = 0; i<size;i++)
        {
            sumCurrent += array[i];
            //若当前和sumCurrent小于arr[i]
            if(array[i]>sumCurrent)
                sumCurrent=array[i];
            //记录下最大和sumMax
            if(sumCurrent>sumMax) 
                sumMax = sumCurrent;
        }
        return sumMax;
    }
};
```
## 3. 旋转数组的最小数字
**题目描述**
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。 输入一个非递减排序（递增，可能相等）的数组的一个旋转，输出旋转数组的最小元素。 例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。 NOTE：给出的所有元素都大于0，若数组大小为0，请返回0。
**思路**：
**思路1**：顺序查找，遍历数组，找到`arr[i]>arr[i+1]`（稍微优化），时间复杂度$O(n)$，需改进。
**思路2**：旋转数组是两个排序的子数组，采用二分查找。时间复杂度$O(logn)$：
- **情况1**：
 - 用指针`p1`指向第一个数组的第一个元素，用指针`p2`指向第二个数组的第2个元素。对于旋转数组，有特性：`arr[p1]>arr[p2]`（`第一个数组>=第二个数组`）；
 - 计算中间元素为指针`p`，如果`arr[p]>=arr[p1]`，则指针`p`在第一个数组中，此时最小元素应该位于中间元素的后面，令`p1=p;`如果`arr[p]<=arr[p1]`，则指针`p`在第二个数组中，此时最小元素应该位于中间元素的前面，令`p2 = p;`；
 - `p1`始终在第一个数组，`p2`始终在第二个数组，故循环终止条件为：`(p1+1)=p2`，`arr[p2]`即为最小元素。
- **情况2**：若旋转数组是将原来的0个元素搬到最后面，即数组有序，即`arr[p1]<arr[p2]`，该情况是情况1中的特例，情况1中的做法是不适用的，直接返回最小元素`arr[0]`。
- **情况3**：当指针`p`，`p1`，`p2`指向的元素相同时，即`arr[p]==arr[p1] && arr[p]==arr[p2]`时，无法判断该将中间的数字是位于第一个数组还是第二个数组，无法继续用二分法，转而对`[p1，p2]`采用顺序查找法。

**代码**：
```C++
//思路1：顺序查找，时间复杂度O(n)
class Solution 
{
public:
    int minNumberInRotateArray(vector<int> rotateArray) 
    {
        int min = 0;
        if(!rotateArray.empty())
        {
            int size = rotateArray.size();
            //顺序查找，遍历数组，找到arr[i+1]<arr[i]
            for(int i = 0;i<size-1;i++)
            {
                if(rotateArray[i+1]<rotateArray[i])
                {
                    min = rotateArray[i+1];
                    break;
                }
            }
        }
        return min;
    }
};
```
```C++
//思路2：二分查找，时间复杂度O(logn)
class Solution {
public:
    int minNumberInRotateArray(vector<int> rotateArray) {
        if(rotateArray.empty())  return 0;
        int size = rotateArray.size();
        int p1 = 0;
        int p2 = size-1;
        int minIndex=0;
        //情况2，数组有序，直接返回arr[0]
        if(rotateArray[p1]<rotateArray[p2]) return rotateArray[0];
        //情况1，二分查找
        while((p1+1)!=p2)
        {
            int p = (p1+p2)/2;
            //情况3，转为对[p1,p2]采用顺序查找
            if(rotateArray[p1]==rotateArray[p] && rotateArray[p]==rotateArray[p2])
                return MinInOrder(rotateArray,p1,p2);
            if(rotateArray[p]>=rotateArray[p1]) p1 = p;
            else if(rotateArray[p]<=rotateArray[p2]) p2 = p;
        }
        return rotateArray[p2];
    }
    int MinInOrder(vector<int>& rotateArray,int p1, int p2)
    {
        int min = 0;
    	for(int i = p1;i<p2;i++)
        {
            	if(rotateArray[i+1]<rotateArray[i])
            	{
                	min = rotateArray[i+1];
                	break;
            	}
        }
        return min;
    }
    
};
```
## 4. 数字在排序数组中出现的次数
**题目描述**
统计一个数字在排序数组中出现的次数。
**思路**：
- **思路1**：顺序遍历数组，时间复杂度$O(n)$，需改进。
- **思路2**：有序数组，使用二分查找。分两次二分查找：
 - **第一次**找出该数字第一次出现的位置:`(mid==start) || (data[mid-1]!=target))`即已经是最左边的位置或其左边的一个数不等于该数；
 - **第二次**找出该数字最后一次出现的位置：`(mid==end) || (data[mid+1]!=target)`即已经是最右边的位置或其右边的一个数不等于该数。
- 可用递归实现，也可以用循环实现。

**代码**：
```C++
//思路2，递归实现
class Solution 
{
public:
    int GetNumberOfK(vector<int> data ,int k) 
    {
        int number = 0;
        if(data.empty()) return number;
        int size = data.size();
        int firstIndex = getFirst(data,0,size-1,k);
        int lastIndex = getLast(data,0,size-1,k);
        if (firstIndex > -1 && lastIndex > -1)
        {
        	number = lastIndex - firstIndex + 1;
        }
        return number;
    }
    int getFirst(vector<int>& data, int start, int end, int target)
    {
        if(start>end) return -1;
        int mid = (start+end)/2;
        if(data[mid]==target)
        {
            if((mid==start) || (data[mid-1]!=target))//让它是第一个出现的位置
            //if((mid==start) || (mid-1>start&&data[mid-1]!=target))//等价
                return mid;
            else end = mid-1;
        }
        else if(data[mid]>target) end = mid-1;
        else start = mid+1;
        return getFirst(data,start,end,target);
    }
    int getLast(vector<int>& data, int start, int end, int target)
    {
        if(start>end) return -1;
        int mid = (start+end)/2;
        if(data[mid]==target)
        {
           if((mid==end) || (data[mid+1]!=target))//让它是最后一个出现的位置
           //if((mid==end) || (mid+1<end&&data[mid+1]!=target))//等价
                return mid;
            else
                start = mid+1;
        }
        else if(data[mid]>target) end = mid-1;
        else start = mid+1;
        return getLast(data,start,end,target);
    }
};
```
## 5. 数组中重复的数字
题目描述
在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。 例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是第一个重复的数字2。


## 6. 数组中只出现一次的数字
题目描述
一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

## 7. 数组中出现次数超过一半的数字
题目描述
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。

## 8. 把数组排成最小的数
**题目描述**
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。
**思路**：
**代码**：
```C++
class Solution {
public:
    string PrintMinNumber(vector<int> numbers) {
        int length = numbers.size();
        if(length==0) return "";
        sort(numbers.begin(),numbers.end(),compare);
        string result;
        for(int i = 0; i<length;i++)
            result += to_string(numbers[i]);
        return result;
    }
    static bool compare(int a, int b)
    {
        string A = to_string(a)+to_string(b);
        string B = to_string(b)+to_string(a);
        return A<B;
    }
};
```
## 9. 调整数组顺序使奇数位于偶数前面
**题目描述**
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有的奇数位于数组的前半部分，所有的偶数位于位于数组的后半部分，并保证奇数和奇数，偶数和偶数之间的相对位置不变。
**思路**：
1.剑指offer书中，是要求调整数组顺序，使得奇数位于偶数前面，没有要求奇数和奇数，偶数和偶数的相对位置不变。
解法是：指针`p1`指向数组头部，`p1`只后移。指针`p2`指向数组尾部，`p2`只前移。当`arr[p1]`为偶数，`arr[p2]`为奇数，则交换。`p1`继续后移，`p2`继续前移，一直循环。终止条件是：`p1`与`p2`交叉。
2.本题目中，要求奇数在前，偶数在后，且奇数相对于奇数，偶数相对于偶数的相对位置不变。则解法是：新建一个数组，先把原数组中的奇数push进去，再把偶数push进去，然后用新数组数据覆盖原数组即可。复杂度O(n)。
**代码**：
```C++
class Solution {
public:
    void reOrderArray(vector<int> &array) {
        if(array.empty()) return;
        int size = array.size();
        //新建一个数组
        vector<int> result;
        //先把原数组中的奇数push进去
        for(int i = 0; i<size;i++)
        {
        	if(array[i]%2==1)
                result.push_back(array[i]);
        }
        //再把原数组中的偶数push进去
        for(int i = 0; i<size;i++)
        {
        	if(array[i]%2==0)
                result.push_back(array[i]);
        }
        //用新数组覆盖原数组
        array=result;
    }
};
```
## 10. 二维数组中的查找
**题目描述**
在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
**思路**：首先选取数组中右上角的数字。如果该数字等于要查找的数字，则查找过程结束；如果该数字大于要查找的数字，则去除该数字所在列；如果该数字小于要查找的数字，则去除该数字所在行。
**代码**：
```C++
class Solution {
public:
    bool Find(int target, vector<vector<int>> array) {
		//防御型编程
        if(array.empty()) return false;
        //计算二维数组的行和列
        int rows = array.size();
        int cols = array[0].size();
        //首先选取数组中右上角的数字
        int i = 0;
        int j = cols-1;
        bool result = false;
        while(i<rows && j>=0)//注意二维数组行列的取值范围为[0~rows-1, 0~cols-1]
        {
            //如果该数字等于要查找的数字，则查找过程结束
            if(array[i][j]== target)
            {
                result = true;
                break;
            }
            //如果该数字大于要查找的数字，则去除该数字所在列
            else if(array[i][j]>target)   
            {
                j--;
            }
            //如果该数字小于要查找的数字，则去除该数字所在行
            else
            {
            	i++;
            }
        }
        return result;
    }
};
```
## 11. 数组中的逆序对
**题目描述**
在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。并将P对1000000007取模的结果输出。 即输出P%1000000007
**输入描述**:
题目保证输入的数组中没有的相同的数字
**数据范围**：
对于%50的数据,$size<=10^4$
对于%75的数据,$size<=10^5$
对于%100的数据,$size<=2\*10^5$
**示例**:
输入
1,2,3,4,5,6,7,0
输出
7
**思路**：

**代码**：
```C++
//不对，未完成
class Solution {
private:
    int count=0;
public:
    int InversePairs(vector<int> data) {
        int n = data.size();
        return InversePairs2(data, n);
    }
    int InversePairs2(vector<int>& data, int n) 
    {
		if(n<2) return 0;//就是为了返回，返回的值无所谓
        int leftLength= n/2;
        int rightLength= n-leftLength;
        vector<int> left;
        vector<int> right;
        for(int i = 0;i<leftLength;i++) left.push_back(data[i]);
        for(int i = leftLength;i<n;i++) right.push_back(data[i]);
        InversePairs2(left,leftLength);    //sorting the left subarray
        InversePairs2(right,rightLength);  //sorting the right subarray
        mergeNew(data,left,leftLength,right,rightLength); //merge left and right into A
        return count%1000000007;
    }
    void mergeNew(vector<int>& data, vector<int>& left, int leftLength, vector<int>& right,int rightLength)
    {
        int i=leftLength+rightLength-1;
		int j = leftLength-1;
        int k = rightLength-1;
        while(j>=0 && k>=0)
        {
            if(left[j]>right[k]) 
            { 
            	data[i--] = left[j--];
            	count+=(k+1);
            }
            else 
            {
            	data[i--] = right[k--];
            }
        }
        while(j>=0) data[i--]=left[j--];
        while(k>=0) data[i--]=right[k--];
    }
};
```
## 12. 顺时针打印矩阵
**题目描述**
输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下矩阵： 
```
1   2  3  4
5   6  7  8
9  10 11 12
13 14 15 16
```
则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.
**思路**：分解成若干个简单的问题。
1.每次打印矩阵的一个圈。关键：循环继续的条件是：`cols>start*2 && row>start*2`
2.将打印一圈分成四步：第一步，从左到右打印一行；第二步，从上到下打印一列；第三步；从右到左打印一行；第四步，从下到上打印一列。
3.四步的前提条件：
第一步总是需要；
第二步的前提条件是终止行号大于起始行号；
第三步的前提条件是该圈内至少两行两列，即终止行号大于起始行号，终止列号大于起始列号；
第四步的前提条件：该圈内至少有三行二列，即终止行号比起始行号至少大2，终止列号大于起始列号。
**代码**：
```C++
class Solution {
public:
    vector<int> printMatrix(vector<vector<int> > matrix) {
		vector<int> result;
        int rows = matrix.size();
        int cols = matrix[0].size();
        if(rows<=0 || cols<=0) return result;
        int start = 0;
        //每次打印矩阵的一个圈
        while(cols>start*2 && rows> start*2)
        {
            PrintMatrixInCircle(matrix,rows,cols,start,result);
            start++;
        }
        return result;
    }
    void PrintMatrixInCircle(vector<vector<int>>& matrix,int rows,int cols,int start,vector<int>& result)
    {
        int endX = cols-start-1;
        int endY = rows-start-1;
        //1.从左到右打印矩阵
        for(int i = start;i<=endX;i++)
        {
            result.push_back(matrix[start][i]);
        }
        //2.从上到下打印一列
        if(start<endY)
        {
            for(int i = start+1;i<=endY;i++)
            {
                result.push_back(matrix[i][endX]);
            }
        }
        //3.从右到左打印一行
        if(start<endX && start< endY)
        {
            for(int i = endX-1;i>=start;i--)
            {
                result.push_back(matrix[endY][i]);
            }
        }
        //4.从下到上打印一列
        if(start<endX && start <endY -1)
        {
            for(int i = endY-1; i>=start+1;i--)
            {
                result.push_back(matrix[i][start]);
            }
        }
    }
};
```
--------
# 六、排序
## 1.数组中出现次数超过一半的数字
题目描述
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。
**思路**：结合数组特性：数组中有一个数字出现的次数超过了数组长度的一半，如果将这个数组排序，那么排序之后位于数组中间的数字一定就是那个出现次数超过数组长度一半的数字。这个数字就是统计学上的中位数，即长度为$n$的数组中第$n/2$大的数字。已经有成熟的时间复杂度为$O(n)$的算法得到数组中任意第$k$大的数字。
第一，防御性编程，判断数组是否有效；第二，利用快速排序中的分割(partition)方法，选主元位置及重排数组。如果返回的主元位置(pIndex)小于数组中间位置((length-1)/2)，则对左半部分进行分割，否则对右半部分进行分割，直到返回的pIndex等于((length-1)/2)；第三，遍历数组，验证该数是否出现了超过一半的次数。
<!-- more --> 
**代码**：
```C++
class Solution {
public:
    int MoreThanHalfNum_Solution(vector<int> numbers) {
    	int size = numbers.size();
        //1.防御性编程，判断数组是否有效；
        if(size==0) return 0;
        int start = 0;
        int end = size-1;
        int middle = end/2;
        //2.利用快速排序中的分割(partition)方法，选主元位置及重排数组
        int pIndex = partition(numbers,start,end);
        //直到返回的pIndex等于((length-1)/2)
        while(pIndex!=middle)
        {
            if(pIndex<middle)
            {
                start = pIndex+1;
                pIndex = partition(numbers,start,end);
            }
            else
            {
                end = pIndex-1;
                pIndex = partition(numbers,start,end);
            }
        }
        int result=numbers[middle];
        //3.遍历数组，验证该数是否出现了超过一半的次数。
        if(isMoreThanHalf(numbers,result,size))
            return result;
        else
            return 0;
    }
    int partition(vector<int>& numbers, int start,int end)
    {
        int pivot = numbers[end];
        int pIndex = start;
        for(int i = start;i<end;i++)
        {
            if(numbers[i]<=pivot)
            {
                int temp = numbers[i];
                numbers[i] = numbers[pIndex];
                numbers[pIndex] = temp;
                pIndex++;
            }
        }
        int temp = numbers[pIndex];
        numbers[pIndex] = pivot;
        numbers[end] = temp;
        return pIndex;
    }
    bool isMoreThanHalf(vector<int>& numbers,int result,int size)
    {
        int count = 0;
        for(int i = 0;i<size;i++)
        {
            if(numbers[i]==result)
                count++;
        }
        if(2*count<=size) return false;
        else return true;
    }
};
```
## 2.最小的k个数
**题目描述**：
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。
**思路**：
思路1：利用快速排序排序数组，位于前面的$k$个数即是最小的$k$个数。时间复杂度为$O(nlogn)$。
第一，防御性编程，如果数组为空或$k>length$，则返回空；第二，利用快速排序，对数组进行排序；第三，输出数组中的前$k$个数。
思路2：
由上题“数组中出现次数超过一半的数字”得到启发，基于快速排序中的分割(partition)方法来解决问题。利用partiton函数，如果返回的$pIndex<(k-1)$，则对右半部分进行partition，否则，对左半部分进行partition，直到返回的主元位置pIndex等于$(k-1)$。这样调整后，位于数组中左边的k个数字就是最小的$k$个数字，但这$k$个数字不一定是排序的。时间复杂度为$O(n)$。
**代码**：
```C++
//实现思路1
class Solution {
public:
    vector<int> GetLeastNumbers_Solution(vector<int> input, int k) {
        vector<int> result;
        int size = input.size();
        //1.防御性编程，如果数组为空或k>size，则返回
        if(size==0 || k>size) return result;
        int start = 0;
        int end = size-1;
        //2.利用快速排序，对数组进行排序
        quickSort(input,start,end);
        //3.输出数组中的前k个数
        for(int i = 0;i<k;i++)
        {
            result.push_back(input[i]);
        }
        return result;
    }
    void quickSort(vector<int>& numbers, int start, int end)
    {
        if(start>end) return;
        int pIndex = partition(numbers,start,end);
        quickSort(numbers,start,pIndex-1);
        quickSort(numbers,pIndex+1,end);
    }
    int partition(vector<int>& numbers, int start,int end)
    {
        int pivot = numbers[end];
        int pIndex = start;
        for(int i = start;i<end;i++)
        {
            if(numbers[i]<=pivot)
            {
                int temp = numbers[i];
                numbers[i] = numbers[pIndex];
                numbers[pIndex] = temp;
                pIndex++;
            }
        }
        int temp = numbers[pIndex];
        numbers[pIndex] = pivot;
        numbers[end] = temp;
        return pIndex;
    }
};
```

# 七、位运算
## 1.数组中只出现一次的数字（#56）
**题目描述**
一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。
**思路**：核心思想：[异或去重](http://blog.csdn.net/ns_code/article/details/27568975)；异或的性质：交换律，结合律，以及`a^a=0`，`a^0=a`
**1.考虑简单情况**：数组中只有一个数字`n`出现了一次，其他都出现了两次。
将数组中所有元素逐个异或，则结果为只出现一次的`n`，即`a^a^b^b^...^n^...^c^c=n`。
**2.本题求解思路**：故将数组分成两部分解决，一部分包含只出现一次的数字`n_1`，一部分包含只出现一次的数字`n_2`，同时保证出现两次的数字在同一数组,而不是分散在l不同的数组。对两数组分别逐元素异或，即分别得到`n_1`，`n_2`。
**3.数组划分方法**：对整个数组，逐元素异或，则结果为`n_1^n_2`，即`n_1`和 `n_2`异或的结果。`n_1`和`n_2`不相同，故结果定不为0，则结果的二进制表示定至少有一位为1（即`n_1`和`n_2`的二进制表示的该位不相同），找出结果的二进制表示中第一次为1的下标索引`index`，通过这个下标索引`index`，对整个数组中的元素进行数组划分。数组中每个元素的二进制表示的第`index`位是1的分为一组，是0的分为另一组。这样保证了`n_1`和`n_2`分别在两个组，且出现两次的数字在同一个组，不会被分到不同的组(因此相同数字的二进制表示的`index`位必定是相同的)。
**代码**：
```C++
class Solution {
public:
    void FindNumsAppearOnce(vector<int> data,int* num1,int *num2) {
		int length = data.size();
        if(data.size()<2) return;
        int result = 0; //初始值为0,a^0=a
        // get num1 ^ num2
        for(int i =0;i<length;i++)
            result ^=data[i];
        // get index of the first bit, which is 1 in resultExclusiveOR
        unsigned int indexOf1 = FindFirstBitIs1(result);
        *num1 = *num2 = 0;
        // divide the numbers in data into two groups,
        // the indexOf1 bit of numbers in the first group is 1,
        // while in the second group is 0
        for(int j=0;j<length;j++)
        {
            if(IsBit1(data[j],indexOf1))
                *num1^=data[j];
            else
                *num2^=data[j];
        }
    }
    //Find the index of first bit which is 1 in num (assuming not 0)
    unsigned int FindFirstBitIs1(int num)
    {
        int indexBit=0;
        while(((num&1)==0) && (indexBit<8*sizeof(int)))
        {
            num = num>>1;
            ++indexBit;
        }
        return indexBit;
    }
    // Is the indexBit bit of num 1?
    bool IsBit1(int num, unsigned int indexBit)
    {
        num=num>>indexBit;
        return (num&1);
    }
};
```
## 2.二进制中1的个数
**题目描述**
输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。
**思路**：
**首先**，直观的思路是：先判断整数二进制表示中的最后一位是不是1；然后将整数右移一位，再次判断，直到整数变为0。判断二进制表示中最后一位是不是1的方法是：`n&1`，因为1的二进制表示中除了最后一位为1以外，其余全为0。
**然后**，右移整数n的做法存在隐患，就是当n是负数的时候，[n右移会在开头补符号位1（算术右移）](http://blog.csdn.net/morewindows/article/details/7354571)，而不是补0。故采用将1循环左移的方法，逐个判断n的二进制表示的第0位到最高位是否为1。
**代码**：
```C++
class Solution {
public:
     int  NumberOf1(int n) {
         int count = 0;
         unsigned int flag = 1;
         //直到1的循环左移为0
         while(flag)
         {
             if(n & flag)
             	count++;
             //左移1
             flag =flag<<1;
         }
         return count;
     }
};
```
# 八、递归
## 1.斐波那契数列（#10）
**题目描述**
大家都知道斐波那契数列，现在要求输入一个整数n，请你输出斐波那契数列的第n项。(n<=39)
**注**：本题中`n=0`时，输出是0。斐波那契数列从第1项开始。即本题斐波那契数列是`0,1,1,2,3,5,...`
**思路**：
**思路1**：最直观的递归，返回`Fibonacci(n-1)+Fibonacci(n-2)`。但有大量冗余计算，时间复杂度过高，无法通过。
**思路2**：带记忆的递归，用辅助数组存储下已经计算过的结果。
**思路3**：迭代计算。
**代码**：
```C++
//思路1：最直观的递归，时间复杂度过高
class Solution {
public:
    int Fibonacci(int n) {
        if(n<=1) return n;
        return Fibonacci(n-1)+Fibonacci(n-2);
    }
};
```
```
//思路2：带记忆的递归，用辅助数组存储下已经计算过的结果
class Solution {
private:
    int F[40];//默认为0
public:
    int Fibonacci(int n) {
        if(F[n]!= 0) 
            return F[n];
        if(n<=1) 
        {
         	F[n]=n;
        	return F[n];
        }
        else
        {
            F[n]=Fibonacci(n-1)+Fibonacci(n-2);
        	return F[n];
        }
    }
};
```
```
//思路3：迭代计算。
class Solution {
public:
    int Fibonacci(int n) {
    	int num1=0;
    	int num2=1;
    	int num3;
        if(n<=1) 
            return n;
		for(int i = 2;i<=n;i++)
        {
            num3 = num1+num2;
            num1 = num2;
            num2 = num3;
        }
        return num3;
    }
};
```
## 2. 青蛙跳台阶问题（#10）
**题目描述**
一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
**思路**：类似斐波那契数列
f(n)=　1, (n=1)
　　　 2, (n=2)
　　　 f(n-1)+f(n-2) ,(n>2,n为整数)
**代码**：
```C++
class Solution {
public:
    int jumpFloor(int number) {
        if(number<=0) return number; //defensive
        if(number==1) return 1;
        if(number==2) return 2;
        return jumpFloor(number-1)+jumpFloor(number-2);
    }
};
```
## 3.变态跳台阶（#10）
**题目描述**
一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法。
**思路**：
因为n级台阶，第一步有n种跳法：跳1级，跳2级，...，跳n级
跳1级，剩下n-1级，则剩下跳法是f(n-1)
跳2级，剩下n-2级，则剩下跳法是f(n-2)
所以f(n)=f(n-1)+f(n-2)+...+f(1)+f(0)
因为f(n-1)=f(n-2)+f(n-3)+...+f(1)+f(0)
所以f(n)=2xf(n-1)
故：
f(n)=　1, (n=1)
　　　 2xf(n-1),(n>1,n为整数)
**代码**：
```C++
class Solution {
public:
    int jumpFloorII(int number) {
        if(number<=0) return number; //defensive
		if(number==1) return 1;
        return 2*jumpFloorII(number-1);
    }
};
```
## 4.矩形覆盖（#10）
**题目描述**
我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？
**思路**：仍旧是斐波那契数列
第一块有两种方式：横着放和竖着放。
横着放后，覆盖方法为f(n-2)，因为下方也被占用，下方必须也横着放。
竖着放后，覆盖方法为f(n-1);
所以总的覆盖方法为f(n)=f(n-1)+f(n-2);
故：
f(n)=　1, (n=1)
　　　 2, (n=2)
　　　 f(n-1)+f(n-2) ,(n>2,n为整数)
**代码**：
```C++
class Solution {
public:
    int rectCover(int number) {
 		if(number<=0) return number; //defensive
        if(number==1) return 1;
        if(number==2) return 2;
        return rectCover(number-1)+rectCover(number-2);
    }
};
```

# 九、回溯

# 十、动态规划与贪婪算法





