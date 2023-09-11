给你链表的头结点 `head` ，请将其按 **升序** 排列并返回 **排序后的链表** 。
**示例 1：**
![[Pasted image 20230911145125.png]]
```txt
输入：head = [4,2,1,3]
输出：[1,2,3,4]
```
**示例 2：**
![[Pasted image 20230911145157.png]]

```txt
输入：head = [-1,5,3,4,0]
输出：[-1,0,3,4,5]
```
**示例 3：**

```txt
输入：head = []
输出：[]
```
**提示：**

- 链表中节点的数目在范围 `[0, 5 * 104]` 内
- `-105 <= Node.val <= 105`

- **快慢指针**
- **归并排序** 

```c++
class Solution {
public:
    ListNode* merge(ListNode* l1, ListNode* l2) {
        ListNode head, *p = &head;
        while(l1 && l2) {
            if(l1->val < l2->val) {
                p->next = l1;
                l1 = l1->next;
            } else {
                p->next = l2;
                l2 = l2->next;
            }
            p = p->next;
        }
        p->next = l1 ? l1 : l2;
        return head.next;
    }

    ListNode* findMid(ListNode* l) {
        ListNode *slow = l, *fast = l;
        while(fast->next && fast->next->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow;
    }

    ListNode* sortList(ListNode* head) {
        if(!head || !head->next) return head;
        ListNode* mid = findMid(head);
        ListNode* l2 = mid->next, *l1 = head;
        mid->next = nullptr;
        l1 = sortList(l1);
        l2 = sortList(l2);
        return merge(l1, l2);
    }
};
```

**相似题目** 
[143. 重排链表 - 力扣（LeetCode）](https://leetcode.cn/problems/reorder-list/description/)

## 143. 重排链表

给定一个单链表 `L` 的头节点 `head` ，单链表 `L` 表示为：
$L_0 → L_1 → … → L_{n - 1} → L_n$ 
请将其重新排列后变为：
$L_{0} → L_{n} → L_{1} → L_{n - 1} → L_{2} → L_{n} - 2 → …$
不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

**示例 1：**

![[Pasted image 20230911151256.png]]
**输入：head = \[1,2,3,4\]
输出：\[1,4,2,3\]
示例 2：**
![[Pasted image 20230911151335.png]]
**输入：head = \[1,2,3,4,5\]
输出：\[1,5,2,4,3\]
提示：**

- 链表的长度范围为 $[1, 5 * 10^4]$
- `1 <= node.val <= 1000`

```c++
class Solution {
public:
    ListNode* findMid(ListNode* head) {
        ListNode *fast = head, *slow = head;
        while(fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow;
    }

    ListNode* reverseList(ListNode* head) {
        ListNode* cur = head, *pre = nullptr;
        while(cur) {
            head = head->next;
            cur->next = pre;
            pre = cur;
            cur = head;
        }
        return pre;
    }

    void reorderList(ListNode* head) {
        ListNode* mid = findMid(head);
        mid = reverseList(mid);
        ListNode* li1 = head, *li2 = mid;
        while(mid->next) {
            mid = mid->next;
            head = head->next;
            li1->next = li2;
            li2->next = head;
            li1 = head;
            li2 = mid;
        }
    }
};
```