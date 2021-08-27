---
title: "[技術] Linked List的pointer思維"
date: 2021-08-27T06:30:15-04:00
draft: true
---

[leetcode 21. Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/)


C:
```c
struct ListNode* mergeTwoLists(struct ListNode* l1, struct ListNode* l2){
    typedef struct ListNode node_t;
    node_t *head;
    node_t **ptr = &head;
    while (l1 && l2) {
        if (l1->val < l2->val) {
            *ptr = l1;
            l1 = l1->next;
            ptr = &((*ptr)->next);
        } else {
            *ptr = l2;
            l2 = l2->next;
            ptr = &((*ptr)->next);
        }
    }
    if (l1) 
        *ptr = l1;
    else
        *ptr = l2;
    return head;
}
```

go:
```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
    head := new(ListNode)
    ptr := &head
    for l1 != nil && l2 != nil {
        if l1.Val < l2.Val {
            *ptr = l1
            l1 = l1.Next
            ptr = &((*ptr).Next)
        } else {
            *ptr = l2
            l2 = l2.Next
            ptr = &((*ptr).Next)
        }
    }
    if l1 == nil {
        *ptr = l2
    } else {
        *ptr = l1
    }
    return head
}

```