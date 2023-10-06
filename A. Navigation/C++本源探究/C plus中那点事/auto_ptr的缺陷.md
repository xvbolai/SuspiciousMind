auto_ptr是从C98残留下来的弃用特性，它是一种针对智能指针 进行标准化的尝试，是为了提供C++11 中的unique_ptr 所提供的语义——专属所有权。正是auto_ptr想要实现的语义导致自身存在问题。

### 1. auto_ptr源码分析

```c++
template class auto_ptr { 
	private: _Tp* _M_ptr; 
	// auto_ptr对象使用_M_ptr_来指向一个heap内的对象 
	public: typedef _Tp element_type; 
	// 显式构造函数，接受一个原生指针来生成auto_ptr对象 
	explicit auto_ptr(element_type* __p = 0) throw() : _M_ptr(__p) { } 
	// 拷贝构造函数，只能接收一个作为左值的auto_ptr对象 
	auto_ptr(auto_ptr& __a) throw() : _M_ptr(__a.release()) { } 
	// 拷贝构造函数，兼容_Tp1*可以隐式转换为_Tp*的auto_ptr对象作为初值 
	template auto_ptr(auto_ptr<_Tp1>& __a) throw() : _M_ptr(__a.release()) { } 
	// 赋值构造函数，同样只能接收一个作为左值的auto_ptr对象 
	auto_ptr& operator=(auto_ptr& __a) throw() { 
		reset(__a.release()); 
		return *this; 
	} 
	template auto_ptr& operator=(auto_ptr<_Tp1>& __a) throw() { 
		reset(__a.release()); 
		return *this; 
	} 
	// auto_ptr对象析构时，销毁其所指物 
	~auto_ptr() { 
		delete _M_ptr; 
	} 
	// 解引用操作 
	element_type& operator*() const throw() { 
		__glibcxx_assert(_M_ptr != 0); 
		return *_M_ptr; 
	} 
	element_type* operator->() const throw() { 
		__glibcxx_assert(_M_ptr != 0); 
		return _M_ptr; 
	} 
	/** * 通过get函数可以获取管理对象 **/ 
	element_type* get() const throw() { 
		return _M_ptr; 
	} 
	/** * release和reset是auto_ptr最重要的成员方法； * 可以看出当auto对象被拷贝或者赋值时，对象所有权会转移； * 因此，千万不要使用by value的方式传递auto_ptr对象 */ 
	element_type* release() throw() { 
		element_type* __tmp = _M_ptr; 
		_M_ptr = 0; return __tmp; 
	} 
	void reset(element_type* __p = 0) throw() { 
		if (__p != _M_ptr) { 
			delete _M_ptr; _M_ptr = __p; 
		} 
	} 
};
```

值得注意的是，由于auto_ptr的拷贝构造函数和赋值构造函数操作会控制权转移，因此auto_ptr的拷贝构 造函数和赋值构造函数使用的入参并不是const_by_reference。这样，在这种情况下可能会编译不过：

```c++
auto_ptr pt(auto_ptr new(int(3)));
```
因为auto_ptr new(int(3))是一个右值，右值不能赋给左值引用。因此auto_prt 中还提供了如下拷 贝构造函数和赋值构造函数，入参为auto_ptr_ref对象，就是为了解决此场景：
```c++
auto_ptr(auto_ptr_ref __ref) throw() : _M_ptr(__ref._M_ptr) { }
auto_ptr& operator=(auto_ptr_ref __ref) throw() { 
	if (__ref._M_ptr != this->get()) { 
		delete _M_ptr; 
		_M_ptr = __ref._M_ptr; 
	} 
	return *this; 
}
```

只要auto_ptr new(int(3))可以隐式的转为auto_ptr_ref就可以构造auto_ptr对象。

关于auto_ptr的使用：

```c++
auto_ptr p(new int(3)); 
auto_ptr ap2(p); // 控制权转移，p已经不能再使用了
```

### 2. auto_ptr的问题

（1）“非自觉遗失所有权”引发的运行时危险
```c++
template void badPrint(std::auto_ptr p) { 
	if(p.get() == nullptr) { 
		std::cout << "nullptr"; 
	} else { 
		std::cout << *p; 
	} 
}

std::auto_ptr ptr(new int); 
*ptr = 42; 
badPrint(ptr); 
// ptr的所有权转移给了p，p在函数调用完毕后会删除其拥有的对象 
*ptr = 18; // runtime error
```

通过源码我们可以发现auto_ptr在copy和assignment操作时会导致所有权转移，因此只要auto_ptr以实 参传递，就会引发致命的运行期错误。

（2）不存在deleter 所表示的语义，只能使用它处理“以new分配之单一对象”

（3）传递给容器也很危险

使用容器保存auto_ptr后，在进行操作时，可能由于“非自觉性遗失所有权”导致容器中保存的原 auto_ptr所管理的对象已经失效。**例如，使用排序算法**。

auto_ptr是从C98残留下来的弃用特性，**可以使用它来学习RAII的原理**，但是不能在代码中使用， unique_ptr可以做auto_ptr能够做的任何事，当你需要使用智能指针时，unique_ptr基本上应是手头首选。因为unique_ptr真正实现了专属所有权语义 ，当你值传递时就必须使用std::move()传递实参。

### auto_ptr类的限制（auto_ptr的缺陷）

1. 不要使用auto_ptr对象保存指向静态分配对象的指针，否则，当auto_ptr对象本身被撤销的时候，它将试图删除指向非动态分配对象的指针，导致未定义的行为。

2. 永远不要使用两个 auto_ptrs 对象指向同一对象，导致这个错误的一种明显方式是，使用同一指针来初始化或者 reset 两个不同的 auto_ptr对象。另一种导致这个错误的微妙方式可能是，使用一个 auto_ptr 对象的 get 函数的结果来初始化或者 reset另一个 auto_ptr 对象。

3. 不要使用 auto_ptr 对象保存指向动态分配数组的指针。当auto_ptr 对象被删除的时候，它只释放一个对象—它使用普通delete 操作符，而不用数组的 delete [] 操作符。

4. 不要将 auto_ptr 对象存储在容器中。容器要求所保存的类型定义复制和赋值操作符，使它们表现得类似于内置类型的操作符：在复制（或者赋值）之后，两个对象必须具有相同值，auto_ptr 类不满足这个要求。


