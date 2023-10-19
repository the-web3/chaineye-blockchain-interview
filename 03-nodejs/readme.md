# 区块链中的 node 面试题目

## 第一部分：JS 与 Node

### 1.以下代码输出的结果是什么

#### 代码片段一
```
try {
 throw new Error('1')
} catch(error) {
 console.log('error')
}
```
输出结果：

```
error
```

#### 代码片段二

```
try {
 setTimeout(function() {
  console.log('b');
 }, 0);
} catch (error) {
 console.log('error');
}
console.log('out try catch')
```
输出结果：

```
out try catch
```

#### 代码片段三

```
try {
 new Promise(() => {
  throw new Error('new promise throw error');
 });
} catch (error) {
 console.log('error');
}
```

输出结果：

```
Error: new promise throw error
```

#### 代码片段四

```
console.log('Start');
setTimeout(() => {
console.log('Timeout');
}, 0);
Promise.resolve().then(() => {
console.log('Promise resolved');
});
console.log('End');
```

输出结果：

```
Start
End
Promise resolved
```

### 2.事件循环机制做的是什么事情？

事件循环机制用于管理异步API的回调函数什么时候回到主线程中执行。

Node.js采用的是异步IO模型。同步API在主线程中执行，异步API在底层的C++维护的线程中执行，异步API的回调函数也会在主线程中执行。

在Javascript应用运行时，众多异步API的回调函数什么时候能回到主线程中调用呢？这就是事件环环机制做的事情，管理异步API的回调函数什么时候回到主线程中执行。


### 3. EventLoop 的六个阶段分别是那六个阶段

3.1.Timers
Timers：用于存储定时器的回调函数(setlnterval,setTimeout)。

3.2.Pendingcallbacks
Pendingcallbacks：执行与操作系统相关的回调函数，比如启动服务器端应用时监听端口操作的回调函数就在这里调用。

3.3.idle，prepare
idle，prepare：系统内部使用。(这个我们程序员不用管)

3.4.Poll
Poll：存储1/O操作的回调函数队列，比如文件读写操作的回调函数。

在这个阶段需要特别注意，如果事件队列中有回调函数，则执行它们直到清空队列，否则事件循环将在此阶段停留一段时间以等待新的回调函数进入。

但是对于这个等待并不是一定的，而是取决于以下两个条件：

- 如果setlmmediate队列（check阶段）中存在要执行的调函数。这种情况就不会等待。
- timers队列中存在要执行的回调函数，在这种情况下也不会等待。事件循环将移至check阶段，然后移至Closingcallbacks阶段，并最终从timers阶段进入下一次循环。

3.5.Check
Check：存储setlmmediate的回调函数。

3.6.Closingcallbacks
Closingcallbacks：执行与关闭事件相关的回调，例如关闭数据库连接的回调函数等。

### 3. 宏任务与微任务有那些

跟浏览器中的js一样，node中的异步代码也分为宏任务和微任务，只是它们之间的执行顺序有所区别。我们再来看看Node中都有哪些宏任务和微任务

宏任务
setlnterval
setimeout
setlmmediate
I/O

微任务
Promise.then
Promise.catch
Promise.finally
process.nextTick

### 4. 简单说一下微任务和宏任务的执行顺序

在node中，微任务的回调函数被放置在微任务队列中，宏任务的回调函数被放置在宏任务队列中。

微任务优先级高于宏任务。当微任务事件队列中存在可以执行的回调函数时，事件循环在执行完当前阶段的回调函数后会暂停进入事件循环的下一个阶段，而会立即进入微任务的事件队列中开始执行回调函数，当微任务队列中的回调函数执行完成后，事件循环才会进入到下一个段开始执行回调函数。

对于微任务我们还有个点需要特别注意。那就是虽然nextTick同属于微任务，但是它的优先级是高于其它微任务，在执行微任务时，只有nextlick中的所有回调函数执行完成后才会开始执行其它微任务。

总的来说就是当主线程同步代码执行完毕后会优先清空微任务(如果微任务继续产生微任务则会再次清空)，然后再到下个事件循环阶段。并且微任务的执行是穿插在事件循环六个阶段中间的，也就是每次事件循环进入下个阶段前会判断微任务队列是否为空，为空才会进入下个阶段，否则先清空微任务队列。

### 5. 说说 ES6 有哪些新特性

类的支持，模块化，箭头操作符，let/const块作用域，字符串模板，解构，参数默认值/不定参数/拓展参数, for-of遍历, generator, Map/Set, Promise

### 6. js类继承的方法有哪些

原型链法，属性复制法和构造器应用法. 另外，由于每个对象可以是一个类，这些方法也可以用于对象类的继承．

原型链法

	function Animal() {
		this.name = 'animal';
	}
	Animal.prototype.sayName = function(){
		alert(this.name);
	};

	function Person() {}
	Person.prototype = Animal.prototype; // 人继承自动物
	Person.prototype.constructor = 'Person'; // 更新构造函数为人
 
属性复制法

	function Animal() {
		this.name = 'animal';
	}
	Animal.prototype.sayName = function() {
		alert(this.name);
	};

	function Person() {}

	for(prop in Animal.prototype) {
		Person.prototype[prop] = Animal.prototype[prop];
	} // 复制动物的所有属性到人量边
	Person.prototype.constructor = 'Person'; // 更新构造函数为人
 
构造器应用法

	function Animal() {
		this.name = 'animal';
	}
	Animal.prototype.sayName = function() {
		alert(this.name);
	};

	function Person() {
		Animal.call(this); // apply, call, bind方法都可以．细微区别，后面会提到．
	}

## 7.js类多重继承的实现方法是怎么样的?

就是类继承里边的属性复制法来实现．因为当所有父类的prototype属性被复制后，子类自然拥有类似行为和属性．


## 8.js 里面的 this 指向的是什么

对象本身

## 9. Js 的 apply, call 和 bind 有什么区别?

三者都可以把一个函数应用到其他对象上，注意不是自身对象．apply,call是直接执行函数调用，bind是绑定，执行需要再次调用．apply和call的区别是apply接受数组作为参数，而call是接受逗号分隔的无限多个参数列表

```
	function Person() {
	}
	Person.prototype.sayName() { alert(this.name); }

	var obj = {name: 'michaelqin'}; // 注意这是一个普通对象，它不是Person的实例
	1) apply
	Person.prototype.sayName.apply(obj, [param1, param2, param3]);

	2) call
	Person.prototype.sayName.call(obj, param1, param2, param3);

	3) bind
	var sn = Person.prototype.sayName.bind(obj);
	sn([param1, param2, param3]); // bind需要先绑定，再执行
	sn(param1, param2, param3); // bind需要先绑定，再执行
```

## 10. caller, callee和arguments分别是什么?

caller, callee之间的关系就像是employer和employee之间的关系，就是调用与被调用的关系，二者返回的都是函数对象引用．arguments是函数的所有参数列表，它是一个类数组的变量．

```
	function parent(param1, param2, param3) {
		child(param1, param2, param3);
	}

	function child() {
		console.log(arguments); // { '0': 'mqin1', '1': 'mqin2', '2': 'mqin3' }
		console.log(arguments.callee); // [Function: child]
		console.log(child.caller); // [Function: parent]
	}

	parent('mqin1', 'mqin2', 'mqin3');
```

## 11. node的构架是什么样子的

主要分为三层，应用app >> V8及node内置架构 >> 操作系统. V8是node运行的环境，可以理解为node虚拟机．node内置架构又可分为三层: 核心模块(javascript实现) >> c++绑定 >> libuv + CAes + http.


coming soon

## 第二部分： node 常用后端开发框架

coming soon

## 第三部分： 数据库

coming soon


## 第三部分： node 与 区块链

coming soon


















