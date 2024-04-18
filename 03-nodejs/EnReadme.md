# node interview questions in blockchain

## Part 1: JS and Node

### 1. What is the output result of the following code?

#### Code snippet 1
```
try {
  throw newError('1')
} catch(error) {
  console.log('error')
}
```
Output result:

```
error
```

#### Code snippet 2

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
Output result:

```
out try catch
```

#### Code snippet three

```
try {
  new Promise(() => {
   throw new Error('new promise throw error');
  });
} catch (error) {
  console.log('error');
}
```

Output result:

```
Error: new promise throw error
```

#### Code snippet four

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

Output result:

```
Start
End
Promise resolved
```

### 2. What does the event loop mechanism do?

The event loop mechanism is used to manage when the callback function of the asynchronous API returns to the main thread for execution.

Node.js uses an asynchronous IO model. The synchronous API is executed in the main thread, the asynchronous API is executed in the thread maintained by the underlying C++, and the callback function of the asynchronous API will also be executed in the main thread.

When a Javascript application is running, when can the callback functions of many asynchronous APIs be called back to the main thread? This is what the event loop mechanism does, managing when the callback function of the asynchronous API returns to the main thread for execution.


### 3. The six stages of EventLoop are those six stages

3.1.Timers
Timers: Callback function (setlnterval, setTimeout) used to store timers.

3.2.Pending callbacks
Pendingcallbacks: Execute callback functions related to the operating system. For example, the callback function that monitors port operations when starting a server-side application is called here.

3.3.idle, prepare
idle, prepare: used internally by the system. (We programmers don’t need to worry about this)

3.4.Poll
Poll: Stores the callback function queue for 1/O operations, such as callback functions for file read and write operations.

Special attention needs to be paid at this stage. If there are callback functions in the event queue, execute them until the queue is cleared, otherwise the event loop will stay in this stage for a while waiting for new callback functions to enter.

But this waiting is not certain, but depends on the following two conditions:

- If there is a calling function to be executed in the setlmmediate queue (check phase). In this case there will be no waiting.
- There is a callback function to be executed in the timers queue, and there will be no waiting in this case. The event loop will move to the check phase, then to the Closingcallbacks phase, and finally from the timers phase to the next loop.

3.5.Check
Check: Stores the callback function of setlmmediate.

3.6.Closingcallbacks
Closingcallbacks: Execute callbacks related to closing events, such as callback functions for closing database connections, etc.

### 3. What are macro tasks and micro tasks?

Like js in the browser, asynchronous code in node is also divided into macro tasks and micro tasks, but the execution order between them is different. Let’s take a look at what macrotasks and microtasks are in Node.

macro task
setlnterval
settimeout
setlmmediate
I/O

microtasks
Promise.then
Promise.catch
Promise.finally
process.nextTick

### 4. Briefly explain the execution sequence of micro tasks and macro tasks

In node, the callback function of microtask is placed in the microtask queue, and the callback function of macrotask is placed in the macrotask queue.

Microtasks have higher priority than macrotasks. When there is an executable callback function in the microtask event queue, the event loop will pause and enter the next stage of the event loop after executing the callback function of the current stage, and will immediately enter the microtask's event queue to start executing the callback function. When the callback function in the microtask queue is executed, the event loop will enter the next segment and start executing the callback function.

There is another point we need to pay special attention to when it comes to microtasks. That is, although nextTick is also a microtask, its priority is higher than other microtasks. When executing a microtask, other microtasks will not start until all callback functions in nextlick are executed.

In general, when the main thread synchronization code is executed, the microtasks will be cleared first (if the microtasks continue to generate microtasks, they will be cleared again), and then go to the next event loop stage. And the execution of microtasks is interspersed among the six stages of the event loop. That is, each time the event loop enters the next stage, it will determine whether the microtask queue is empty. If it is empty, it will enter the next stage. Otherwise, the microtasks will be cleared first. queue.

### 5. Let’s talk about the new features of ES6

Class support, modularization, arrow operators, let/const block scope, string templates, destructuring, parameter default values/indefinite parameters/expanded parameters, for-of traversal, generator, Map/Set, Promise

### 6. What are the methods inherited by js classes?

Prototype chaining method, attribute copying method and constructor application method. In addition, since each object can be a class, these methods can also be used for inheritance of object classes.

prototype chain method

function Animal() {
this.name = 'animal';
}
Animal.prototype.sayName = function(){
alert(this.name);
};

function Person() {}
Person.prototype = Animal.prototype; // Person inherits from animal
Person.prototype.constructor = 'Person'; // Update constructor to person
 
attribute copy method

function Animal() {
this.name = 'animal';
}
Animal.prototype.sayName = function() {
alert(this.name);
};

function Person() {}

for(prop in Animal.prototype) {
Person.prototype[prop] = Animal.prototype[prop];
} //Copy all attributes of the animal to the human side
Person.prototype.constructor = 'Person'; // Update constructor to be person
 
Constructor application method

function Animal() {
this.name = 'animal';
}
Animal.prototype.sayName = function() {
alert(this.name);
};

function Person() {
Animal.call(this); // Apply, call, and bind methods are all available. The subtle differences will be mentioned later.
}

## 7. What is the implementation method of multiple inheritance in js classes?

This is achieved through the attribute copying method in class inheritance. Because when all prototype properties of the parent class are copied, the subclass will naturally have similar behaviors and properties.


## 8. What does this in js point to?

object itself

## 9. What is the difference between apply, call and bind in JS?

All three can apply a function to other objects. Note that it is not the own object. apply, call is to directly execute the function call, bind is to bind, and execution needs to be called again. The difference between apply and call is that apply accepts an array as a parameter, while call accepts an unlimited list of parameters separated by commas.

```
function Person() {
}
Person.prototype.sayName() { alert(this.name); }

var obj = {name: 'michaelqin'}; // Note that this is a normal object, it is not an instance of Person
1) apply
Person.prototype.sayName.apply(obj, [param1, param2, param3]);

2) call
Person.prototype.sayName.call(obj, param1, param2, param3);

3) bind
var sn = Person.prototype.sayName.bind(obj);
sn([param1, param2, param3]); // bind needs to be bound first and then executed
sn(param1, param2, param3); // bind needs to be bound first and then executed
```

## 10. What are caller, callee and arguments respectively?

The relationship between caller and callee is like the relationship between employee and employee, that is, the relationship between calling and being called. Both return function object references. arguments is a list of all parameters of the function, which is an array-like variable.

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

## 11. What is the architecture of node?

It is mainly divided into three layers, application app >> V8 and node built-in architecture >> operating system. V8 is the environment in which node runs, and can be understood as a node virtual machine. The built-in architecture of node can be divided into three layers: core module (javascript implementation) >> c++ binding >> libuv + CAes + http.


## 12. What is the difference between apply, call and bind in js?

All three can apply a function to other objects. Note that it is not the own object. apply, call is to directly execute the function call, bind is to bind, and execution needs to be called again. The difference between apply and call is that apply accepts an array as a parameter, while call accepts an unlimited number of parameter lists separated by commas.

```
function Person() {}
Person.prototype.sayName() { alert(this.name); }

var obj = {name: 'michaelqin'}; // Note that this is a normal object, it is not an instance of Person
1) apply
Person.prototype.sayName.apply(obj, [param1, param2, param3]);

2) call
Person.prototype.sayName.call(obj, param1, param2, param3);

3) bind
var sn = Person.prototype.sayName.bind(obj):
sn([param1, param2, param3]); // bind needs to be bound first and then executed
sn(param1, param2, param3); // bind needs to be bound first and then executed
```

## 13. What are caller, callee and arguments respectively?
The relationship between caller and callee is like the relationship between employer and employee, that is, the relationship between calling and being called. Both return function object references. arguments is a list of all parameters of the function, which is an array-like variable.
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

## 13. The difference between ES MAP and SET

JavaScript's default object representation {} can be regarded as the Map or Dictionary data structure in other languages, that is, a set of key-value pairs. But there is a small problem with JavaScript objects, that is, the keys must be strings. But in fact, Number or other data types are also very reasonable as keys. To solve this problem, the latest ES6 specification introduces a new data type Map.

Map is a structure of key-value pairs and has extremely fast search speed.

Set is similar to Map in that it is also a collection of keys but does not store values. Since keys cannot be repeated, there are no duplicate keys in Set.

## 14. Let’s briefly talk about the functions of set, WeakSet, map and WeakMap.

##### Set:

- Members are unique, ordered and non-duplicate.
- [value, value], the key value and the key name are consistent (or only the key value, no key name).
- Can be traversed, the methods are: add, delete, has.

##### WeakSet:

- Members are objects.
- Members are all weak references and can be recycled by the garbage collection mechanism. They can be used to save DOM nodes and are not likely to cause memory leaks.
- Cannot be traversed, methods include add, delete, and has.

##### Map:

- Essentially a collection of key-value pairs, similar to a collection.
- It can be traversed, and there are many methods to convert it to various data formats.

##### WeakMap:

- Only objects are accepted as key names (except null), and other types of values ​​are not accepted as key names.
- The key name is a weak reference, the key value can be arbitrary, the object pointed to by the key name can be garbage collected, and the key name is invalid at this time.
- Cannot be traversed, methods include get, set, has, delete.

## 15. JS closures, design patterns, etc.
I won’t write the answer here, it’s relatively simple.
