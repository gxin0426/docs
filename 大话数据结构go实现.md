

## 0.知识点总结

### 1.概念

#### 1.逻辑结构和物理结构

- 四种逻辑结构： 
  - 集合结构
  - 线性结构
  - 树形结构
  - 图形结构
- 两种物理结构
  - 顺序存储结构
  - 链式存储结构

#### 2.数据类型

- 原子类型：不可以再分解的基本类型，包括整型，实型，字符型
- 结构类型：由若干个类型组合而成，是可以在分解的。例如整型数组

#### 3.算法

- 算法的特性：输入，输出，有穷性，确定性，可行性
- 算法的设计要求：正确性，可读性，健壮性，时间效率高和存储量低

#### 4.推导大O阶

- 用常数1取代运行时间中的所有加法常数
- 在修改后的运行次数函数中，只保留最高阶项
- 如果最高阶项存在且不是1，则去除与这个项相乘的常数
- O(1)<O(logn)<O(n)<O(nlogn)<O(n<sup>2</sup>)<O(n<sup>3</sup>)<O(2<sup>n</sup>)<O(n!)<O(n<sup>n</sup>)

#### 5.go语言整数取值范围

~~~go

	fmt.Println("int8 : ", math.MinInt8, "----", math.MaxInt8) //  -1 << 7    1 << 7 - 1
	fmt.Println("int16 : ", math.MinInt16, "----", math.MaxInt16) //-1 << 15  1 << 15 - 1
	fmt.Println("int32 : ", math.MinInt32, "----", math.MaxInt32)
	fmt.Println("int64 : ", math.MinInt64, "----", math.MaxInt64)
	fmt.Println("uint8 : ", math.MaxUint8) //1 << 8 -1
	fmt.Println("uint16 : ", math.MaxUint16) // 1 << 16 - 1
	fmt.Println("uint32 : ", math.MaxUint32)
	fmt.Println("uint64 : ", uint64(math.MaxUint64))
res:
	int8 :  -128 ---- 127
    int16 :  -32768 ---- 32767
    int32 :  -2147483648 ---- 2147483647
    int64 :  -9223372036854775808 ---- 9223372036854775807
    uint8 :  0-255
    uint16 :  0-65535
    uint32 :  0-4294967295
    uint64 :  0-18446744073709551615
~~~



## 1.线性表

- **单向链表**

~~~go
//go实现单向表
package main

import "fmt"

type Element int
type LinkNode struct {
	Data Element
	Next *LinkNode
}

func New() *LinkNode {
	return  &LinkNode{}
}

func (head *LinkNode) Insert(i int, e Element) bool {
	p := head

	j := 1
	for p != nil && j < i {
		p = p.Next
		j++
	}
	if p == nil || j > i {
		return false
	}

	s := &LinkNode{e, nil}
	s.Next = p.Next
	p.Next = s
	return true
}

func (head *LinkNode)Traverse(){
	p := head
	for p != nil {
		fmt.Println(p.Data)
		p = p.Next
	}
	fmt.Println("_____________________")
}

func (head *LinkNode) Delete(i int) bool {
	p := head

	if i => p.Len(){
		return false
	}

	j := 1
	for p != nil && j < i {
		p = p.Next
		j++
	}

	if p == nil || j > i {
		return false
	}

	p.Next = p.Next.Next
	return true
}

func (head *LinkNode) Get(i int)Element {
	p := head

	if i == p.Len(){
		return -100001
	}

	for j := 0; j < i; j++{
		if p == nil {
			return -100001
		}
		p = p.Next
	}
	return p.Data
}


func (head *LinkNode)Len() int {
	q := 1
	p := head
	for  p.Next != nil {
		q++
		p = p.Next
	}
	return q
}
~~~

- **双向链表**

~~~go
//双向链表
package main

import "fmt"

type ListNode struct {
	prev *ListNode
	next *ListNode
	value int
}

func NewListNode(v int) *ListNode{
	listNode := &ListNode{
		prev:  nil,
		next:  nil,
		value: v,
	}
	return listNode
}

func (n *ListNode)Prev() *ListNode {
	return n.prev
}

func (n *ListNode)Next() *ListNode{
	return n.next
}

func (n *ListNode) GetValue() int {
	if n == nil {
		return 0
	}
	return n.value
}

type List struct {
	head *ListNode
	tail *ListNode
	len int
}

func NewList() *List{
	list := &List{
	}
	return  list
}

func (l *List) Head() *ListNode{
	return l.head
}

func (l *List) Tail() *ListNode{
	return l.tail
}

func (l *List) Len() int{
	return l.len
}

func (l *List) RPush(v int) {
	n := NewListNode(v)
	if l.Len() == 0 {
		l.head = n
		l.tail = n
	}else {
		tail := l.tail
		tail.next = n
		n.prev = tail
		l.tail = n
	}
	l.len++
}

func (l *List) LPop() *ListNode{
	if l.Len() == 0 {
		return nil
	}
	n := l.head
	if n.next == nil {
		l.head = nil
		l.tail = nil
	}else {
		l.head = l.head.next
	}
	l.len--
	return n
}

func (l *List)Index(i int) *ListNode{

	if i < 0 {
		return  nil
	}

	n := l.head
	for ; i > 0 && n != nil; i-- {
		n = n.next
	}
	return n
}
~~~

- 静态链表
- 循环链表

## 2.栈与队列

- 栈：在表尾插入和删除
- 队列：在一端插入 另一端删除

- **顺序存储实现栈和队列**

~~~go
//适用场景 LeetCode刷题
var queue []int
var stack []int
//入队列 入栈
queue = append(queue, 1)
stack = append(stack, 1)
//出队列 出栈
queue = queue[1: len(queue)]
stack = stack[:len(stack)-1]
~~~

- 链是存储实现栈和队列

~~~go
//适用场景 生产环境
import (
	"container/list"
	"fmt"
)

func main(){
	queue := list.New()
	stack := list.New()

	//在队列首压入
	queue.PushFront(int8(11))
	queue.PushFront(15)
    //在栈首压入
	stack.PushFront(12)
	stack.PushFront(16)
	//出队列 出栈 返回的数据是结构类型 Value 需要断言成相应的类型
    //在队列尾移出
	n1 := queue.Back()
	v, ok := n1.Value.(int8)
	if ok {
		fmt.Printf("%T\n", v)
		fmt.Println(v)
	}
	queue.Remove(n1)
    
    //在栈首移出
	n2 := stack.Front()
	fmt.Printf("%v", n2.Value)
	stack.Remove(n2)
}
~~~

## 3.串

- 朴素模式匹配

~~~go
// s 主串 t子串 O(n+m)
func match(s string, t string, pos int) int {

	i := 0
	j := 0
	if pos != 0 {
		i = pos
	}
	for (i < len(s) && j < len(t)) {
		if s[i] == t[j]{
			i++
			j++
		}else{
			i = i - j + 2
			j = 1
		}
	}
	if j == len(t) {
		return i - len(t)
	}else {
		return  0
	}
}
~~~

- KMP算法

~~~go
func getNext(t string)[]int{

	next := make([]int, len(t), len(t))
	next[0] = -1
	next[1] = 0
	i := 0
	j := 1
	for j < len(t) - 1{
		if i == -1 || t[i] == t[j]{
			i++
			j++

				next[j] = i
			}else {
				i = next[i]
			}
		}
	return next
}

func kmpSearch(s, t string) int{
	i, j := 0, 0
	next := getNext(t)
	for i < len(s) && j < len(t){
		if j == -1 || s[i] == t[j] {
			i++
			j++

		}else {
			j = next[j]
		}
	}
	if j == len(t){
		return i - j
	}else {
		return  -1
	}
}
~~~



## 4.树



## 5.图



## 6.查找



## 7.排序

### 1.冒泡



### 2.简单选择



### 3.直接插入



### 4.希尔



### 5.堆



### 6.归并



### 7.快速



### 8.排序总结

1. 排序分为内排序和外排序 外排序需要和内外存之间多次交换数据才能进行 
2. 内排序分为：插入排序、交换排序、选择排序、归并排序四类
   1. 插入排序：直接插入排序、希尔排序
   2. 选择排序：简单选择排序、堆排序
   3. 交换排序： 冒泡排序、快速排序
   4. 归并排序： 归并排序

3. 稳定排序： 冒泡、简单选择、直接插入、归并
4. 不稳定排序：希尔、堆、快速
5. 时间复杂度
   1. O(n<sup>2</sup>) ： 冒泡、简单选择、直接插入
   2. O(nlogn) ~ O(n<sup>2</sup>) ： 希尔排序
   3. O(nlogn)：堆、归并、快速

6. 算法的简单性
   1. 简单算法：冒泡、简单选择、直接插入
   2. 改进算法：希尔、堆、归并、快速