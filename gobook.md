## 0.变量的声明方式

- string

  ~~~go
  s := ""
  var s string
  var s = ""
  var s string = ""
  ~~~

  

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

####1.正则表达式 regexp

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

####2.sync 锁

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

#### 3.iota

#### 4.math包

~~~go
fmt.Println(math.Abs(float64(i))) //绝对值
fmt.Println(math.Ceil(5.0))       //向上取整
fmt.Println(math.Floor(5.8))      //向下取整
fmt.Println(math.Mod(11, 3))      //取余数，同11%3
fmt.Println(math.Modf(5.26))      //取整数，取小数
fmt.Println(math.Pow(3, 2))       //x的y次方
fmt.Println(math.Pow10(4))        // 10的n次方
fmt.Println(math.Sqrt(8))         //开平方
fmt.Println(math.Cbrt(8))         //开立方
fmt.Println(math.Pi)              //pai
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

func justifyType(x interface{}){
    switch v := x.(type){
        case string: 
        	fmt.Printf("x is a string value is %v\n",v)
        case int:
        	fmt.Printf("x is a int value is %v\n",v)
        case float32:
        	fmt.Printf("x is a float32 value is %v\n",v)
        default:
        	fmt.Println("unknow type！")
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

- 值接收者和指针接受者的区别

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

- 类型和接口的关系

1. 一个类型可以实现多个接口

~~~go
type Sayer interface{
    say()
}
type Mover interface{
    move()
}

type dog struct{
    name string
}
func (d *dog) move() {
    
}
func (d *dog) say() {
    
}
~~~

2. 多个类型实现一个接口

~~~go
type people struct{
    name string
}
func (p *people) move {
    
}
~~~

- 空接口和断言

~~~go
var x interface{}
x = "hello"
ret , ok := x.(string)
if !ok {
    fmt.Println("输出的不是string类型")
}else{
    fmt.Println(ret)
}
~~~

## 8.读取输入输出

- **<u>Scan Sscan Fscan</u>**   

~~~go
Scan：从标准输入os.Stdin中读取数据 包括 Scan() Scanf() Scanln()
Sscan: 从字符串读取数据 包括Sscan() Sscanf() Sscanln()
Fscan: 从io.Reader中读取数据 包括Fscan() Fscanf() Fscanln()
ln: 在遇到换行符的时候停止
Scan: 将换行符当成空格处理
f: 根据给定的format格式读取 就像Printf一样 遇到换行符终止

fmt.Scanf("%s : %d",&name,&age)
在输入时，必须按照以下格式进行输入：首先至少一个空格，然后一个冒号，再至少一个空格

#Sscan用法
var(
	name string
    age int
)
input := "gao 27"
fmt.Sscan(input,&name,&age)
fmt.Println(name,age)
#Sscanf
input := "gaoxin : 27"
fmt.Sscanf(input,"%s : % %d",&name,&age)
#bufio包的读取标准输入（主要作用操作缓冲io）
var inputReader *bufio.Reader
var input string
var err error

func main(){
    inputReader = bufio.NewReader(os.Stdin)
    fmt.Println("输入姓名：")
    input , err := inputReader.ReadString('\n')
    if err == nil{
        fmt.Printf("The input was %s\n",input)
    }
    fmt.Printf("your name is %s", input)
	switch input {
	case "phi\r\n":
		fallthrough
	case "ghi\r\n":
		fallthrough
	case "gg\r\n":
		fmt.Printf("wel %s", input)
	}
}
//其中NewReader()创建一个bufio.Reader实例，表示创建一个从给定文件中读取数据的读取器对象。然后调用读取器#对象(Reader实例)的ReadString()方法，这个方法以\n作为分隔符，它的分隔符必须只能是单字符，且必须使用单引号包围，因为它会作为byte读取。ReadString()读取来自os.Stdin的内容后将其保存到input变量中，同时返回是否出错的信息。ReadString()只有一种情况会返回err：没有遇到分隔符。
//ReadString会将读取的内容包括分隔符都一起放进缓冲中，如果读取文件时读到了结尾，则会将整个文件内容放进缓冲，并将文件终止标识符io.EOF放进设置为err。
~~~

- 读文件

~~~go
inputFile, err := os.Open("abc.txt")
	if err != nil {
		fmt.Println(err)
		return
	}
	defer inputFile.Close()
	inputReader := bufio.NewReader(inputFile)
	for {
        //buf := make([]byte,1024)
        //n,err := inputReader.Read(buf)
        //if (n==0) {break}
		inputString, err := inputReader.ReadString('\n')
		fmt.Println(inputString)
		if err == io.EOF {
			return
		}
	}
~~~

- 读写文件

~~~go
buf , err := ioutil.ReadFile("file")
err = ioutil.WriteFile("file_copy",buf,0644)
~~~

- 按列读取文件中的数据 使用Fscanln

~~~go
file, er := os.Open("abc.txt")
	if er != nil {
		panic(er)
	}
	defer file.Close()
	var col1, col2, col3 []string
	for {
		var v1, v2, v3 string
		_, er = fmt.Fscanln(file, &v1, &v2, &v3)
		if er != nil {
			fmt.Println(er)
			break
		}
		col1 = append(col1, v1)
		col2 = append(col2, v2)
		col3 = append(col3, v3)

	}

	fmt.Println(col1)
	fmt.Println(col2)
	fmt.Println(col3)
~~~

- 读压缩文件

~~~go
fName := "MyFile.gz"
	var r *bufio.Reader
	fi, err := os.Open(fName)
	if err != nil {
		fmt.Fprintf(os.Stderr, "%v, Can't open %s: error: %s\n", os.Args[0], fName,
			err)
		os.Exit(1)
	}
	fz, err := gzip.NewReader(fi)
	if err != nil {
		r = bufio.NewReader(fi)
	} else {
		r = bufio.NewReader(fz)
	}

	for {
		line, err := r.ReadString('\n')
		if err != nil {
			fmt.Println("Done reading file")
			os.Exit(0)
		}
		fmt.Println(line)
	}
~~~

- 写文件（三要素：文件句柄 bufio Writer）

~~~go
outputFile, err := os.OpenFile("abc.txt", os.O_WRONLY|os.O_CREATE, 0666)
	if err != nil {
		fmt.Println(err)
		return
	}

	defer outputFile.Close()

	outputWriter := bufio.NewWriter(outputFile)  
	opString := "hello world haha \n"  

	for i := 0; i < 10; i++ {  
		outputWriter.WriteString(opString)
	}
	outputWriter.Flush()
/*****************/
//也可以使用Fprintf写入文件
fmt.Fprintf(outputFile, "Some test data.\n")
/********************/
//另一种方式
f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0666)
	defer f.Close()
	f.WriteString("hello, world in a file\n")
~~~

- 拷贝文件

~~~go
func CopyFile(srcName, desName string) (numByte int64, err error) {
	src, err := os.Open(srcName)
	if err != nil {
		return
	}
	defer src.Close()
	dst, err := os.Create(desName)
	if err != nil {		return
	}
	defer dst.Close()
	return io.Copy(dst, src)
} 
~~~

- 从命令行读取参数

~~~go
var who string
	if len(os.Args) > 1 {
		who = strings.Join(os.Args[0:], " ")
	}
	fmt.Println("Good Morning", who)
~~~

- 用切片读写文件

~~~go
func cat(f *os.File) {
	const NBUF = 512
	var buf [NBUF]byte
	for {
		nr, _ := f.Read(buf[:])
		os.Stdout.Write(buf[0:nr])
	}
}
~~~

- Fprintf

~~~go
// unbuffered
fmt.Fprintf(os.Stdout, "%s\n", "hello world! - unbuffered")
// buffered: os.Stdout implements io.Writer
buf := bufio.NewWriter(os.Stdout)
// and now so does buf.
fmt.Fprintf(buf, "%s\n", "hello world! - buffered")
buf.Flush()
~~~

- json

~~~go
pa := &Address{"private", "shanghai"}
wa := &Address{"work", "Boom"}
vc := VCard{"gx", []*Address{pa, wa}} 
js, _ := json.Marshal(vc) 
fmt.Printf("json fmat: %s", js) 
file, _ := os.OpenFile("vcard.json", os.O_CREATE|os.O_WRONLY, 0666) 
defer file.Close() 
//file.WriteString(string(js)) 
enc := json.NewEncoder(file) 
err := enc.Encode(vc) 
if err != nil { 
	fmt.Println(err) 
} 
~~~

##9.错误处理

- 定义错误

~~~go
err := errors.New("错误")
~~~

- 栗子

~~~go
func Sqrt(f float64)(float64,error){
    if f < 0 {
        return 0, errors.New("a negative number")
        //return 0, fmt.Errorf("a negative number %g", f)
    }
    return f*f, nil
}
//如果有不同错误条件可能发生，那么对实际的错误使用类型断言或类型判断（type-switch）是很有用的，并且可以根据错误场景做一些补救和恢复操作
if e, ok := err.(*os.PathError); ok {
    //remedy sitution
}
//或者
switch err := err.(type){
    case ParseError:
    	PrintParseError(err)
	case PathError:
    	PrintPathError(err)
    default:
    	fmt.Printf("Not a special error : %s\n",err)
    
}
//第二个例子：从命令行读取输入时，如果加了 help 标志，我们可以用有用的信息产生一个错误(不会用)
if len(os.Args) > 1 && (os.Args[1] == "-h" || os.Args[1] == "--help") {
	err = fmt.Errorf("usage: %s infile.txt outfile.txt", filepath.Base(os.Args[0]))
	return
}
~~~

- 运行时异常和panic

~~~go
//recover内建函数被用于从panic或者错误场景中恢复；让程序从panicking重新获得控制权，停止终止过程进而恢复正常运行。
//recover 只能在defer修饰的函数中使用 用于取得panic调用中传递过来的错误值
//panic会导致栈被展开直到defer修饰的recover()被调用或者程序终止
func protect(g func()){
    defer func(){
        log.Println("done")
        if err := recover(); err != nil{
            log.Printf("%v",err)
        }
    }()
    log.Println("start")
    g() 
}
//完整栗子
func main() {
	fmt.Println("start")
	test()
	fmt.Println("end")
}
func badCall() {
	panic("failure\n")
}
func test() {
	defer func() {
		if e := recover(); e != nil {
			fmt.Print(e)
		}
	}()

	badCall()
	fmt.Println("complete\r\n")
}
~~~

## 10.协程和通道

- 创建通道

~~~go
ch1 := make(chan string)
~~~

- 通信操作符 <-

```go
//<- ch 可以单独调用获取通道的（下一个）值，当前值会被丢弃，但是可以用来验证，所以以下代码是合法的
if <- ch != 1000{...}
```

- 信号量模式

~~~go
func compute(ch chan int){
	ch <- someComputation() // when it completes, signal on the channel.
}
func main(){
	ch := make(chan int) 	// allocate a channel.
	go compute(ch)		// stat something in a goroutines
	doSomethingElseForAWhile()
	result := <- ch
}
//不返回结果
ch := make(chan int)
go func(){
	// doSomething
	ch <- 1 // Send a signal; value does not matter
}()
doSomethingElseForAWhile()
<- ch	// Wait for goroutine to finish; discard sent value.
~~~

~~~go
//下边的代码，用完整的信号量模式对长度为N的 float64 切片进行了 N 个doSomething() 计算并同时完成，通道 sem 分配了相同的长度（切包含空接口类型的元素），待所有的计算都完成后，发送信号（通过放入值）。在循环中从通道 sem 不停的接收数据来等待所有的协程完成。
	type Empty interface{}
	var empty Empty
	data := make([]float64, 10)
	res := make([]float64, 10)
	sem := make(chan Empty, 10)
	for i := 0; i < 10; i++ {
		data[i] = rand.Float64()
	}
    //并行的for循环
	for i, xi := range data {
		go func(i int, xi float64) {
			res[i] = doSomething(i, xi)
			sem <- empty
		}(i, xi)
	}
	// wait for goroutines to finish
	for i := 0; i < 10; i++ {
		<-sem
	}
//注意闭合：i、xi 都是作为参数传入闭合函数的，从外层循环中隐藏了变量 i 和 xi。让每个协程有一份 i 和 xi 的拷贝；另外，for 循环的下一次迭代会更新所有协程中 i 和 xi 的值。切片 res 没有传入闭合函数，因为协程不需要单独拷贝一份。切片 res 也在闭合函数中但并不是参数
~~~

~~~go
//打印素数
// Send the sequence 2, 3, 4, ... to channel 'ch'.
func generate(ch chan int) {
	for i := 2; ; i++ {
		ch <- i // Send 'i' to channel 'ch'.
	}
}

// Copy the values from channel 'in' to channel 'out',
// removing those divisible by 'prime'.
func filter(in, out chan int, prime int) {
	for {
		i := <-in // Receive value of new variable 'i' from 'in'.
		if i%prime != 0 {
			out <- i // Send 'i' to channel 'out'.
		}
	}
}

// The prime sieve: Daisy-chain filter processes together.
func main() {
	ch := make(chan int) // Create a new channel.
	go generate(ch)      // Start generate() as a goroutine.
	for {
		prime := <-ch
		fmt.Print(prime, " ")
		ch1 := make(chan int)
		go filter(ch, ch1, prime)
		ch = ch1
	}
}
//lizi2
package main

import (
	"fmt"
)

// Send the sequence 2, 3, 4, ... to returned channel
func generate() chan int {
	ch := make(chan int)
	go func() {
		for i := 2; ; i++ {
			ch <- i
		}
	}()
	return ch
}

// Filter out input values divisible by 'prime', send rest to returned channel
func filter(in chan int, prime int) chan int {
	out := make(chan int)
	go func() {
		for {
			if i := <-in; i%prime != 0 {
				out <- i
			}
		}
	}()
	return out
}

func sieve() chan int {
	out := make(chan int)
	go func() {
		ch := generate()
		for {
			prime := <-ch
			ch = filter(ch, prime)
			out <- prime
		}
	}()
	return out
}

func main() {
	primes := sieve()
	for {
		fmt.Println(<-primes)
	}
~~~

- 使用通道或者锁实现同步

~~~go
func Worker(in, out chan *Task) {
    for {
        t := <-in
        process(t)
        out <- t
    }
}
~~~

- 惰性求值

~~~go
func main() {
	resume = integers()
	fmt.Println(generateInteger())
	fmt.Println(generateInteger())
}
var resume chan int
func integers() chan int {
	yield := make(chan int)
	count := 0
	go func() {
		for {
			yield <- count
			count++
		}

	}()
	return yield
}
func generateInteger() int {
	return <-resume
}


type Any interface{}
type EvalFunc func(Any) (Any, Any)

func main() {
    evenFunc := func(state Any) (Any, Any) {
        os := state.(int)
        ns := os + 2
        return os, ns
    }
    
    even := BuildLazyIntEvaluator(evenFunc, 0)
    
    for i := 0; i < 10; i++ {
        fmt.Printf("%vth even: %v\n", i, even())
    }
}


//通过巧妙地使用空接口、闭包和高阶函数，我们能实现一个通用的惰性生产器的工厂函数BuildLazyEvaluator（这个应该放在一个工具包中实现）
func BuildLazyEvaluator(evalFunc EvalFunc, initState Any) func() Any {
    retValChan := make(chan Any)
    loopFunc := func() {
        var actState Any = initState
        var retVal Any
        for {
            retVal, actState = evalFunc(actState)
            retValChan <- retVal
        }
    }
    retFunc := func() Any {
        return <- retValChan
    }
    go loopFunc()
    return retFunc
}

func BuildLazyIntEvaluator(evalFunc EvalFunc, initState Any) func() int {
    ef := BuildLazyEvaluator(evalFunc, initState)
    return func() int {
        return ef().(int)
    }
}
~~~

- future 模式

~~~go
//old
func InverseProduct(a Matrix, b Matrix) {
    a_inv := Inverse(a)
    b_inv := Inverse(b)
    return Product(a_inv, b_inv)
}
//new
func InverseProduct(a Matrix, b Matrix) {
    a_inv_future := InverseFuture(a)   // start as a goroutine
    b_inv_future := InverseFuture(b)   // start as a goroutine
    a_inv := <-a_inv_future
    b_inv := <-b_inv_future
    return Product(a_inv, b_inv)
}
func InverseFuture(a Matrix) {
    future := make(chan Matrix)
    go func() {
        future <- Inverse(a)
    }()
    return future
}
~~~

## 11.常见错误

####误用短声明变量导致变量覆盖

~~~go
var remeber bool = false
if something{
    remember := true //错误
}
//remember在if内部为新声明变量 if 外部的remember不会变为true
//观察下面的代码
func shadow() (err error) {
    x, err := check1() // x是新创建变量，err是被赋值
if err != nil {
    return // 正确返回err
}
if y, err := check2(x); err != nil { // y和if语句中err被创建
    return // if语句中的err覆盖外面的err，所以错误的返回nil！
} else {
    fmt.Println(y)
}
    return
}
~~~

####字符串的使用

~~~go
//对字符串频繁操作时 go中的字符串是不可变的（类似于java）使用 a += b形式连接字符串效率低 应该使用一个字符数组 将字符串内容写入一个内存中
var b bytes.Buffer
for condition{
    b.WriteString(str) //将字符串str写入缓存buffer
}
return b.String()
~~~

####defer正确用法

~~~go
//错误方式
for _,file := range files{
    if f, err = os.Open(file); err != nil{
        return
    }
    //当循环结束时文件没有关闭
    defer f.Close()
    //对文件进行操作
    f.Process(data)
}
//正确做法
for _, file := range files{
    if f, err = os.Open(file); err != nil{
        return
    }
    //对文件进行操作
    f.Process(data)
    //关闭文件
    f.Close()
}
//defer仅在函数返回时才会执行，在循环的结尾或其他一些有限范围的代码内不会执行
~~~

####new make

~~~go
- 切片 映射 通道 使用make
- 数组 结构体 所有的值类型 使用new
~~~

####不需要将一个指向切片的指针传递给函数

~~~go
//切片实际是一个指向潜在数组的指针，把切片作为参数传递给函数 实际是传递一个指向变量的指针 在函数内可以改变这个变量 而不是传递数据的拷贝
//true
func findBiggest(listOfNumbers []int) int{}
//false
func findBiggest(lsitOfNumbers *[]int) int{}
~~~

####使用指针指向接口类型

~~~go
type nexter interface {
	next() byte
}
func nextFew1(n nexter, num int) []byte {
	var b []byte
	for i := 0; i < num; i++ {
		b[i] = n.next()  
	}
	return b
}
func nextFew2(n *nexter, num int) []byte {
	var b []byte
	for i := 0; i < num; i++ {
		b[i] = n.next()// 编译错误:n.next未定义（*nexter类型没有next成员或next方法）
	}
	return b
}
//永远不要使用一个指针指向一个接口类型，因为它已经是一个指针
~~~

####闭包协程的使用

~~~go
//A
for k := range values {
    func() {
        fmt.Print(k, " ")
    }()
}
fmt.Println()
//B
for k := range values {
    go func() {
        fmt.Print(k, " ")
    }()
}
fmt.Println()
time.Sleep(5e9)
//C
for k := range values {
    go func(k interface{}) {
        fmt.Print(k, " ")
    }(k)
}
fmt.Println()
time.Sleep(5e9)
//D
for k := range values {
    val := values[k]
    go func() {
        fmt.Print(val, " ")
    }()
}
time.Sleep(1e9)
}
var values = [5]int{10, 11, 12, 13, 14}
//版本A调用闭包5次打印每个索引值，版本B也做相同的事，但是通过协程调用每个闭包。按理说这将执行得更快，因为闭包是并发执行的。如果我们阻塞足够多的时间，让所有协程执行完毕，版本B的输出是：4 4 4 4 4。为什么会这样？在版本B的循环中，k变量实际是一个单变量，表示每个数组元素的索引值。因为这些闭包都只绑定到一个变量，这是一个比较好的方式，当你运行这段代码时，你将看见每次循环都打印最后一个索引值4，而不是每个元素的索引值。因为协程可能在循环结束后还没有开始执行，而此时k值是4。
//版本C的循环写法才是正确的：调用每个闭包时将k作为参数传递给闭包。k在每次循环时都被重新赋值，并将每个协程的k放置在栈中，所以当协程最终被执行时，每个索引值对协程都是可用的。注意这里的输出可能是0 2 1 3 4或者0 3 1 2 4或者其他类似的序列，这主要取决于每个协程何时开始被执行。
//在版本D中，我们输出这个数组的值，为什么版本B不能而版本D可以呢？
//因为版本D中的变量声明是在循环体内部，所以在每次循环时，这些变量相互之间是不共享的，所以这些变量可以单独的被每个闭包使用。
~~~

####错误处理

~~~go
//错误的方式
... err1 := api.Func1()
if err1 != nil {
    fmt.Println("err: " + err.Error())
    return
}
err2 := api.Func2()
if err2 != nil {
...
    return
}
//正确的错误处理方式
func httpRequestHandler(w http.ResponseWriter, req *http.Request){
    err := func() error{
        if req.Method != "GET"{
            return errors.New("unexcept get")
        }
        if input := parseInput(req); input != "command"{
            return errors.New("malformed command")
        }
    }()
    if err != nil{
        w.WriteHeader(400)
        io.WriteString(w,err)
        return
    }
    dosomething()
}
~~~

####错误或者bool返回的形式

~~~go
//在函数返回时检测错误
e.g. : int , err := strconv.Atoi(str) os.Open(file)//user recover() recover program
//检测映射中是否存在一个键值
if v,isPresent = map1[k1]; isPresent{
    Process(v) 
}
//检测一个接口类型变量varI是否包含了类型T：类型断言 varI必须是一个接口变量
if value, ok := varI.(T); ok{
    Process(value)
}
//检测一个通道ch是否关闭
for input := range ch {
    Process(input)
}
or
for{
    if input, open := <-ch; !open{
        break
    }
    Process(input)
}
~~~

#### 常见错误 invalid memory address or nil pointer dereference

~~~go
func main() {
	var i *int
	*i = 1
    fmt.Println(i, &i, *i)
}
//1.报错原因：报这个错的原因是 go 初始化指针的时候会为指针 i 的值赋为 nil ，但 i 的值代表的是 *i 的地址， nil 的话系统还并没有给 *i 分配地址，所以这时给 *i 赋值肯定会出错
//2.解决方法： 解决这个问题非常简单，在给指针赋值前可以先创建一块内存分配给赋值对象即可
func main() {
	var i *int
	i = new(int)
	*i = 1
	fmt.Println(i, &i, *i)
}
//第二种情况
有可能是引入文件出现格式错误
正确："views/block/home_block.html"  错误路径： "view/block/home_block.html"
~~~

## 12.出于性能考虑的使用代码片段

- 字符串

~~~go
//修改字符串中的一个字符
str := "hello"
c := []byte(str)
c[0] = 'a'
s2 := string(c)
//获取子串
substr := str[0:2]
//遍历字符串
for i := 0; i < len(str); i++ {
    //var b byte
    b := str[i]
    fmt.Println(string(b))
}
for k, v := range str{}

//获取字符串的字节数：len(str)

//获取字符数：utf8.RuneCountInString(str) 

//拼接字符串
var buffer bytes.Buffer
buffer.WriteString("abc")
fmt.Println(buffer.String())
//这种实现方式比使用 += 要更节省内存和 CPU，尤其是要串联的字符串数目特别多的时候
~~~

- 数组与切片

~~~go
//创建
arr ：= new([len]type)
slice1 := make([]type)
//初始化
arr1 := [...]type{1,2,3,4,5}
arrkeyValue := [len]type{k1:v1,k2:v2}
slice1 = arr1[start:end]

//二维数组或者切片中查找V 二维数组的声明和复制
var s [3][3]int
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        s[i][j] = rand.Int()
    }
}
for i := 0; i < 3; i++ {
    for j := 0; j < 3; j++ {
        fmt.Printf("s[%d][%d] :  %d\n", i, j, s[i][j])
    }
}
Found:
for row := range s {
    for column := range s[row] {
        if s[row][column] == 6129484611666145821 {
            fmt.Printf("zhaodaole : %d", s[row][column])
            break Found
        }
    }
}
~~~

- 映射

~~~go
//创建
map1 := make(map[type]type)
//初始化
map1 := map[string]int{"wang":1,"li":2}
//检测k1是否存在
val1 , isPresent = map1[k1]
//delete
delete(map1,k1)
~~~

- struct

~~~go
//通常情况下，为每个结构体定义一个构建函数，并推荐使用构建函数初始化结构体
type struct1 struct{
    num1 int
    num2 float32
    num3 string
}
func NewStruct1(n int, f float32, name string) *struct1{
    return &struct1{n, f, name}
}
ms := NewStruct1(19, 15.5, "wang")
~~~

- 接口

~~~go
//如何检测一个值v是否实现了接口Stringer
if v, ok := v.(Stringer); ok{
    fmt.Println("implements String(): %s\n",v.String())
}
//接口实现一个类型分类函数
func classifier(items ...interface{}){
    for i, x := range items{
        switch x.(type){
            case bool:
            	fmt.Printf("param #%d is a bool\n", i)
            case float64:
            	fmt.Printf("param #%d is a float64\n", i)
            case int, int64:
            	fmt.Printf("param #%d is a int\n", i)
            default: 
            	fmt.Println("unknown")
        }
    }
}
~~~

- 文件

~~~go
//如何打开一个文件并读取
file, err := os.Open("input.dat")
if err != nil{
    fmt.Println(err)
    return
}
defer file.Close()
iReader := bufio.NewReader(file)
for {
    str, err := iReader.ReadString('\n')
    if err != nil{
        return
    }
    fmt.Printf("the input is %s\n", str)
}
~~~

- 通道

~~~go
//创建
ch := make(chan type , buf)
//遍历
for v := range ch {
    //todo
}
//检测通道是否关闭
for{
    if input, ok := <-ch; !ok{
        break
    }
    fmt.Printf("%s",input)
}
//一个通道让主程序等待协程完成（信号量模式）
ch := make(chan int)
go func() {
    //todo
    ch <- 1
}()
//do else
<- ch // 如果希望程序一直阻塞 在匿名函数中省略 ch <- 1
//通道的工厂模板：以下函数是一个通道工厂，启动一个匿名函数作为协程以生产通道
//通道迭代器模板
//如何限制并发处理请求的数量
//如何在多核CPU上实现并行计算
//如何终止一个协程 runtime.Goexit()
//简单的超时模板
timeout := make(chan bool, 1)
go func(){
    time.Sleep(1e9)
    timeout <- true
}()
select{
    case <- ch:
    // a read from ch has occured
    case <- timeout:
    //the read from ch has timed out
}
//如何使用输入通道和输出通道代替锁
//如何在同步调用运行时间过长时将之丢弃
//如何在通道中使用计时器和定时器
//典型的服务器后端模型

/*******************/
//程序出错时终止程序
if err != nil{
    fmt.Printf("program stopping with error %v", err)
    os.Exit(1)
}
if err != nil{
    panic("ERROR occurred:" + err.Error())
}
~~~



## 13.常用工具

#### 1.pprof是golang程序一个性能分析的工具，可以查看堆栈、cpu信息等 

#### 2.go常用插件

~~~go
go get -u -v golang.org/x/tools/cmd/godoc
go get -u -v github.com/nsf/gocode
go get -u -v github.com/golang/lint/golint
go get -u -v github.com/lukehoban/go-outline
go get -u -v github.com/rogpeppe/godef
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v golang.org/x/tools/cmd/gorename
go get -u -v golang.org/x/tools/cmd/guru
go get -u -v github.com/tpng/gopkgs
go get -u -v github.com/newhook/go-symbols
go get -u -v github.com/derekparker/delve/cmd/dlv
go get -u -v github.com/josharian/impl
go get -u -v github.com/cweill/gotests/gotests
go get -u -v github.com/lukehoban/go-find-references
go get -u -v github.com/fatih/gomodifytags
go get -u -v github.com/haya14busa/goplay/cmd/goplay
~~~

##14.单向链表

- 链表中存储着两个东西： 一个值和一个指向列表中的下一个节点的引用

~~~go
// package main
 
// import (
// 	"fmt"
// 	_"os"
// )
 
// /**
// golang实现单链表
// 单链表是一种链式存取的数据结构，用一组地址任意的存储单元存放线性表中的数据元素。
// 链表中的数据以节点来表示，每个节点的构成：元素(数据元素的映像)+ 指针(指向后继元素存储位置)。元素是存储数据的存储单元，指针是连接每个节点的地址数据。
// 节点结构：
// |--data--|--next--|
// data域存放节点值的数据域，next域存放节点的直接后继地址(位置)的指针域(链域)
// 头指针head和终端节点
// 单链表中每个几点的存储地址是存放在其前趋节点next域中。而开始节点无前趋，所以应该设置头指针head指向开始节点。链表由头指针唯一确定，单链表可以用头指针的名字来命名。
// 终端节点无后继，所以终端节点的指针域为空，即NULL。
//  */
 
// type Node struct {
// 	// 值
// 	Data interface{}
// 	// 后继节点指针
// 	Next *Node
// }
 
// // 链表是否为空
// func IsEmpty(node *Node) bool {
// 	return node == nil
// }
 
// // 是否是最后一个节点
// func IsLast(node *Node) bool  {
// 	return node.Next == nil
// }
 
// // 查找指定节点的前一个节点
// func FindPrevious(data interface{}, node *Node) *Node  {
// 	tmp := node
// 	for tmp.Next != nil && tmp.Next.Data != data {
// 		tmp = tmp.Next
// 	}
// 	return tmp
// }
// // 查找某个值在哪个节点
// func Find(data interface{}, node *Node) *Node  {
// 	tmp := node
// 	for tmp.Data != data {
// 		tmp = tmp.Next
// 	}
// 	return tmp
// }
 
// // 插入节点:在指定节点插入节点
// /**
// position:指定的节点位置
//  */
// func Insert(data interface{}, position *Node)  {
// 	// 新建一个节点
// 	tmpCell := new(Node)
// 	if tmpCell == nil {
// 		fmt.Println("err: out of space")
// 	}
// 	// 给新建的节点的值域赋值为传入的data值
// 	tmpCell.Data = data
// 	// 新建的节点的next指针指向的是指定节点position的next
// 	tmpCell.Next = position.Next
// 	// 指定节点position的后继节点变成了新建的节点
// 	position.Next = tmpCell
// }
 
// // 删除节点
// func Delete(data interface{}, node *Node)  {
// 	preview := FindPrevious(data, node)
// 	tmp := Find(data, node)
// 	preview.Next = tmp.Next
// 	tmp = nil
// }
 
// // 删除链表
// func DeleteList(node **Node)  {
// 	p := node
// 	for *p != nil {
// 		tmp := *p
// 		*p = nil
// 		*p = tmp.Next
// 	}
// }
 
// //打印列表
// func PrintList(list *Node) {
// 	p := list
// 	for p != nil {
// 		fmt.Printf("%d-%v-%p ", p.Data, p, p.Next)
// 		p = p.Next
// 	}
// 	fmt.Println()
// }
 
// func main() {
// 	headNode := &Node{
// 		Data:0,
// 		Next:nil,
// 	}
// 	list := headNode
// 	Insert(1, headNode)
// 	Insert(2, headNode)
// 	Insert(3, headNode)
// 	PrintList(list)
// 	//fmt.Println(IsEmpty(list))
// 	//fmt.Println(IsLast(headNode))
// 	//p := Find(0, list)
// 	//Insert(2, p)
// 	//PrintList(list)
// 	//Delete(1, list)
// 	//PrintList(list)
// 	//DeleteList(&list)
// 	//PrintList(list)
// }


package main
import "fmt"
import "errors"
type Post struct{
    body string
    publishDate int64
    next *Post
}

type Feed struct{
    length int
    start *Post
    end *Post
}

func (f *Feed)Append(newPost *Post){

    if f.length == 0{
        f.start = newPost
        f.end = newPost
    }else{
        lastPost := f.end
        lastPost.next = newPost
        f.end = newPost
    
    }
    f.length++

}

func (f *Feed)Remove(publishDate int64){
    if f.length ==0{
        panic(errors.New("feed is empty"))
    }

    
    var prePost *Post
    currentPost := f.start

    if currentPost.publishDate == publishDate{ 
        f.start = f.start.next 
        f.length-- 
        return 
    }

    for currentPost.publishDate != publishDate{
        if currentPost.next == nil{
            panic(errors.New("no such post found"))
        }
        prePost = currentPost
        currentPost = currentPost.next
    }
    prePost.next = currentPost.next
    f.length--
}

func (f *Feed)Insert(newPost *Post){
    if f.length == 0{
        f.start = newPost
    }else{
        var prePost *Post
        currentPost := f.start

        for currentPost.publishDate < newPost.publishDate{
            prePost = currentPost
            currentPost = prePost.next
        }

        prePost.next = newPost
        newPost.next = currentPost

    }

    f.length++
}

func main(){

    f := &Feed{}
    p1 := Post{
        body: "hahahasb",
        publishDate: 3,
    }
    f.Append(&p1)

    p2 := Post{
        body: "hahahasb2",
        publishDate: 5,
    }
    p3 := Post{
        body: "hahahasb2",
        publishDate: 7,
    }

    p4 := Post{
        body: "hahahasb5",
        publishDate: 6,
    }
    f.Append(&p2)
    f.Append(&p3)
    f.Insert(&p4)
    fmt.Println(f.length)
    fmt.Println(f.start)
    fmt.Println(f.start.next)
    fmt.Println(f.start.next.next)
    fmt.Println(f.start.next.next.next)
    f.Remove(3)
    fmt.Println(f.length)
    fmt.Println(f.start)
    fmt.Println(f.start.next)
    fmt.Println(f.start.next.next)
}
~~~

| action  | array | linked list |
| :-----: | :---: | :---------: |
| Access  | O(1)  |    O(n)     |
| Search  | O(n)  |    O(n)     |
| Prepend | O(1)  |    O(1)     |
| Append  | O(n)  |    O(1)     |
| Delete  | O(n)  |    O(n)     |

## 15.二叉树

![](image\go\erchashu.png)

- 概念

~~~shell
#完全二叉树： 叶子节点都在最底下两层，最后一层的叶子节点都是靠左排列， 并且除了最后一层， 其它层的节点个数都要达到最大，这种二叉树叫做完全二叉树。
#满二叉树： 叶子节点全部在最底层，除了叶子节点之外， 每个节点都有左右两个子节点，这种树叫做满二叉树
#平衡二叉树： 平衡二叉树又被称作AVL树，他是一棵空树或者他的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一个平衡二叉树
~~~

- 遍历

![](image\go\demo01.png)

~~~shell
#前序遍历： 先访问当前节点 再前序遍历左子树 最后再遍历右子树 根——左——右 1 2 4 5 7 8 3 6
#中序遍历： 先中序遍历左子树 然后再访问当前节点 最后再中序遍历右子树 左——根——右 4 2 7 8 5 1 3 6
#后序遍历： 先后序遍历左子树 然后再后序遍历右子树 最后再访问当前节点 左——右——根 4 8 7 5 2 6 3 1
#层次遍历： 从第一层开始 依次遍历每层 直到结束 1 2 3 4 5 6 7 8 
~~~

~~~代码
package main
import (
    "fmt"
)
type Node struct{
    Value int
    Left *Node
    Right *Node
}
func (n *Node) Print(){
    fmt.Print(n.Value, " ")
}
func (n *Node)SetValue(v int){
    if n== nil{
        fmt.Println("setting value to nil.node")
        return
    }
    n.Value = v
}
//前序遍历
func (n *Node) PreOrder(){
    if n == nil{
        return
    }
    n.Print()
    n.Left.PreOrder()
    n.Right.PreOrder()
}
//中序遍历
func (n *Node)MidOrder(){
    if n == nil{
        return
    }
    n.Left.MidOrder()
    n.Print()
    n.Right.MidOrder()
}
//后序遍历
func (n *Node)PostOrder(){
    if  n == nil{
        return
    }
    n.Left.PostOrder()
    n.Right.PostOrder()
    n.Print()
}
//层次遍历（广度优先遍历）
func (n *Node)BreadFirstSearch(){
    if n == nil{
        return
    }
    res := []int{}
    nodes := []*Node{n}
    for len(nodes) > 0{
        curNode := nodes[0]
        nodes = nodes[1:]
        res = append(res, curNode.Value)
        if curNode.Left != nil {
            nodes = append(nodes, curNode.Left)
        }
        if curNode.Right != nil{
            nodes = append(nodes,curNode.Right)
        }
    }
    for _, v := range res{
        fmt.Print(v, " ")
    }
}
//层数 对任意一个子树的根节点来说 他的深度=左右子树深度的最大值+1
func (n *Node) Layer() int{
    if n == nil{
        return 0
    }
    leftLayer := n.Left.Layer()
    rightLayer := n.Right.Layer()

    if leftLayer > rightLayer{
        return leftLayer + 1
    }else{
        return rightLayer + 1
    }
}
//层数 非递归实现
func (n *Node) LayersByQueue() int{
    if n == nil{
        return 0
    }
    layers := 0
    nodes := []*Node{n}
    for len(nodes) > 0{
        layers++
        size := len(nodes)
        count := 0
        for count < size {
            count++ 
            curNode := nodes[0]
            if curNode.Left != nil{
                nodes = append(nodes, curNode.Left)
            }
            if curNode.Right != nil{
                nodes = append(nodes, curNode.Right)
            }
        }
    }
    return layers
}
func CreateNode(v int) *Node{
    return &Node{Value : v}
}
func main(){
    root := Node{Value : 3}
    root.Left = &Node{}
    root.Left.SetValue(0)
    root.Left.Right = CreateNode(2)
    root.Right = &Node{5, nil, nil}
    root.Right.Left = CreateNode(4)

    fmt.Print("\n前序遍历： ")
    root.PreOrder()

    fmt.Print("\n中序遍历： ")
    root.MidOrder()

    fmt.Print("\n后序遍历： ")
    root.PostOrder()

    fmt.Print("\n层次遍历： ")
    root.BreadFirstSearch()
} 
~~~

## 16.位运算

~~~shell
1.位移操作 << : 向左移位 可以看做是乘以2的几次方
2.位移操作 >> : 向右移位 可以看做除以2的几次方
3.and操作 & : 都为1 才是1
4.or操作 | : 只要有一个是1 那么就是1
5.取反 ^ : 有一个为1 则为1 两个1或者两个两个0 全是0
6.标志位操作 &^ : a &^ b = (a^b) & b 其实就是清除标记位

func main(){
		const a = 2   //0010
	const b = 6   //0110
	const c = 11  //1011
	var d = a ^ c //1001 9
	Println(d)
	d = b &^ c //0110 1011
	Println(d) //0100
	Println(1 << 2) // 0001 --> 0100
	a1 := 4
	a1 |= 1 << 4
	Println(a1) //20
	a1 |= 1 << 3
	Println(a1) //28
}
~~~

## 17. 部署go项目的命令

~~~shell
$ nohup ./goweb &> goweb.log
$ nohup ./go > out.log 2>&1 &
#这个意思是把标准错误2重定向到标准输出1中，而标准输出又导入文件out.log中，以结果是标准错误和标准输出都导入文件out.log里面了 一般很大的stdout和stderr当你不关心的时候可以利用/dev/null 定向到 
$ nohup ./command > /dev/null 2>&1
~~~

## 18.字符串 整数 浮动见的转换

~~~go
//string > int
int, err := strconv.Atoi(string)
//string > int64
int64, err := strconv.ParseInt(string, 10, 64)
//第二个参数为基数（2-36）(基数是指string 所表示的数值的进制)
//第三个参数大小表示期望转换的结果类型 值为 0 8 16 32 64
//分别对应着 int int8 int16 int32 int64
//int > string
string := strconv.Itoa(int)
//等价于
string := strconv.FormatInt(int64(int), 10)
//int64 > string
string := strconv.FormatInt(int64, 10)
//第二个参数为基数 可选2-36
//对于无符号整型 可以使用FormatUint(i uint, base int)

//float > string
string := strconv.FormatFloat(float32, 'E',-1,32)
string := strconv.FormatFloat(float64, 'E',-1,64)
//'b'(-ddddp±ddd, 二进制指数)
//'e'(-d.dddde±dd, 十进制指数)
//'E'(-d.dddde±dd, 十进制指数)
//'f'(-ddd.dddd, 没有指数)

//string > float64
float64, err := strconv.ParseFloat(string, 64)
//string > float32
float32, err := strconv.ParseFloat(string, 32)

~~~

## 19. go mod

~~~go

~~~

## go语言程序设计（黑色版）知识总结

####1.变量 指针 new函数 变量的声明周期

~~~ go
//因为一个指针包含变量的地址 所以传递一个指针参数给函数 能够让函数更新间接传递的变量值 例如 这个函数递增一个指针参数所指向的变量 然后返回此变量的新值，于是他可以在表达式中使用：
func incr(p *int)int{
    *p++ //递增p所指向的值，p自身保持不变
    return *p
}

v := 1
incr(&v)  //副作用， v现在等于2
fmt.Println(incr(&v)) // 3
~~~

####2. 赋值

~~~go
//操作符也具有和函数类似的返回错误的行为 其中有map查询 类型断言 通道接收操作
v, ok = m[key]
v, ok = x.(T)
v, ok = <-ch
//像变量声明一样 可以将不需要的赋值给空标识符
_, err = io.Copy(dst, src) //丢弃字节个数
_, ok = x.(T)
//type声明定义一个新的命名类型 它和某个已有类型使用相同的底层类型
type name underlying-type
//类型的声明通常是在包级别 这里命名的类型是整个包可见 如果名字是导出的（开头字母大写） 其他包也可以访问他

//包的作用和其他语言中的库或模块类似 用于支持模块化 封装 编译隔离和重用 一个包的源代码保存在多个.go结尾的文件中
~~~

### 基本数据

go数据类型分为四大类 ： 基础类型 	聚合类型	引用类型	接口类型

基础类型包括： 数字	字符串	布尔型

聚合类型： 数组	结构体

引用是一大分类 其中包括多种不同类型： 指针	slice	map	函数	通道（channel）	他们的共同点事全都间接指向程序变量和状态 于是操作所引用数据就会遍及该数据的全部引用

接口类型： 



 