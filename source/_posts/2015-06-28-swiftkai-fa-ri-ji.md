---
layout: post
title: "Swift开发日记"
date: 2015-06-28 17:48:09 +0800
comments: true
categories: 
---

###枚举（Enumeration）
> [枚举（Enumerations） - The Swift Programming Language 中文版 - 极客学院Wiki](http://wiki.jikexueyuan.com/project/swift/chapter2/08_Enumerations.html)

1. 枚举的定义
	
		enum CompassPoint {
			case North
			case South
			case East
			case West
		}

2. 枚举的使用
	* 通过已定义枚举变量可以找到其枚举类型，因此在之后的赋值或控制流时可以忽略其枚举类型，直接以 . 开头
	
			var direction = CompassPoint.North
			switch direction {
			case .North:
			case .South:
			...
			}

	* 如果控制流没有枚举出所有枚举值，则需要default语句，否则会报错

<!--more-->
3. 相关值
	* 即可以定义如下类型的枚举：

			enum Barcode {
				case UPCA(Int, Int, Int, Int )
				case QRCode(String)
			}
	
	* 赋值和使用方式如下：

			var product = Barcode.UPCA(8, 1111, 2222, 3)
			product = Barcode.QRCode("ABCDEFG")

			// 控制流
			switch product {
			case .UPCA(let numberSystem, let manufacturer, let product, let check):
			case let .QRCode(productCode):
			}

4. 原始值的隐式赋值

	* 首先定义枚举时可以给枚举值指定原始值

			enum ASCIIControlCharacter: Character {
				case Tab = "\t"
			    case LineFeed = "\n"
			    case CarriageReturn = "\r"
			}
	* 在使用原始值为整数或者字符串类型的枚举时，Swift会自动赋值。
		- Int类型隐式赋值为依次递增1，如果第一个值没有赋值，将会被自动置位0

				enum Planet: Int {
					// Mercury: 0, Venus: 1 ...
					case Mercury, Venus, Earth, Mars, Jupiter, Saturn, Uranus, Neptune
				}
		- String类型隐式赋值则为该成员的名称

				enum CompassPoint: String {
					// North: "North", South: "South" ...
					case North, South, East, West
				}

		- 比如Alamofire里面有一段代码即为隐式赋值：

				public enum Method: String {
				    case OPTIONS, GET, HEAD, POST, PUT, PATCH, DELETE, TRACE, CONNECT
				}

###函数
1. 函数定义
		
		func sayHello(personName: String) -> String {
			let greeting = "Hello, " +  personName + "!"
			return greeting
		}

2. 函数名称
	
		sayHello(_:)

3. 多重输入参数&多参量参数
	* 多重输入参数，参数类型相同，调用时不需要外部参数名

			func halfOpenRangeLength(start: Int, end: Int) -> Int {
			    return end - start
			}
			print(halfOpenRangeLength(1, 10))
			// prints "9"

	* 多参量参数

			func sayHello(personName: String, alreadyGreeted: String) -> String {
				if alreadyGreeted {
					return "say hello again!"
				} else {
					return sayHello(personName)
				}
			}
			print(sayHello("Tim", alreadyGreeted: true))
			// prints "Hello again, Tim!"
	* 本地参数名称&外部参数名称
		- 函数参数可以有一个本地参数名（用于函数内部实现）和一个外部参数名（用于函数调用）
		- 一般来说，第一个参数忽略外部参数名，第二个参数起，外部参数名与其本地参数名相同

				func someFunction(firstParameterName: Int, secondParameterName: String) {
					...
				}
				someFunction(1, secondParameterName: "two")
		- 如果要区分本地参数名和外部参数名，则外部参数名在前，而这以空格分隔。

				func sayHelloToBoth(to person: String, and anotherPerson: String) {
					...
				}
				sayHelloToBoth(to: "Tom", and: "Jerry")
		- 如果指定了外部参数名，则调用时必须使用外部参数名
		- **忽略外部参数名**，如果不想为第二个及后续的参数设置外部参数名，可用一个下划线(_)代替

				func sayHelloToBoth(person: String, _ anotherPerson: String) {
					...
				}
				sayHelloToBoth("Tom", "Jerry")

		- 默认参数值，如果给一个参数设置了默认参数值，则调用时可忽略这个参数，将带有默认值的参数放在函数参数列表的最后。这样可以保证在函数调用时，非默认参数的顺序是一致的，同时使得相同的函数在不同情况下调用时显得更为清晰。

				func someFunction(parameterWithDefault: Int = 12) {
				    // function body goes here
				    // if no arguments are passed to the function call,
				    // value of parameterWithDefault is 12
				}
				someFunction(6) // parameterWithDefault is 6
				someFunction() // parameterWithDefault is 12

4. 返回值
	* 无返回值函数
		- sayGoodbye(_:) 
		- 可以省略返回值
		- 严格上来说，虽然没有返回值被定义，sayGoodbye(_:) 函数依然返回了值。没有定义返回类型的函数会返回特殊的值，叫 Void。它其实是一个空的元组（tuple），没有任何元素，可以写成()。
	* 多返回值
		- 返回一个元组（注意：这个函数还体现了另一个语法，可变参数）

				func minMax(numbers: Int...) -> (min: Int, max: Int) {
					...
				}

				if let bounds = minMax(3, 4, 2, 6) {
					print("min is \(bounds.min), and max is \(bounds.max)")
				}
5. 举个Alamofire中的函数例子说明：
	
		public func request(
		    method: Method,
		    _ URLString: URLStringConvertible,
		    parameters: [String: AnyObject]? = nil,
		    encoding: ParameterEncoding = .URL,
		    headers: [String: String]? = nil)
		    -> Request
		{
		    return Manager.sharedInstance.request(
		        method,
		        URLString,
		        parameters: parameters,
		        encoding: encoding,
		        headers: headers
		    )
		}

	* 调用时可以是这样：

		Alamofire.request(.GET, "http://private-f4f5c-onebox1.apiary-mock.com/questions/0")

	* 第一个参数method，默认忽略外部参数名
	* 第二个参数URLString，在函数声明时指定了忽略外部参数名
	* 第三、四、五个参数都指定了默认值，所以调用时可以直接不传外部参数
	* 另外，类似 [String: AnyObject]? 的定义标示参数类型是一个 Key为String类型，值是任意值的字典 或者 nil

6. notifications
	* [iOS8: Where To Remove Observer for NSNotification in Swift](http://natashatherobot.com/ios8-where-to-remove-observer-for-nsnotification-in-swift/)
7. 注意 [AnyObject] & Array & NSArray 间的区别
8. selector
	* 单纯用string来标示
	* [@selector() in Swift? - Stack Overflow](http://stackoverflow.com/questions/24007650/selector-in-swift)

###博客站
1. [Swifter - Swift 必备 tips](http://swifter.tips/)
2. [Natasha The Robot](http://natashatherobot.com/)
3. [Swift & the Objective-C Runtime - NSHipster](http://nshipster.cn/swift-objc-runtime/)
4. [Swift Blog - Apple Developer](https://developer.apple.com/swift/blog/)

###开发技巧
1. Prefer ‘let’
2. string interpolation
3. Inferred Typing
4. any properties you declare must be set to an initial value when you declare them, or in your initializer
5. 命令行输入：lldb repl 调起 REPL(Read–Eval–Print Loop) ，推出命令：":" -> "quit"

###Swift特点
1. 编译型语言；
2. LLVM；
1. 类型安全，同OC，变量和方法都有明确的返回，并且变量使用之前需要初始化；
2. 先进的语法体系：闭包、多返回、泛型、函数式编程理念；
3. 函数成为一种类型；
4. Playground；
5. iOS7及以上支持；

###问题记录
1. let 如何提高编译效率？
2. 如何做到 Inferred Typing
3. 继承：与NSObject对比
4. 动态类型（Runtime）
5. 为什么iOS6及以下不支持？

---