---
layout: post
title: "Swift的 ? 和 ! "
date: 2015-10-10 17:12:37 +0800
comments: true
categories: swift

---

Swift语言中的常量变量的声明和使用方式借鉴了像Ruby这样的动态语言，也保留了静态语言的一些特性。

* 使用let定义常量
* 使用var定义变量
* **普通值**类型的变量声明之后不会赋默认值，必须初始化才能使用
* 除普通值类型外，还可以声明**Optional值**类型的变量，也就是本文所讲的内容。

Swift的常量变量使用中，Optional值理解起来是比较困难的，尤其是?和!这样的语法糖，代码写起来赏心悦目，但是学习起来就需要下点功夫。

>?和!只是Optional值的语法糖，而!可以看做是?的进一步的特殊形式，所以后面说明的顺序是：Optional值 -> ?语法糖 -> !语法糖 -> 使用注意事项

<!--more-->

###什么是Optinal值（?语法糖）

首先，我们来声明一个普通值的变量。
		
	var str : String

不给str赋初值，直接使用的话编译器会报错：

	// error: Variable 'str' used before being initialized
	print(str)

如果我们这样定义，就没有问题了。

	var str2 : Optional<String>
			
	// 通常我们使用?语法糖来实现上面的语法：
	// var str2 : String?
			
	print(str2)		// "nil\n"
		
"Optional\<String>"就是一个Optional值，通过这个格式可以看出：Optional本身是一个枚举，\<String>是这个枚举里面定义的一个泛型。

Optional在Swift2.0中的定义如下：

	public enum Optional<Wrapped> : _Reflectable, NilLiteralConvertible {
	    case None
	    case Some(Wrapped)
	    /// Construct a `nil` instance.
	    public init()
	    /// Construct a non-`nil` instance that stores `some`.
	    public init(_ some: Wrapped)
	    /// If `self == nil`, returns `nil`.  Otherwise, returns `f(self!)`.
	    @warn_unused_result
	    @rethrows public func map<U>(@noescape f: (Wrapped) throws -> U) rethrows -> U?
	    /// Returns `nil` if `self` is nil, `f(self!)` otherwise.
	    @warn_unused_result
	    @rethrows public func flatMap<U>(@noescape f: (Wrapped) throws -> U?) rethrows -> U?
	    /// Create an instance initialized with `nil`.
	    public init(nilLiteral: ())
	}

通过上面的定义我们可以看到两个枚举类型：.None和.Some(Wrapped)。这样就很好理解了，声明的时候如果使用了?或者!，这个变量的类型就不简简单单是String类型，而是一个Optinal.Some<String>的枚举，因此也就解释了为什么对于这样的变量直接进行比较、属性访问、方法调用时编译器会报各种各样的错误。

###封装和拆包

其实，**Swift里面的nil就是Optional.None**；非nil就是Optional.Some(Wrapped)，即变量里面存储的不是普通值（原始值），而是对原始值的**封装（Wrap）**，这也是Optional值类型变量在使用时需要先**拆包(Unwrap)**的原因。

因此 var str2 : Optional<String> 形式定义的变量，作为Optional值使用，例如用于比较、打印，可直接使用。
	
	// if (str2 != nil)
	if (str2 != Optional.None) {    // 与上面意思相等
		...
	}
		
这里强调一点是Swift2.0里面Optional值不能直接用作判断条件，必须和nil比较，如下的语句编译器会报错：

	// error: Optinal type cannot be used as a boolean.
	// if str2 {
		
Optinal值还可以直接打印：
		
	print(str2)		// "nil\n"

###Optinal值的使用（拆包）

如果要作为普通值使用，例如**方法调用、属性访问、下标访问、打印原始值**，则需要先拆包，再使用。由于Swift里面没有显示的拆包方法，因此需要通过?和!两个语法糖来实现。

* 属性访问，?的作用可以理解为不是简单的拆包，而是先判断当前变量是否为nil，如果为nil，则忽略后续操作，如果不为nil，则执行后面的操作。

		str2?.capitalizedString		// nil
		
		// 此处由于在整个上下文中我还没有对str2赋值，所以编译器会报错
		// error: Execution was interrupted, reason: EXC_BAD_INSTRUCTION
		// str2!.capitalizedString		
		
		str2 = "test"
		str2!.capitalizedString		// "Test"

* 方法调用，在方法调用的时候?的作用类似于OC中的isResponseToSelector:，如果是nil则忽略后面的方法调用，如果非nil则执行方法调用。
	
		var str3 : String?
		str3 = "hello"
		print(str3)		// "Optional("hello")\n"
		
		str3?.appendContentsOf(" swift")
		print(str3)		// "Optional("hello swift")\n"

* 下标访问

		var ary1 : Array<Int>?
		
		// 未经拆包访问下标编译器会报错
		// error: Cannot subscript a value from type 'Array<Int>?'
		// ary1[0]
		
		ary1?[0]	// nil
		
		ary1 = [1, 2, 3]
		ary1?[1]	// 2
		
* 打印原始值，打印时不能用?号，Swift中?号只能用来访问属性、调用方法、访问下标元素。
		
		var str4 : String?
		
		// error: '?' must be followed by a call, member lookup, or subscript
		// print(str4?)
		
  只能用!打印Optional的原始值，当然，如果什么都不加，则打印的是原始值本身。
		
		str4 = "make a cup of tea"
		print(str4!)	// "make a cup of tea\n"
		print(str4)		// "Optional("make a cup of tea")\n"
  		
  打印未主动初始化的Optinal值同样会报错。
  	
		var str5 : String?
		// error: Execution was interrupted, reason: EXC_BAD_INSTRUCTION
		// print(str5!)

以上就是?语法糖（即Optinal值）在变量声明和变量使用两个方面的用法，有了上面的基础，!语法糖可以简单总结为两句话：

###!语法糖

1. 声明时指定!，则表示这个变量不仅是一个Optinal值，并被初始化为nil，而且这个Optinal值在使用的时候都会被编译器隐式的加上!，即强制拆包。

		var str6 : String!
		str6 = "str6"
		print(str6)		// "str6\n"
		
	注意到上面打印出来的是拆包后的内容，而不是"Optinal("str6")\n"。
	
	隐式拆包的Optinal值（Implicitly Unwrapped Optionals）的调用方式可以有如下三种，当然通常我们使用的是第一种：
	
		str6.capitalizedString		// "Str6"
		str6?.capitalizedString		// "Str6"
		str6!.capitalizedString		// "Str6"
		
	前面提到!只不过是一种语法糖，真正对应的其实是ImplicitlyUnwrappedOptional这种枚举类型，如下两种方式声明得到的结果是一样的：
	
		// var str7 : String!
		var str7 : ImplicitlyUnwrappedOptional<String>
		
2. 使用时加上!号表示强制拆包，由于对隐式拆包的Optinal值强制拆包是没有意义的（虽然编译上没有报错），因此这里的强制拆包通常用于使用?声明的Optinal值。 

		var str8 : String?				
		str8 = "Swift"
		
		if (str8 != nil) {
			// if判断从逻辑上保证str8不为空，可以强制拆包
		    str8!.appendContentsOf(" is fun")
		}

值得注意的是：对于nil强制拆包会导致Crash。因此**!这个东西是把双刃剑**，有过大型程序经验的同学都了解：今天的逻辑保证x不为空，不代表明天的逻辑还能保证x不为空。

###?和!用于类型转换

?和!还有另一种用法就是用于类型转换，看下下面的例子。

	var anybody : AnyObject
	anybody = "still a String"

如果要把AnyObject赋值给一个Optinal.Some<String>，则需要as后面跟上?来进行类型转换，?的作用是判断anybody是否为一个String，如果是则赋值给justTry，如果不是则justTry值为nil。

	var justTry : String?
	justTry = anybody as? String	// "still a String"

如果要把AnyObject赋值给一个普通String，则需要as后面跟上!来强制类型转换，!的作用是强制转换并赋值给secondTry。

	var secondTry : String
	secondTry = anybody as! String		// "still a String"
		
如果我们这里重新给anybody赋一个Int值，那么发现!转换编译器会报错，而?转换的结果为nil

	anybody = 1
	// error: Execution was interrupted, reason: singal SIGABRT.
	// let anotherTry = anybody as! String
	
	let tryAgain = anybody as? String	// nil
	
	// 不带?和!的转换同样会报错
	//let failedTry = anybody as String

###小贴士

1. 在思考Optinal值的nil比较时突然想到既然普通值不能与nil比较，那么普通值如何判断是否为nil呢?再细一想是自己犯二了，普通值在声明的时候compile-time就保证了不会为空，自然也不需要判断；
2. 上面已经提到在Swift中，nil其实是一个枚举值Optional.None，还有一个有意思的值：Void，它其实是()，一个空元组；
3. 强制拆包（!）不要轻易使用，倒也还没想到必须使用的场景，不知道后续会不会出现关于这一点的开发规范；
4. 可以考虑Optinal值在定义的时候增加一个特殊的命名规范和普通值加以区分，虽然现在编译器给的错误提示挺全面，但是代码的可阅读性也需要考虑。

###什么时候需要Optional值

1. 属性的初始化操作不在声明时和init中进行时；
2. 对已定义的Optinal变量使用时；
3. 对变量做类型转换时；
2. 当然，任何想用的时候只要得到编译器的同意都可以用。
