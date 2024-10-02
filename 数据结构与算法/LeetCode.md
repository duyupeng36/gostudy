# LeetCode

## 简单题

| 题号                                                                        | 类型   | 思路         |
| :------------------------------------------------------------------------ | ---- | :--------- |
| [206-反转链表](https://leetcode.cn/problems/reverse-linked-list/description/) | 数据结构 | 链表的头插法     |
| [141-环形链表](https://leetcode.cn/problems/linked-list-cycle/description/)   | 数据结构 | 快慢双指针      |
| [67-二进制求和](https://leetcode.cn/problems/add-binary/description/)          | 数学   | 位运算或模拟竖式计算 |

### 反转链表

```go
func reverseList(head *ListNode) *ListNode {  
    // 边界条件。链表为空，或者只有一个节点，不需要反转  
    if head == nil || head.Next == nil {  
       return head  
    }  
  
    var result *ListNode  // 结果  
    var tempNode *ListNode   // 记录当前处理节点
  
    result = head // 记录第一个节点
    head = head.Next  
  
    result.Next = nil // 必须的，否则结果存在环形，导致结果匹配失败
  
    for head != nil{  
       tempNode = head  // 记录当前节点
       head = head.Next   // 头指针指向下一个节点
  
       tempNode.Next = result  // 将当前按节点插入到新链表开头
       result = tempNode  
    }  
  
    return result  
}
```

### 环形链表

```go
package main  
  
func hasCycle(head *ListNode) bool {  
  
    // 边界条件：链表为空和只有一个节点，一定不会存储环  
    if head == nil || head.Next == nil {  
       return false  
    }  
  
    // 慢指针指向第一个节点，快指针指向第二个节点  
    // 慢指针移动一次，快指针移动两次  
    // 当 快指针与慢指针相遇时，一定存在环  
    for slow, fast := head, head.Next; fast != slow; slow, fast = slow.Next, fast.Next.Next {  
       // 当快指针为 nil 或 快指针的 next 为nil，则链表遍历完成，不存在环  
       if fast == nil || fast.Next == nil {  
          return false  
       }  
    }  
    return true  
}
```

### 二进制求和

注意，二进制加法在不进位的情况下，结果与 **位异或** 的结果相同。**进位** 等于 **位与** 的结果 **左移** 一位

```python
class Solution:
    def addBinary(self, a: str, b: str) -> str:
        x, y = int(a, 2), int(b, 2)
        while y:
            ans = x ^ y
            carry = (x & y) << 1
            x, y = ans, carry
        
        return bin(x)[2:]
```

> [!tip] 采用位运算需要注意，字符串长度超过 $64$ 位时，某些语言可能无法进行计算
> 
> 采用 Python，可以使用 Python 的整数不限制长度的特性
> 

更通用，并且算法鲁棒性更好的是通过 **模拟竖式计算**

```go
func addBinary(a string, b string) string {
	//   0b1011
	// + 0b0110
	//  0b10001

	ans := ""
	carry := 0  // 当前的进位
	lenA := len(a)
	lenB := len(b)
	n := max(lenA, lenB)

	// 反序遍历 a 和 b
	// 第 i 位就是 (ai + bi + carry)
	// 进位为 (ai + bi + carry) / 2
	for i := 0; i < n; i++ {

		if i < lenA {
			carry += int(a[lenA - i - 1] - '0')
		}

		if i < lenB {
			carry += int(b[lenB - i - 1] - '0')
		}
		ans = strconv.Itoa(carry % 2) + ans
		carry /= 2
	}

	if carry > 0 {
		ans = "1" + ans
	}
	return ans
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```

## 中等题




### 困难题




