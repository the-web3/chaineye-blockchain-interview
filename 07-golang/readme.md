# go 面试题目汇总

## 第一题： 交替打印数字和字母
**问题描述**

使用两个 `goroutine` 交替打印序列，一个 `goroutine` 打印数字， 另外一个 `goroutine` 打印字母， 最终效果如下：

```bash
12AB34CD56EF78GH910IJ1112KL1314MN1516OP1718QR1920ST2122UV2324WX2526YZ2728
```

**解题思路**

问题很简单，使用 channel 来控制打印的进度。使用两个 channel ，来分别控制数字和字母的打印序列， 数字打印完成后通过 channel 通知字母打印, 字母打印完成后通知数字打印，然后周而复始的工作。

**源码参考**

```go
	letter,number := make(chan bool),make(chan bool)
	wait := sync.WaitGroup{}

	go func() {
		i := 1
		for {
			select {
			case <-number:
				fmt.Print(i)
				i++
				fmt.Print(i)
				i++
				letter <- true
			}
		}
	}()
	wait.Add(1)
	go func(wait *sync.WaitGroup) {
		i := 'A'
		for{
			select {
			case <-letter:
				if i >= 'Z' {
					wait.Done()
					return
				}

				fmt.Print(string(i))
				i++
				fmt.Print(string(i))
				i++
				number <- true
			}

		}
	}(&wait)
	number<-true
	wait.Wait()
```

**源码解析**

这里用到了两个`channel`负责通知，letter负责通知打印字母的goroutine来打印字母，number用来通知打印数字的goroutine打印数字。wait用来等待字母打印完成后退出循环。

也可以分别使用三个 channel 来控制数字，字母以及终止信号的输入.

```go

package main

import "fmt"

func main() {
	number := make(chan bool)
	letter := make(chan bool)
	done := make(chan bool)

	go func() {
		i := 1
		for {
			select {
			case <-number:
				fmt.Print(i)
				i++
				fmt.Print(i)
				i++
				letter <- true
			}
		}
	}()

	go func() {
		j := 'A'
		for {
			select {
			case <-letter:
				if j >= 'Z' {
					done <- true
				} else {
					fmt.Print(string(j))
					j++
					fmt.Print(string(j))
					j++
					number <- true
				}
			}
		}
	}()

	number <- true

	for {
		select {
		case <-done:
			return
		}
	}
}
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第二题：判断字符串中字符是否全都不同

**问题描述**

请实现一个算法，确定一个字符串的所有字符【是否全都不同】。这里我们要求【不允许使用额外的存储结构】。
给定一个string，请返回一个bool值,true代表所有字符全都不同，false代表存在相同的字符。
保证字符串中的字符为【ASCII字符】。字符串的长度小于等于【3000】。


**解题思路**

这里有几个重点，第一个是`ASCII字符`，`ASCII字符`字符一共有256个，其中128个是常用字符，可以在键盘上输入。128之后的是键盘上无法找到的。

然后是全部不同，也就是字符串中的字符没有重复的，再次，不准使用额外的储存结构，且字符串小于等于3000。

如果允许其他额外储存结构，这个题目很好做。如果不允许的话，可以使用golang内置的方式实现。

**源码参考**

通过`strings.Count` 函数判断：

```
func isUniqueString(s string) bool {
	if strings.Count(s,"") > 3000{
		return  false
	}
	for _,v := range s {
		if v > 127 {
			return false
		}
		if strings.Count(s,string(v)) > 1 {
			return false
		}
	}
	return true
}
```

通过`strings.Index`和`strings.LastIndex`函数判断：

```
func isUniqueString2(s string) bool {
	if strings.Count(s,"") > 3000{
		return  false
	}
	for k,v := range s {
		if v > 127 {
			return false
		}
		if strings.Index(s,string(v)) != k {
			return false
		}
	}
	return true
}
```

通过位运算判断

```
func isUniqString3(s string) bool {
	if len(s) == 0 || len(s) > 3000 {
		return false
	}
	// 256 个字符 256 = 64 + 64 + 64 + 64
	var mark1, mark2, mark3, mark4 uint64
	var mark *uint64
	for _, r := range s {
		n := uint64(r)
		if n < 64 {
			mark = &mark1
		} else if n < 128 {
			mark = &mark2
			n -= 64
		} else if n < 192 {
			mark = &mark3
			n -= 128
		} else {
			mark = &mark4
			n -= 192
		}
		if (*mark)&(1<<n) != 0 {
			return false
		}
		*mark = (*mark) | uint64(1<<n)
	}
	return true
}
```


**源码解析**

以上三种方法都可以实现这个算法。

第一个方法使用的是golang内置方法`strings.Count`,可以用来判断在一个字符串中包含的另外一个字符串的数量。

第二个方法使用的是golang内置方法`strings.Index`和`strings.LastIndex`，用来判断指定字符串在另外一个字符串的索引未知，分别是第一次发现位置和最后发现位置。

第三个方法使用的是位运算来判断是否重复，时间复杂度为o(n)，相比前两个方法时间复杂度低

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第三题：翻转字符串

**问题描述**

请实现一个算法，在不使用【额外数据结构和储存空间】的情况下，翻转一个给定的字符串(可以使用单个过程变量)。

给定一个string，请返回一个string，为翻转后的字符串。保证字符串的长度小于等于5000。

**解题思路**

翻转字符串其实是将一个字符串以中间字符为轴，前后翻转，即将str[len]赋值给str[0],将str[0] 赋值 str[len]。

**源码参考**

```
func reverString(s string) (string, bool) {
    str := []rune(s)
    l := len(str)
    if l > 5000 {
        return s, false
    }
    for i := 0; i < l/2; i++ {
        str[i], str[l-1-i] = str[l-1-i], str[i]
    }
    return string(str), true
}
```

**源码解析**

以字符串长度的1/2为轴，前后赋值
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第四题：判断两个给定的字符串排序后是否一致

**问题描述**

给定两个字符串，请编写程序，确定其中一个字符串的字符重新排列后，能否变成另一个字符串。
这里规定【大小写为不同字符】，且考虑字符串重点空格。给定一个string s1和一个string s2，请返回一个bool，代表两串是否重新排列后可相同。
保证两串的长度都小于等于5000。

**解题思路**

首先要保证字符串长度小于5000。之后只需要一次循环遍历s1中的字符在s2是否都存在即可。

**源码参考**

```
func isRegroup(s1,s2 string) bool {
	sl1 := len([]rune(s1))
	sl2 := len([]rune(s2))

	if sl1 > 5000 || sl2 > 5000 || sl1 != sl2{
		return false
	}

	for _,v := range s1 {
		if strings.Count(s1,string(v)) != strings.Count(s2,string(v)) {
			return false
		}
	}
	return true
}
```

**源码解析**

这里还是使用golang内置方法 `strings.Count` 来判断字符是否一致。
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

第五题：字符串替换问题

**问题描述**

请编写一个方法，将字符串中的空格全部替换为“%20”。
假定该字符串有足够的空间存放新增的字符，并且知道字符串的真实长度(小于等于1000)，同时保证字符串由【大小写的英文字母组成】。
给定一个string为原始的串，返回替换后的string。

**解题思路**

两个问题，第一个是只能是英文字母，第二个是替换空格。

**源码参考**

```
func replaceBlank(s string) (string, bool) {
	if len([]rune(s)) > 1000 {
		return s, false
	}
	for _, v := range s {
		if string(v) != " " && unicode.IsLetter(v) == false {
			return s, false
		}
	}
	return strings.Replace(s, " ", "%20", -1), true
}
```

**源码解析**

这里使用了golang内置方法`unicode.IsLetter`判断字符是否是字母，之后使用`strings.Replace`来替换空格。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第六题：机器人坐标问题

**问题描述**

有一个机器人，给一串指令，L左转 R右转，F前进一步，B后退一步，问最后机器人的坐标，最开始，机器人位于 0 0，方向为正Y。
可以输入重复指令n ： 比如 R2(LF) 这个等于指令 RLFLF。
问最后机器人的坐标是多少？

**解题思路**

这里的一个难点是解析重复指令。主要指令解析成功，计算坐标就简单了。

**源码参考**

```
package main

import (
	"unicode"
)

const (
	Left = iota
	Top
	Right
	Bottom
)

func main() {
	println(move("R2(LF)", 0, 0, Top))
}

func move(cmd string, x0 int, y0 int, z0 int) (x, y, z int) {
	x, y, z = x0, y0, z0
	repeat := 0
	repeatCmd := ""
	for _, s := range cmd {
		switch {
		case unicode.IsNumber(s):
			repeat = repeat*10 + (int(s) - '0')
		case s == ')':
			for i := 0; i < repeat; i++ {
				x, y, z = move(repeatCmd, x, y, z)
			}
			repeat = 0
			repeatCmd = ""
		case repeat > 0 && s != '(' && s != ')':
			repeatCmd = repeatCmd + string(s)
		case s == 'L':
			z = (z + 1) % 4
		case s == 'R':
			z = (z - 1 + 4) % 4
		case s == 'F':
			switch {
			case z == Left || z == Right:
				x = x - z + 1
			case z == Top || z == Bottom:
				y = y - z + 2
			}
		case s == 'B':
			switch {
			case z == Left || z == Right:
				x = x + z - 1
			case z == Top || z == Bottom:
				y = y + z - 2
			}
		}
	}
	return
}

```

**源码解析**

这里使用三个值表示机器人当前的状况，分别是：x表示x坐标，y表示y坐标，z表示当前方向。
L、R 命令会改变值z，F、B命令会改变值x、y。
值x、y的改变还受当前的z值影响。

如果是重复指令，那么将重复次数和重复的指令存起来递归调用即可。
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第七题、下面代码能运行吗？为什么。

```go
type Param map[string]interface{}

type Show struct {
	Param
}

func main1() {
	s := new(Show)
	s.Param["RMB"] = 10000
}
```

**解析**

共发现两个问题：

1. `main` 函数不能加数字。
2. `new` 关键字无法初始化 `Show` 结构体中的 `Param` 属性，所以直接对 `s.Param` 操作会出错。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第八题：请说出下面代码存在什么问题。

```go
type student struct {
	Name string
}

func zhoujielun(v interface{}) {
	switch msg := v.(type) {
	case *student, student:
		msg.Name
	}
}
```

**解析：**

golang中有规定，`switch type`的`case T1`，类型列表只有一个，那么`v := m.(type)`中的`v`的类型就是T1类型。

如果是`case T1, T2`，类型列表中有多个，那`v`的类型还是多对应接口的类型，也就是`m`的类型。

所以这里`msg`的类型还是`interface{}`，所以他没有`Name`这个字段，编译阶段就会报错。具体解释见： <https://golang.org/ref/spec#Type_switches>

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第九题：写出打印的结果。

```go
type People struct {
	name string `json:"name"`
}

func main() {
	js := `{
		"name":"11"
	}`
	var p People
	err := json.Unmarshal([]byte(js), &p)
	if err != nil {
		fmt.Println("err: ", err)
		return
	}
	fmt.Println("people: ", p)
}
```

**解析：**

按照 golang 的语法，小写开头的方法、属性或 `struct` 是私有的，同样，在`json` 解码或转码的时候也无法上线私有属性的转换。

题目中是无法正常得到`People`的`name`值的。而且，私有属性`name`也不应该加`json`的标签。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第十题：下面的代码是有问题的，请说明原因。

```go
type People struct {
	Name string
}

func (p *People) String() string {
	return fmt.Sprintf("print: %v", p)
}

func main() {
 	p := &People{}
	p.String()
}
```

**解析：**

在golang中`String() string` 方法实际上是实现了`String`的接口的，该接口定义在`fmt/print.go`  中：

```go
type Stringer interface {
	String() string
}
```

在使用 `fmt` 包中的打印方法时，如果类型实现了这个接口，会直接调用。而题目中打印 `p` 的时候会直接调用 `p` 实现的 `String()` 方法，然后就产生了循环调用。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第十题：请找出下面代码的问题所在。

```go
func main() {
	ch := make(chan int, 1000)
	go func() {
		for i := 0; i < 10; i++ {
			ch <- i
		}
	}()
	go func() {
		for {
			a, ok := <-ch
			if !ok {
				fmt.Println("close")
				return
			}
			fmt.Println("a: ", a)
		}
	}()
	close(ch)
	fmt.Println("ok")
	time.Sleep(time.Second * 100)
}
```


**解析：**

在 golang 中 `goroutine` 的调度时间是不确定的，在题目中，第一个写 `channel` 的 `goroutine` 可能还未调用，或已调用但没有写完时直接 `close` 管道，可能导致写失败，既然出现 `panic` 错误。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第十一题：请说明下面代码书写是否正确。

```go
var value int32

func SetValue(delta int32) {
	for {
		v := value
		if atomic.CompareAndSwapInt32(&value, v, (v+delta)) {
			break
		}
	}
}
```


    
**解析：**

`atomic.CompareAndSwapInt32` 函数不需要循环调用。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第十二题、下面的程序运行后为什么会爆异常。

```go
type Project struct{}

func (p *Project) deferError() {
	if err := recover(); err != nil {
		fmt.Println("recover: ", err)
	}
}

func (p *Project) exec(msgchan chan interface{}) {
	for msg := range msgchan {
		m := msg.(int)
		fmt.Println("msg: ", m)
	}
}

func (p *Project) run(msgchan chan interface{}) {
	for {
		defer p.deferError()
		go p.exec(msgchan)
		time.Sleep(time.Second * 2)
	}
}

func (p *Project) Main() {
	a := make(chan interface{}, 100)
	go p.run(a)
	go func() {
		for {
			a <- "1"
			time.Sleep(time.Second)
		}
	}()
	time.Sleep(time.Second * 100000000000000)
}

func main() {
	p := new(Project)
	p.Main()
}
```

**解析：**

有一下几个问题：

1. `time.Sleep` 的参数数值太大，超过了 `1<<63 - 1` 的限制。
2. `defer p.deferError()` 需要在协程开始出调用，否则无法捕获 `panic`。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第十三题、请说出下面代码哪里写错了

```go
func main() {
	abc := make(chan int, 1000)
	for i := 0; i < 10; i++ {
		abc <- i
	}
	go func() {
		for  a := range abc  {
			fmt.Println("a: ", a)
		}
	}()
	close(abc)
	fmt.Println("close")
	time.Sleep(time.Second * 100)
}
```

**解析：**

协程可能还未启动，管道就关闭了。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第十四题、请说出下面代码，执行时为什么会报错

```go
type Student struct {
	name string
}

func main() {
	m := map[string]Student{"people": {"zhoujielun"}}
	m["people"].name = "wuyanzu"
}

```

**解析：**

map的value本身是不可寻址的，因为map中的值会在内存中移动，并且旧的指针地址在map改变时会变得无效。故如果需要修改map值，可以将`map`中的非指针类型`value`，修改为指针类型，比如使用`map[string]*Student`.

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第十五题、请说出下面的代码存在什么问题？

```go
type query func(string) string

func exec(name string, vs ...query) string {
	ch := make(chan string)
	fn := func(i int) {
		ch <- vs[i](name)
	}
	for i, _ := range vs {
		go fn(i)
	}
	return <-ch
}

func main() {
	ret := exec("111", func(n string) string {
		return n + "func1"
	}, func(n string) string {
		return n + "func2"
	}, func(n string) string {
		return n + "func3"
	}, func(n string) string {
		return n + "func4"
	})
	fmt.Println(ret)
}
```

**解析：** 

依据4个goroutine的启动后执行效率，很可能打印111func4，但其他的111func*也可能先执行，exec只会返回一条信息。


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第十六题、下面这段代码为什么会卡死？

```go
package main

import (
    "fmt"
    "runtime"
)

func main() {
    var i byte
    go func() {
        for i = 0; i <= 255; i++ {
        }
    }()
    fmt.Println("Dropping mic")
    // Yield execution to force executing other goroutines
    runtime.Gosched()
    runtime.GC()
    fmt.Println("Done")
}
```

**解析：**

Golang 中，byte 其实被 alias 到 uint8 上了。所以上面的 for 循环会始终成立，因为 i++ 到 i=255 的时候会溢出，i <= 255 一定成立。

也即是， for 循环永远无法退出，所以上面的代码其实可以等价于这样：
```go
go func() {
    for {}
}
```

正在被执行的 goroutine 发生以下情况时让出当前 goroutine 的执行权，并调度后面的 goroutine 执行：

- IO 操作
- Channel 阻塞
- system call
- 运行较长时间

如果一个 goroutine 执行时间太长，scheduler 会在其 G 对象上打上一个标志（ preempt），当这个 goroutine 内部发生函数调用的时候，会先主动检查这个标志，如果为 true 则会让出执行权。

main 函数里启动的 goroutine 其实是一个没有 IO 阻塞、没有 Channel 阻塞、没有 system call、没有函数调用的死循环。

也就是，它无法主动让出自己的执行权，即使已经执行很长时间，scheduler 已经标志了 preempt。

而 golang 的 GC 动作是需要所有正在运行 `goroutine` 都停止后进行的。因此，程序会卡在 `runtime.GC()` 等待所有协程退出。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第十七题、写出下面代码输出内容。

```go
package main

import (
	"fmt"
)

func main() {
	defer_call()
}

func defer_call() {
	defer func() { fmt.Println("打印前") }()
	defer func() { fmt.Println("打印中") }()
	defer func() { fmt.Println("打印后") }()

	panic("触发异常")
}
```

**解析：**

`defer` 关键字的实现跟go关键字很类似，不同的是它调用的是`runtime.deferproc`而不是`runtime.newproc`。

在`defer`出现的地方，插入了指令`call runtime.deferproc`，然后在函数返回之前的地方，插入指令`call runtime.deferreturn`。

goroutine的控制结构中，有一张表记录`defer`，调用`runtime.deferproc`时会将需要defer的表达式记录在表中，而在调用`runtime.deferreturn`的时候，则会依次从defer表中出栈并执行。

因此，题目最后输出顺序应该是`defer` 定义顺序的倒序。`panic` 错误并不能终止 `defer` 的执行。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第十八题、 以下代码有什么问题，说明原因

```go
type student struct {
	Name string
	Age  int
}

func pase_student() {
	m := make(map[string]*student)
	stus := []student{
		{Name: "zhou", Age: 24},
		{Name: "li", Age: 23},
		{Name: "wang", Age: 22},
	}
	for _, stu := range stus {
		m[stu.Name] = &stu
	}
}
```

**解析：**

golang 的 `for ... range` 语法中，`stu` 变量会被复用，每次循环会将集合中的值复制给这个变量，因此，会导致最后`m`中的`map`中储存的都是`stus`最后一个`student`的值。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第十九题、下面的代码会输出什么，并说明原因

```go
func main() {
	runtime.GOMAXPROCS(1)
	wg := sync.WaitGroup{}
	wg.Add(20)
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println("i: ", i)
			wg.Done()
		}()
	}
	for i := 0; i < 10; i++ {
		go func(i int) {
			fmt.Println("i: ", i)
			wg.Done()
		}(i)
	}
	wg.Wait()
}

```

**解析：**

这个输出结果决定来自于调度器优先调度哪个G。从runtime的源码可以看到，当创建一个G时，会优先放入到下一个调度的`runnext`字段上作为下一次优先调度的G。因此，最先输出的是最后创建的G，也就是9.

```go
func newproc(siz int32, fn *funcval) {
	argp := add(unsafe.Pointer(&fn), sys.PtrSize)
	gp := getg()
	pc := getcallerpc()
	systemstack(func() {
		newg := newproc1(fn, argp, siz, gp, pc)

		_p_ := getg().m.p.ptr()
        //新创建的G会调用这个方法来决定如何调度
		runqput(_p_, newg, true)

		if mainStarted {
			wakep()
		}
	})
}
...

	if next {
	retryNext:
		oldnext := _p_.runnext
        //当next是true时总会将新进来的G放入下一次调度字段中
		if !_p_.runnext.cas(oldnext, guintptr(unsafe.Pointer(gp))) {
			goto retryNext
		}
		if oldnext == 0 {
			return
		}
		// Kick the old runnext out to the regular run queue.
		gp = oldnext.ptr()
	}
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十题、下面代码会输出什么？

```go
type People struct{}

func (p *People) ShowA() {
	fmt.Println("showA")
	p.ShowB()
}
func (p *People) ShowB() {
	fmt.Println("showB")
}

type Teacher struct {
	People
}

func (t *Teacher) ShowB() {
	fmt.Println("teacher showB")
}

func main() {
	t := Teacher{}
	t.ShowA()
}


```


**解析：**

输出结果为`showA`、`showB`。golang 语言中没有继承概念，只有组合，也没有虚方法，更没有重载。因此，`*Teacher` 的 `ShowB` 不会覆写被组合的 `People` 的方法。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十一题、下面代码会触发异常吗？请详细说明

```go
func main() {
	runtime.GOMAXPROCS(1)
	int_chan := make(chan int, 1)
	string_chan := make(chan string, 1)
	int_chan <- 1
	string_chan <- "hello"
	select {
	case value := <-int_chan:
		fmt.Println(value)
	case value := <-string_chan:
		panic(value)
	}
}
```

**解析：**

结果是随机执行。golang 在多个`case` 可读的时候会公平的选中一个执行。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十二题、下面代码输出什么？

```go
func calc(index string, a, b int) int {
	ret := a + b
	fmt.Println(index, a, b, ret)
	return ret
}

func main() {
	a := 1
	b := 2
	defer calc("1", a, calc("10", a, b))
	a = 0
	defer calc("2", a, calc("20", a, b))
	b = 1
}
```

**解析：**

输出结果为：

```
10 1 2 3
20 0 2 2
2 0 2 2
1 1 3 4
```

`defer` 在定义的时候会计算好调用函数的参数，所以会优先输出`10`、`20` 两个参数。然后根据定义的顺序倒序执行。


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十三题、请写出以下输入内容

```go
func main() {
	s := make([]int, 5)
	s = append(s, 1, 2, 3)
	fmt.Println(s)
}

```

**解析：**

输出为 `0 0 0 0 0 1 2 3`。

`make` 在初始化切片时指定了长度，所以追加数据时会从`len(s)` 位置开始填充数据。


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十四题、下面的代码有什么问题?

```go
type UserAges struct {
	ages map[string]int
	sync.Mutex
}

func (ua *UserAges) Add(name string, age int) {
	ua.Lock()
	defer ua.Unlock()
	ua.ages[name] = age
}

func (ua *UserAges) Get(name string) int {
	if age, ok := ua.ages[name]; ok {
		return age
	}
	return -1
}
```

**解析：**

 在执行 Get方法时可能被thorw。
 
 虽然有使用sync.Mutex做写锁，但是map是并发读写不安全的。map属于引用类型，并发读写时多个协程见是通过指针访问同一个地址，即访问共享变量，此时同时读写资源存在竞争关系。会报错误信息:“fatal error: concurrent map read and map write”。
 
因此，在 `Get` 中也需要加锁，因为这里只是读，建议使用读写锁 `sync.RWMutex`。


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十五题、下面的迭代会有什么问题？

```go
func (set *threadSafeSet) Iter() <-chan interface{} {
	ch := make(chan interface{})
	go func() {
		set.RLock()

		for elem := range set.s {
			ch <- elem
		}

		close(ch)
		set.RUnlock()

	}()
	return ch
}
```

**解析：**

默认情况下 `make` 初始化的 `channel` 是无缓冲的，也就是在迭代写时会阻塞。


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十六题、以下代码能编译过去吗？为什么？

```go
package main

import (
	"fmt"
)

type People interface {
	Speak(string) string
}

type Student struct{}

func (stu *Student) Speak(think string) (talk string) {
	if think == "bitch" {
		talk = "You are a good boy"
	} else {
		talk = "hi"
	}
	return
}

func main() {
	var peo People = Student{}
	think := "bitch"
	fmt.Println(peo.Speak(think))
}
```

**解析：**


编译失败，值类型 `Student{}` 未实现接口`People`的方法，不能定义为 `People`类型。

在 golang 语言中，`Student` 和 `*Student` 是两种类型，第一个是表示 `Student` 本身，第二个是指向 `Student` 的指针。


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十七题、以下代码打印出来什么内容，说出为什么。。。

```go
package main

import (
	"fmt"
)

type People interface {
	Show()
}

type Student struct{}

func (stu *Student) Show() {

}

func live() People {
	var stu *Student
	return stu
}

func main() {
	if live() == nil {
		fmt.Println("AAAAAAA")
	} else {
		fmt.Println("BBBBBBB")
	}
}
```

**解析：**

跟上一题一样，不同的是`*Student` 的定义后本身没有初始化值，所以 `*Student` 是 `nil`的，但是`*Student` 实现了 `People` 接口，接口不为 `nil`。


