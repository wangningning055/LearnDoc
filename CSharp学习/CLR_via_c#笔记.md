---
title: CLR via 笔记
---

# CLR介绍
- CLR公共语言运行时，是一个可由多种语言共同使用的运行时，包括了内存管理，程序集加载，异常处理和多线程等。
- 无论是c# basic 还是IL代码，最终都会被处理为托管模块（中间语言和元数据），统一被CLR执行。
- CLR是面向程序集执行的，程序集包含了程序的自描述内容（元数据），代码（IL代码）以及资源等，在一个方法首次调用时，CLR会先
  编译IL代码（存在exe文件中）为本机的CPU指令（JIT编译），再把编译好的CPU指令放到分派好的内存块中，在进行结构构造和引用处理，JIT编译这个操作只会
  在首次调用的时候才会出现，只再次调用都只会使用内存中已有的CPU指令

## 生成打包部署和管理应用程序和类型
- exe文件内容（程序集）：PE32头，CLR头， 元数据， IL代码

### 关于元数据
- 元数据是几个表构成的二进制数据块，分为定义表引用表和清单表

### 程序集的部署
- 简单的程序部署可以将程序集合并到一个文件中，使用文件夹中的批处理程序来执行完整的程序
- 简单的控制配置，可以将配置文件（XML）一同和程序集放在一起，通过解析XML中的配置来决定程序集的管理和加载策略
- 在启动程序时CLR会在子文件中挨个搜索exe和dll文件，必要时需要在配置文件中定义clr的搜索行为，避免不必要的性能开销

## 共享程序集和强命名程序集
强命名程序集除了一般程序集含有的内容外，还包含了发布者用私钥进行的签名


```c#
using System; //命名空间
using System.Globalization;
using System.Text;
using System.Runtime.InteropServices;
using System.Reflection;
using sys_io = System.IO;//通过命名空间给类名简化
using System.Diagnostics.CodeAnalysis; 
using System.IO;
using System.Collections.Generic;
using System.Threading;
using System.Text.Json;
```
# C#运行概述
- CLR执行模型：面向运行时的语言（c#）会被统一翻译成托管模块（中间语言和元数据），托管模块是一种可移植的执行体
- 托管模块的组成部分： PE32, CLR头，元数据，IL代码
- 关于程序集：CLR并不直接和托管模块交互，而是和程序集（assembly）交互，程序集是多个模块和资源文件的分组，是程序重用和分组的最小单位，可以理解为组件；‘、
- JIT编译器： 实时编译，遇到一个没有执行的方法时Jit编译获取IL代码，再转换为CPU指令，这个CPU指令会存在内存里面，之后再次执行相关方法的时候就不会再进行jit编译了，JIT的实时编译能力具有生成新的代码的能力（写），因此一般苹果系统不适用，需要使用AOT编译
- IL代码：类似于面向对象的汇编语言，基于栈
- 可访问性修饰符：public ，private， protect， internal（只在同一个程序集（assembly）下可访问）
- CLS各个语言的公共兼容部分的最小子集
- 元数据内容： 定义表，引用表，清单表
- 程序集(assembly) 是重用、版本控制和应用安全性的基本单元，拆分程序集有益于减小工作集，减少编译时间，缓解进程空间碎片化
- [assembly: CLSCompliant(true)] //标签，表示告诉编译器检查CLS的兼容性，以此来规范的编写跨语言兼容性代码
```c#
class MainClass
{

	private static void Main(string[] args)
	{
		Console.WriteLine("Hello, World!");
		TestClass testClass = new TestClass();//CLR通过new来创建对象
		testClass.TestFunc();
		Console.Read();
	}

}

public class TestClass: Object //可简化，c#所有的类型都从Object派生
{
	public void TestFunc()
	{
		ValAndRefClass b = new ValAndRefClass();
		//b.TestFunc3();
		BaseClass a = new BaseClass();
		BaseClass a2 = new BaseClass();
		a.TestFunc();
		
		EventTestTriggerClass eventTestTriggerClass = new EventTestTriggerClass();
		eventTestTriggerClass.EventTrigger();

		InterfaceTestClass interfaceTestClass = new InterfaceTestClass();
		interfaceTestClass.InterfaceFunc1();
		interfaceTestClass.InterfaceFunc2();
		IInterfaceTest2<int> interfaceTest2 = interfaceTestClass;
		interfaceTest2.InterfaceFunc2();
		interfaceTest2.InterfaceFuncT(1);
		IInterfaceTest2<string> interfaceTest3 = interfaceTestClass;
		interfaceTest3.InterfaceFuncT("112121");

		BaseTypeClass baseTypeClass = new BaseTypeClass();
		baseTypeClass.CharTest();
		baseTypeClass.StringTest();
		baseTypeClass.EnumTestFunc();
		baseTypeClass.ArrayTestFunc();
		baseTypeClass.DelegateTestFunc();
		baseTypeClass.AttributeTestFunc();

		CoreMechanismClass coreMechanismClass = new CoreMechanismClass();
		coreMechanismClass.EXceptionTestFunc();
		coreMechanismClass.CustodianTestFunc();
		coreMechanismClass.ReflectionTestFunc();
		coreMechanismClass.SerializableTestFunc();

		ThreadProcessClass threadProcessClass = new ThreadProcessClass();
		threadProcessClass.ThreadBaseFunc();
	}
}
```

# 设计类型


## 基元类型，值类型和引用类型相关
###  类型基础
- 所有类型都由System.Object派生
- new关键字从内存里面分配一块空间给对应的类型，并执行初始化后返回实例化后的指针给目标
```c#

public class ValAndRefClass{

	TestClass testClass = new TestClass();
	void TestFunc()
	{
		bool a = (testClass is Object); //is进行类型判断

		Object b = testClass as Object; //as进行类型转换，永远不会抛出异常，转换失败会返回null，这样操作比隐式类型检查性能要高
		if(b == null)
		{
			//类型转换失败
		}
	}
```
## 关于栈和堆
- 栈主要是用来传递存储方法的参数和局部变量的
- 类分配出来的空间主要是在堆上的，即对象本身，但ClR在堆上保存一个对象时不仅仅保存了静态字段，还有类型对象指针，同步块索引以及方法索引
- 值类型一般会直接在栈上分配，值类型都继承自ValueType
```c#
	void TestFunc2()
	{
		int a = 10; //在栈上存了个a
		TestClass testClass = new TestClass(); //在堆上分配了个TestClass类型的对象，并初始化返回一个对象指针放在位于栈上的变量testClass上
		testClass.TestFunc(); //JIT编译器开始查找有没有编译过TestFunc方法，编译过就直接调用编译过的代码，没编译过就JIT编译然后再调用
	}
```
## 关于值类型和引用类型
- 值类型：在栈上分配，引用类型：在堆上分配
### 装箱和拆箱：
- 装箱：
值类型转换成引用类型，
编译器首先会自动生成装箱所需的IL代码
1.在托管堆中分配内存，内存大小为值的大小外加类型对象指针和同步索引块所需要的内存大小
2 值类型的内容复制到新内存中，注意是复制过去
3 返回对象地址
- 拆箱：
1 获取已装箱的各个字段的地址
2 把值在复制回去
- 拆箱只是获取字段地址，代价比装箱低得多
- 装箱是一个很耗费的操作，代码实现上需要尽可能的避免频繁的装箱和拆箱
```c#
	public void TestFunc3()
	{
		List<Object> tempStructs = new List<Object>();
		TempStruct tempStruct = new TempStruct();
		tempStructs.Add(tempStruct);// 装箱
		TempStruct tempStruct1 = (TempStruct)tempStructs[0]; // 拆箱

		List<TempStruct> tempStructs1 = new List<TempStruct>();
		tempStructs1.Add(tempStruct);// 明确了是一个struct的List则不会造成装箱
		

		ReferenceEquals(tempStructs, tempStructs);//在判断c#中两个对象的引用是否相等时，务必调用ReferenceEquals而不是==
		//关于GetHashCode
		TestClass testClass = new TestClass();
		TestClass testClass1 = new TestClass();

		TempStruct tempStruct2 = new TempStruct();
		TempStruct tempStruct3 = new TempStruct();
		Console.WriteLine(testClass.GetHashCode()); //依照内存地址计算,hashCode两个类每次运行不一定相同
		Console.WriteLine(testClass1.GetHashCode()); //依照内存地址计算,hashCode两个类每次运行不一定相同

		Console.WriteLine("____________________");

		Console.WriteLine(tempStruct3.GetHashCode()); //依照具体内部值计算，两个结构体内容相同hashCode就相同
		Console.WriteLine(tempStruct2.GetHashCode()); //依照具体内部值计算，两个结构体内容相同hashCode就相同
	}
```
- 结构体也是值类型，在设计代码的具体结构体的时候，明确结构体内满足这几种条件：
- 结构体的的参数不会被自身改变（甚至尽量让结构体内的字段为readonly），
- 结构体是一个简单类型的集合
- 结构体不需要继承或派生
- 结构体常常需要作为参数传递时需要结构体的实例较小（16字节或更小）
- 结构体所有的方法都应该是隐式密封的（不可重写
```c#
	struct TempStruct
	{
		public  int a;
		public  char b;
		public BaseClass baseClass;
	}
}
```



## 类型和成员继承相关

### 定义类的基本原则：
- 除非确定该类是基类并会被派生，否则都应该将该类定义为sealed（密封类，不允许派生）
- 默认将类定义为internal类，除非确定这个类会被别的程序集引用（非显示指定情况下c#编译器默认将类定义为internal）
- 类的字段代表了类的状态，尽量将类的字段设置为private，使类状态不会被外部轻易改变
- 尽量将类操作自己的方法定义为private，虚方法在最后考虑，因为虚方法需要依赖派生实现行为，同时虚方法的调用代价更高（c#无法inline）
- 尽量不要使用嵌套类型
```c#
public class BaseClass
{


	const int a = 100; //常量的定义会导致生成元数据，在编译时编译器会查找常量符号的值并嵌入代码，因此常量不需要运行时内存分配
	readonly int b; //只读字段，只允许在构造的时候赋值一次，当字段是引用时，不可改变的是引用地址而不是引用对象具体的内容
	public int publicData = 10;
	public float publicData2 = 1.1f;
	public int cc;
	private static int c;
	private int data;

	//不要不要不要不要在属性的get set方法里做赋值和取值以外的任何操作，因为额外的操作会引起外界对字段和方法的歧义
	public int Data 
	{
		get{return data;}
		set{data = value;}
	}
	public void TestFunc()
	{
		BaseClass baseClass = new BaseClass();
		BaseClass baseClass2 = new BaseClass();
		BaseClass baseClassadd = baseClass + baseClass2;
		Console.WriteLine("重载操作符相加后的结果：" + baseClassadd.publicData);
		int data = 35;
		baseClass = data;
		Console.WriteLine("转换后的结果：" + baseClass.publicData);
		baseClass.publicData = 45;
		data = baseClass;
		Console.WriteLine("转换后的结果：" + data);
		float data2 = 2.47f;
		baseClass = (BaseClass)data2;
		Console.WriteLine("转换后的结果：" + baseClass.publicData2);
		baseClass.publicData2 = 8.66f;
		data2 = (float)baseClass;
		Console.WriteLine("转换后的结果：" + data2);

		BaseClass extensionClass = new BaseClass();
		extensionClass.ExtensionFunc2(1);

	}
```
- 构造函数，不能被继承
- 不要在构造器中调用虚方法，因为如果被实例化的类型重写了虚方法，在构造器中就会执行新的虚方法的实现，但此时继承结构中的字段尚未完全初始化,此时调用虚方法会导致不可预测的行为
```c#
	public BaseClass()
	{
		Console.WriteLine("我是实例构造器");
		b = 100;
	}
	//这是类型构造器，只有在类型首次被引用调用时才会被CLR调用一次
	//类型构造器是线程安全的，每个AppDomain且只执行一次，可以在这里作为单例的实例化入口
	//类型构造器内部禁止和别的类型构造器进行互相引用，因为类型构造器的调用顺序是根据声明顺序来的，并由CLR自行调用，时机并不可控
	static BaseClass()
	{
		Console.WriteLine("我是类型构造器");
		c = 100; //类型构造器只能访问静态字段
	}

	//可变数量的参数
	//可变参数会对性能有影响，毕竟数组在堆上面分配，大多数情况下尽量避免高频使用可变参数
	public void MutiParam(params int[] values)
	{
		for(int i = 0; i < values.Length; i++)
		{
			Console.WriteLine(values[i]);
		}
	}
	//匿名类型，在方法中临时创建的类型
	public void NoneNameClass()
	{
		var o = new {name ="我是匿名类", age = 50};
		Console.WriteLine($"name: {0}, age = {1}", o.name, o.age);
	}
	//操作符重载
	//重载方法必须定义位public static
	public static BaseClass operator +(BaseClass b1, BaseClass b2)
	{
		b1.publicData += b2.publicData;
		return b1;
	}

	//这是转换构造器
	//一般是用来实现两个风牛马不相及的类之间的数据转换
	public Int32 ToInt32(BaseClass baseClass)
	{
		return baseClass.publicData;
	}
	public BaseClass ToBaseClass(Int32 data)
	{
		BaseClass baseClass = new BaseClass();
		baseClass.publicData = data;
		return baseClass;
	}
	//这里是转换操作符的重载
	public static implicit operator Int32(BaseClass baseClass)
	{
		Console.WriteLine("隐式调用了BaseClass到int的转换");
		return baseClass.publicData;
	}
	public static implicit operator BaseClass(Int32 data)
	{
		Console.WriteLine("隐式调用了Int到BaseClass的转换");
		BaseClass baseClass = new BaseClass();
		baseClass.publicData = data;
		return baseClass;
	}
	public static explicit operator float(BaseClass baseClass)
	{
		Console.WriteLine("显式调用了BaseClass到float的转换");
		return baseClass.publicData2;
	}

	public static explicit operator BaseClass(float data)
	{
		Console.WriteLine("显式调用了float到BaseClass的转换");
		BaseClass baseClass = new BaseClass();
		baseClass.publicData2 = data;
		return baseClass;
	}

}
```
### 扩展方法
- 这是扩展方法，必须在静态类中定义，第一个参数用this关键字声明对应的要扩展的类
- 扩展方法可以像类中定义的普通方法一样通过实例调用
```c#
public static class ExtensionClass
{
	public static void ExtensionFunc2(this BaseClass cl, int data)
	{
		Console.WriteLine("我是扩展方法 + " + data);
	}
}
```
## 事件相关
- EventTestTriggerClass， RegisterEventTest，EventBaseClass，EventTestArgs四个类
- 这几个类仅仅只是展示了EventHandler和EventArgs是怎么配合使用的，正式的事件系统会有正式的注册接口反注册接口和触发接口以及事件管理
- event关键字相当于对委托的一种包装,相当于声明了这个委托只能在这个类的内部被触发
- EvenHandler是一个具体的委托类型,常常和event声明以及TEventArgs配合使用实现事件的发布和订阅
```c#
//这个类负责触发事件
public class EventTestTriggerClass
{
	public void EventTrigger()
	{
		RegisterEventTest registerEventTest = new RegisterEventTest();
		registerEventTest.RegisterEvent();

		EventTestArgs eventTestArgs = new EventTestArgs();
		eventTestArgs.Fill("王1111", 10);
		registerEventTest.eventBaseClass.TriggerEvent(eventTestArgs);
	}
}

//这个类负责事件的注册
public class RegisterEventTest
{
	public EventBaseClass eventBaseClass = new EventBaseClass();
	public void RegisterEvent()
	{
		eventBaseClass.TestEvent += SpecificFunc;
		eventBaseClass.TestEvent += SpecificFunc2;
	}
	public void UnRegisterEvent()
	{
		eventBaseClass.TestEvent -= SpecificFunc;
		eventBaseClass.TestEvent -= SpecificFunc2;
	}

	//EventHandler的委托是(Object sender, EventArgs evt)形式的，所以事件触发也要是这个形式
	public void SpecificFunc(Object? sender, EventTestArgs evt)
	{
		Console.WriteLine($"事件1触发了，name:{evt.Name}, age:{evt.Age}");
	}
	public void SpecificFunc2(Object? sender, EventTestArgs evt)
	{
		Console.WriteLine($"事件2触发了，name:{evt.Name}, age:{evt.Age}");
	}
}

//这个类负责处理事件具体的触发
public class EventBaseClass{
	public event EventHandler<EventTestArgs> TestEvent;
	public EventBaseClass()
	{
		TestEvent += BaseTrigger;
	}

	public void TriggerEvent(EventTestArgs e)
	{
		//通过引用来触发事件，防止触发中途事件为空，线程安全
		EventHandler<EventTestArgs> eventHandler = TestEvent;
		if(eventHandler != null)
		{
			TestEvent(this,e);
		}
	}
	public void BaseTrigger(Object? sender, EventArgs evt)
	{
		//这是基类里面的基本触发函数
	}

}

//这个类定义了具体事件触发时所需要携带的信息，即事件的参数
public class EventTestArgs:EventArgs
{
	private string name = "";
	private int age = -1;
	public string Name{
		get{return name;}
	}
	public int Age{
		get{return age;}
	}
	public void Fill(string _name, int _age)
	{
		name = _name;
		age = _age;
	}
	public void Reset()
	{
		name = "";
		age = -1;
	}

```

## 泛型相关
```c#
	//这是一个泛型类,同时使用了where约束了泛型T必须实现了TInterface的泛型接口,并且约束T类型必须有一个无参的公共构造函数
	public class TClass<T> where T:TInterface<T>, new()
	{
		public T Value;
		//泛型方法：这是一个TT作为参数类型的泛型方法,约束了TT类型必须继承自BaseClass,并且实现了TInterface接口
		public void TFunc<TT>(TT args)where TT : BaseClass, TInterface<T>
		{

		}
		//泛型方法：这是一个TT作为参数类型的泛型方法,约束了TT类型是一个引用类型
		public void TFunc2<TT>(TT? args)where TT : class
		{
			args = default;
		}
		//泛型方法：这是一个TT作为参数类型的泛型方法,约束了TT类型是一个值类型
		public void TFunc3<TT>(TT args)where TT : struct
		{
			args = default(TT);
		}
	}
	//这是一个泛型接口
	public interface TInterface<T>
	{
		public void Func(T val);
	}
}
```


## 接口
- CLR不支持多继承，所以对于多继承菱形继承等情况，需要使用接口来实现
- 接口只提供方法签名，并不提供方法的具体实现，但凡继承了接口的类，都要把接口内的方法全部实现
- 接口的约定俗成的命名方式是I开头
- 接口常常配合泛型使用，添加统一接口约束的泛型可以统一通过泛型调用接口方法，泛型的约束是接口的常用使用场景
- 泛型接口在处理值类型的时候会很有优势，会减少很多装箱和拆箱，同时泛型接口可以多次实现
- 当不清楚是否应该选择接口还是基类时，可以定义接口的同时实现继承接口的基类，这样既可以选择继承基类，也可以选择实现接口

```c#
public interface IInterfaceTest
{
	public void InterfaceFunc1();
	public void InterfaceFuncCommon();

}
public interface IInterfaceTest2<T>
{
	public void InterfaceFuncCommon();
	public void InterfaceFunc2();
	public void InterfaceFuncT(T val);

}
```

- 接口支持多继承, 泛型接口IInterfaceTest2支持多种实现和继承
```c#
public class InterfaceTestClass : IInterfaceTest, IInterfaceTest2<int>, IInterfaceTest2<string>
{
	//继承接口后必须把接口的内容一一实现
	public void InterfaceFunc1()
	{
		Console.WriteLine("我是普通的接口方法");
	}
	public void InterfaceFunc2()
	{
		Console.WriteLine("我是具体类的接口方法");

	}

	//当不同接口拥有相同的方法签名定义的时候，可以通过显式的方式来定义不同的接口方法，显式实现的方法必须将类转换为对应的接口类型才能调用
	//显式的接口方法实现需要谨慎使用，因为涉及到很多类型转换，容易造成装箱，同时不能被继承类调用（同样需要转换成具体接口类型才能调用）
	//因此显式的接口方法实现应该要尽量少的使用
	void IInterfaceTest.InterfaceFuncCommon()
	{
	}
	void IInterfaceTest2<int>.InterfaceFuncCommon()
	{
	}
	void IInterfaceTest2<string>.InterfaceFuncCommon()
	{
	}

	//这里显式的实现了接口的方法，这样在调用接口方法时会根据当前类型是具体类还是接口类型来调用不同的方法
	void IInterfaceTest2<int>.InterfaceFunc2()
	{
		Console.WriteLine("我是显式实现的接口方法");

	}
	//泛型接口的多实现
	public void InterfaceFuncT(int val)
	{
		Console.WriteLine("我是泛型接口的int实现");

	}
	//泛型接口的多实现
	public void InterfaceFuncT(string val)
	{
		Console.WriteLine("我是泛型接口的string实现");

	}

}
```

# 基本类型
```c#
public class BaseTypeClass
{

	public void CharTest()
	{
		char a = '1';
		char b = '是';
		char c = '+';
		char d = '^';
		char e = '$';

		//UnicodeCategory是在System.Globalization命名空间下的类型，通过Char.GetUnicodeCategory可以获取char的具体类型
		Action<char> logTypeFunc = (char data) =>{
			UnicodeCategory type = Char.GetUnicodeCategory(data);
			Console.WriteLine($"字符类型是{type}");
		};
		logTypeFunc(a);
		logTypeFunc(b);
		logTypeFunc(c);
		logTypeFunc(d);
		logTypeFunc(e);

		//Char还提供了方便的IsNumber等可以直接判断的函数
		bool isNumber = Char.IsNumber(a);
		bool IsLetter = Char.IsLetter(b);

		//类型转换
		int convertInt = (int)a;
		Console.WriteLine($"转换后的int是{convertInt}");
		char convertChar = (char)convertInt;
		Console.WriteLine($"转换后的char是{convertChar}");

		convertInt = Convert.ToInt32(a);
		Console.WriteLine($"转换后的int是{convertInt}");
		convertChar = Convert.ToChar(convertInt);
		Console.WriteLine($"转换后的char是{convertChar}");
	}
```

### 字符串
- 字符串是引用类型,每一个string都是一个对象,string字符串是不可变的,每生成一个新的字符串都会导致新的对象生成
- CLR底层针字符串不可变特性定制了字符串留用机制,即相同的字符串内容共用一个字符串对象
- 执行大量字符串操作时,需要使用StringBuilder,StringBuilder通过使用char数组来避免在string连接过程中产生大量废弃的string驻留对象
```c#
	public void StringTest()
	{

		//字符串连接：字符串的字面值进行+连接，编译器会在编译时连接，最后只会生成一个字符串,这里内存里面只会有hello world一个字符串
		string concatStringTest1 = "hello" + " world";

		string concat1 = "aaa";
		string concat2 = "bbb";
		//运行时不要用+来连接字符串，这样会生成concat1， concat2，concatRes三个对象
		string concatRes = concat1 + concat2;



		//c#的字符串也定义了转义符，带有回车和换行的可以这么写
		string newLineTest = "hello\n\rworld";
		Console.WriteLine(newLineTest);
		//但是并不推荐这样做，跨平台可能出问题，可以使用Environment里面的newLine属性来做，这样跨平台不会出问题
		//Environment里面还有很多别的东西，例如Environment.CommandLine + Environment.MachineName + Environment.NewLine + Environment.UserName;
		newLineTest = "hello" + Environment.NewLine + "world";
		Console.WriteLine(newLineTest);
		//有时候字符里面包含转义字符串,但是又希望转义的字符串也是字符串,可以使用@(逐字字符串)来表示后面是一整个字符串(即忽略转义符)
		string totalString = @"hello\n\rworld";
		Console.WriteLine(totalString);

		//两者的hashCode是相同的，实际是同一个对象
		string sameStr1 = "qwert";
		string sameStr2 = "qwert";
		Console.WriteLine($"1: {sameStr1.GetHashCode()}, 2:{sameStr2.GetHashCode()}");

		//在进行频繁的string操作时，需要用到stringBuilder
		//stringBuilder可以看作是一个大小可变的Char数组，通过char数组来表示可变内容的string

		StringBuilder stringBuilder = new StringBuilder();
		//默认容量是16，容量不够时会倍增
		Console.WriteLine($"里面的字符长度：{stringBuilder.Length}, 容量：{stringBuilder.Capacity}");
		stringBuilder.Append("wawawaw");
		stringBuilder.AppendLine();
		stringBuilder.Append("erererer");
		//最后通过tostring来返回最终的string对象
		string builderRes = stringBuilder.ToString();
		Console.WriteLine(builderRes);

		//关于字符编码，utf-16即unicode编码，把一个字符编码成两个字节，不对字符产生影响，性能出色
		//utf-8，字符部分编码为1，2，3，4字节，这两种编码格式是目前最常用的编码格式
		string codeString = "我是准备编码的字符串，我要被编成utf-8";

		Encoding encodingUTF8 = Encoding.UTF8;
		byte[] UTF8bytes = encodingUTF8.GetBytes(codeString);
		Console.WriteLine($"utf8结果：{BitConverter.ToString(UTF8bytes)}"); 

		string deCodingUTF8 = encodingUTF8.GetString(UTF8bytes);
		Console.WriteLine($"utf8解码结果：{deCodingUTF8}"); 
	}
```
### 枚举
- 枚举类型和位标志,枚举类型是值类型
- 枚举内部定义的值都是常量值
```c#
	public enum EnumTest
	{
		white,
		red,
		blued,
		black
	}
	//枚举类型具有基元类型，c#默认为int，也可以是byte或者long等
	public enum EnumByte:byte
	{
		enum1,
		enum2
	}
	//很多时候枚举类型用位来表示,表示某种开关或者状态
	//Flags标签可以让c#认为这个枚举是标志位,在调用不同枚举之间的|操作后的tostring时,会按对应的逻辑处理
	[Flags]
	public enum EnumBit
	{
		//一般会有一个状态完全归零的状态位None
		None = 0,
		bit0 = 1,
		bit1 = 1 << 1,
		bit2 = 1 << 2,
		bit3 = 1 << 3,
		bit4 = 1 << 4,
		bit5 = 1 << 5,
		bit6 = 1 << 6,

		//有时候也会拿已有的位来表示新的位,以减少代码量
		bit7 = EnumBit.bit1 | EnumBit.bit2,
		bit8 = EnumBit.bit0 | EnumBit.bit2

	}
	public void EnumTestFunc()
	{
		//遍历枚举
		EnumTest[] enumTests = (EnumTest[])Enum.GetValues(typeof(EnumTest));
		for(int i = 0; i < enumTests.Length; i++)
		{
			Console.WriteLine($"枚举名：{enumTests[i]}, 枚举值{(int)enumTests[i]}");
		}
		//通过isDefine方法来判断枚举是否被定义,可以判断是否有对应的枚举名，也可以判断是否有对应的枚举值
		//isDefine方法很方便，但是很慢，因为执行的是带有大小写的检查，同时内部的实现有通过反射，不要在频繁调用的地方使用
		bool isDefine1 = Enum.IsDefined(typeof(EnumTest), "white");
		bool isDefine2 = Enum.IsDefined(typeof(EnumTest), 10);
		Console.WriteLine($"判断1：{isDefine1}, 判断2{isDefine2}");

		//位标志的枚举可以让使用者身上仅仅使用一个字段来表示同时有多个状态
		//flagState表示身上同时有bit1和bit2的状态,在判断时,仅需要和对应的位枚举进行与操作,即可获得对应的状态是否开启
		//只要&出来的结果大于0,则说明有状态
		int flagState = (1 << 2) + 1;
		string flagStateStr = Convert.ToString(flagState, 2);
		int res1 = flagState & (int)EnumBit.bit0;
		int res2 = flagState & (int)EnumBit.bit2;
		int res3 = flagState & (int)EnumBit.bit8;
		int res4 = flagState & (int)EnumBit.bit3;
	
		Console.WriteLine($"基本状态是:{flagStateStr}");
		Console.WriteLine($"结果是:{res1}");
		Console.WriteLine($"结果是:{res2}");
		Console.WriteLine($"结果是:{res3}");
		Console.WriteLine($"结果是:{res4}");

		EnumBit enumBit = EnumBit.bit1;
		EnumBit enumBit2 = EnumBit.bit3;

		EnumBit enumBit3 = enumBit | enumBit2;
		EnumBit enumBit4 = enumBit & enumBit2;
		EnumBit enumBit5 = enumBit | EnumBit.bit2;

		Console.WriteLine($"枚举是:{enumBit3}");
		Console.WriteLine($"枚举是:{enumBit4}");
		Console.WriteLine($"枚举是:{enumBit5}");
	}
```
### 数组
```c#
	public void ArrayTestFunc()
	{
		int[] array1 = new int[10];
		//二维数组
		int[,] array2Vec = new int[10, 10];

		//交错数组(数组嵌套)
		int[][] arrayCross = new int[10][];
		arrayCross[1] = new int[2];

		//数组初始化器
		arrayCross[2] = new int [3]{1,2,3};
		arrayCross[3] = new int [4]{6,7,8,9};

		//数组拷贝,ArayCopy是浅拷贝
		Array.Copy(arrayCross[2], arrayCross[3], 3);

		Console.WriteLine("数组结果是:" + arrayCross[3].ToString());

		//对于引用类型的数组,声明时会自动为这个数组类型实现IEnumerable,ICollection,IList接口
		//所以这个数组引用也可以当作参数传到对应的迭代函数中
		BaseClass[] baseClassArray = new BaseClass[10];
		Action<IEnumerable<BaseClass>> actionEnumerable =(baseClass) =>{};
		Action<ICollection<BaseClass>> actionCollection =(baseClass) =>{};
		Action<IList<BaseClass>> actionList =(baseClass) =>{};
		actionEnumerable(baseClassArray);
		actionCollection(baseClassArray);
		actionList(baseClassArray);

		
	}
```
### 委托
- 在声明委托时,实际是定义了一个对应名字的委托类,类里面实现了构造器,Invoke,BeginInvoke,EndInvoke
- 委托内部有_target持有实例的对象,_methodPtr持有函数指针,_invocationList持有委托链
- 所以委托实际上是调用的方法和该方法持有的对象的包装
- 注意delegate和Delegate的区别,小写的代表的是c#关键字,表示声明一个委托,大写代表的是委托类型,所有带有委托声明的字段,他的类型都是继承自Delegate
- 如果存在在调用委托时并不知道委托的类型,可以通过MethodInfo和DynamicInvoke以及反射来声明一个未指定类型的委托并调用

- 尽量使用c#提供的Action<>(无返回值泛型委托)和Func<>(有返回值泛型委托)来定义委托,可以减少委托的堆积和增加复用性,最多可以提供16个参数的实现
```c#
    public delegate int DelegateInstance1(int data);
	public delegate void Action();
	public delegate void Action<T>(T a);
	public delegate T  Func<T>();
	public delegate T1 Func<T1, T2>(T2 a);

	public void DelegateTestFunc()
	{
		DelegateInstance1 delegateInstance1 = new DelegateInstance1(DelegateTestFunc1);
		delegateInstance1(1);
		delegateInstance1 += DelegateTestFunc2;
		delegateInstance1 += DelegateTestFunc3;
		//使用()直接调用和使用Invoke调用没有区别,使用()直接调用编译器会生成Invoke来调用
		delegateInstance1(1);

		//委托链调用时,会按添加顺序依次调用,只返回最后一个函数的返回值,中间的返回值都被丢弃掉了
		//同时如果委托链中有一个委托抛出异常或者阻塞很长一段时间,会导致委托链断掉,后续的委托方法都不会被调用
		int res = delegateInstance1.Invoke(1);
		Console.WriteLine($"最后的委托调用结果是{res}");

		//鉴于委托链有可能中断的情况,可以使用GetInvocation方法来获取List中的每个Invoke对象,从而挨个调用
		Delegate[] delegates = delegateInstance1.GetInvocationList();
		foreach(DelegateInstance1 funcs in delegates)
		{
			funcs(1);
		}

		//可以使用系统自带的Action和Func,同时委托可以使用匿名函数来赋值,增加代码的可读性
		Action<int> action = (a) =>{
			Console.WriteLine($"我的lambda表达式的匿名函数,{a}");
		};
		Func<int, int> func = (a) =>
		{
			Console.WriteLine($"我的lambda表达式的匿名函数,{a}");
			return a + 10;
		};
		action(2);
		int funcRes = func(2);

	}
	public int DelegateTestFunc1(int data)
	{
		Console.WriteLine("我是委托调用1");
		data = data + 1;
		return data;
	}
	public int DelegateTestFunc2(int data)
	{
		Console.WriteLine("我是委托调用2");
		data = data + 2;
		//如果中途抛出异常,后续的委托将不会再执行
		//throw new Exception();
		return data;
	}
	public int DelegateTestFunc3(int data)
	{
		Console.WriteLine("我是委托调用3");
		data = data + 3;
		return data;
	}
```
### 定制特性
- 定制特性可以宣告式的为自己的代码添加注解来实现特殊的功能,定制特性带来的可扩展的元数据信息能在运行时查询,从而动态的改变代码的执行方式
- 例如[DllImport] 表示方法的实现在指定Dll的非托管代码中
- 例如[Serializable] 表示一个实例的字段可以序列化和反序列化
- 所有的定制特性其实都是一个类的实例，类的实现都要继承自Attribute类，实际使用时名称有所简化，DllImport的实际特性类名叫DllImportAttribute
- 特性最后通过反射提供的isDefine和GetCustomAttributes来做判断，注意在AOT编译场景中，这两个API不能使用，需要自行实现特性的注册与查找
- 这里给方法定义了DllImport特性（名称简化了后面的Attribute），并调用构造器把"ExternDll"传进去，同时设置DllImport特性类的字段CharSet和SetLastError内容
```c#
	[DllImport("ExternDll", CharSet = CharSet.None, SetLastError = true)]
	extern  static void AttributeExpFunc();
```
- 自定义一个自己的特性类,里面需要有一个公共的构造器，可以添加自己的属性和字段
```c#
	sealed class TestAttribute :Attribute
	{
		public TestAttribute(string name)
		{
			Name = name;
		}
		public string Name;
		public int Type;

	}
	//应用测试的自制特性
	[Test("A", Type = 1)]
	public class AttributeUseClass
	{

	}
```
#### 特性的具体判断
- 通过isDefine或者GetCustomAttributes来获取到特性的具体内容，从而让类根据不同的特性来做出不同的行为
```c#
	public void AttributeTestFunc()
	{
		AttributeUseClass attributeUseClass = new AttributeUseClass();
		//使用Type的isDefine方法来检查是否添加了特性，使用GetCustomAttributes获取特性内容
		//注意在禁止反射（禁止JIT编译）的情况下，这两个接口不能使用
		if(attributeUseClass.GetType().IsDefined(typeof(TestAttribute), false))
		{
			Console.WriteLine("类添加了特性");
			Object[] objects = attributeUseClass.GetType().GetCustomAttributes(false);
			foreach (TestAttribute testAttribute in objects)
			{
				Console.WriteLine($"当前特性的内容是：{testAttribute.Name}, {testAttribute.Type}");
			}
		}
		else
		{
			Console.WriteLine("类没有添加特性");
		}

	}
```
### 可空值类型
```c#
	//c#的值类型是不允许为null的，由此需要用可空值类型来表示
	void NullableValueTest()
	{
		//定义一个可空的int
		Nullable<int> a = null;
		//简化定义
		int? b = null;
		//空接合操作符？？,上下两句话等价
		int? c = a == null? a: b;
		c = a??b;
	}
}
```

# 核心机制
## 异常处理
- 异常常常代表着某些操作失败，并伴随着程序的不正确状态，异常处理需要捕获具体的异常并实际处理，尝试把程序的状态复原到正常
- 如果是不可复原状态的异常,就不应该捕获,而应该直接终止程序并在测试阶段解决这个bug
```c#
public class CoreMechanismClass
{
	//

	public void EXceptionTestFunc()
	{
		//在这里检查，有报错会抛出异常
		try{
			Console.WriteLine("我是try块的内容");
			throw new Exception();
		}
		//在这里捕获异常
		//实际使用中需要去捕获具体的异常并具体进行回滚处理,在真正运行而非测试的程序环境下,永远都不要直接捕获Exception基类异常
		catch(Exception e){

			Console.WriteLine("我是catch块的内容,捕获到了异常：" + e.StackTrace);
			string message = e.Message;//这个字段表示了异常的内容是什么
			string? trace = e.StackTrace;//这个字段表示了异常的堆栈
		}
		//在这里做try块里面的清理工作，try块打开了文件，这里就该关闭文件，不然未捕获的异常发生后，文件一直都关闭不了，直到下次垃圾回收
		//这里的代码无论是否抛出异常，都会保证执行,回收锁，关闭io，关闭文件都可以放在这里执行
		finally
		{
			Console.WriteLine("我是finally块的内容");

		}
		//假如上面有未捕获的异常，这里的内容将不会再执行
		Console.WriteLine("我是后续的方法的内容");

	}
```
## 托管堆
- CLR要求所有的对象都从托管堆分配，进程刚一开始时，CLR划分一个地址空间为托管堆区域，一个区域被非垃圾对象填满后，CLR会继续划分区域，直到内存被填满
- 期间CLR会维护一个NextObjPtr指针，该指针指向下一个新对象的地址，由此可见，CLR在绝大部分情况下，在短时间内一起分配的对象地址是紧凑的，他们也应该在差不多的同一时间被访问
- 在新的new操作符时，CLR发现第0代满了，会触发垃圾回收
### 回收概述:
- 一般的托管回收使用的是引用计数的方法，即被引用一次计数加1，消除一次引用计数减一，计数为0就回收，但是引用计数解决不了循环应用的问题，两个不再需要的
垃圾对象互相引用导致计数永远不为0而得不到回收,所以CLR使用的是引用跟踪的方式,即计算对象是否可达,是否可达的判断标准是该引用类型的变量(根)是否引用了具体的对象
- CLR开始垃圾回收时,会暂停当前线程中的所有线程,然后标记托管堆上的所有的对象为不可达(即需要回收),然后CLR检查所有的活动根,该根对应的引用对象在同步索引块上
被标记为可达,并检查该对象引用了哪些根,并同样标记为可达,之后再遍历活动根的适合,遇到已经标记为可达的就跳过检查,这样就避免了循环引用的问题,检查完毕后,
CLR会进行GC压缩,即把可达的对象内存移到连续的地址空间下,然后将对应的活动根的指针移到改变后的具体对象的地址位置,内存压缩是为了解决堆的空间碎片化问题(过于碎片化的空间无法分配过大的对象)
### 回收分代:
- 如果每次垃圾回收都遍历整个堆的话,代价会过大,所以CLR的垃圾回收有代的概念,关于代,有这样的共识:对象越新,生存期越短,对象越老,生存期越长,回收堆的一部分,比回收整个堆快
所以新分配的对象没有经历过垃圾回收，他们就是第0代，当第0代预算满了，触发回收，剩下存活的对象将内存整理压缩被放到第1代内存中（最早分配的连续内存地址），之后新的对象分配
依然是第0代，0代预算满了继续触发垃圾回收再将剩余的存活对象放到第1代，直到第一代的预算也满了，就触发第0代和第1代的垃圾回收，将第一代存活的对象以同样的方式放到第二代，第0
代存活的对象放到第一代，CLR最多只有2代，在整个垃圾回收期间，每次都会计算整个堆的对象到达图，分代主要针对于内存移动回收和压缩，同时CLR会根据代内对象的回收频率大小等动态的调整代的预算
### 大对象堆：
- CLR认为大于等于85000字节（当下CLR的标准，之后可能会变，总之就是很大）的对象是个超大对象，在内存上移动它代价过于大，所以这种大对象CLR会分派到大对象堆，大对象总是第2代，即
非常不易被回收的对象，大对象一般是超长的字符串或者网络传输来的io字节数组
### Finalize方法（~classname方法）与dispose：
- finalize方法形式上类似于c++的析构函数，实际内部原理完全不同，该方法是在垃圾回收完毕才执行的，调用时机不明确，同时会导致对象及对象的引用全部升代，因此几乎不推荐使用
- 实在某些对象需要在被销毁时进行某些代码退出，可以使用SafeHandle类
- 实现IDisposable接口来使用Dispose方法来手动标记一个类之后永远都不会被使用了，但是依然不建议使用
### 关于对象池：
- 垃圾回收总是在第0代满了就发生，而垃圾回收性能消耗又很大，所以常见的方法是程序自己维护对象池，通过对对象的状态重置和再设来复用对象，对象池会固定引用对象来避免垃圾回收
以及对象的堆分配，但是池并不是越多越好，长久且大量的对象池会导致CLR内存吃紧，甚至出现内存无法分配报错导致程序崩溃（OutOfMemoryException），所以合理的内存池设计应该在内存吃紧的时候将空闲对象设置为弱引用，或者对池进行清洗，以缓解内存压力
```c#
	public void CustodianTestFunc()
	{
		//手动回收，手动回收一般对性能影响巨大，确认程序中确实产生了一次大量的垃圾对象，才可以手动调用一次垃圾回收
		//指定回收2代以及小于等于2代的内存，Optimized模式表示只有存在大量垃圾时才会真正执行回收，否则这次函数调用什么都不会发生
		GC.Collect(2,GCCollectionMode.Optimized);

		//检查某一代垃圾回收了多少次评判有时候GC频繁但效果差的时候
		GC.CollectionCount(1);
		//检查托管堆当前使用了多少内存
		GC.GetTotalMemory(false);

		var info = GC.GetGCMemoryInfo();
		//上次垃圾回收的最大内存预算
		Console.WriteLine($"最大内存预算：{ info.TotalAvailableMemoryBytes}");
		//上次垃圾回收时用了的内存总量
		Console.WriteLine($"内存使用量：{info.MemoryLoadBytes}");
		//当前整个堆的大小
		Console.WriteLine($"当前堆的大小{info.HeapSizeBytes}");

	}
	//这是一个给内存填满的测试，在做池的总管理时，需要考虑到内存炸掉的情况每次分配新的new对象时，需要估算当前的内存占用
	//unity可以用SystemInfo.systemMemorySize;来获取当前设备的物理内存
	public class MemoryTestClass()
	{
		//一个byte是8bit, 1024个byte是1kb
		byte[] a = new byte[10 * 1024 * 1024];
	}
	public void MemoryTestFunc()
	{
		List<MemoryTestClass> baseClasses = new List<MemoryTestClass>();
		while(true)
		{
			try
			{
				baseClasses.Add(new MemoryTestClass());
				
				Console.WriteLine($"已分配{baseClasses.Count * 10}MB");
			}
			catch(OutOfMemoryException e)
			{
				Console.WriteLine($"堆炸了{e.Message}");
				//立即进行池的固定对象的释放，然后再重新分配
				baseClasses.Clear();
				//对list大小缩减
				baseClasses.TrimExcess();
				//立即进行垃圾回收
				GC.Collect();
				try
				{
					baseClasses.Add(new MemoryTestClass());
				}
				catch(Exception ee)
				{
					Console.WriteLine($"尝试再分配堆炸了{ee.Message}");
					break;
				}
				
			}
			finally
			{
				//Console.WriteLine($"最终执行代码");
			}
		}
	}
```

## 反射
- 反射允许在运行时发现并使用编译时还不了解的类型和成员
- 反射的运行时才发现类型的方式破坏了类型安全性
- 反射因为不是编译期间的，所以类型不明确，报错难以查找，在AOT环境下常常要注意代码裁剪，甚至不适用反射
- 反射的调用与引用不会在编译期被察觉，所以单纯只会被反射调用的类可能在AOT编译的时候被裁剪掉，导致调用报空
- 反射的速度很慢，反射大部分情况依赖字符串查找，，程序集大量的不分大小写的字符串查找很耗时，同时反射调用方法也会因为参数解包和类型确认而更花费时间
- 反射很多时候和JIT编译相关联，在AOT场景下部分反射行为会被禁止或不推荐（System.Reflection.Emit， MakeGenericType() / MakeGenericMethod()，
- System.Linq.Dynamic，Assembly.LoadFrom磁盘程序集加载），在有可能需要AOT编译的情况下，反射要小心使用，所有类型在调用前都要是被IL生成过的
- 因此具体实现中需要尽量避免使用反射，能用接口实现的动态实例构造尽量不考虑反射，能switch case就不考虑反射
```c#
	//[DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.PublicConstructors)]
	public class TestReflectionGenClass
	{
		public string StrVal = "test";
		public  int IntVal = 3;
		//即使是new的情况下，AOT也有可能裁剪掉构造器，所以需要这样打个标签
		//[DynamicDependency(DynamicallyAccessedMemberTypes.PublicConstructors, typeof(TestReflectionGenClass))]
		public TestReflectionGenClass()
		{
			IntVal = 1;
			Console.WriteLine($"反射调用了无参构造函数");
		}
		//[DynamicDependency(DynamicallyAccessedMemberTypes.PublicConstructors, typeof(TestReflectionGenClass))]
		public TestReflectionGenClass(int data1, string data2)
		{
			IntVal = data1;
			StrVal = data2;
			Console.WriteLine($"反射调用了有参构造函数：{data1}， {data2}");
		}
		public void TestFunc(int data)
		{
			Console.WriteLine($"我是被反射调用的方法：{data}");
		}
	}
	public void ReflectionTestFunc()
	{
		//获取一个对象的类型,typeof和GetType()获得的实际上是一个类型对象的引用
		//尽量使用typeof，而不是GetType
		BaseClass baseClass = new BaseClass();
		bool isType = typeof(BaseClass) == baseClass.GetType();

		FindType();
		GenClassByType(typeof(TestReflectionGenClass));
	}
	//通过反射来查找自己要的类型
	public void FindType()
	{
		Assembly[] assemblies = AppDomain.CurrentDomain.GetAssemblies();
		foreach(Assembly assembly in assemblies)
		{
			Type[] types = assembly.GetTypes();
			foreach(Type typeSingle in types)
			{
				if(typeSingle.ToString().Contains("TestReflectionGenClass"))
				{
					Console.WriteLine($"找到的类型：{typeSingle.ToString()}");
					return;
				}
			}
		}
		Console.WriteLine($"没找到：");

	}
	//通过反射生成具体的类，并获取到类的字段，方法，构造器
	public void GenClassByType(
		[DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.PublicParameterlessConstructor)]
		Type type)
	{
		//对于只有会被反射生成的类，AOT编译时会认为这些类没有被引用过，是“不被需要的”，所以编译阶段会把这个类的代码裁剪掉，所以对于那些仅仅会被反射生成的类
		//不只是代码，构造函数也一样，没有被使用过的构造函数也会被裁剪
		//裁剪会发生在整个类的各种地方，一个构造器，一个方法，某些字段，一旦完全没调用过，就有可能被裁剪
		//在unity开发环境下，也可以使用UnityEngine.Scripting;空间下的[Preserve]标签来确保类型不会被裁剪Copy CopyUsed Link
		//在AOT环境下可以设置裁剪力度，过高的裁剪力度会导致即使用new声明了部分构造函数也会被裁剪，可以在.csproj里面设置<TrimmerDefaultAction>CopyUsed</TrimmerDefaultAction> 
		//其中Copy表示全复制，CopyUsed表示尽量保留，部分裁剪，Link表示最大力度的裁剪
		TestReflectionGenClass dummyClass = new TestReflectionGenClass();
		TestReflectionGenClass dummyClass2 = new TestReflectionGenClass(1, "2");

		//简单直接的生成类型实例,尽量直接用Type生成，字符串查找代价太大
		Object? obj = Activator.CreateInstance(type);
		//带有有参构造函数的生成
		Object? obj11 = Activator.CreateInstance(type, [2, "aaa"]);
		//也可以通过字符串生成（别的Assembly里面）
		//Object? obj2 = Activator.CreateInstance("AssemblyName", "TypeName");
		//或Assembly直接生成
		Object? obj3 = Assembly.GetExecutingAssembly().CreateInstance("TestReflectionGenClass");
		TestReflectionGenClass? testReflectionGenClass = obj as TestReflectionGenClass;
		TestReflectionGenClass? testReflectionGenClass2 = obj3 as TestReflectionGenClass;

		if (testReflectionGenClass != null)
		{
			testReflectionGenClass.TestFunc(3);
		}
		if (testReflectionGenClass2 != null)
		{
			testReflectionGenClass2.TestFunc(3);
		}

		//但是一般不直接用CreateInstance，一般会直接保存构造器，然后一直用构造器来生成对应的类,这样会省很多
		//获取无参构造函数
		ConstructorInfo? constructorInfo1 = type.GetConstructor(Type.EmptyTypes);
		Object? objByConstructor = constructorInfo1?.Invoke(null);

		//获取有参构造函数
		ConstructorInfo? constructorInfo2 = type.GetConstructor([typeof(int), typeof(string)]);
		Object? objByConstructor2 = constructorInfo2?.Invoke([1, "222"]);//这里装箱了一次
		
		//读写类里面具体的字段和方法
		if(objByConstructor != null)
		{
			FieldInfo? fileInfo = objByConstructor.GetType().GetField("StrVal");
			if(fileInfo != null)
			{
				string? str = fileInfo.GetValue(objByConstructor) as string;
				Console.WriteLine($"我读到了类的字段：{str}");
				fileInfo.SetValue(objByConstructor, "rrrrr");
				string? str2 = fileInfo.GetValue(objByConstructor) as string;
				Console.WriteLine($"我读到了类的字段2：{str2}");
			}
			MethodInfo? methodInfo = objByConstructor.GetType().GetMethod("TestFunc");
			methodInfo?.Invoke(objByConstructor, [3]);
			//通过构造委托来让反射方法调用代价最小化
			if(methodInfo != null)
			{
				Action<int> action = methodInfo.CreateDelegate<Action<int>>(objByConstructor);
				action(10000);
			}
		}
		// 可以使用handler来避免Info的大对象缓存导致的内存压力
		//RuntimeTypeHandle runtimeTypeHandle = Type.GetTypeHandle(type);
		//RuntimeFieldHandle runtimeFieldHandle = fileInfo.FieldHandle;
		//RuntimeMethodHandle runtimeMethodHandle = methodInfo.MethodHandle;


	}
```
## 运行时序列化
- 序列化是将对象转换成字节流的过程，反序列化则是将字节流转换回对象的过程
- 序列化后对象可以保存在系统，磁盘，剪贴板上，在程序下一次运行的时候直接恢复对- 象状态
- 序列化以后也可以直接通过网络将对应的对象直接传到另一个端上运行
- 序列化一个是基于字节流的处理，可以进行字节流的加密和压缩
- C#当前已经弃用BinaryFormatter，不要再用字节流进行序列化，c#原生提供了JSON和XML序列化
- 在unity的场景下Serializable标签表示这个类可以序列化SerializeField是让私有字段也能序列化，并显示在Inspector上
```c#

	public class SerializableTestClass
	{
		public int a = 100;
		public string b = "333";
		private int c = 40;
		public SerializableTestClass()
		{
		}
		public SerializableTestClass(int _a , string _b)
		{
			a = _a;
			b = _b;
		}
		public void SetC(int data)
		{
			c = data;
		}
		public void ConsolData()
		{
			Console.WriteLine($"我是序列化对象，我的内容是：{a}， {b}, {c}");
		}
	}
	public void SerializableTestFunc()
	{
		SerializableTestClass serializableTestClass = new SerializableTestClass(40, "哇大碗大碗的");
		serializableTestClass.SetC(90);
		//使用JSON序列化,使用json序列化的类必须要有一个无参的构造函数，Json只能序列化公有字段
		string jsonStr = JsonSerializer.Serialize(serializableTestClass);
		SerializableTestClass? serializableTestClass1 = JsonSerializer.Deserialize<SerializableTestClass>(jsonStr);
		serializableTestClass1?.ConsolData();

	}
} 
```

# 线程处理机制
## 线程基础
- 进程是资源，线程是任务，程序启动开启进程，进程里面可以调度多个线程
- 一个线程里面存了内核对象，用户和内核栈，TEB（环境数据），dll引用等，过多的线程可能会导致内存压力
- 对于单核的cpu，一旦新建了一个线程，cpu会给这个线程一段时间执行（一个时间片），一旦时间（一般是30ms）到了，cpu会切换上下文到另一个线程去执行，时间片流转
- 但是多核的cpu会执行真正的并发，比如8核的cpu，会真正并发的执行最多8个线程，所以只有多核cpu多线程才能真正实现性能提升
- 创建一个线程对象巨巨巨贵无比，要使用线程池来管理和复用线程
- 上下文切换比较耗时，尽量避免上下文切换
- 因此线程池的出现主要解决的就是创建线程对象和上下文切换带来的性能损耗
```c#
public class ThreadProcessClass
{

	public class ThreadTestClass()
	{
		public void ConsolData()
		{
			Console.WriteLine("我是线程启用的方法");
		}
	}
	public void ThreadFunc1(Object? a)
	{
		ThreadTestClass? threadTestClass = a as ThreadTestClass;
		threadTestClass?.ConsolData();
	}
	public void ThreadBaseFunc()
	{
		//简单启一个线程
		//构造一个Thread对象，构造函数里面的参数是	public delegate void ParameterizedThreadStart(object? obj);的委托，传入的方法要匹配
		Thread thread = new Thread(ThreadFunc1);
		ThreadTestClass threadTestClass = new ThreadTestClass();
		//调用start，正式启用线程
		thread.Start(threadTestClass);

		ThreadPoolFunc();
		TaskTestFunc();
		AsyncTestFunc();
		ThreadLockTestFunc();
		PositiveLockTestFunc();
		MixThreadLockFunc();
	}

	//上文说到线程的创建和销毁其实是一个非常耗时的操作，因此有线程池，通过线程池来避免线程的频繁创建和销毁
	public void ThreadPoolFunc()
	{
		int context = 100;
		//向线程池里面塞一个方法
		ThreadPool.QueueUserWorkItem((a) =>{

			Thread.Sleep(2000);
			context += 100;
			Console.WriteLine($"我是线程正在执行的回调方法:{a}, {context}");

		}, 8);
		//取消线程操作,需要new一个CancellationTokenSource对象，他持有的Token字段是CancellationToken类型的结构体，调用CancellationTokenSource的Cancel方法，会让
		//Token里面的IsCancellationRequested为true，把Token通过被调用的线程方法传到线程里面，因为这是跨线程通知的，线程里面就可以根据IsCancellationRequested的状态
		//来决定是否退出当前线程的操作
		//CancellationTokenSource的有参构造函数支持传递时间，时间到后自动终止
		CancellationTokenSource cts = new CancellationTokenSource(5000);
		Action<CancellationToken, int> cancelAction = (token, data) =>{
			Console.WriteLine("我是可以被取消的线程，我开始运行");
			for(int i = 0; i < data; i++)
			{
				if(token.IsCancellationRequested)
				{
					Console.WriteLine($"我是可以被取消的线程，我被取消了");
					break;
				}
				Console.WriteLine($"我是可以被取消的线程，我正在运行：{i}");
				Thread.Sleep(1000);
			}
			Console.WriteLine("运行完毕");
		};
		ThreadPool.QueueUserWorkItem((obj) =>{cancelAction(cts.Token, 10);});
		Console.ReadLine();
		cts.Cancel();
		Console.WriteLine("我是主线程的最后的退出");

	}
```
## Task(任务)
- 使用ThreadPool.QueueUserWorkItem往线程池添加任务的时候，使用者并不知道什么时候能完成，也拿不到完成的返回值，所以需要用到Task
```c#
	public void TaskTestFunc()
	{
		//创建简单的task
		Task task1 = new Task(() =>{
			Thread.Sleep(3000);
			Console.WriteLine("我是一个基础的Task任务调度");
		});
		task1.Start();

		Task task2 = new Task((a) =>{
			Thread.Sleep(2000);
			Console.WriteLine($"我是一个带参的Task任务调度：{a}");

		}, 20);
		task2.Start();

		Task.Run(() =>{
			Console.WriteLine($"我是一个简化的Task写法");
			
		});

		//创建一个有返回值的Task
		Task<int> task3 = new Task<int>(() => {
			Thread.Sleep(3000);
			Console.WriteLine("我是有返回值的线程");
			return 20;
		});

		task3.Start();

		//显式的等待,会阻塞当前线程，直到对应的task执行完毕
		task3.Wait();
		//实际上线程去访问另一个task的Result，也会导致自己进入阻塞，直到Result有值，Result属性内部本就有wait
		Console.WriteLine($"线程结果是：{task3.Result}");


		//同样使用CancellationTokenSource来取消task
		CancellationTokenSource cts = new CancellationTokenSource();
		Action<CancellationToken, int> action = (token, data) =>{
			for(int i = 0; i < data; i++)
			{
				Console.WriteLine($"我是能取消的task, 正在执行{i}， {data}");
				if(token.IsCancellationRequested)
				{
					Console.WriteLine($"取消task");
					break;
				}
				Thread.Sleep(1000);
			}
			Console.WriteLine("我是能被取消的task，已经执行完了");
		};
		Task task4 = new Task(() =>{action(cts.Token, 10);});
		task4.Start();
		Console.ReadLine();
		cts.Cancel();

		//一个Task完成可以自动接续后面的任务,可以做一个任务链,这样的话新的任务也会自动创建task来进行，就不会因为wait而阻塞主线程
		//TaskContinuationOptions.OnlyOnRanToCompletion这个参数表示只有上一个任务线程执行完成，才会继续执行下一个任务
		Task<int> taskContinue = Task.Run(() =>{
			Console.WriteLine("我是任务链的第一个");
			Thread.Sleep(3000);
			return 100;
		});
		Task<int> taskContinue2 = taskContinue.ContinueWith((task) =>{
			Console.WriteLine($"我是任务链的第二个，上一个方法结果是{task.Result}");
			Thread.Sleep(3000);
			return 200;
		}, TaskContinuationOptions.OnlyOnRanToCompletion);
		Task taskContinue3 = taskContinue2.ContinueWith((task) =>{
			Console.WriteLine($"我是任务链的第三个，上一个方法结果是{task.Result}");
			Thread.Sleep(3000);
			Console.WriteLine("任务链结束");
		}, TaskContinuationOptions.OnlyOnRanToCompletion);

		//Task任务可以建立父子任务，在任务里面再新建任务的时候可以使用TaskCreationOptions.AttachedToParent来将任务附加到父任务上
		//同样在调用continue任务的时候，也能通过枚举指定continue任务为子任务
		Task<int[]> taskParent = new Task<int[]>(() => {
			int[] res = new int[3];
			Console.WriteLine("开始执行父任务");

			Task<int> childTask1 = new Task<int>(() =>{
				Console.WriteLine("开始执行子任务1");
				Thread.Sleep(1000);
				Console.WriteLine("子任务1执行完");
				return 100;
			}, TaskCreationOptions.AttachedToParent);

			Task<int> childTask2 = new Task<int>(() =>{
				Console.WriteLine("开始执行子任务2");
				Thread.Sleep(2000);
				Console.WriteLine("子任务2执行完");
				return 200;
			}, TaskCreationOptions.AttachedToParent);

			Task<int> childTask3 = new Task<int>(() =>{
				Console.WriteLine("开始执行子任务3");
				Thread.Sleep(3000);
				Console.WriteLine("子任务3执行完");
				return 300;
			}, TaskCreationOptions.AttachedToParent);
			childTask1.Start();
			childTask2.Start();
			childTask3.Start();
			res[0] = childTask1.Result;
			res[1] = childTask2.Result;
			res[2] = childTask3.Result;
			return res;
		});
		//把显示放在ContinueWith里面，防止阻塞主线程
		taskParent.ContinueWith((_taskParent) =>{
			Console.WriteLine($"最后的结果是：{_taskParent.Result[0]}, {_taskParent.Result[1]}, {_taskParent.Result[2]}");
		});
		taskParent.Start();

		//Parallel方法，简化多线程的使用,Parallel会让里面的操作都并行执行
		//Parallel.For
		Parallel.For(0, 20, (i)=>{
			Console.WriteLine($"我是Parallel里面的For循环方法{i}");
		});
		List<int> lists = new List<int>();
		for(int i = 0; i< 10; i++)
		{
			lists.Add(i);
		}
		//Parallel.ForEach
		Parallel.ForEach(lists, (i) =>{
			Console.WriteLine($"我是Parallel里面的ForEach循环方法{i}");
		});

		//Parallel.Invoke
		Parallel.Invoke(
			() => {Console.WriteLine("我是Parallel里面的invoke方法1");},
			() => {Console.WriteLine("我是Parallel里面的invoke方法2");},
			() => {Console.WriteLine("我是Parallel里面的invoke方法3");},
			() => {Console.WriteLine("我是Parallel里面的invoke方法4");}
		);
	
		//Timer相关,new Timer第一个参数是回调(兼容TimerCallback)，第二个参数是传个回调的参数，第三个参数是多少秒以后调用，第四个参数是两次调用的最低间隔，-1表示无限时间长度
		//这里的设置表示这个timer定义了但是还没启用
		Action<object?> action1 = (obj) =>{
			Console.WriteLine($"我是Timer调用的方法{obj}");
			//可以在具体调度方法里再次改变timer调用的时间间隔，这里设置为每五秒调用一次
			//timer.Change(5000, 3000);
		};
		Timer timer = new Timer((obj) =>{action1(obj);} ,5, Timeout.Infinite, -1);
		
		//启用，3秒后调用，只调用一次
		timer.Change(3000, Timeout.Infinite);

		Console.WriteLine("我是主线程的后续");
	}
```
## 异步函数
- io常常是阻塞的操作，对于一个线程来说，等待io的过程其实是什么都不干的，会浪费资源，可以让这个线程先做别的，然后io到达后再执行具体逻辑，这之间使用的就是异步async / await 
- 关于异步，方法声明async表示我这个方法里面可能会有异步操作，await是会将现在的方法执行挂起，等待结束后还给调用者线程
- c#底层会把标记为异步的方法体展开成类，并保存方法的状态，通过状态机来控制方法的执行位置
```c#
	public void AsyncTestFunc()
	{
		Console.WriteLine("我是主线程开始执行");
		//如果外部函数还是需要异步，那就加上async和await，不需要等待就直接调用，让异步函数里面自行处理自己的等待结果，并忽略返回的task
		Task task = IOThread();
		//如果外部函数需要阻塞，就用Wait
		IOThread().Wait();
		Console.WriteLine("我是主线程接着执行");

	}
	//创建一个异步函数，异步函数内的状态机的保存与继续执行是通过task来实现的，所以异步函数必须返回一个task对象
	public async Task IOThread()
	{
		Console.WriteLine("开始模拟等待异步数据");
		//这里把这个耗时的操作包装成Task丢给线程池
		string res = await Task.Run(() => FuncAsyncGetStr());
		Console.WriteLine("最后的结果是：" + res);
	}
	//某个操作很费时
	public string FuncAsyncGetStr()
	{
		Thread.Sleep(5000);
		return "我是最终的数据";
	}
```
## 多线程锁
- 同一份数据如果能被两个线程同时访问，就会有数据损坏的风险，因此需要给数据加锁，构造加锁以及锁的释放等的过程就是线程的同步构造
- 自旋锁, 乐观锁，互斥锁，混合锁,异步锁
```c#
	public void ThreadLockTestFunc()
	{
		//关于原子操作：
		//一次操作可以被一个cpu指令执行完毕，那这个就是一次原子操作，如果是多个cpu指令才能完成的操作，多线程读取时可能会被中途打断导致读取出错误的数据，就需要某种标记，
		//让多个cpu指令为一次原子操作，不可被打断，来保证数据的完整性
		//例如在一个32位系统上给一个64位的变量赋值，就会被拆成两个指令，先写高位，再写低位
		//因此在别的线程读取这个数据的时候，a的指令可能会被打断，导致读取到0x1122334400000000或者0x0000000055667788，这在64位系统里面写128位数据是同一个道理
		long longData = 0x1122334455667788;

		//可以使用某些操作来保证原子性
		Interlocked.Exchange(ref longData, 10);

		//除此之外，编译器优化还会导致代码的执行顺序改变，假如一个值被存到寄存器里面之后，另一个线程又改了值，之前的cpu寄存器并不知道，
		// 这样在另一个线程从寄存器读值时，可能就会出错，因此需要使用Volatile下的读和写来确保代码的执行顺序
		//volatile的使用规则是确保volatile.Write强制禁止这个写之前的写被乱序排到这个写之后
		//volatile.Read强制禁止这个读之后的读被乱序到这个读的前面
		int a = 0;
		int b = 0;

		//对于类来说，可以加volatile关键字来表示这个字段是保证顺序的
		//volatile int a = 1;

		//假设这是一个线程
		a = 100;
		b = 100;
		//这是另一个线程,更改顺序后会导致a == 100的时候，可能b还是0，最后打印出来个0
		if(a == 100)
			Console.WriteLine(b);

		//使用volatile保证顺序
		a = 100;
		Volatile.Write(ref b, 100);//保证了在b写入数据时，a已经是100

		if(Volatile.Read(ref a) == 100)//保证在读取b之前，先读取了a
			Console.Write(b);

		//针对于data，使用自旋锁
		int data = 0;
		MySpineLock mySpineLock = new MySpineLock();
		Thread thread1 = new Thread(() =>{
			Console.WriteLine($"我是线程1，我拿了锁准备执行:{data}");
			//要操作data了，就加锁
			mySpineLock.Enter();
			Console.WriteLine($"我是线程1，我拿了锁开始执行:{data}");
			Thread.Sleep(3000);
			data += 1000; 
			//操作完毕释放锁
			mySpineLock.Exit();
			Console.WriteLine($"我是线程1，我执行完毕:{data}");
		});

		Thread thread2 = new Thread(() => {
			Console.WriteLine($"我是线程2，我拿了锁准备执行:{data}");
			mySpineLock.Enter();
			Console.WriteLine($"我是线程2，我拿了锁开始执行:{data}");
			data += 1000; 
			mySpineLock.Exit();
			Console.WriteLine($"我是线程2，我执行完毕:{data}");
		});
		thread1.Start();
		thread2.Start();

	}
```
### c#自带的lock互斥锁
- 可以使用c#自带的lock其实是Monitor.Enter和Exit的语法糖，这个锁可能会让线程陷入内核挂起等待
```c#
	public class LockObj
	{
		//这个obj用来当锁
		public static readonly object lockObj = new object();
		int data = 100;
		public void ProcessData()
		{
			//这里一次只允许一个线程进入
			lock(lockObj)
			{

			}
		}
		public void LockObjTestFunc()
		{

		}
	}
```
### 自旋锁
- 自旋锁，也是一种互斥锁，当一个线程去访问正在被访问的数据时，会进入自旋状态，直到锁被释放，这个期间线程一直占用着cpu空跑，所以自旋锁只适合在多核cpu且数据读写极快的情况下使用
```c#
	public class MySpineLock
	{
		int isLocked = 0;
		public void Enter()
		{
			while(true)
			{
				//Interlocked.CompareExchange(ref isLocked, 1, 0) ,这里意思是如果isLocked是0，那就把isLocked设置为1，
				if(Interlocked.CompareExchange(ref isLocked, 1, 0) == 0)
				{
					//获得锁，从循环中跳出
					break;
				}
				//否则就一直呆在循环里，Thread.SpinWait表示cpu执行一次极短的等待，可以稍微让出几个cpu时间片好让别的线程执行
				Thread.SpinWait(1);
			}
		}
		public void Exit()
		{
			//高速cpu已经写完，并立即同步到主内存，这里用写屏障是因为要确保持有锁的线程数据都已经更新完毕，才释放锁
			Volatile.Write(ref isLocked, 0);
		}
	}
```
### c#自带的自旋锁
```c#
	public class SpineLockClass
	{
		//使用c#自带的自旋锁
		SpinLock spinLock = new SpinLock();
		int data;
		void WriteData()
		{
			bool isFree = false;
			try{
				spinLock.Enter(ref isFree);
				data = 100;

			}
			//锁的释放放到finally里面，保证锁一定会被释放
			finally
			{
				if(isFree)
					spinLock.Exit();
			}
		}
	}
```
### 乐观锁
- 这个东西的实现有争议，先不考虑这类型锁的具体内容
- 乐观锁，数据直接拿，直接改，之后再判断版本号对不对(也可以直接判断数据有没有改)，如果不对就回退，适合竞争不激烈的情况下使用
```c#
	public class PositiveLockClass
	{
		public void UpdateData(PositiveLockData data, int processData)
		{
			//volatile内存屏障读取，保证自己读到的是最新的
			int oldData = Volatile.Read(ref data.data);
			int newData = oldData;
			do{
				//一开始操作的时候获取一次当前的数据，以便操作结束后进行比对，防止线程被挂起导致数据改变
				newData = Volatile.Read(ref oldData);

				//这里执行一系列操作，这整个期间线程都有可能被挂起
				int target = newData + processData;
				Console.WriteLine($"执行数据更改{target}");
				//这是一个原子操作，再次去获取数据，并和自己刚一开始拿到的数据进行比对，如果改变了，那就说明当前的计算数据不可信，如果一致，就直接写入数据
				//这个对比操作必须放在最后
				oldData = Interlocked.CompareExchange(ref data.data, target, newData);
			}
			while(newData != oldData);//数据改变了，那就继续重写
		}
	}
	public class PositiveLockData
	{
		 public int data = 10; //这是数据
	}
	public void PositiveLockTestFunc()
	{
		PositiveLockData positiveLockData = new PositiveLockData();
		PositiveLockClass positiveLockClass = new PositiveLockClass();
		Thread thread1 = new Thread(() => {
			Console.WriteLine($"准备执行数据更改1000");
			positiveLockClass.UpdateData(positiveLockData, 1000);
			Console.WriteLine($"数据更改完毕1000");

		});
		Thread thread2 = new Thread(() => {
			Console.WriteLine($"准备执行数据更改3000");
			positiveLockClass.UpdateData(positiveLockData, 3000);
			Console.WriteLine($"数据更改完毕3000");
		});
		Thread thread3 = new Thread(() => {
			Console.WriteLine($"准备执行数据更改4000");
			positiveLockClass.UpdateData(positiveLockData, 4000);
			Console.WriteLine($"数据更改完毕4000");
		});

		thread1.Start();
		thread2.Start();
		thread3.Start();
		//强制主线程等待
		thread1.Join();
		thread2.Join();
		thread3.Join();
		Console.WriteLine($"最后的乐观锁执行结果是：{positiveLockData.data}");
	}
```
### 混合锁
- 除了上面的用户模式的锁以外，还有内核模式的锁，当用户模式的锁涉及到比较强烈的锁竞争和锁饥饿的情况下，需要转为内核模式的锁，但是一般情况下不要使用内核模式，代价较大
- 上面的lock关键字的互斥锁就是一个内核模式的锁，lock实际上是Monitor.Enter和Exit的语法糖
- 因此使用的应该是混合锁，当锁竞争不强烈时，允许锁在cpu上自旋一段时间，超过一定时间后，改为内核模式
```c#
	public sealed class MixThreadLockTestClass
	{
		private static object lockObj = new object(); //锁对象
		private const int spineNum = 100000000; //最大自旋次数，超过这个自旋次数就丢到内核挂起
		private int isLock = 0; //自旋锁的状态
		public void Enter()
		{
			int curSpineNum = 0; //当前线程的自旋等待时间

			while(true)
			{
				if(Interlocked.CompareExchange(ref isLock, 1, 0) == 0)
				{
					//获取到锁
					Console.WriteLine("获得锁，开始执行");
					break;
				}
				curSpineNum++;

				if(curSpineNum > spineNum)
				{
					//超过自旋等待时间，把线程挂起丢到内核里面
					lock(lockObj)
					{
						Console.WriteLine($"超过自旋等待时间，挂起:{curSpineNum}");
						curSpineNum = 0;
						Monitor.Wait(lockObj);
					}
				}

				//进行一次自旋等待
				Thread.SpinWait(1);
			}
		}

		public void Exit()
		{
			//退出锁，并立即更新cpu内存
			Volatile.Write(ref isLock, 0);
			//尝试唤醒一个挂起的线程
			lock(lockObj)
			{
				Monitor.Pulse(lockObj);
			}
		}

	}
	public void MixThreadLockFunc()
	{
		MixThreadLockTestClass mixThreadLockTestClass = new MixThreadLockTestClass();
		int data = 10;
		Thread thread1 = new Thread(() =>{
			Console.WriteLine($"我是线程1，我拿了锁准备执行:{data}");
			//要操作data了，就加锁
			mixThreadLockTestClass.Enter();
			Console.WriteLine($"我是线程1，我拿了锁开始执行:{data}");
			Thread.Sleep(8000);
			data += 1000; 
			Console.WriteLine($"我是线程1，我执行完毕:{data}");
			//操作完毕释放锁
			mixThreadLockTestClass.Exit();
		});

		Thread thread2 = new Thread(() => {
			Console.WriteLine($"我是线程2，我拿了锁准备执行:{data}");
			mixThreadLockTestClass.Enter();
			Console.WriteLine($"我是线程2，我拿了锁开始执行:{data}");
			data += 1000; 
			Thread.Sleep(3000);
			Console.WriteLine($"我是线程2，我执行完毕:{data}");
			mixThreadLockTestClass.Exit();
		});
		thread1.Start();
		thread2.Start();
		thread1.Join();
		thread2.Join();
		Console.WriteLine($"最后的结果是：{data}");
	}
```
### 双检锁
- 单例模式如果是需要懒汉式而且线程安全的，可以使用双检锁来实现
- 双检锁单例模式
```c#
	public class ThreadSingletonClass
	{
		private static ThreadSingletonClass instance;
		private static readonly object lockObj = new object();
		public ThreadSingletonClass Instance
		{
			get{
				if(instance == null)
				{
					lock{
						instance = new ThreadSingletonClass();
					}
				}
				return instance;
			}
		}
	}
}
```