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

## 第五题：字符串替换问题

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

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第二十八题、在 golang 协程和channel配合使用

> 写代码实现两个 goroutine，其中一个产生随机数并写入到 go channel 中，另外一个从 channel 中读取数字并打印到标准输出。最终输出五个随机数。

**解析**

这是一道很简单的golang基础题目，实现方法也有很多种，一般想答让面试官满意的答案还是有几点注意的地方。

1. `goroutine` 在golang中式非阻塞的
2. `channel` 无缓冲情况下，读写都是阻塞的，且可以用`for`循环来读取数据，当管道关闭后，`for` 退出。
3.  golang 中有专用的`select case` 语法从管道读取数据。

示例代码如下：

```go
func main() {
    out := make(chan int)
    wg := sync.WaitGroup{}
    wg.Add(2)
    go func() {
        defer wg.Done()
        for i := 0; i < 5; i++ {
            out <- rand.Intn(5)
        }
        close(out)
    }()
    go func() {
        defer wg.Done()
        for i := range out {
            fmt.Println(i)
        }
    }()
    wg.Wait()
}
```

如果不想使用 `sync.WaitGroup`, 也可以用一个 `done` channel.
```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	random := make(chan int)
	done := make(chan bool)

	go func() {
		for {
			num, ok := <-random
			if ok {
				fmt.Println(num)
			} else {
				done <- true
			}
		}
	}()

	go func() {
		defer close(random)

		for i := 0; i < 5; i++ {
			random <- rand.Intn(5)
		}
	}()

	<-done
	close(done)
}
```

## 第二十九题、实现阻塞读且并发安全的map

GO里面MAP如何实现key不存在 get操作等待 直到key存在或者超时，保证并发安全，且需要实现以下接口：

```go
type sp interface {
    Out(key string, val interface{})  //存入key /val，如果该key读取的goroutine挂起，则唤醒。此方法不会阻塞，时刻都可以立即执行并返回
    Rd(key string, timeout time.Duration) interface{}  //读取一个key，如果key不存在阻塞，等待key存在或者超时
}
```

**解析：**

看到阻塞协程第一个想到的就是`channel`，题目中要求并发安全，那么必须用锁，还要实现多个`goroutine`读的时候如果值不存在则阻塞，直到写入值，那么每个键值需要有一个阻塞`goroutine` 的 `channel`。

[实现如下：](../src/q010.go) 

```go
type Map struct {
	c   map[string]*entry
	rmx *sync.RWMutex
}
type entry struct {
	ch      chan struct{}
	value   interface{}
	isExist bool
}

func (m *Map) Out(key string, val interface{}) {
	m.rmx.Lock()
	defer m.rmx.Unlock()
	item, ok := m.c[key]
	if !ok {
		m.c[key] = &entry{
			value: val,
			isExist: true,
		}
		return
	}
	item.value = val
	if !item.isExist {
		if item.ch != nil {
			close(item.ch)
			item.ch = nil
		}
	}
	return
}
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第三十题、高并发下的锁与 map 的读写

场景：在一个高并发的web服务器中，要限制IP的频繁访问。现模拟100个IP同时并发访问服务器，每个IP要重复访问1000次。

每个IP三分钟之内只能访问一次。修改以下代码完成该过程，要求能成功输出 success:100

```go
package main
 
import (
	"fmt"
	"time"
)
 
type Ban struct {
	visitIPs map[string]time.Time
}
 
func NewBan() *Ban {
	return &Ban{visitIPs: make(map[string]time.Time)}
}
func (o *Ban) visit(ip string) bool {
	if _, ok := o.visitIPs[ip]; ok {
		return true
	}
	o.visitIPs[ip] = time.Now()
	return false
}
func main() {
	success := 0
	ban := NewBan()
	for i := 0; i < 1000; i++ {
		for j := 0; j < 100; j++ {
			go func() {
				ip := fmt.Sprintf("192.168.1.%d", j)
				if !ban.visit(ip) {
					success++
				}
			}()
		}
 
	}
	fmt.Println("success:", success)
}
```

**解析**

该问题主要考察了并发情况下map的读写问题，而给出的初始代码，又存在`for`循环中启动`goroutine`时变量使用问题以及`goroutine`执行滞后问题。

因此，首先要保证启动的`goroutine`得到的参数是正确的，然后保证`map`的并发读写，最后保证三分钟只能访问一次。

多CPU核心下修改`int`的值极端情况下会存在不同步情况，因此需要原子性的修改int值。

下面给出的实例代码，是启动了一个协程每分钟检查一下`map`中的过期`ip`，`for`启动协程时传参。

```go
package main

import (
	"context"
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)

type Ban struct {
	visitIPs map[string]time.Time
	lock      sync.Mutex
}

func NewBan(ctx context.Context) *Ban {
	o := &Ban{visitIPs: make(map[string]time.Time)}
	go func() {
		timer := time.NewTimer(time.Minute * 1)
		for {
			select {
			case <-timer.C:
				o.lock.Lock()
				for k, v := range o.visitIPs {
					if time.Now().Sub(v) >= time.Minute*1 {
						delete(o.visitIPs, k)
					}
				}
				o.lock.Unlock()
				timer.Reset(time.Minute * 1)
			case <-ctx.Done():
				return
			}
		}
	}()
	return o
}
func (o *Ban) visit(ip string) bool {
	o.lock.Lock()
	defer o.lock.Unlock()
	if _, ok := o.visitIPs[ip]; ok {
		return true
	}
	o.visitIPs[ip] = time.Now()
	return false
}
func main() {
	success := int64(0)
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	ban := NewBan(ctx)

	wait := &sync.WaitGroup{}

	wait.Add(1000 * 100)
	for i := 0; i < 1000; i++ {
		for j := 0; j < 100; j++ {
			go func(j int) {
				defer wait.Done()
				ip := fmt.Sprintf("192.168.1.%d", j)
				if !ban.visit(ip) {
					atomic.AddInt64(&success, 1)
				}
			}(j)
		}

	}
	wait.Wait()

	fmt.Println("success:", success)
}
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第三十一题、写出以下逻辑，要求每秒钟调用一次proc并保证程序不退出?

```go
package main

func main() {
    go func() {
        // 1 在这里需要你写算法
        // 2 要求每秒钟调用一次proc函数
        // 3 要求程序不能退出
    }()

    select {}
}

func proc() {
    panic("ok")
}
```

**解析**

题目主要考察了两个知识点：

1. 定时执行执行任务
2. 捕获 panic 错误

题目中要求每秒钟执行一次，首先想到的就是 `time.Ticker`对象，该函数可每秒钟往`chan`中放一个`Time`,正好符合我们的要求。

在 `golang` 中捕获 `panic` 一般会用到 `recover()` 函数。  

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	go func() {
		// 1 在这里需要你写算法
		// 2 要求每秒钟调用一次proc函数
		// 3 要求程序不能退出

		t := time.NewTicker(time.Second * 1)
		for {
			select {
			case <-t.C:
				go func() {
					defer func() {
						if err := recover(); err != nil {
							fmt.Println(err)
						}
					}()
					proc()
				}()
			}
		}
	}()

	select {}
}

func proc() {
	panic("ok")
}

```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第三十二题、为 sync.WaitGroup 中Wait函数支持 WaitTimeout 功能.

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    wg := sync.WaitGroup{}
    c := make(chan struct{})
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(num int, close <-chan struct{}) {
            defer wg.Done()
            <-close
            fmt.Println(num)
        }(i, c)
    }

    if WaitTimeout(&wg, time.Second*5) {
        close(c)
        fmt.Println("timeout exit")
    }
    time.Sleep(time.Second * 10)
}

func WaitTimeout(wg *sync.WaitGroup, timeout time.Duration) bool {
    // 要求手写代码
    // 要求sync.WaitGroup支持timeout功能
    // 如果timeout到了超时时间返回true
    // 如果WaitGroup自然结束返回false
}
```


**解析**

首先 `sync.WaitGroup` 对象的 `Wait` 函数本身是阻塞的，同时，超时用到的`time.Timer` 对象也需要阻塞的读。

同时阻塞的两个对象肯定要每个启动一个协程,每个协程去处理一个阻塞，难点在于怎么知道哪个阻塞先完成。

目前我用的方式是声明一个没有缓冲的`chan`，谁先完成谁优先向管道中写入数据。

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	wg := sync.WaitGroup{}
	c := make(chan struct{})
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go func(num int, close <-chan struct{}) {
			defer wg.Done()
			<-close
			fmt.Println(num)
		}(i, c)
	}

	if WaitTimeout(&wg, time.Second*5) {
		close(c)
		fmt.Println("timeout exit")
	}
	time.Sleep(time.Second * 10)
}

func WaitTimeout(wg *sync.WaitGroup, timeout time.Duration) bool {
	// 要求手写代码
	// 要求sync.WaitGroup支持timeout功能
	// 如果timeout到了超时时间返回true
	// 如果WaitGroup自然结束返回false
	ch := make(chan bool, 1)

	go time.AfterFunc(timeout, func() {
		ch <- true
	})

	go func() {
		wg.Wait()
		ch <- false
	}()
	
	return <- ch
}
```


## 第三十三题、对已经关闭的的chan进行读写，会怎么样？为什么？

## 题目

对已经关闭的的 chan 进行读写，会怎么样？为什么？

## 回答

- 读已经关闭的 chan 能一直读到东西，但是读到的内容根据通道内关闭前是否有元素而不同。
    - 如果 chan 关闭前，buffer 内有元素还未读 , 会正确读到 chan 内的值，且返回的第二个 bool 值（是否读成功）为 true。
    - 如果 chan 关闭前，buffer 内有元素已经被读完，chan 内无值，接下来所有接收的值都会非阻塞直接成功，返回 channel 元素的零值，但是第二个 bool 值一直为 false。
- 写已经关闭的 chan 会 panic


## 示例

### 1. 写已经关闭的 chan

```go
func main(){
    c := make(chan int,3)
    close(c)
    c <- 1
}
//输出结果
panic: send on closed channel

goroutine 1 [running]
main.main()
...
```

- 注意这个 send on closed channel，待会会提到。

### 2. 读已经关闭的 chan

```go
package main
import "fmt"

func main()  {
    fmt.Println("以下是数值的chan")
    ci:=make(chan int,3)
    ci<-1
    close(ci)
    num,ok := <- ci
    fmt.Printf("读chan的协程结束，num=%v， ok=%v\n",num,ok)
    num1,ok1 := <-ci
    fmt.Printf("再读chan的协程结束，num=%v， ok=%v\n",num1,ok1)
    num2,ok2 := <-ci
    fmt.Printf("再再读chan的协程结束，num=%v， ok=%v\n",num2,ok2)
    
    fmt.Println("以下是字符串chan")
    cs := make(chan string,3)
    cs <- "aaa"
    close(cs)
    str,ok := <- cs
    fmt.Printf("读chan的协程结束，str=%v， ok=%v\n",str,ok)
    str1,ok1 := <-cs
    fmt.Printf("再读chan的协程结束，str=%v， ok=%v\n",str1,ok1)
    str2,ok2 := <-cs
    fmt.Printf("再再读chan的协程结束，str=%v， ok=%v\n",str2,ok2)

    fmt.Println("以下是结构体chan")
    type MyStruct struct{
        Name string
    }
    cstruct := make(chan MyStruct,3)
    cstruct <- MyStruct{Name: "haha"}
    close(cstruct)
    stru,ok := <- cstruct
    fmt.Printf("读chan的协程结束，stru=%v， ok=%v\n",stru,ok)
    stru1,ok1 := <-cs
    fmt.Printf("再读chan的协程结束，stru=%v， ok=%v\n",stru1,ok1)
    stru2,ok2 := <-cs
    fmt.Printf("再再读chan的协程结束，stru=%v， ok=%v\n",stru2,ok2)
}

```

输出结果

```bash
以下是数值的chan
读chan的协程结束，num=1， ok=true
再读chan的协程结束，num=0， ok=false
再再读chan的协程结束，num=0， ok=false
以下是字符串chan
读chan的协程结束，str=aaa， ok=true
再读chan的协程结束，str=， ok=false
再再读chan的协程结束，str=， ok=false
以下是结构体chan
读chan的协程结束，stru={haha}， ok=true
再读chan的协程结束，stru=， ok=false
再再读chan的协程结束，stru=， ok=false
```


## 多问一句

### 1. 为什么写已经关闭的 `chan` 就会 `panic` 呢？

```go
//在 src/runtime/chan.go
func chansend(c *hchan,ep unsafe.Pointer,block bool,callerpc uintptr) bool {
    //省略其他
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("send on closed channel"))
    }   
    //省略其他
}
```

- 当 `c.closed != 0` 则为通道关闭，此时执行写，源码提示直接 `panic`，输出的内容就是上面提到的 `"send on closed channel"`。

### 2. 为什么读已关闭的 chan 会一直能读到值？

```go
func chanrecv(c *hchan,ep unsafe.Pointer,block bool) (selected,received bool) {
    //省略部分逻辑
    lock(&c.lock)
    //当chan被关闭了，而且缓存为空时
    //ep 是指 val,ok := <-c 里的val地址
    if c.closed != 0 && c.qcount == 0 {
        if receenabled {
            raceacquire(c.raceaddr())
        }
        unlock(&c.lock)
        //如果接受之的地址不空，那接收值将获得一个该值类型的零值
        //typedmemclr 会根据类型清理响应的内存
        //这就解释了上面代码为什么关闭的chan 会返回对应类型的零值
        if ep != null {
            typedmemclr(c.elemtype,ep)
        }   
        //返回两个参数 selected,received
        // 第二个采纳数就是 val,ok := <- c 里的 ok
        //也就解释了为什么读关闭的chan会一直返回false
        return true,false
    }   
}
```
- `c.closed != 0 && c.qcount == 0` 指通道已经关闭，且缓存为空的情况下（已经读完了之前写到通道里的值）
- 如果接收值的地址 `ep` 不为空
    - 那接收值将获得是一个该类型的零值
    - `typedmemclr` 会根据类型清理相应地址的内存
    - 这就解释了上面代码为什么关闭的 chan 会返回对应类型的零值

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第三十四题、简单聊聊内存逃逸？

### 问题

知道golang的内存逃逸吗？什么情况下会发生内存逃逸？

### 回答

golang程序变量会携带有一组校验数据，用来证明它的整个生命周期是否在运行时完全可知。如果变量通过了这些校验，它就可以在栈上分配。否则就说它 逃逸 了，必须在堆上分配。

能引起变量逃逸到堆上的典型情况：

- **在方法内把局部变量指针返回** 局部变量原本应该在栈中分配，在栈中回收。但是由于返回时被外部引用，因此其生命周期大于栈，则溢出。
- **发送指针或带有指针的值到 channel 中。**  在编译时，是没有办法知道哪个 `goroutine` 会在 `channel` 上接收数据。所以编译器没法知道变量什么时候才会被释放。
- **在一个切片上存储指针或带指针的值。** 一个典型的例子就是 `[]*string` 。这会导致切片的内容逃逸。尽管其后面的数组可能是在栈上分配的，但其引用的值一定是在堆上。
- **slice 的背后数组被重新分配了，因为 append 时可能会超出其容量( cap )。** slice 初始化的地方在编译时是可以知道的，它最开始会在栈上分配。如果切片背后的存储要基于运行时的数据进行扩充，就会在堆上分配。
- **在 interface 类型上调用方法。**  在 interface 类型上调用方法都是动态调度的 —— 方法的真正实现只能在运行时知道。想像一个 io.Reader 类型的变量 r , 调用 r.Read(b) 会使得 r 的值和切片b 的背后存储都逃逸掉，所以会在堆上分配。

### 举例

**通过一个例子加深理解，接下来尝试下怎么通过 `go build -gcflags=-m` 查看逃逸的情况。**

```go
package main
import "fmt"
type A struct {
 s string
}
// 这是上面提到的 "在方法内把局部变量指针返回" 的情况
func foo(s string) *A {
 a := new(A) 
 a.s = s
 return a //返回局部变量a,在C语言中妥妥野指针，但在go则ok，但a会逃逸到堆
}
func main() {
 a := foo("hello")
 b := a.s + " world"
 c := b + "!"
 fmt.Println(c)
}
```

执行go build -gcflags=-m main.go

```bash
go build -gcflags=-m main.go
# command-line-arguments
./main.go:7:6: can inline foo
./main.go:13:10: inlining call to foo
./main.go:16:13: inlining call to fmt.Println
/var/folders/45/qx9lfw2s2zzgvhzg3mtzkwzc0000gn/T/go-build409982591/b001/_gomod_.go:6:6: can inline init.0
./main.go:7:10: leaking param: s
./main.go:8:10: new(A) escapes to heap
./main.go:16:13: io.Writer(os.Stdout) escapes to heap
./main.go:16:13: c escapes to heap
./main.go:15:9: b + "!" escapes to heap
./main.go:13:10: main new(A) does not escape
./main.go:14:11: main a.s + " world" does not escape
./main.go:16:13: main []interface {} literal does not escape
<autogenerated>:1: os.(*File).close .this does not escape
```

- `./main.go:8:10: new(A) escapes to heap` 说明 `new(A)` 逃逸了,符合上述提到的常见情况中的第一种。
- `./main.go:14:11: main a.s + " world" does not escape` 说明 b 变量没有逃逸，因为它只在方法内存在，会在方法结束时被回收。
- `./main.go:15:9: b + "!" escapes to heap` 说明 c 变量逃逸，通过`fmt.Println(a ...interface{})`打印的变量，都会发生逃逸，感兴趣的朋友可以去查查为什么。

以上操作其实就叫逃逸分析。下篇文章，跟大家聊聊怎么用一个比较trick的方法使变量不逃逸。方便大家在面试官面前秀一波。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------


## 第三十五题、 字符串转成byte数组，会发生内存拷贝吗？

### 问题

字符串转成byte数组，会发生内存拷贝吗？

### 回答

字符串转成切片，会产生拷贝。严格来说，只要是发生类型强转都会发生内存拷贝。那么问题来了。

频繁的内存拷贝操作听起来对性能不大友好。有没有什么办法可以在字符串转成切片的时候不用发生拷贝呢？

### 解释

```go
package main

import (
 "fmt"
 "reflect"
 "unsafe"
)

func main() {
 a :="aaa"
 ssh := *(*reflect.StringHeader)(unsafe.Pointer(&a))
 b := *(*[]byte)(unsafe.Pointer(&ssh))  
 fmt.Printf("%v",b)
}
```

**`StringHeader` 是字符串在go的底层结构。**

```go
type StringHeader struct {
 Data uintptr
 Len  int
}
```

**`SliceHeader` 是切片在go的底层结构。**

```go
type SliceHeader struct {
 Data uintptr
 Len  int
 Cap  int
}
```

那么如果想要在底层转换二者，只需要把 StringHeader 的地址强转成 SliceHeader 就行。那么go有个很强的包叫 unsafe 。

1. `unsafe.Pointer(&a)`方法可以得到变量a的地址。
2. `(*reflect.StringHeader)(unsafe.Pointer(&a))` 可以把字符串a转成底层结构的形式。
3. `(*[]byte)(unsafe.Pointer(&ssh))` 可以把ssh底层结构体转成byte的切片的指针。
4. 再通过 `*`转为指针指向的实际内容。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第三十六题、 http包的内存泄漏

### 问题

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"runtime"
)

func main() {
	num := 6
	for index := 0; index < num; index++ {
		resp, _ := http.Get("https://www.baidu.com")
		_, _ = ioutil.ReadAll(resp.Body)
	}
	fmt.Printf("此时goroutine个数= %d\n", runtime.NumGoroutine())


}
```

上面这道题在不执行`resp.Body.Close()`的情况下，泄漏了吗？如果泄漏，泄漏了多少个goroutine?

### 怎么答

不进行resp.Body.Close()，泄漏是一定的。但是泄漏的goroutine个数就让我迷糊了。由于执行了6遍，每次泄漏一个读和写goroutine，就是12个goroutine，加上main函数本身也是一个goroutine，所以答案是13.
然而执行程序，发现答案是3，出入有点大，为什么呢？

### 解释

我们直接看源码。golang 的 http 包。

```go
http.Get()

-- DefaultClient.Get
----func (c *Client) do(req *Request)
------func send(ireq *Request, rt RoundTripper, deadline time.Time)
-------- resp, didTimeout, err = send(req, c.transport(), deadline) 
// 以上代码在 go/1.12.7/libexec/src/net/http/client:174 

func (c *Client) transport() RoundTripper {
	if c.Transport != nil {
		return c.Transport
	}
	return DefaultTransport
}
```

- 说明 `http.Get` 默认使用 `DefaultTransport` 管理连接。

DefaultTransport 是干嘛的呢？

```go
// It establishes network connections as needed
// and caches them for reuse by subsequent calls.
```

- `DefaultTransport` 的作用是根据需要建立网络连接并缓存它们以供后续调用重用。

那么 `DefaultTransport` 什么时候会建立连接呢？

接着上面的代码堆栈往下翻

```go
func send(ireq *Request, rt RoundTripper, deadline time.Time) 
--resp, err = rt.RoundTrip(req) // 以上代码在 go/1.12.7/libexec/src/net/http/client:250
func (t *Transport) RoundTrip(req *http.Request)
func (t *Transport) roundTrip(req *Request)
func (t *Transport) getConn(treq *transportRequest, cm connectMethod)
func (t *Transport) dialConn(ctx context.Context, cm connectMethod) (*persistConn, error) {
    ...
	go pconn.readLoop()  // 启动一个读goroutine
	go pconn.writeLoop() // 启动一个写goroutine
	return pconn, nil
}
```

- 一次建立连接，就会启动一个读goroutine和写goroutine。这就是为什么一次`http.Get()`会泄漏两个goroutine的来源。
- 泄漏的来源知道了，也知道是因为没有执行close

**那为什么不执行 close 会泄漏呢？**

回到刚刚启动的读goroutine 的 `readLoop()` 代码里

```go
func (pc *persistConn) readLoop() {
	alive := true
	for alive {
        ...
		// Before looping back to the top of this function and peeking on
		// the bufio.Reader, wait for the caller goroutine to finish
		// reading the response body. (or for cancelation or death)
		select {
		case bodyEOF := <-waitForBodyRead:
			pc.t.setReqCanceler(rc.req, nil) // before pc might return to idle pool
			alive = alive &&
				bodyEOF &&
				!pc.sawEOF &&
				pc.wroteRequest() &&
				tryPutIdleConn(trace)
			if bodyEOF {
				eofc <- struct{}{}
			}
		case <-rc.req.Cancel:
			alive = false
			pc.t.CancelRequest(rc.req)
		case <-rc.req.Context().Done():
			alive = false
			pc.t.cancelRequest(rc.req, rc.req.Context().Err())
		case <-pc.closech:
			alive = false
        }
        ...
	}
}
```

其中第一个 body 被读取完或关闭这个 case:

```go
alive = alive &&
    bodyEOF &&
    !pc.sawEOF &&
    pc.wroteRequest() &&
    tryPutIdleConn(trace)

```

bodyEOF 来源于到一个通道 waitForBodyRead，这个字段的 true 和 false 直接决定了 alive 变量的值（alive=true那读goroutine继续活着，循环，否则退出goroutine）。

**那么这个通道的值是从哪里过来的呢？**


```go
// go/1.12.7/libexec/src/net/http/transport.go: 1758
		body := &bodyEOFSignal{
			body: resp.Body,
			earlyCloseFn: func() error {
				waitForBodyRead <- false
				<-eofc // will be closed by deferred call at the end of the function
				return nil

			},
			fn: func(err error) error {
				isEOF := err == io.EOF
				waitForBodyRead <- isEOF
				if isEOF {
					<-eofc // see comment above eofc declaration
				} else if err != nil {
					if cerr := pc.canceled(); cerr != nil {
						return cerr
					}
				}
				return err
			},
		}
```

- 如果执行 earlyCloseFn ，waitForBodyRead 通道输入的是 false，alive 也会是 false，那 readLoop() 这个 goroutine 就会退出。
- 如果执行 fn ，其中包括正常情况下 body 读完数据抛出 io.EOF 时的 case，waitForBodyRead 通道输入的是 true，那 alive 会是 true，那么 readLoop() 这个 goroutine 就不会退出，同时还顺便执行了 tryPutIdleConn(trace) 。

```go
// tryPutIdleConn adds pconn to the list of idle persistent connections awaiting
// a new request.
// If pconn is no longer needed or not in a good state, tryPutIdleConn returns
// an error explaining why it wasn't registered.
// tryPutIdleConn does not close pconn. Use putOrCloseIdleConn instead for that.
func (t *Transport) tryPutIdleConn(pconn *persistConn) error
```

- tryPutIdleConn 将 pconn 添加到等待新请求的空闲持久连接列表中，也就是之前说的连接会复用。

那么问题又来了，什么时候会执行这个 `fn` 和 `earlyCloseFn` 呢？

```go
func (es *bodyEOFSignal) Close() error {
	es.mu.Lock()
	defer es.mu.Unlock()
	if es.closed {
		return nil
	}
	es.closed = true
	if es.earlyCloseFn != nil && es.rerr != io.EOF {
		return es.earlyCloseFn() // 关闭时执行 earlyCloseFn
	}
	err := es.body.Close()
	return es.condfn(err)
}
```

- 上面这个其实就是我们比较收悉的 resp.Body.Close() ,在里面会执行 earlyCloseFn，也就是此时 readLoop() 里的 waitForBodyRead 通道输入的是 false，alive 也会是 false，那 readLoop() 这个 goroutine 就会退出，goroutine 不会泄露。
  
```go
b, err = ioutil.ReadAll(resp.Body)
--func ReadAll(r io.Reader) 
----func readAll(r io.Reader, capacity int64) 
------func (b *Buffer) ReadFrom(r io.Reader)


// go/1.12.7/libexec/src/bytes/buffer.go:207
func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error) {
	for {
		...
		m, e := r.Read(b.buf[i:cap(b.buf)])  // 看这里，是body在执行read方法
		...
	}
}
```

- 这个`read`，其实就是 `bodyEOFSignal` 里的

```go
func (es *bodyEOFSignal) Read(p []byte) (n int, err error) {
	...
	n, err = es.body.Read(p)
	if err != nil {
		... 
    // 这里会有一个io.EOF的报错，意思是读完了
		err = es.condfn(err)
	}
	return
}


func (es *bodyEOFSignal) condfn(err error) error {
	if es.fn == nil {
		return err
	}
	err = es.fn(err)  // 这了执行了 fn
	es.fn = nil
	return err
}
```

- 上面这个其实就是我们比较收悉的读取 body 里的内容。 ioutil.ReadAll() ,在读完 body 的内容时会执行 fn，也就是此时 readLoop() 里的 waitForBodyRead 通道输入的是 true，alive 也会是 true，那 readLoop() 这个 goroutine 就不会退出，goroutine 会泄露，然后执行 tryPutIdleConn(trace) 把连接放回池子里复用。
  
### 总结

- 所以结论呼之欲出了，虽然执行了 6 次循环，而且每次都没有执行 Body.Close() ,就是因为执行了ioutil.ReadAll()把内容都读出来了，连接得以复用，因此只泄漏了一个读goroutine和一个写goroutine，最后加上main goroutine，所以答案就是3个goroutine。
- 从另外一个角度说，正常情况下我们的代码都会执行 ioutil.ReadAll()，但如果此时忘了 resp.Body.Close()，确实会导致泄漏。但如果你调用的域名一直是同一个的话，那么只会泄漏一个 读goroutine 和一个写goroutine，这就是为什么代码明明不规范但却看不到明显内存泄漏的原因。
- 那么问题又来了，为什么上面要特意强调是同一个域名呢？改天，回头，以后有空再说吧。

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## 第三十七题 sync.Map 的用法

### 问题

```go
package main

import (
	"fmt"
	"sync"
)

func main(){
	var m sync.Map
	m.Store("address",map[string]string{"province":"江苏","city":"南京"})
        v,_ := m.Load("address")
	fmt.Println(v["province"]) 
}
```

- A，江苏；
- B`，v["province"]`取值错误；
- C，`m.Store`存储错误；
- D，不知道

## 解析

`invalid operation: v["province"] (type interface {} does not support indexing)`
因为 `func (m *Map) Store(key interface{}, value interface{})`
所以 `v`类型是 `interface {}` ，这里需要一个类型断言

```go
fmt.Println(v.(map[string]string)["province"]) //江苏
```


# 注意：golang 的所有面试题目来自网络，并非 The Web3 社区原创，若有违权，请联系删除。

