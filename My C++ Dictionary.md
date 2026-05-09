### MY C++ DICTIONARY

#### 常用于错误处理的 C++ 容器

std::optional
	可选的值 即一个可能存在也可能不存在的值 作为一个函数的返回值 std::nullopt
	可以直接在if语句中使用，相当于类型转换到 bool，等同于 hasValue 成员函数
      

	or_else(),如果 optional 对象中不含有值，那么会调用 or_else 的实参，一个可调用对象，将它执行后的结果作为返回值
	  	and_then(),对返回的 optional 对象值进一步运算，返回一个新的 optional 对象
	transform(), 对 optional 值进变换

std::variant<ResultType, ErrorType>	变体类型
	variant 意为变体，变种，相异的，比方说病毒的变种(variant),或者一个产品的不同型号，
	在std::variant中指同一类东西的不同可能形态，表示这个值有多个可能的‘类型变体’（type variants）
	std::variant<int, std::string, double>
	类似于一个盒子（variant）
	里面可以装：int string double 但是同一时间只能装一个，必须知道现在装的是哪种类型
	本质是：类型安全（type-safe）的 union

	正常返回，包含的值是 ResultType,否则是 ErrorType

// std::expected<ExpectedType, UnExpectedType> 

      std::expected<double, std::string> result = std::unexpected("缺考");
      .error() 未期望值

#### 多线程

>   对于共享数据对象，不同线程会对其进行修改操作

std::atomic		原子变量

只对一个整数计数器进行读写，每次操作只是简单的赋值，加减

简单的场景，使用std::atomic更加轻量且高效

原子类型模板，保证对单个变量的读写操作具有原子性，提供一系列原子操作方法，从而再多线程环境中能够安全的访问和修改共享数据，而无需显式调用互斥锁

**创建**	

std::atomic<int> ac(0); 支持基本类型，指针类型，自定义类型

对于自定义类型，如果类中包含以下成员，则无法使用std::atomic模板参数 （通过memcpy正确拷贝）浅拷贝？

		- 类型自定义的拷贝构造，拷贝赋值，移动赋值，析构函数
		- 类型包含虚函数
		- 包含引用成员

![image-20260320212113342](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260320212115734.png)

trivially copyable 平凡拷贝 （trivially 意为琐碎地，微不足道地，平凡地）

**基本操作**

load() 是原子性的 读取值

store() 设置值 或者 = 

可以隐式变换数据类型

exchange() 读取旧值，设置新值，返回旧值

不同类型支持的原子操作是不同的

使用std::memory_order_relaxed表示只保证原子性，不保证执行顺序

**CAS操作**

Compare And Exchange 原子操作，用来实现无锁同步

![image-20260320213952630](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/20260320213954686.png)

compare_exchange_strong(T& expected, T desired);

compare_exchange_weak(T& expected, T desired); 会出现虚假失败，使用循环，多次判断

![image-20260320214437816](https://cdn.jsdelivr.net/gh/dengyu32/note_images/images/image-20260320214437816.png)



std::mutex

复杂共享数据，修改的操作不是一步完成的，而是多个步骤组成的逻辑整体

