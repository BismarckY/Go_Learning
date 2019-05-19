#### Golang Key Notes

Go first searches for package directory inside `**GOROOT**/src` directory and if it doesn’t find the package, then it looks for `**GOPATH**/src`

go install <package>, If a package does not contain file with `main` package declaration, then Go creates a **package archive** (`.a`) file inside `pkg` directory.

`go run` command compiles and executes a program. We know, `go install` command compiles packages and creates binary executable files or package archive files. This is to avoid compilation of package(s) every single time a (*program where these packages are imported*) is compiled . `go install` pre-compiles a package and Go refers to `.a` files.

You can have multiple `init` functions in a file or a package. Order of the execution of `init` function in a file will be according to the order of their appearances.

You can have `init` function anywhere in the package. These `init` functions are called in lexical file name order (**alphabetical order**).

Hence, **main job of** `**init**` **function is to initialize global variables** that can not be initialized in global context. For example, initialization of an array.

![image-20190409151228431](/Users/i327087/Library/Application Support/typora-user-images/image-20190409151228431.png)

*A stack is like a notebook where Go compiler writes down* `deferred functions` *to execute at the end of current function execution. This stack follows* `Last In First Out (LIFO)` *order of execution. Which means, any task pushed first, will execute in the end.*

A function in Go is also a type. If two function accept same parameters and return same values, then these two functions are of same type.

## Since there is no class-object architecture and closest thing to class we have is structure. Function with struct receiver is a way to achieve methods in go.

Syntax for defining a method is

```go
func (r Type) functionName(...Type) Type {
    ...
}
```

On major difference between function and method is many methods can have same name while no two functions with same name can be defined in a package.

#### Method can accept both pointer and value

When a function has value argument, it will only accept value of the parameter. If you passed a pointer to the function which expects a value, it will not work. This is also true when function accepts pointer, you simply can not pass value to it.

In case of method, that’s not the case. We can define method with value or pointer receive and call it on pointer or value.

## Slice

 **slice is just a reference to an array**

Syntax to define a slice is pretty similar to that of an `array` **but without specifying the elements count**. Hence `s` is an an slice

```go
var s []int
```

But in case of `slice`, zero value of slice defined like above is `nil`. Below program will return `true`.

![image-20190409163901835](/Users/i327087/Library/Application Support/typora-user-images/image-20190409163901835.png)

**Process & Thread**

While executing code, thread store variables (*data*) inside memory region known as `stack` which `scratch space` where variables hold temporary space. A stack is created at compile time and is normally of a fixed size, preferably 1-2 MB. While stack of a thread can be used by only that thread and will not be shared with other thread. A heap is a property of a process and it is available to use by any thread. Heap is a shared memory space where data from one thread can be access by other threads as well.

goroutines behave like threads but technically; it is a abstraction over threads.

When we run a go program, go **runtime** will create few threads on a core on which all the goroutines are multiplexed (*spawned*). At any point in time, one thread will be executing one goroutine and if that goroutine is blocked, then it will be swapped out for another goroutine that will execute on that thread instead. This is like **thread scheduling** but handled by **go runtime** and this is much faster.

It is advised in most of the cases, to run all your goroutines on one core but if you need to divide goroutines among available CPU cores of your system, you can use GOMAXPROCS environment variable or call to runtime using function `runtime.GOMAXPROCS(n)` where `n` is number of cores to use. But you may sometime feel that setting `GOMAXPROCS > 1` is making your program slower. It truly depends on the nature of your program but you can find solution or explanation of your problem on internet. In practical terms, programs that spend more time communicating on channels than doing computation will experience performance degradation when using multiple cores, OS threads and processes.

Go has an `M:N` scheduler that can also utilize multiple processors. At any time, `M` goroutines need to be scheduled on `N` OS threads that runs on at most on `GOMAXPROCS` numbers of processors. At any time, at most only one thread is allowed to run per core. But scheduler can create more threads if required, but that rarely happens. If your program doesn’t start any additional goroutines, it will naturally run in only one thread no matter how many cores you allow it to use.



**MAP**

But the thing about `nil` map is, we can’t add values to it because like slices, `map` can not hold any data, rather they references internal data structure that holds the data. So, in case of `nil` map, internal data structure is missing and assigning any data to it will case runtime panic `panic: assignment to entry in nil map`. You can use `nil` map as a variable to store another non `nil` map.

In case `array` or `slice`, when you are trying to access out of index element (*when index does not exist*), go will throw an error. But not in case of `map`. When you are trying to access value by the `key` which is not present in the map, go will not throw an error, instead it will return `zero` value of `valueType`.

So, to check if a key exist in the map or not, go provide another syntax which returns 2 values.

```go
value, ok := m[key]
```

can use `len` function of Map. but *There is nothing like capacity in* `map` *because go completely takes control of internal data structure of* `map`*. Hence don't try to use* `cap` *function on* `map`*.*

#### Delete `map` element

Unlike `slice` where you need to use a **hack** to delete an element, go provide easier function `delete` to delete `map` element. Syntax of a delete function is as following

```go
func delete(m map[Type]Type1, key Type)
```

If the key does not exist in the map,  go will not throw an error while executing* `delete` *function.*

#### ☛ Maps comparison

Like `slice`, map can be only compared with `nil`. If you are thinking to iterate over a map and match each element, you are in grave trouble. But if you are in dire need to compare two maps, use `reflect` package’s `DeepEqual`function (<https://golang.org/pkg/reflect/>).

*Order of retrieval of elements in map is random when used in* `for` *iteration. Hence there is no guarantee that each time, they will be in order. That explains why we can’t compare two maps.*

 All **comparable types** such as boolean, integer, float, complex, string etc. can also be keys. 

#### Maps are referenced type

Like `slice`, map references internal data structure. When, you copy a map to new map, the internal data structure is not copied, just referenced.

you can use `%c` format string in `Printf` statement. You can also use `%v` format string to see byte value and `%T` to see data type of the value.

**Chan- Channel**

A channel can transport data of **only one data type**. No other data types are allowed to be transported from that channel.

```
Var C chan int
C := make (chan int)
```

 Channels by default are **pointers** Mostly, when you want to communicate with a goroutine, you pass the channel as argument to the function or method. Hence when goroutine receives that channel as argument, you don’t need to dereference it to push or pull data from that channel.

When some data is written to the channel, goroutine is blocked until some other goroutine reads it from that channel. At the same time, as we seen in `concurrency chapter`, channel operations tells scheduler to schedule another goroutine, that’s why program doesn’t block forever on same goroutine. These features of channel is very useful in goroutines communication as it prevent us writing manual locks and hacks to make them work with each other.

When the buffer size is non-zero `n`, **goroutine is not blocked until after buffer is full**.

By default channels are *unbuffered*, meaning that they will only accept sends (`chan <-`) if there is a corresponding receive (`<- chan`) ready to receive the sent value. *Buffered channels* accept a limited number of values without a corresponding receiver for those values.

# 极客时间Notes

代码包的名称一般会与源码文件所在的目录同名。如果不同名，那么在构建、安装的过程中会以代码包名称为准。

在工作区中，一个代码包的导入路径实际上就是从 src 子目录...

一般情况下，Go 语言的源码文件都需要被存放在环境变量 GOPATH 包含的某个工作区（目录）中的 src 目录下的某个代码包（目录）中。

那么在安装后如果产生了归档文件（以“.a”为扩展名的文件），就会放进该工作区的pkg子目录；如果产生了可执行文件，就放进工作区的bin子目录。

![image-20190514213759760](/Users/i327087/Library/Application Support/typora-user-images/image-20190514213759760.png)

类型推断与短变量声明

![image-20190514215807829](/Users/i327087/Library/Application Support/typora-user-images/image-20190514215807829.png)





​	

