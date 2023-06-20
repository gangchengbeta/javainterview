## GO学习笔记

### 字符串
#### Q1. 什么是string
字符串的本质是**不可修改的byte数组**
#### Q2. string 的遍历

```go
func LearnString() {
	s := "GOLANG"
	fmt.Println(s)
	for i, ele := range s {
		fmt.Printf("%d %c\n", i, ele)
	}
	// s[0] = `f`   cannot assign to s[0] (value of type byte)
	fmt.Println(len(s)) // 6
	a := "golang你好"
	// 一般来说 一个汉字占3个字节
	fmt.Println(len(a))   // 12
	arr := []rune(a)      // 将string转换为rune切片 可以求得正确的长度
	fmt.Println(len(arr)) // 8
	for _, ele := range a {
		fmt.Printf("%c\n", ele)
	}
}
```

#### Q3. string 的拼接

```go
	// 字符串的拼接
	// 1. 使用+号 
	c := s + a
	fmt.Println(c)
	// 2. 使用fmt.Sprintf
	d := fmt.Sprint(s, a)
	fmt.Println(d)
	// 3. 使用strings.Join
	e := strings.Join([]string{s, a}, "")
	fmt.Println(e)
```



![image-20230620113724079](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com/java/image-20230620113724079.png)

### 指针


#### Q1. 什么是指针

>在计算机科学中，指针（Pointer）是一种变量，它存储了一个内存地址的值。简单来说，指针是一个用于存储内存地址的变量。这个地址可以是任何数据（如整数、字符、数组、对象等）在内存中的位置。
>
>通过使用指针，可以实现对内存中数据的间接访问。指针可以用来传递数据、修改数据，以及动态分配和释放内存等。在许多编程语言中，特别是低级语言如 C 和 C++，指针是一项重要的概念。
>
>指针通常包括两个部分：地址和类型。地址是内存位置的值，类型表示该地址上存储的数据的类型。指针的操作包括获取地址（取址操作符 &）、获取地址上的值（间接引用操作符 *）以及指针的赋值和比较等。
>
>指针的使用需要小心谨慎，因为错误的指针操作可能会导致程序崩溃或产生难以调试的错误。正确地使用指针可以提高程序的效率和灵活性，但也需要程序员对内存管理有一定的了解和责任。

####  Q2. Go 的指针和 C 语言的指针有什么区别

1. 指针声明和操作符：在 C 语言中，使用星号（*）来声明指针类型和进行间接引用操作，而在 Go 语言中，指针类型使用 `*` 进行声明，但间接引用操作使用`.` 运算符，而不是 `*`
2. 空指针：在 C 语言中，可以将指针设置为 NULL，表示指针不指向任何有效的内存地址。而在 Go 语言中，使用 `nil` 表示空指针
3. 内存管理：在 C 语言中，程序员负责手动分配和释放内存，使用 `malloc()` 和 `free()` 等函数。而在 Go 语言中，有垃圾回收器负责自动管理内存，程序员无需手动释放内存。这种自动内存管理可以减少内存泄漏和悬挂指针等问题
4. 指针运算：在 C 语言中，可以对指针进行算术运算，例如指针的加法和减法。而在 Go 语言中，指针运算是受限的，不允许进行指针的算术运算(Go保证安全性)
5. 安全性：Go 语言在设计上注重安全性和简洁性，对指针的使用进行了限制，避免了一些常见的错误和安全隐患，如空指针引用和悬挂指针等。这使得 Go 语言相对于 C 语言更容易编写安全和可靠的代码

总体而言，Go 语言对指针的处理更加简化和安全，提供了更高级别的内存管理和类型安全性，而 C 语言则更加底层和灵活，允许更多的指针操作和内存控制。

---

### 函数

函数本质是一个指针 指向其函数的内存地址

#### Q1. 匿名函数:

```go
//匿名函数
	var getSum = func(n1, n2 int) (sum, difference int) {
		sum = n1 + n2
		difference = n1 - n2
		return
	}
	// 立即调用的匿名函数
	res3, res4 := func(n1, n2 int) (sum, difference int) {
		sum = n1 + n2
		difference = n1 - n2
		return
	}(2, 3)
```

#### Q2. defer: 延迟执行函数

1. 延迟执行的函数会被压入一个栈中 return之后按照先进后出的顺序调用
2. 延迟执行的函数其参数会立即求值
3. defer常用于资源释放、文件关闭、解锁以及记录时间等操作

#### Q3. recover 错误捕捉 函数能防止异常错误退出:

```go
func DeferRecover() {
	defer func() {
		err := recover()
		if err != nil {
			fmt.Println("err=", err)
		}
	}()
	n := 0
	fmt.Println(3 / n)
}
```
#### Q4.  什么是闭包

>Go 语言支持函数闭包（Function Closure）。闭包是指一个函数值（函数变量）捕获并绑定了其周围的函数体内的变量。这意味着闭包函数可以访问并操作其外部函数中声明的变量，即使外部函数已经返回，闭包仍然可以使用这些变量。
>
>简言之:
>
>当我们谈论闭包时，可以将其想象成一个包裹（closure），里面装着一个函数和一些与函数相关的数据。这个函数可以访问并操作这些数据，即使这些数据在函数外部也是可见的。
>
>闭包通常用于在函数内部创建函数，并且这些函数可以访问函数外部的变量。这就像是在函数内部构建了一个小的「记忆」，可以保存一些数据状态。每当我们调用闭包时，它会记住之前的状态，并在后续调用中保持这些状态的连续性。

闭包在 Go 语言中具有以下特点：

1. 函数变量：在 Go 语言中，函数可以作为一种类型，可以被赋值给变量，作为参数传递给其他函数或者作为返回值。通过将函数赋值给变量，我们可以创建闭包。

2. 引用外部变量：闭包函数可以访问和使用其外部函数中声明的变量，包括函数参数和局部变量。闭包通过引用捕获这些变量，因此即使外部函数执行完毕，闭包仍然可以使用这些变量。

3. 保持状态：闭包函数可以在不同的时间点被调用，而它所捕获的变量将保持其最新的状态。这使得闭包可以用于实现保持状态或者记忆的功能，例如计数器或者延迟执行。

   ```go
   func counter() func() int {
       count := 0
       return func() int {
           count++
           return count
       }
   }
   
   func main() {
       c := counter()
       fmt.Println(c()) // 输出: 1
       fmt.Println(c()) // 输出: 2
       fmt.Println(c()) // 输出: 3
   }
   ```

   在这个例子中，我们定义了一个 `counter` 函数，它返回一个匿名函数。这个匿名函数引用了外部函数中的 `count` 变量。每次调用 `counter` 函数时，都会创建一个新的闭包实例，并且每个实例都保留了自己的 `count` 变量。每次调用闭包函数时，它会增加 `count` 的值并返回。

   闭包的妙处在于它可以记住之前的状态。在这个例子中，无论我们调用多少次闭包函数，它都会记住每次调用的计数值，并保持连续增加。这是因为**闭包函数持有对外部变量的引用**，而不是简单地创建新的变量副本。





### 数组



### Map

#### Q1: map是什么

在Go语言中，map是一种无序的键值对的集合。它是一种引用类型，可以像其他引用类型一样通过make函数进行初始化。map的键可以是任何可比较类型，例如字符串、数字、结构体等，而值可以是任何类型。map的底层实现是哈希表(类似Java的HashMap?)，因此可以在O(1)时间复杂度内完成插入、查找和删除操作。如果需要使用map，请确保在使用前进行初始化

> Java的HashMap 和Golang的map的异同
>
> Java的HashMap和Go的map都是键值对的集合，但是它们在实现上有一些不同。Java的HashMap是基于哈希表实现的，而Go的map也是基于哈希表实现的。但是，Java的HashMap允许键和值为null，而Go的map不允许键为nil。此外，Java的HashMap是线程不安全的，需要使用Collections.synchronizedMap()方法或者ConcurrentHashMap来保证线程安全，而Go的map是并发安全的，可以在多个goroutine中同时使用。

#### Q2: map的增删改查和遍历

```go
// Map 键值对
func Map() {

	var m0 map[string]string
	fmt.Println(m0)
	m0 = make(map[string]string, 16)
	fmt.Println(m0)
	m1 := map[string]string{
		"name": "张三",
		"age":  "18",
		"id":   strings.ReplaceAll(uuid.New().String(), "-", ""), // 生成uuid
	}
	fmt.Println(m1)

	// map的插入
	m1["password"] = "123"
	fmt.Println(m1)
	// 修改
	m1["age"] = "100"
	fmt.Println(m1)

	// 查找
	v, ok := m1["name1"]
	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("不存在")
	}

	// map的遍历
	for key, value := range m1 {
		fmt.Println(key, "=", value)
	}
	// 删除 通过delete(map,key)函数删除单个元素
	// 删除不存在的元素也不会报错
	delete(m1, "name")
	fmt.Println(m1)
	// 删除所有元素 通过make()函数重新make一次 或者指向nil
	m1 = nil
	m2 := make(map[string]string, 16)
	fmt.Println(m1)
	fmt.Println(m2)
}
```



![image-20230620114318713](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com/java/image-20230620114318713.png)

### 结构体

###  切片

#### Q1: 切片是什么
在go的源文件中,切片底层是一个结构体

```go
type slice struct {
	array unsafe.Pointer // 指向底层数组
	len   int  // 底层数组的长度
	cap   int  // 底层数组的长度
}
```

#### Q2: 切片的赋值/拷贝

当 brr:=arr时,会拷贝slice1给slice2,但是他们Pointer 指针指向的是同一个底层数组 因此操作切片2的同时切片1的内容也会发生变化

![image-20230620105124794](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com/java/image-20230620105124794.png)

对切片的修改结果会相互影响

```go
func AppendDemo() {
	arr := make([]int, 3, 5)
	arr[0], arr[1], arr[2] = 2, 7, 9
	//底层数组 : 2, 7, 9, 8
	brr := append(arr, 8)
	// brr和arr共享底层数组 因此修改arr会影响brr 反之亦然
	fmt.Println(len(arr))
	fmt.Println(len(brr)) 
	//底层数组 : 2, 7, 9, 10
	arr = append(arr, 10)
	fmt.Println(arr)
	fmt.Println(brr)
}
```

![image-20230620110146241](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com/java/image-20230620110146241.png)

当切片扩容后.对切片的修改则不会再相互影响

```go
func AppendDemo() {
	arr := make([]int, 3, 5)
	arr[0], arr[1], arr[2] = 2, 7, 9
	//底层数组 : 2, 7, 9, 8
	brr := append(arr, 8)
	// brr和arr共享底层数组 因此修改arr会影响brr 反之亦然
	fmt.Println(len(arr)) // 3
	fmt.Println(len(brr)) // 4
	//底层数组 : 2, 7, 9, 10
	arr = append(arr, 10)
	fmt.Println(arr)
	fmt.Println(brr)
	fmt.Println("brr 插入 	容量满")
	brr = append(brr, 15)
	arr[0] = 100
	brr[1] = 200
	fmt.Println("容量满 arr", arr) //容量满 arr [100 200 9 10]
	fmt.Println("容量满 brr", brr) //容量满 brr [100 200 9 10 15]
	//根据以上运行结果可知 当容量未满时，arr和brr始终共享底层数组

	fmt.Println("brr 插入  需要扩容")
	brr = append(brr, 16)
	arr[2] = 200
	brr[3] = 300
	fmt.Println("需要扩容 arr", arr) //需要扩容 arr [100 200 200 10]
	fmt.Println("需要扩容 brr", brr) //需要扩容 brr [100 200 9 300 15 16]
	// 根据以上运行结果可知 当容量满时，arr和brr不再共享底层数组
	// 底层会重新分配一个数组，将原数组的值拷贝到新数组中 将新数组的地址赋值给brr的Pointer 从而对brr的修改不会影响arr

}
```

![image-20230620111647876](https://cscgblog-1301638685.cos.ap-chengdu.myqcloud.com/java/image-20230620111647876.png)

#### Q3. 切片的遍历

同数组遍历一致 可使用普通for循环或者for...range 遍历

