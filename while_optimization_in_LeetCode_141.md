# LeetCode 141 中的 while 循环优化

## LeetCode 141 分析

题目是判断链表是否有环. 

假设让两个速度不同的人在一个起点同时出发, 如果他们还能再次相遇则证明有环. 因此可以设置两个指针 slow 和 fast,  slow 每次前进一个节点, fast 每次前进两个节点, 如果两个指针再次相遇则证明有环. 

## 代码实现

### Version 1

我们将 **如果指针相遇了, 那么一定是环** 作为循环条件. 但这个条件引入了一个问题, 两个指针如果初始都在 head, 那么此时它们就是相遇的, 因此要先将两个指针移动一次.

```python
def hasCycle(head: Optional[ListNode]) -> bool:
    # 移动一次指针, 移动前检查空值.
    if head is None or head.next is None:
        return False
    slow = head.next
    fast = head.next.next

    # 如果指针相遇了 (slow is fast), 那么一定是环.
    while slow is not fast:
        # 移动一次指针, 移动前检查空值.
        if fast is None or fast.next is None:
            return False
        slow = slow.next
        fast = fast.next.next
    return True
```

### Version 2

V1 中第一次移动指针和 while 中移动指针的代码是重复的, V2 中我们将合并这段重复的代码 (实际上这是一个 do-while 循环, 但 Python 中并没有 do-while) . 将循环条件改为 **如果指针不是第一次相遇, 那么一定是环**, 就能解决此问题.

```python
def hasCycle(head: Optional[ListNode]) -> bool:
    slow = fast = head
    is_first = True
	
    # 如果指针不是第一次相遇 (not is_first and slow is fast), 那么一定是环. 
    while is_first or slow is not fast:
        is_first = False
        # 移动一次指针, 移动前检查空值.
        if fast is None or fast.next is None:
            return False
        slow = slow.next
        fast = fast.next.next
    return True
```

### V3

V2 中虽然解决了问题, 但引入了一个辅助变量 `is_first`, V3 中我们将不使用此辅助变量. V2 中实际上有两个跳出循环的条件, **判断指针是否是第一次相遇** 和 **判断 fast 指针能否继续移动**. 我们尝试交换两个条件的位置, 将循环条件改为 **如果 fast 不能移动了, 那么一定不是环**.

```python
def hasCycle(head: Optional[ListNode]) -> bool:
    slow = fast = head

    # 如果 fast 不能移动了 (fast is None or fast is not None), 那么一定不是环.
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        # 相遇了.
        if slow is fast:
            return True
    return False
```

## 总结

在使用 while 时, 应比较不同循环条件, 特别是当 while 中有其他跳出循环的条件时, 一个好的循环条件, 可以让代码逻辑更加清晰. 另外, 在 Python 中可以使用一个辅助变量, 模拟 do-while.