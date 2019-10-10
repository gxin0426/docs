## 1.go.Printf

`%v `默认方式打印变量的值

`%+d` 带符号的整型，`fmt.Printf("%+d", 255)`输出`+255` 

 `%q` 打印单引号

 `%o` 不带零的八进制

 `%#o` 带零的八进制

 `%x` 小写的十六进制

 `%X` 大写的十六进制

 `%#x` 带0x的十六进制

 `%U` 打印Unicode字符

 `%#U` 打印带字符的Unicode

 `%b` 打印整型的二进制

`%T` 打印变量的类型

### Float

-  `%f` (=`%.6f`) 6位小数点
-  `%e` (=`%.6e`) 6位小数点（科学计数法）
-  `%g` 用最少的数字来表示
-  `%.3g` 最多3位**数字**来表示
-  `%.3f` 最多3位**小数**来表示

### String

-  `%s` 正常输出字符串
-  `%q` 字符串带双引号，字符串中的引号带转义符
-  `%#q` 字符串带反引号，如果字符串内有反引号，就用双引号代替
-  `%x` 将字符串转换为小写的16进制格式
-  `%X` 将字符串转换为大写的16进制格式
-  `% x` 带空格的16进制格式

### String Width (以5做例子）

-  `%5s` 最小宽度为5
-  `%-5s` 最小宽度为5（左对齐）
-  `%.5s` 最大宽度为5
-  `%5.7s` 最小宽度为5，最大宽度为7
-  `%-5.7s` 最小宽度为5，最大宽度为7（左对齐）
-  `%5.3s` 如果宽度大于3，则截断
-  `%05s` 如果宽度小于5，就会在字符串前面补零

### Struct

-  `%v` 正常打印。比如：`{sam {12345 67890}}` 
-  `%+v` 带字段名称。比如：`{name:sam phone:{mobile:12345 office:67890}` 
-  `%#v` 用Go的语法打印。
   比如`main.People{name:”sam”, phone:main.Phone{mobile:”12345”, office:”67890”}}` 

### Boolean

-  `%t` 打印true或false

### Pointer

-  `%p` 带0x的指针
-  `%#p` 不带0x的指针
-  \t 空四个字符 相当于tab键



## 2.数组和切片

### 数组与切片声明格式

```go
//array
var identifier [len]type
//slice
v := make([]int, 10, 50)
```

### 函数中指针的使用

```go
package main
import "fmt"
func f(a [3]int) { fmt.Println(a) }
func fp(a *[3]int) { fmt.Println(a) }

func main() {
	var ar [3]int
	f(ar) 	// passes a copy of ar
	fp(&ar) // passes a pointer to ar
}
```

- 当参数是指针时 传参要记得传地址（&）

 ### 利用匿名函数调试

~~~go
where := func() {
	_, file, line, _ := runtime.Caller(1)
log.Printf("%s:%d", file, line) 
} 
where() 
//some code 
//方法2
log.SetFlags(log.Llongfile)
log.Print("")
fmt.Println("sskhahdfhksdfkjsd")
~~~

## 3.map

- 声明一个map 

~~~go
maplit := map[string]int{}  
maplitr := make(map[int]int)   
var maplit map[string]int   
//判断是否一个key存在
k,ispresent := map1[key1]   
//当 ispresent为true key1存在  
//删除 
delete(map1,key1)  
//for range   
for key,value := range map1{   
  
}

//map类型的slice
items := make([]map[int]int, 5)
	for i:= range items {
		items[i] = make(map[int]int, 1)
		items[i][1] = 2
	}
~~~

## 4.网络编程 tcp

~~~go
// tcp coding step
//1.监听端口 2.接受 3. 读取数据
1.listen, err := net.Listen("tcp", "0.0.0.0:8889") 
2.conn, err := listen.Accept() 
3._, err := conn.Read(buf[0:4]) 
~~~

## 5.package

- 正则表达式 regexp

```go
//define regexp regular
reg := "[0-9]+.[0-9]+"
re , err := regexp.Compile(reg)
str := re.ReplaceAllString(astring,"##.##") //astring is a string
f := func(s string) string{
v, _ := strconv.ParseFloat(s, 32)
return strconv.FormatFloat(v * 2, 'f', 2, 32)
}
str2 := re.ReplaceAllStringFunc(astring,f)
```

- sync 锁

~~~go
type Info struct {
	mu sync.Mutex
	// ... other fields, e.g.: Str string
}
func Update(info *Info) {
	info.mu.Lock()
    // critical section:
    info.Str = // new value
    // end critical section
    info.mu.Unlock()
}

~~~

## 6.struct and method

- 使用new函数给一个新的结构体变量分配内存，它返回指向已分配内存的指针 <u>var t *T = new(T)</u> 

- 选择器 selector

~~~go
add := &Address{
	doorId: 200,
	smtp:   900,
}
//add 的类型是*add
&Address 和 new(Address) 是等价的
~~~

![](image\go\newcshstruct.png)

 ![](image\go\structzmlcsh.png)

 

 ~~~go
//Go 语言中，结构体和它所包含的数据在内存中是以连续块的形式存在的，即使结构体中嵌套有其他的结构体，这在性能上带来了很大的优势。不像 Java 中的引用类型，一个对象和它里面包含的对象可能会在不同的内存空间中，这点和 Go 语言中的指针很像。下面的例子清晰地说明了这些情况：
type Rect1 struct {Min, Max Point }
type Rect2 struct {Min, Max *Point }
 ~~~

![](image\go\memorybuju.png)

- **递归结构体（没看懂）**



- 结构体工厂

~~~go
type File struct {
    fd      int     // 文件描述符
    name    string  // 文件名
}
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }

    return &File{fd, name}
}
//调用
f := NewFile(10, "./test.txt")
~~~

- 强制使用工厂方法

~~~go
package matrix
type matrix struct {
    ...
}
func NewMatrix(params) *matrix {
    m := new(matrix) // 初始化 m
    return m
}
//其他包调用
package main
import "matrix"
...
wrong := new(matrix.matrix)     // 编译失败（matrix 是私有的）
right := matrix.NewMatrix(...)  // 实例化 matrix 的唯一方式
~~~

- 匿名字段和嵌入结构体(与其他语言的继承对比)

~~~go
type innerS struct {
	in1 int
	in2 int
}

type outerS struct {
	b    int
	c    float32
	int  // anonymous field
	innerS //anonymous field
}
~~~

- method

~~~go
//方法和函数的区别
函数将变量作为参数：Function1(recv)
方法在变量上被调用：recv.Method1()
//接受者需要和方法在同一个包下 当不在同一个包时 可以使用下面的代码
package main
import (
	"fmt"
	"time"
)
type myTime struct {
	time.Time //anonymous field
}
func (t myTime) first3Chars() string {
	return t.Time.String()[0:3]
}
func main() {
	m := myTime{time.Now()}
	// 调用匿名Time上的String方法
	fmt.Println("Full time now:", m.String())
	// 调用myTime.first3Chars
	fmt.Println("First 3 chars:", m.first3Chars())
}
~~~

- 方法和未导出字段

~~~go
//使用get set方法
package person

type Person struct {
	firstName string
	lastName  string
}
func (p *Person) FirstName() string {
	return p.firstName
}
func (p *Person) SetFirstName(newName string) {
	p.firstName = newName
}
//另一个包
package main
import (
	"./person"
	"fmt"
)
func main() {
	p := new(person.Person)
	// p.firstName undefined
	// (cannot refer to unexported field or method firstName)
	// p.firstName = "Eric"
	p.SetFirstName("Eric")
	fmt.Println(p.FirstName()) // Output: Eric
}
~~~

```go
type Engine interface {
	Start()
	Stop()
}

type Car struct {
	Engine
}
```

- 类型中的string（）方法和格式化描述符

~~~go
type TwoInts struct {
	a int
	b int
}
func (tn *TwoInts) String() string {
	return "(" + strconv.Itoa(tn.a) + "/" + strconv.Itoa(tn.b) + ")"
}
~~~

- 查看当前的内存状态

~~~go
var m runtime.MemStats
runtime.ReadMemStats(&m)
fmt.Printf("%d Kb\n", m.Alloc/1024)
~~~

## 7.接口反射

- 定义格式

~~~go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
}
~~~

- 类型判断

~~~go
//在处理来自于外部的、类型未知的数据时，比如解析诸如 JSON 或 XML 编码的数据，类型测试和转换会非常有用
func classifier(items ...interface{}) {
	for i, x := range items {
		switch x.(type) {
		case bool:
			fmt.Printf("para %d is a bool\n", i)
		case float64:
			fmt.Printf("para %d is a float64\n", i)
		case int32:
			fmt.Printf("para %d is a int32\n", i)
		case string:
			fmt.Printf("para %d is a string\n", i)
		default:
			fmt.Println("unknow")
		}
	}
}
~~~

- print函数检验是否可以打印自身

~~~go
type Stringer intterface{
    String() string
}
if sv,ok := v.(Stringer); ok{
    fmt.Printf("v implemets String(): %s\n",sv.String())
}
~~~

- 使用方法集与接口

~~~go
package main
type List []int
func (l List) len() int {
	return len(l)
}
func (l *List) Append(val int) {
	l.Append(val)
}
type Appender interface {
	Append(int)
}
func CountInfo(a Appender, start, end int) {
	for i := start; i <= end; i++ {
		a.Append(i)
	}
}
type Lener interface {
	len() int
}
func LongEnough(l Lener) bool {
	return l.len()*10 > 42
}
func main() {
	var lst List
	plst := new(List)
    var ttt Appender
    //报错 cannot use lst (type List) as type Appender in assignment:
	//List does not implement Appender (Append method has pointer receiver)
    ttt = lst 
	var tt Lener
	tt = plst
    aa := tt.len()
	LongEnough(lst)
	//报错 值接受者不能调用指针接受者的方法 反则可以
    CountInfo(lst, 1, 10)
	CountInfo(plst, 1, 10)
	LongEnough(plst)
	lst.Append(20)
}
总结：类型*t 的可调用方法集包含接受者为*t和t的所有方法集
     类型t的可调用方法集包含接受者为t的所有方法
     类型t的可调用方法集不包含接受者*t的方法
~~~

- 空接口

~~~go
//空接口实现二叉树
package main
import "fmt"
type Node struct {
	le   *Node
	data interface{}
	ri   *Node
}
func NewNode(left, right *Node) *Node {
	return &Node{left, nil, right}
}
func (n *Node) SetData(data interface{}) {
	n.data = data
}
func main() {
	root := NewNode(nil, nil)
	root.SetData("root node")
	// make child (leaf) nodes:
	a := NewNode(nil, nil)
	a.SetData("left node")
	b := NewNode(nil, nil)
	b.SetData("right node")
	root.le = a
	root.ri = b
	fmt.Printf("%v\n", root) // Output: &{0x125275f0 root node 0x125275e0}
}
~~~







 

 