---
title: C++11中的值类型与移动语义
category: c-c++
tags: [ C++11, C++ ]
layout: post
comments: true
---

在学习C++17的时候，在油条学长的指导下重新、更加深入地学习了C++11引入的全新的左值、右值类型系统。尽管我以前也有看过左右值相关的资料，但是一直停留一扫而过没有放在心上的程度。这次趁着还没有彻底忘记来温习并记录一下。

## 值类型的发展

为什么C、C++存在值类型，其存在的意义、目的是什么。为什么到了C++11又要重新定义一个全新的左右值类型系统。为了搞清楚这些问题，我需要去了解值类型的发展历史。

### CPL

一开始，左右值的概念来自一种叫做CPL的程序设计语言（C语言的爷爷），这时候的左值、右值正如它们的名字，左值是能放在赋值号左侧被赋值的对象，右值是可以放在等号右侧的对象。本质上，所有的对象均可作为右值，但是只有部分对象可以作为左值。

### C

在不考虑`const`的引入使得某些左值不能被赋予新值的情况下，C语言的左值和CPL的系统完全相同，而右值的概念被取消。

### C++98

C++98除了重新引入了右值，对于值的处理上和C一样。因为多了引用这个东西，所以左右值的区分变得复杂起来。

C++和C相比一个比较重要的改变是前缀自增、组合赋值运算的返回值变成了左值引用。这个改动使得C++里可以进行链式自增、组合赋值。而C中，由于返回值要不为指针，要不为拷贝，不可能再进行自增和赋值（以前写C的时候被坑过）。

### C++11

引入了移动语义、右值引用，增加了“亡值”的概念。值类型系统在C++11瞬间变得复杂。

C++11的值类型系统包括左值`lvalue`，纯右值`prvalue`，亡值`xvalue`，其中左值和亡值属于泛左值`glvalue`，纯右值和亡值属于右值`rvalue`。这里的纯右值实际上就是C++11以前的右值。而亡值引入，则的意思是将要死亡的值，将要被移动，是引入了右值引用造成的：返回为右值引用的函数表达式为亡值。以及对右值对象取成员、右值数组取索引均为亡值。

## 引用

C++作为面向对象的C语言，为了能让自定义类也能像内置类型一样有很自然的内置运算符，增加了运算符重载，为了对操作运算符函数的返回值的操作显得更加自然，增加了引用这个东西。

然后在C++11中，为了在代码中区分开移动（浅拷贝+清空原对象）和深拷贝，增加了右值引用。

### 左值引用

简单来说，左值引用就是别名，声明一个左值引用的格式为`type &lreferred = lvalue;`左值引用只能绑定在左值上，一旦绑定就无法再改变引用指向的对象，修改左值引用就是修改被绑定的对象。下面的代码是简单的示例。

{% highlight cpp %}
int x = 0;
int &r = x;
r = 42;
assert(x == 42);
{% endhighlight %}

左值引用最主要的用处在函数传参和函数返回值中。

在C中，函数传参都是传入一份参数的拷贝，也就是传入到函数的不是调用函数处传入的对象，而是在栈上新建了一块和传入对象各个值都相同的内存块，对这块内存进行的操作不会影响到原来的对象。所以在C里面，想要修改传入的对象，我们需要传入指向对象的指针。

而在C++中，我们不去考虑引用的本质，可以假装引用就是我们传入的对象，对引用的修改可以直接反应到传入的对象中。

一个简单的例子，交换变量的函数在C和C++中的区别。

{% highlight cpp %}
void swap(int *l, int *y) {
	/* swap function in C */
	int t = *l; *l = *r; *r = t;
}

template <typename T>
void swap(int &l ,int &r) {
	// swap function in C++
	T t = l; l = r; r = l;
}
{% endhighlight %}

不用管我template的那块，首先C++的swap就简单很多，不需要有一堆复杂的解引用。特别是在传入的对象是个指针，而我们需要修改指针值的时候，不需要有二重指针这种令人眼花缭乱的东西。

而引用被引入到C++最初的目的是为了使运算符重载的传参和返回更加自然，再来个例子，如果C++里没有引用，我们的`operator[]`可能要长下面这样。

{% highlight cpp %}
template <typename T, size_t N>
struct MyArray {
	T arr[N];

	T *operator[](size_t p) {
		return arr + p;
	}
};

template <typename T, size_t N>
void reverse(MyArray<T, N> *a) {
	for (size_t i = 0; i != N >> 1; ++i) {
		T t = *(*a)[i];
		*(*a)[i] = *(*a)[N - 1 - i];
		*(*a)[N - 1 - i] = t;
	}
}
{% endhighlight %}

大概没人希望自己的代码里有一坨的解引用和括号。

如果你觉得这样的代码还可以接受的话，再去试着写一下不用引用的`std::vector<std::string>`快速排序？

### 右值引用

C++11的右值引用，类似于左值引用，可是它只能绑定到右值上，格式也类似，`type &&rreferred = rvalue`。还是个简单的例子。

{% highlight cpp %}
int a = 0;
//int &&r = a;  // compile error
int &&r = std::move(a); // equals to static_cast<int &&>(a)
r = 42;
assert(a == 42);
{% endhighlight %}

然而里除了一步将a强转成右值引用，和左值引用又有什么区别呢？区别就在于类型不同。因为两者的类型不同，所以在函数调用的时候，会调用不同的重载函数——左值引用调用左值引用参数的函数，右值引用调用右值引用重载的函数。除此之外就没有区别了。

既然没有区别，为什么C++11要引入这么复杂的值类型系统呢？答案是：为了速度。还是原来的swap函数，考虑下面这种情况。

{% highlight cpp %}
template <typename T>
void swap(T &l, T &r) {
	T t = l; l = r; r = t;
}

int main() {
#define str16 "################"
#define str64 str16 str16 str16 str16
#define str256 str64 str64 str64 str64
#define str1024 str256 str256 str256 str256
	std::string s1 = str1024;
#undef str16
#undef str64
#undef str256
#undef str1024

#define str16 "****************"
#define str64 str16 str16 str16 str16
#define str256 str64 str64 str64 str64
#define str1024 str256 str256 str256 str256
	std::string s2 = str1024;
#undef str16
#undef str64
#undef str256
#undef str1024
	::swap(s1, s2);
	return 0;
}
{% endhighlight %}

在这里，`::swap`函数首先对`t`从`l`进行了一次1024字节的深拷贝构造，这也就是把`l`所有的值都拷贝到一块新申请的内存里，然后再将`r`的值深拷贝到`l`，最后再将`t`的值深拷贝回`r`。而我这个程序里，`l`和`r`都是长度为1024的字符串，于是`::swap`函数就进行了总计3072字节的内存拷贝和一次内存申请。而如果`l`和`r`是更大的字符串，`l`和`r`的大小不相同，这里就会导致更多的内存移动和内存申请——这将大大拖慢程序速度。

但是没有那么复杂，本质上，我们只需要交换两个`std::string`里指向内存区域的指针，再改几个描述这段内存的成员变量即可，可是C++11以前，`std::string`没有提供这种实现了这种方法的函数，我们也不可能去访问这些不存在的（`private`）成员，于是我们只能用一开始的方式简单粗暴地进行拷贝。

所以在C++11中，为了使编译器知道什么时候直接浅拷贝，什么时候深拷贝，从标准中引入了移动语义和右值引用。而在编译的后端，编译器也能放心地进行更彻底的优化。

### 移动语义`std::move`

C++11的`std::move`函数，其实就是一个`static_cast<typename remove_reference<T>::type &&>(t)`，也就是对于任何类型，强制转换成其无引用类型的右值引用，除此之外什么都没干。真正要干活的，重载了右值引用为参数的函数，由这些函数来执行移动。

为了更清楚地说明，还是举个例子。这里重写一个只有构造函数的vector。方便起见，直接用struct，全成员public。

{% highlight cpp %}
#include <iostream>

template <typename T>
struct MyVector {
	T *vec;
	size_t size;

	MyVector(size_t count, const T &value) {
		this->size = count;
		this->vec = new T[count];
		for (size_t i = 0; i != count; ++i) {
			this->vec[i] = value;
		}
	}

	MyVector(MyVector<T> &other) {
		this->size = other.size;
		this->vec = new T[other.size];
		for (size_t i = 0; i != other.size; ++i) {
			this->vec[i] = other.vec[i];
		}
	}

	MyVector(MyVector<T> &&other) {
		this->size = other.size;
		this->vec = other.vec;
		other.size = 0; other.vec = nullptr;
	}
};

int main() {
	MyVector<char> v0(10, 'x');
	std::cout << "============" << '\n';
	std::cout << "v0.size = " << v0.size << '\n';
	std::cout << "v0.vec  = " << static_cast<void*>(v0.vec) << '\n';
	MyVector<char> v1 = v0;
	std::cout << "============" << '\n';
	std::cout << "v0.size = " << v0.size << '\n';
	std::cout << "v0.vec  = " << static_cast<void*>(v0.vec) << '\n';
	std::cout << "v1.size = " << v1.size << '\n';
	std::cout << "v1.vec  = " << static_cast<void*>(v1.vec) << '\n';
	MyVector<char> v2 = std::move(v0);
	std::cout << "============" << '\n';
	std::cout << "v0.size = " << v0.size << '\n';
	std::cout << "v0.vec  = " << static_cast<void*>(v0.vec) << '\n';
	std::cout << "v1.size = " << v1.size << '\n';
	std::cout << "v1.vec  = " << static_cast<void*>(v1.vec) << '\n';
	std::cout << "v2.size = " << v2.size << '\n';
	std::cout << "v2.vec  = " << static_cast<void*>(v2.vec) << '\n';
	std::cout << "============" << '\n';
	return 0;
}
{% endhighlight %}

运行结果应该如下所示（除了地址不一样）

{% highlight text %}
================
v0.size = 10
v0.vec  = 0x634c20
================
v0.size = 10
v0.vec  = 0x634c20
v1.size = 10
v1.vec  = 0x635050
================
v0.size = 0
v0.vec  = 0
v1.size = 10
v1.vec  = 0x635050
v2.size = 10
v2.vec  = 0x634c20
================
{% endhighlight %}

STL中的对于右值引用的使用大概也就是这个逻辑。

### 完美转发`std::forward`

另外还有一个`std::forward`，称其为“完美转发”，其实它等价与`static_cast<T &&>`，和`std::move`的区别就在于少了一个`remove_reference`。为了更详细地说明它们的区别，上表格。

|被引用对象类型|引用类型|最终对象类型|
|---|---|---|---|
|`T`|`&`|`T&`|
|`T`|`&&`|`T&&`|
|`T&`|`&`|`T&`|
|`T&`|`&&`|`T&`|
|`T&&`|`&`|`T&`|
|`T&&`|`&&`|`T&&`|

注意到，左值引用对象的右值引用结果还是左值引用，右值引用对象的右值引用结果还是右值引用，所以`std::forward`其实并没有改变引用类型。

由于任何有名字的量都是左值，所以在使用参数调用别的重载函数的时候会遇上麻烦，这时候就要用`std::forward`来获取这个变量原来的引用类型传入。

{% highlight cpp %}
#include <iostream>
#include <list>
#include <vector>

struct adjacency_list {
	std::vector<std::list<int>> vli;
	template <typename T>
	adjacency_list(T &&g)
		: vli(std::forward<T>(g)) { }
};

template <typename T>
void show2D(const T &vv) {
	for (const auto &it1: vv) {
		for (const  auto &it2: it1) {
			std::cout << it2 << ' ';
		}
		std::cout << '\n';
	}
}

int main() {
	std::vector<std::list<int>> vli = { {0, 1}, {2, 3} };
	std::cout << "** vli **\n"; show2D(vli);

	std::cout << "=== copy ===\n";
	adjacency_list cpy(vli);
	std::cout << "** vli **\n"; show2D(vli);
	std::cout << "** cpy **\n"; show2D(cpy.vli);

	std::cout << "=== move ===\n";
	adjacency_list mov(std::move(vli));
	std::cout << "** vli **\n"; show2D(vli);
	std::cout << "** cpy **\n"; show2D(cpy.vli);
	std::cout << "** mov **\n"; show2D(mov.vli);

	return 0;
}
{% endhighlight %}

上面的代码中，`adjacency_list`中的构造函数的函数参数为`T &&`，由于任何引用的右值引用还是本身，所以这个可以接受任何引用并且不改变引用类型。然后再在`vli`的构造函数中直接用`std::forward`传递`g`。

如果用了`std::forward`，那么输出结果应该和下面一样。

{% highlight text %}
** vli **
0 1 
2 3 
=== copy ===
** vli **
0 1 
2 3 
** cpy **
0 1 
2 3 
=== move ===
** vli **
** cpy **
0 1 
2 3 
** mov **
0 1 
2 3 
{% endhighlight %}

可以看出，第二个使用了拷贝构造，第三个用了移动构造。

而如果没有使用`std::forward`，最后那个mov中会使用拷贝构造，因为`g`是一个左值——任何变量都是左值。这就是`std::forward`的完美转发。

## 值类型

这里更详细地列举一下C++11中的值类型内容。摘抄自[cppref](http://zh.cppreference.com/w/cpp/language/value_category){:target="_blank"}

* 泛左值glvalue，**g**eneralized **l**eft **value**，指的是可以通过取地址或者一些其它手段确认是不是指向某一个实体的表达式。
* 右值rvalue, **r**ight **value**，可被移动构造函数、移动赋值运算符或者其它实现了移动语义的其它函数重载绑定的表达式被称之为右值。
* 左值lvalue，**l**eft **value**，不能被移动的泛左值。
* 亡值xvalue，e**x**piring **value**，可以被确认多个亡值或左值是否为同个实体，也可以被移动。
* 纯右值prvalue，**p**ure **r**ight **value**，不可通过取地址或其它手段获知其是否为同一实体。

### 左值

* 当前作用域下的所有的有名字的变量。
* 字符串字面常量。
* 返回值为左值引用的函数。
* `static_cast<T&>`等强制转换为左值引用的强制转换表达式。
* `++a`, `--a`，内建前置自增自减表达式。
* `a = b`, `a += b`等，内建赋值、复合赋值表达式。
* `*p`，解引用指针。
* `a.m`, `p->m`, `a.*mp`, `p->*mp`，当`a`为左值时的取成员表达式。
* 返回**函数右值引用**的函数调用。
* 强制转换为**函数右值引用**的强制转换表达式。
* `a, b`, `a ? b : c`，返回值为上述值的逗号表达式和三目表达式。

### 纯右值

* 除了字符串字面量以外的所有直接值。
* 返回值为非引用的函数。
* `static_cast<T>`等转换为非引用类型的强制转换表达式。
* `a++`, `a--`等内建后置自增自减表达式。
* `a + b`, `a || b`, `a < b`等所有内建二元算数、逻辑、比较表达式。
* `&a`取地址表达式。
* `a.m`, `p->m`, `a.*mp`, `p->*mp`，`m`为成员枚举符、`m`和`mp`为非静态成员函数的情况。
* `this`指针。
* `[]{}`等，Lambda表达式。
* `a, b`, `a ? b : c`，返回值为上述表达式的逗号表达式或三目表达式。

### 亡值

* `static_cast<T&&>(a)`
* `std::move(a)`等，返回对象右值引用的函数。
* `a[n]`，内建的下标表达式，其中一个为数组右值（关于这个，要知道`a[n]`和`n[a]`是同一个东西）。
* `a.m`，需要`a`为右值且`m`为非引用类型的非静态数据成员时该表达式为亡值。
* `a.*mp`，需要`a`为右值该表达式为亡值。
* `a ? b : c`，返回值为上的三目运算符。

