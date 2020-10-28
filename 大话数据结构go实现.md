

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

####朴素模式匹配

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

####KMP算法

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

#### 1.概念

- 结点拥有的子树数称为结点的度（degree）
- 树表示法：双亲表示法 孩子表示法 孩子兄弟表示法

#### 2.二叉树

- 满二叉树：所有分支结点都存在左子树和右子树 并且所有叶子都在同一层上
- 完全二叉树：对一棵有n个结点的二叉树按层序编号，如果编号i的结点与同样深度的满二叉树中编号i的结点在二叉树中位置完全相同，则这颗二叉树称为完全二叉树
  - 叶子节点只能出现在最下两层
  - 最下层的叶子一定集中在左部连续位置
  - 倒数二层，若有叶子结点，一定在右部连续位置
  - 如果节点度为1 则该节点只有左孩子
  - 同样结点数的二叉树， 完全二叉树的深度最小
- 二叉树性质1：在二叉树的第i层上至多有2<sup>i-1</sup>个结点
- 二叉树性质2：深度为k的二叉树至多有2<sup>k</sup>-1个结点
- 二叉树性质3：对于任意一棵二叉树T，如果其终端结点数为n<sub>0</sub>，度为2的结点数为n<sub>2</sub>，则 n<sub>0</sub> = n<sub>2</sub> + 1
- 二叉树性质4：具有n个结点的完全二叉树的深度为不大于log<sub>2</sub>n的最大整数+1

~~~go
package main

import "fmt"

type Node struct {
	V int
	Left *Node
	Right *Node
}

func NewNode(v int) *Node {
	n :=  &Node{V:v}
	return n
}

func (n *Node)Print(){
	fmt.Println("v : ",n.V)
}
//前序遍历
func (n *Node) PreList(){
	if n == nil {
		return
	}
	n.Print()
	n.Left.PreList()
	n.Right.PreList()
}
//中序遍历
func (n *Node)MidList(){
	if n == nil {
		return
	}
	n.Left.MidList()
	n.Print()
	n.Right.MidList()
}
//后序遍历
func (n *Node)PostList(){
	if n == nil {
		return
	}
	n.Left.PostList()
	n.Right.PostList()
	n.Print()
}
//层次遍历 广度优先
func (n *Node) BFS(){
	if n == nil {
		return
	}
	res := []int{}
	nodes := []*Node{n}
	for len(nodes) > 0 {
		curNode := nodes[0]
		nodes = nodes[1:]
		res = append(res, curNode.V)
		if curNode.Left != nil {
			nodes = append(nodes, curNode.Left)
		}
		if curNode.Right  != nil {
			nodes = append(nodes, curNode.Right)
		}
	}
	for _, v := range res{
		fmt.Println("v : ", v)
	}
}
//层数
func (n *Node) LayerByQueue() int{
	if n == nil {
		 return  0
	}
	layers := 0
	nodes := []*Node{n}
	for len(nodes) > 0 {
		layers++
		size := len(nodes)
		fmt.Println("nodes : ", nodes)
		fmt.Println("size :", size)
		count := 0
		for count < size {
			count++
			curNode := nodes[0]
			nodes = nodes[1:]
			if curNode.Left != nil {
				nodes = append(nodes, curNode.Left)
			}
			if curNode.Right != nil {
				nodes = append(nodes, curNode.Right)
			}
		}
	}
	return layers
}
func main(){
	n := NewNode(3)
	n2 := NewNode(4)
	n3 := NewNode(5)
	n4 := NewNode(6)
	n5 := NewNode(7)
	n6 := NewNode(8)
	n7 := NewNode(9)

	n.Left = n2
	n.Right = n3
	n.Left.Left = n4
	n.Left.Right = n5
	n.Right.Left = n6
	n.Right.Right = n7

	res := n.LayerByQueue()
	fmt.Println(res)
}
~~~

- **树转换为二叉树**
  - **加线** 在所有兄弟结点之间加一条连线
  - **去线** 对树中每个结点，只保留它与第一个孩子结点的连线 删除它与其他孩子结点之间的连线
  - **层次调整** 以树的根为轴心 将整棵树顺时针旋转一定的角度 注意第一个孩子是二叉树结点的左孩子 兄弟转换过来的孩子是结点的右孩子

![](image\alg\treeReverseBtree.png)

#### 3.哈夫曼树

 [https://pyihe.github.io/2020/08/29/Golang%E8%B5%AB%E5%A4%AB%E6%9B%BC%E6%A0%91%E5%8F%8A%E5%85%B6%E7%BC%96%E7%A0%81.html](https://pyihe.github.io/2020/08/29/Golang赫夫曼树及其编码.html) 

~~~go
package main

import (
	"fmt"
	"sort"
)

type HafumanList []*HafumanNode

func (h HafumanList) Len() int {
	return len(h)
}

func (h HafumanList) Swap(i, j int) {
	h[i], h[j] = h[j], h[i]
}

func (h HafumanList) Less(i, j int) bool {
	return h[i].Weight < h[j].Weight
}

type HafumanNode struct {
	Weight int         //权值
	Data interface{}   //数值
	Parent *HafumanNode  //父节点
	Left *HafumanNode    //左孩子
	Right *HafumanNode   //右孩子
}

type HafumanTree struct {
	root *HafumanNode       //根节点
	leaf HafumanList		//所有叶子节点
	src map[interface{}]int     //叶子节点的源数据 key为数据 value 为权重
	codeSet map[interface{}]string  //叶子节点数据的编码集 key为数据 value为通过构造哈夫曼树得到的数据的编码
}

func NewHafumanTree(src map[interface{}]int) *HafumanTree{
	var tree = &HafumanTree{
		root:    nil,
		leaf:    nil,
		src:     src,
		codeSet: nil,
	}
	tree.init()
	tree.build()
	tree.parse()
	return tree
}

//根据数据进行哈夫曼编码
func (h *HafumanTree) Coding(target interface{})(res string){
	if target == nil {
		return
	}

	var s string
	switch t := target.(type) {
	case string:
		s = t
	case []byte:
		s = string(t)
	default:
		return
	}

	for _, t := range s {
		v := string(t)
		if c, ok := h.codeSet[v]; !ok {
			panic("invalid code: " + v)
		}else {
			res += c
		}
	}
	return res
}

// 根据哈夫曼编码获取数据
func (h *HafumanTree) UnCoding(target  string)(res string){
	node := h.root
	for i := 0; i < len(target); i++{
		switch target[i]{
		case '0':
			node = node.Left
		case '1':
			node= node.Right
		}
		if node.Left == nil && node.Right == nil {
			res = res + node.Data.(string)
			node = h.root
		}
	}
	return res
}

// 初始化所有叶子节点
func (h *HafumanTree) init(){
	if len(h.src) <= 1 {
		panic("invalid len src")
	}

	h.codeSet = make(map[interface{}]string)
	h.leaf = make(HafumanList, len(h.src))
	var i int
	for data, weight := range h.src {
		var node = &HafumanNode{
			Weight: weight,
			Data:   data,
			Parent: nil,
			Left:   nil,
			Right:  nil,
		}
		h.leaf[i] = node
		i++
	}
	sort.Sort(h.leaf)
}

//构造哈夫曼树
//src： key: data  value ： weight
func (h *HafumanTree) build() {
	nodeList := h.leaf
	//根据哈夫曼树的规则构造哈夫曼树
	for nodeList.Len() > 1 {
		var temp = &HafumanNode{
			Weight: nodeList[0].Weight + nodeList[1].Weight,
			Data:   nil,
			Parent: nil,
			Left:   nodeList[0],
			Right:  nodeList[1],
		}
		nodeList[0].Parent = temp
		nodeList[1].Parent = temp
		//将生成的新节点插入节点序列中
		nodeList = regroup(nodeList[2:], temp)
	}
	h.root = nodeList[0]
}
//获取每个byte的编码 目的是为了下次需要编码的时候不用再次遍历树以获取每个byte的编码了
//在哈夫曼树中的所有节点要么没有孩子结点 要么有两个孩子结点
//此处的编码由底至顶获取 也可以由顶至底获取
func (h *HafumanTree)parse(){
	if h.root == nil {
		return
	}
	var temp *HafumanNode
	var code string
	for _, n := range h.leaf {
		temp = n
		for temp.Parent != nil {
		if temp == temp.Parent.Left {
			code = "0" +code
		}else {
			code = "1" +code
		}
		temp = temp.Parent
		}
		h.codeSet[n.Data] = code
		code = ""
	}

}

//重组 将生成的结点放入既有的list 排序后返回 权值最小的始终在最前面
func regroup(src HafumanList, temp *HafumanNode) HafumanList {
	//将temp添加进src 然后取出weight最小的一个
	length := len(src)
	res := make(HafumanList, len(src)+1)
	if length == 0 {
		res[0] = temp
		return res
	}

	if src[length-1].Weight <= temp.Weight {
		copy(res, src)
		res[length] = temp
		return res
	}
	for i := range src {
		 if src[i].Weight <= temp.Weight{
		 	res[i] = src[i]
		 }else {
		 	res[i] = temp
		 	copy(res[i+1:], src[i:])
		 	break
		 }
	}
	return res
}
func main() {
	 m := make(map[interface{}]int)
	 m["A"] = 10
	 m["B"] = 20
	 m["C"] = 50
	 m["D"] = 5
	 m["F"] = 40
	 m["E"] = 20
	 m["G"] = 15
	 h := NewHafumanTree(m)
	 fmt.Println(h.UnCoding("0011010000010"))
	 fmt.Println(h.codeSet)
}
~~~

#### 4.二叉查找树

 https://flaviocopes.com/golang-data-structure-binary-search-tree/ 

binTree.go

~~~go
package bintree

import (
	"fmt"
	"sync"
)

/*
Insert(v)			// 向二叉搜索树的合适位置插入节点
Search(k)			// 检查序号为 k 的元素在树中是否存在
Remove(v)			// 移除树中所有值为 v 的节点
Min()				// 获取二叉搜索树中最小的值
Max()				// 获取二叉搜索树中最大的值
InOrderTraverse()	// 中序遍历树
PreOrderTraverse()	// 先序遍历树
PostOrderTraverse()	// 后续遍历树
String()			// 在命令行格式化打印出二叉树
 */

type Node struct {
	value int      //中序遍历的节点序号
	left *Node
	right *Node
}

type binSearchTree struct {
	root *Node
	lock sync.RWMutex
}

func (b *binSearchTree) Insert(v int) {
	b.lock.Lock()
	defer b.lock.Unlock()
	n := &Node{
		value: v,
		left:  nil,
		right: nil,
	}

	if b.root == nil {
		b.root = n
	}else {
		insertNode(b.root, n)
	}
}

func insertNode(n, newN *Node){
	if newN.value < n.value {
		if n.left == nil {
			n.left = newN
		}else {
			insertNode(n.left, newN)
		}
	}else {
		if n.right == nil {
			n.right = newN
		}else {
			insertNode(n.right, newN)
		}
	}
}

func (b *binSearchTree) String() {
	b.lock.Lock()
	defer b.lock.Unlock()
	fmt.Println("------------------------------------------------")
	stringify(b.root, 0)
	fmt.Println("------------------------------------------------")
}

func stringify(n *Node, level int) {
	if n != nil {
		format := ""
		for i := 0; i < level; i++ {
			format += "       "
		}
		format += "---[ "
		level++
		stringify(n.left, level)
		fmt.Printf(format+"%d\n", n.value)
		stringify(n.right, level)
	}
}

//中序遍历
func (b *binSearchTree) InOrderTraverse(f func(int)){
	b.lock.Lock()
	defer b.lock.Unlock()
	inOrderTraverse(b.root, f)
}

func inOrderTraverse(n *Node, f func(int)){
	if n != nil {
		inOrderTraverse(n.left, f)
		f(n.value)
		inOrderTraverse(n.right, f)
	}
}

func (b binSearchTree) Min() *int {
	b.lock.RLock()
	defer b.lock.RUnlock()
	n := b.root
	if n == nil {
		return nil
	}
	for {
		if n.left == nil {
			return &n.value
		}
		n = n.left
	}
}

func (b *binSearchTree) Max() *int {
	b.lock.RLock()
	defer b.lock.RUnlock()
	n := b.root
	if n == nil {
		 return  nil
	}
	for {
		if n.right == nil {
			return &n.value
		}
		n = n.right
	}
}

func (b *binSearchTree) Search(v int) bool {
	b.lock.RLock()
	defer b.lock.RUnlock()
	return search(b.root, v)
}

func search(n *Node, v int) bool {
	if n == nil {
		return false
	}

	if v < n.value {
		return search(n.left, v)
	}
	if v > n.value{
		return search(n.right, v)
	}
	return true
}

func (b *binSearchTree) Remove(v int){
	b.lock.RLock()
	defer b.lock.RUnlock()
	remove(b.root, v)
}

func remove( n *Node, v int) *Node{
	if n == nil {
		return nil
	}
	if v < n.value {
		n.left = remove(n.left, v)
		return n
	}
	if v > n.value {
		n.right = remove(n.right, v)
	}
	if n.left == nil && n.right == nil {
		n = nil
		return nil
	}
	if n.left == nil {
		n = n.right
		return n
	}
	if n.right == nil {
		 n = n.left
		 return n
	}
	leftmostright := n.right
	for{
		if leftmostright != nil && leftmostright.left != nil {
			leftmostright = leftmostright.left
		}else {
			break
		}
	}
	n.value= leftmostright.value
	n.right = remove(n.right, n.value)
	return n
}
~~~

binTree_test.go

~~~go
package bintree

import (
	"fmt"
	"testing"
)

var b binSearchTree

func TillTree(b *binSearchTree) {
	b.Insert(8)
	b.Insert(4)
	b.Insert(2)
	b.Insert(1)
	b.Insert(3)
	b.Insert(6)
	b.Insert(5)
	b.Insert(7)
	b.Insert(10)
	b.Insert(9)
}

func TestInsert(t *testing.T){

	TillTree(&b)
	b.String()

	b.Insert(11)
	b.String()
}


func isSameSlice(a, b []int) bool {
	if a == nil && b == nil {
		return true
	}
	if a == nil || b == nil {
		return false
	}
	if len(a) != len(b) {
		return false
	}
	for i := range a {
		if a[i] != b[i] {
			return false
		}
	}
	return true
}

func TestInOrderTraverse(t *testing.T)  {
	var res []int
	b.InOrderTraverse(func(i int){
		res = append(res, i)
	})

	if !isSameSlice(res, []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11}) {
		fmt.Println([]string{"1", "2", "3", "4", "5", "6", "7", "8", "9", "10", "11"})
		t.Errorf("order incorrect got %v", res)
	}
}


func TestMin(t *testing.T){
	if *b.Min() != 1 {
		t.Errorf("min should be 1")
	}
}

func TestMax(t *testing.T){
	if *b.Max() != 11 {
		t.Errorf("max should be 11")
	}
}

func TestSearch(t *testing.T){
	if !b.Search(1) || !b.Search(6) || !b.Search(9) {
		t.Errorf("serach not working")
	}
}

func TestRemove(t *testing.T){
	b.Remove(1)
	if *b.Min() != 2 {
		t.Errorf("min should be 2")
	}
}
~~~

####5.AVL树（平衡二叉搜索树）

~~~go
package bintree

import "fmt"

type AVLTree struct {
	root *AVLNode
}

type AVLNode struct {
	key int
	value int
	h int
	left *AVLNode
	right *AVLNode
}

//add方法
func (t *AVLTree) Add(key int, value int){
	t.root = t.root.add(key, value)
}

func (n *AVLNode) add(key int, value int)*AVLNode {
	if n == nil {
		return &AVLNode{key, value, 1, nil, nil}
	}
	if key < n.key {
		n.left = n.left.add(key, value)
	}else if key > n.key {
		n.right = n.right.add(key, value)
	}else {
		n.value = value
	}
	return n.rebalanceTree()
}

//remove方法
func (t *AVLTree) Remove(key int) {
	t.root.remove(key)
}

func (n *AVLNode)remove(key int) *AVLNode {
	if n == nil {
		return nil
	}
	if key < n.key {
		n.left = n.left.remove(key)
	}else if key > n.key {
		n.right = n.right.remove(key)
	} else {
		if n.left != nil && n.right != nil {
			rightMinNode := n.right.Min()
			n.key = rightMinNode.key
			n.value = rightMinNode.value
			n.right = n.right.remove(rightMinNode.key)
		}else if n.left != nil {
			n = n.left
		}else  if n.right != nil {
			n = n.right
		}else {
			n = nil
			return n
		}
	}
	return n.rebalanceTree()
}

//update方法
func (t *AVLTree)Update(oldKey int, newKey int, newValue int){
	t.root.remove(oldKey)
	t.root.add(newKey, newValue)
}

//search方法
func (t *AVLTree)Search(key int) (node *AVLNode){
	return t.root.search(key)
}

func (n *AVLNode) search(key int) *AVLNode{
	if n == nil {
		return  nil
	}
	if key < n.key {
		return n.left.search(key)
	}else if key > n.key {
		return n.right.search(key)
	}else {
		return n
	}

}

//遍历
func (t *AVLTree) InOrderTraverse(){
	t.root.inorder()
}

func (n *AVLNode) inorder() {
	if n.left != nil {
		n.left.inorder()
	}
	fmt.Print(n.key, " ")
	if n.right != nil {
		n.right.inorder()
	}
}


func (n *AVLNode) Min() *AVLNode{
	if n.left != nil {
		return n.left.Min()
	}else {
		return n
	}
}

func (n *AVLNode) rebalanceTree() *AVLNode{
	if n == nil {
		return n
	}
	n.recalculateHeight()
	balanceFactor := n.left.getHeight() - n.right.getHeight()
	if balanceFactor == -2 {
		if n.right.left.getHeight() > n.right.right.getHeight(){
				n.right = n.right.rRotate()
		}
		return n.lRotate()
	}else if balanceFactor == 2 {
		if n.left.right.getHeight() > n.left.left.getHeight(){
			n.left = n.left.lRotate()
		}
		return n.rRotate()
	}
	return n
}



func (n *AVLNode) rRotate() *AVLNode{
	newRoot := n.left
	n.left = newRoot.right
	newRoot.right = n

	n.recalculateHeight()
	newRoot.recalculateHeight()
	return newRoot
}

func (n *AVLNode) lRotate() *AVLNode {
	newRoot := n.right
	n.right = newRoot.left
	newRoot.left = n
	n.recalculateHeight()
	newRoot.recalculateHeight()
	return newRoot
}

func (n *AVLNode) getHeight() int {
	if n == nil {
		return 0
	}
	return n.h
}

func (n *AVLNode) recalculateHeight(){
	n.h = 1 + max(n.left.getHeight(), n.right.getHeight())
}



func max(a int, b int) int{
	if a > b {
		return a
	}
	return b
}

func (b *AVLTree) String2() {

	fmt.Println("------------------------------------------------")
	stringify2(b.root, 0)

	fmt.Println("------------------------------------------------")
}

func stringify2(n *AVLNode, level int) {
	if n != nil {
		format := ""
		for i := 0; i < level; i++ {
			format += "       "
		}
		format += "---[ "
		level++
		stringify2(n.left, level)
		fmt.Printf(format+"%d\n", n.key)
		stringify2(n.right, level)
	}
}
~~~

~~~go
//测试
package bintree

import "testing"

var avl AVLTree

func fillAVLTree(b *AVLTree) {
	b.Add(8,8)
	b.Add(4, 4)
	b.Add(2, 2)
	b.Add(1, 1)
	b.Add(3, 3)
	b.Add(6, 6)
	b.Add(5, 5)
	b.Add(7, 7)
	b.Add(10, 10)
	b.Add(9, 9)
}


//func TestAdd(t *testing.T){
//	fillAVLTree(&avl)
//	avl.String2()
//
//	avl.Add(11, 11)
//	avl.Add(12, 12)
//
//	avl.String2()
//}

func TestRemove2(t *testing.T){
	fillAVLTree(&avl)
	avl.String2()

	avl.Remove(4)
	avl.String2()

}
~~~



- 第二种方法（没看呢）

~~~go
// AVL Tree in Golang
package main
  
import (
    "encoding/json"
    "fmt"
)
 
type Key interface {
    Less(Key) bool
    Eq(Key) bool
}
  
 
type Node struct {
    Data    Key
    Balance int
    Link    [2]*Node
}
  
 
func opp(dir int) int {
    return 1 - dir
}
  
// single rotation
func single(root *Node, dir int) *Node {
    save := root.Link[opp(dir)]
    root.Link[opp(dir)] = save.Link[dir]
    save.Link[dir] = root
    return save
}
  
// double rotation
func double(root *Node, dir int) *Node {
    save := root.Link[opp(dir)].Link[dir]
  
    root.Link[opp(dir)].Link[dir] = save.Link[opp(dir)]
    save.Link[opp(dir)] = root.Link[opp(dir)]
    root.Link[opp(dir)] = save
  
    save = root.Link[opp(dir)]
    root.Link[opp(dir)] = save.Link[dir]
    save.Link[dir] = root
    return save
}
  
// adjust valance factors after double rotation
func adjustBalance(root *Node, dir, bal int) {
    n := root.Link[dir]
    nn := n.Link[opp(dir)]
    switch nn.Balance {
    case 0:
        root.Balance = 0
        n.Balance = 0
    case bal:
        root.Balance = -bal
        n.Balance = 0
    default:
        root.Balance = 0
        n.Balance = bal
    }
    nn.Balance = 0
}
  
func insertBalance(root *Node, dir int) *Node {
    n := root.Link[dir]
    bal := 2*dir - 1
    if n.Balance == bal {
        root.Balance = 0
        n.Balance = 0
        return single(root, opp(dir))
    }
    adjustBalance(root, dir, bal)
    return double(root, opp(dir))
}
  
func insertR(root *Node, data Key) (*Node, bool) {
    if root == nil {
        return &Node{Data: data}, false
    }
    dir := 0
    if root.Data.Less(data) {
        dir = 1
    }
    var done bool
    root.Link[dir], done = insertR(root.Link[dir], data)
    if done {
        return root, true
    }
    root.Balance += 2*dir - 1
    switch root.Balance {
    case 0:
        return root, true
    case 1, -1:
        return root, false
    }
    return insertBalance(root, dir), true
}
  
// Insert a node into the AVL tree.
func Insert(tree **Node, data Key) {
    *tree, _ = insertR(*tree, data)
}
 
// Remove a single item from an AVL tree.
func Remove(tree **Node, data Key) {
    *tree, _ = removeR(*tree, data)
}
 
func removeBalance(root *Node, dir int) (*Node, bool) {
    n := root.Link[opp(dir)]
    bal := 2*dir - 1
    switch n.Balance {
    case -bal:
        root.Balance = 0
        n.Balance = 0
        return single(root, dir), false
    case bal:
        adjustBalance(root, opp(dir), -bal)
        return double(root, dir), false
    }
    root.Balance = -bal
    n.Balance = bal
    return single(root, dir), true
}
  
func removeR(root *Node, data Key) (*Node, bool) {
    if root == nil {
        return nil, false
    }
    if root.Data.Eq(data) {
        switch {
        case root.Link[0] == nil:
            return root.Link[1], false
        case root.Link[1] == nil:
            return root.Link[0], false
        }
        heir := root.Link[0]
        for heir.Link[1] != nil {
            heir = heir.Link[1]
        }
        root.Data = heir.Data
        data = heir.Data
    }
    dir := 0
    if root.Data.Less(data) {
        dir = 1
    }
    var done bool
    root.Link[dir], done = removeR(root.Link[dir], data)
    if done {
        return root, true
    }
    root.Balance += 1 - 2*dir
    switch root.Balance {
    case 1, -1:
        return root, true
    case 0:
        return root, false
    }
    return removeBalance(root, dir)
}
  
 
type intKey int 
func (k intKey) Less(k2 Key) bool { return k < k2.(intKey) }
func (k intKey) Eq(k2 Key) bool   { return k == k2.(intKey) }
  
func main() {
    var tree *Node
    fmt.Println("Empty Tree:")    
    avl,_ := json.MarshalIndent(tree, "", "   ")
    fmt.Println(string(avl))
 
    fmt.Println("\nInsert Tree:")
    Insert(&tree, intKey(4))
    Insert(&tree, intKey(2))
    Insert(&tree, intKey(7))
    Insert(&tree, intKey(6))
    Insert(&tree, intKey(6))
    Insert(&tree, intKey(9))
    avl,_ = json.MarshalIndent(tree, "", "   ")
    fmt.Println(string(avl))
  
    fmt.Println("\nRemove Tree:")
    Remove(&tree, intKey(4))
    Remove(&tree, intKey(6))
    avl,_ = json.MarshalIndent(tree, "", "   ")
    fmt.Println(string(avl))
}
~~~

####6.红黑树

**红黑树性质**

- 节点是红色或者黑色
- 根节点是黑色
- 每个叶子节点都是黑色的空节点（nil节点）
- 每个红色节点的两个子节点都是黑色。（从每个叶子到根的所有路径上不能有两个连续的红色节点）
- 从任一节点到其每个叶子的所有路径都包括相同数目的黑色节点
- 红黑树从根到叶子的最长路径不会超过最短路径的2倍

#### 7.B树

#### 8.B+树



#### 



## 5.图

#### 定义

- **图**是由定点的无穷非空集合和顶点之间边的集合组成G(V, E)其中 G表示一个图 V是图G顶点的集合 E是图G中边的集合
- 无向图&有向图
- 有向边（弧）连接顶点A到D的有向边就是弧 A是弧尾 D是弧头 <A, D>表示弧 注意不能写成<D, A>
- 无向边用小括号“()”表示，而有向边则是用尖括号“<>”表示
- **对于具有n个顶点和e条边数的图，无向图0≤e≤n(n-1)/2，有向图0≤e≤n(n-1)**
- 有些图的边或弧具有与它相关的数字，这种与图的边或弧相关的数叫做权（Weight）。这些权可以表示从一个顶点到另一个顶点的距离或耗费。这种带权的图通常称为网（Network）
- 无向图中，边数是各顶点度数和的一半
- 有向图 出度 入度
- 第一个顶点和最后一个顶点相同的路径称为回路或环（Cycle）
- 序列中顶点不重复出现的路径称为简单路径
- 除了第一个顶点和最后一个顶点之外，其余顶点不重复出现的回路，称为简单回路或简单环
- 无向图中的极大连通子图称为连通分量
- 向图中的极大强连通子图称做有向图的强连通分量
- 在有向图G中，如果对于每一对vi、vj∈V、vi≠vj，从vi到vj和从vj到vi都存在路径，则称G是强连通图
- 有向图中的极大强连通子图称做有向图的强连通分量

![](image\alg\Image 28.png)

#### 图的存储结构

- **邻接矩阵** ：图的邻接矩阵（Adjacency Matrix）存储方式是用两个数组来表示图。一个一维数组存储图中顶点信息，一个二维数组（称为邻接矩阵）存储图中的边或弧的信息。

![](image\alg\linjiejuzhen.png)

- **邻接表**
  - 1.图中顶点用一个一维数组存储。当然，顶点也可以用单链表来存储，不过数组可以较容易地读取顶点信息，更加方便。
  - 2.图中每个顶点vi的所有邻接点构成一个线性表，由于邻接点的个数不定，所以用单链表存储，无向图称为顶点vi的边表，有向图则称为顶点vi作为弧尾的出边表。

![](image\alg\linjiebiao.png)



- 带权邻接表

![](image\alg\daiquanlinjiebiao.png)

- 十字链表

![](image\alg\shizilianbiao.png)

- 邻接多重表

![](image\alg\linjieduochongbiao.png)

#### 广度优先搜索（链表）

~~~go
package graph

import "fmt"

/*
朋友圈通过字典实现，以“你”（you）作为根节点，字典的键是朋友圈属主，值是朋友圈所有朋友名字，
通过一个列表方式实现，名字按字母顺序排序。
其次，传递创建的朋友圈给breadthFirstSearch函数，该函数是广度优先搜索算法的具体实现，
在函数内部，首先取出you的所有朋友，如果朋友数为0，查找失败，返回false。如果朋友数不为0，
则从you的所有朋友中取出一个朋友，并将朋友从待查找的朋友列表中删除，然后创建一个字典记录被查找过的朋友，
避免再次查找。如果该朋友没有被检查过，则检查该朋友是否是售货员（名字以字母y结尾）。
如果是售货员，查找成功，返回true。如果该朋友不是售货员，
将该朋友的所有朋友又添加到待查找朋友列表中，继续查找，直到结束，实现一种类似Z字形的搜索路径。
由示例中可以看到，查找到的售货员是peggy，而不是jonny。
因为这里的朋友名字是按字母顺序排序，所以优先查找了bob的朋友，
而不是claire的朋友，即peggy是朋友圈中距离you最近的售货员朋友。
*/

func createFriendCircle() map[string][]string {
	fc := make(map[string][]string)
	fc["you"] = []string{"alice", "bob", "claire"}
	fc["bob"] = []string{"anuj", "peggy"}
	fc["alice"] = []string{"peggy"}
	fc["claire"] = []string{"thom", "jonny"}
	fc["anuj"] = []string{}
	fc["peggy"] = []string{}
	fc["thom"] = []string{}
	fc["jonny"] = []string{}
	return fc
}

func personIsSeller(name string) bool {
	return name[len(name)-1] == 'y'
}


func BFS(fc map[string][]string) bool {
	searchList := fc["you"]
	if len(searchList) == 0 {
		return false
	}

	serachd := make(map[string]bool)

	for {
		person := searchList[0]
		searchList = searchList[1:]
		_, found := serachd[person]
		if !found {
			fmt.Println("bianli :    ", person)
			if personIsSeller(person) {
				fmt.Println(person + " is seller")
				return true
			}else {
				searchList = append(searchList, fc[person]...)
				fmt.Println("searchList:                 ", searchList)
				serachd[person] = true
			}
		}
		if len(searchList) == 0 {
			return false
		}

	}
	return false
}
~~~

#### 深度优先搜索（链表）

~~~go

~~~

#### 深度&广度（邻接矩阵）

~~~go
package graph

import (
	"container/list"
	"fmt"
)

const  inf = 999

type Graph struct {
	vexs []string
	vexnum int
	edgnum int
	matrix [6][6]int
}

func InitG(g *Graph) {
	g.matrix[0][1] = 6
	g.matrix[0][2] = 1
	g.matrix[0][3] = 5

	g.matrix[1][0] = 6
	g.matrix[1][2] = 5
	g.matrix[1][4] = 3

	g.matrix[2][0] = 1
	g.matrix[2][1] = 5
	g.matrix[2][3] = 5
	g.matrix[2][4] = 6
	g.matrix[2][5] = 4

	g.matrix[3][0] = 5
	g.matrix[3][2] = 5
	g.matrix[3][5] = 2

	g.matrix[4][1] = 5
	g.matrix[4][2] = 6
	g.matrix[4][5] = 6

	g.matrix[5][2] = 4
	g.matrix[5][3] = 2
	g.matrix[5][4] = 6

	g.vexnum = 6
	g.edgnum = 10

	g.vexs = []string{"v0", "v1", "v2", "v3", "v4", "v5"}
}

//深度优先遍历
func DFS(g *Graph, visit *[]bool, i int){
	fmt.Println(g.vexs[i])
	for j := 0; j < g.vexnum; j++{
		if g.matrix[i][j] != inf && !(*visit)[j]{
			(*visit)[j] = true
			DFS(g, visit, j)
		}
	}
}

func DFSF(g *Graph) {
	visit := make([]bool, 10, 10)
	fmt.Println(visit)
	visit[0] = true
	DFS(g, &visit, 0)
}

//广度优先
func BFS(g *Graph) {
	listq := list.New()

	visit := make([]bool, 10, 10)
	visit[0] = true
	listq.PushBack(0)
	fmt.Println("listq", listq)
	for listq.Len() > 0 {
		index := listq.Front()
		fmt.Println(g.vexs[index.Value.(int)])
		for i:= 0; i < g.vexnum; i++{
			if !visit[i] && g.matrix[index.Value.(int)][i] != inf {
				visit[i] = true
				listq.PushBack(i)
			}
		}
		listq.Remove(index)
	}

}
~~~

- test

~~~go
package graph

import (
	"fmt"
	"testing"
)

func TestInitG(t *testing.T){
	var g Graph
	InitG(&g)

	for i := 0; i < 6; i++{
		for j := 0; j < 6; j++{
			fmt.Print(g.matrix[i][j], " ")
		}
		fmt.Println()
	}
}

func TestDFSF(t *testing.T) {
	var g Graph
	for i := 0; i < 6; i++{
		for j := 0; j < 6; j++{
			g.matrix[i][j] = inf
		}
	}
	InitG(&g)
	DFSF(&g)
}

func TestBFS(t *testing.T) {
	var g Graph
	for i := 0; i < 6; i++{
		for j := 0; j < 6; j++{
			g.matrix[i][j] = inf
		}
	}
	InitG(&g)
	BFS(&g)
}
~~~

## 6.查找



## 7.排序

### 相关概念

- **排序稳定性**
  - **稳定排序**：假设ki=kj(1≤i≤n,1≤j≤n,i≠j），且在排序前的序列中ri领先于rj（即i<j）。如果排序后ri仍领先于rj，则称所用的排序方法是稳定的
  - **不稳定排序**：反之，若可能使得排序后的序列中rj领先ri，则称所用的排序方法是不稳定的
- **内排序和外排序**：根据排序过程中待排序的记录是否全部被放置在内存中 分为内排序和外排序。本章主要讨论内排序
- 算法的复杂度
  - **内排序分为**：插入排序 交换排序 选择排序 归并排序
  - **简单算法**： 冒泡 简单选择 直接插入
  - **改进算法**： 希尔 堆 归并 快速



### 1.冒泡排序

~~~go
package mysort
//严格意义来说不算冒泡 更像是最简单的交换排序
func maopao1(a []int) [] int{

	for i := 0; i < len(a); i++{
		for j  := i + 1; j < len(a); j++{
			if a[j] < a[i] {
				a[j], a[i] = a[i], a[j]
			}
		}
	}
	return a
}
//冒泡排序
func BubbleSort(a []int) []int{
	for i := 1; i < len(a); i++{
		for j := len(a) - 1; j >= i; j--{
			if a[j] < a[j-1]{
				a[j], a[j-1] = a[j-1], a[j]
			}
		}
	}
	return a
}
//优化冒泡
func BubbleSort2(a []int) []int {
	flag := true
	for i := 0; i < len(a) && flag; i++{
		flag = false
		for j := len(a) - 1; j >= i; j-- {
			if a[j] < a[j-1]{
				a[j], a[j-1] = a[j-1], a[j]
				flag = true
			}
		}
	}
	return a
}
~~~

**测试**

~~~go
package mysort

import "testing"

func TestMaopao1(t *testing.T){
	b := []int{2, 3, 4, 5, 6, 7}
	a := []int{5, 7, 3, 6, 4, 2}
	if !isSameSlice(maopao1(a), b) {
		t.Errorf("fail")
	}
}

func TestBubbleSort2(t *testing.T){
	b := []int{2, 3, 4, 5, 6, 7}
	a := []int{5, 7, 3, 6, 4, 2}
	if !isSameSlice(maopao1(a), b) {
		t.Errorf("fail")
	}
}
func TestBubbleSort(t *testing.T) {
	b := []int{2, 3, 4, 5, 6, 7}
	a := []int{5, 7, 3, 6, 4, 2}
	if !isSameSlice(maopao1(a), b) {
		t.Errorf("fail")
	}
}

func isSameSlice(a, b []int) bool {
	if a == nil && b == nil {
		return true
	}else if a == nil || b == nil {
		return false
	}

	if len(a) != len(b) {
		return false
	}

	for i := 0; i < len(a); i++{
		if b[i] != a[i] {
			return false
		}
	}
		return true
}

~~~

**时间复杂度分析**：   最好情况（本来有序）O(n)     最坏情况（待排序为逆序）：O(n<sup>2</sup>)

### 2.简单选择排序

~~~go
package mysort

func SelectSort(a []int) []int {
	var min int
	for i := 0; i <= len(a) - 1; i++{
		min = i
		for j := i + 1; j <= len(a) -1; j++{
			if a[min] > a[j]{
				min = j
			}
		}
		if min != i {
			a[min], a[i] = a[i], a[min]
		}
	}
	return a
}
~~~

**测试**

~~~go
package mysort

import (
	"fmt"
	"testing"
)

func TestSelectSort(t *testing.T) {
	b := []int{2, 3, 4, 5, 6, 7}
	a := []int{5, 7, 3, 6, 4, 2}
	if !isSameSlice2(SelectSort(a), b) {
		fmt.Println(SelectSort(a))
		t.Errorf("fail")
	}
}

func isSameSlice2(a, b []int) bool {
	if a == nil && b == nil {
		return true
	}else if a == nil || b == nil {
		return false
	}

	if len(a) != len(b) {
		return false
	}

	for i := 0; i < len(a); i++{
		if b[i] != a[i] {
			return false
		}
	}
	return true
}
~~~

- 复杂度分析：O(n<sup>2</sup>) 性能略好于冒泡

### 3.直接插入排序

~~~GO
package mysort

func InsertSort(a []int) []int {
	var temp int
	var j int
	for i := 1; i < len(a); i++{
		temp = a[i]
		for j = i; j > 0 && temp < a[j-1]; j-- {
				a[j] = a[j-1]
		}
		a[j] = temp
	}
	return a
}
~~~

- 测试

~~~go
package mysort

import (
	"fmt"
	"testing"
)

func TestInsertSort(t *testing.T) {
	b := []int{2, 3, 4, 5, 6, 7, 25}
	a := []int{5, 7, 3, 6, 4, 25, 2}
	fmt.Println(InsertSort(a))
	if !isSameSlice3(InsertSort(a), b) {
		fmt.Println(a)
		t.Errorf("fail")
	}
}

func isSameSlice3(a, b []int) bool {
	if a == nil && b == nil {
		return true
	}else if a == nil || b == nil {
		return false
	}

	if len(a) != len(b) {
		return false
	}

	for i := 0; i < len(a); i++{
		if b[i] != a[i] {
			return false
		}
	}
	return true
}
~~~

- **复杂度分析**：最好情况：O(n)   最坏情况： O(n<sup>2</sup>)   直接插入排序法比冒泡和简单选择排序的性能要好一些

### 4.希尔排序

~~~go
package mysort

import "fmt"

func ShellSort(a []int) []int {
	l := len(a)
	h := l / 2
	var temp int
	var j int
	for h >= 1 {
		for i := h; i < l; i++{
			fmt.Println(a)
			temp = a[i]
			for j = i; j >= h && temp < a[j-h]; j = j - h {
				a[j] = a[j-h]
			}
			a[j] =temp
		}
		h = h / 2
	}
	return a
}
~~~

- 测试

~~~go
package mysort

import (
	"fmt"
	"testing"
)
func TestShellSort(t *testing.T) {
	a := []int{5, 7, 8, 4, 16, 2, 1, 10, 34, 6, 3, 13}
	fmt.Println(ShellSort(a))
}
~~~

- **复杂度分析**：O(n<sup>3/2</sup>)

### 5.堆排序

- 概念
  -  **最大堆：根结点的键值是所有堆结点键值中最大者的堆。** **（小根堆）**
  -  **最小堆：根结点的键值是所有堆结点键值中最小者的堆。** **（大根堆）**
  -   **每个结点的值都大于或等于其左右孩子结点的值，称为大顶堆** 
  -   **每个结点的值都小于或等于其左右孩子结点的值，称为小顶堆** 

~~~shell
#主要是理解思路，思路有了代码则是水到渠成。

#堆排序实际是数组排序，使用数组的下标构造成一个二叉树，想法很有意思。

#加入有数组a，那可以把a[0]视为根节点，它的子节点为a[2*0+1]，a[2*0+2]，即对于任何一个节点a[i]，则有子节点#a[2*i+1]和a[2*i+2]。

#1. 构建一个大顶堆，构建成功后a[0]便是最大的。

#2. 之后交换a[0]和a[len(a)-1]

#3. 然后在调整a[:len(a)-2]（把a[len(a)-1]排除在外）为大顶堆

#4. 重复上面的步骤

#5. 得到有序数组
~~~



~~~go
package sort

func adjustHeap(a []int, parent int, length int){
	for{
		left := parent*2 + 1
		right := parent*2 + 2
		idx := parent
		if left < length && a[left] > a[idx] {
			idx = left
		}

		if right < length && a[right] > a[idx] {
			idx = right
		}
		if idx == parent {
			 break
		}
		a[parent], a[idx] = a[idx], a[parent]
		parent = idx
	}
}



func buildHeap(a []int){
	for i := len(a)/2 - 1; i >= 0; i-- {
		adjustHeap(a, i, len(a))
	}
}

func HeapSort(a []int) []int {
	buildHeap(a)
	for i := len(a)-1; i >= 0; i-- {
		a[0], a[i] = a[i], a[0]
		adjustHeap(a, 0, i)
	}
	return a
}
~~~

- 复杂度分析：O(nlogn)

### 6.归并排序

~~~go
package mysort

func MergeSort(a []int) []int{
	if len(a) < 2 {
		return a
	}

	m := len(a) / 2
	l := MergeSort(a[:m])
	r := MergeSort(a[m:])
	return merge(l, r)
}

func merge(a, b []int) (c []int){
	i, j := 0, 0

	for i < len(a) && j < len(b) {
		if a[i] <= b[j] {
			c = append(c, a[i])
			i++
		}else {
			c = append(c, b[j])
			j++
		}
	}
	c = append(c, a[i:]...)
	c = append(c, b[j:]...)
	return
}
~~~

- **复杂度分析**： O(nlogn)

### 7.快速排序

~~~go
func QuickSort(a []int) []int {
	if len(a) < 2 {
		return a
	}else {
		pivot := a[0]
		var less []int
		var more []int
		for _, v := range a[1:]{
			if v <= pivot {
				less = append(less, v)
			}else {
				more = append(more, v)
			}
		}
		var  res  []int
		res = append(res, QuickSort(less)...)
		res = append(res, pivot)
		res = append(res, QuickSort(more)...)
		return res
	}
}

//快速排序重点在于写partition函数 填坑法和遍历法比较好理解
//遍历
func partition3(nums []int, start, end int) int{
	index := start
	pivot := nums[end]
	for i := start; i <= end; i++ {
		if nums[i] < pivot{
			nums[i], nums[index] = nums[index], nums[i]
			index++
		}
	}
	nums[index], nums[end] = nums[end], nums[index]
	fmt.Println(nums)
	return index
}

//填坑
func partition1(nums []int, l, r int) int {
	i := l
	j := r
	p := nums[l]
	for i < j {
		for nums[j] >= p && i < j  {
			j--
		}
		nums[i] = nums[j]
		for nums[i] < p && i < j {
			i++
		}
		nums[j] = nums[i]
	}
	nums[i] = p
	return i
}


func sort(nums []int, i, j int) {
	if i >= j {
		return
	}
		mid := partition3(nums, i, j)
		sort(nums, i, mid-1)
		sort(nums, mid+1, j)

}

func Qsort(nums []int){
	sort(nums, 0, len(nums)-1)
}

~~~

### 8.排序总结

1. 排序分为内排序和外排序 外排序需要和内外存之间多次交换数据才能进行 
2. 内排序分为：插入排序、交换排序、选择排序、归并排序四类
   1. 插入排序：直接插入排序、希尔排序
   2. 选择排序：简单选择排序、堆排序
   3. 交换排序： 冒泡排序、快速排序
   4. 归并排序： 归并排序

3. 稳定排序： 冒泡、简单选择、直接插入、归并
4.  时间复杂度
   1. O(n<sup>2</sup>) ： 冒泡、简单选择、直接插入
   2. O(nlogn) ~ O(n<sup>2</sup>) ： 希尔排序
   3. O(nlogn)：堆、归并、快速
6. 算法的简单性
   1. 简单算法：冒泡、简单选择、直接插入
   2. 改进算法：希尔、堆、归并、快速

# 9.基于go的heap包构造优先队列

~~~go
//heap
package heapAndPriorityQueue

type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] > h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[:n-1]
	return x
}
~~~

~~~go
//priotityqueue
package heapAndPriorityQueue

import "container/heap"

type Item struct {
	value    string
	priority int
	index    int
}

type PriorityQueue []*Item

func (pq PriorityQueue) Len() int {return  len(pq)}
func  (pq PriorityQueue) Less(i, j int) bool {return pq[i].priority > pq[j].priority}
func (pq PriorityQueue) Swap(i, j int) {
	pq[i], pq[j] = pq[j], pq[i]
	pq[i].index = i
	pq[j].index = j
}

func (pq *PriorityQueue) Push(x interface{}) {
	n := len(*pq)
	item := x.(*Item)
	item.index = n
	*pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() interface{} {
	old := *pq
	n := len(old)
	item := old[n-1]
	item.index = -1
	*pq = old[:n-1]
	return item
}

func (pq *PriorityQueue) update(item *Item, value string, priority int) {
	item.value = value
	item.priority = priority
	heap.Fix(pq, item.index)
}
~~~

# 10.常用的几种算法和解题思路

### 1.动态规划

- 两种解题思路：自顶向下（递归+记忆化） 自底向上

- leetcode70题 爬楼梯
- 91题解码方法
- 198 打家劫舍
- 剑指offer46 把数字翻译成字符串

- 剑指offer47礼物的最大价值