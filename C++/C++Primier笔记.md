---
title: C++ Primier笔记
---

## 基础语法

### h头
```c++
//#include<iostream> //常被引用的头文件里面尽量不用include别的不必要的东西
//using namespace std; 头文件里面不应该有using，因为头文件会拷贝到所有的引用文件中，头文件的using会被带到别的文件中导致bug

//const int* Constdata; //指向常量的指针
//int* const Constptr = &constInt; //常量指针

class TestClass
{
public:
	int data;
	float wawaa;
	std::string dataStr;
};


void FirstFunc(int data1, std :: string data2);

void Copydata(int &num, std :: string &str1, const std :: string &str2); // 对于超级大的参数定义成引用来避免拷贝，定义成const引用来避免修改


void PrintArry(const int* head, const int* end); // 数组作为参数传递是传的首地址指针，有时候可以传首尾指针来确定范围,对于不需要更改的定义为指向常量的指针

auto RetureTest(int data) -> int; //后置返回值定义，表明函数返回一个int,后置返回值定义是为了简化返回值是指针时的情况
auto RetureTest2(int data) -> int*; //后置返回值定义，表明函数返回一个int指针
auto RetureTes3(int data) -> int(*)[10]; //后置返回值定义，表明函数返回一个指向大小为10的int数组指针

auto DefultParamters(int aaa, int bbb = 100);


inline auto TestInlineFunc(int data);  //这是一个内联函数，编译时会在调用该函数的地方等价的展开，对于一些短小简单的函数，可以声明为内联函数，会更省内存和性能


int FuncPtr(int a, int b); //函数指针测试，指针指向这个函数


void FuncPtrTest(int bb, int aa); //函数指针测试,返回值是一个函数指针

void FunPtrTest2(int a, void (*func)(int a, int b)); //函数指针测试，参数是一个函数指针

auto FuncPtrTest3(int a, int b) -> void(*)(int , int );



typedef int xxl; //类型别名，简化代码
typedef void testFuncpteRename(int a, int b); //这个类型是函数类型
typedef void (*testFuncpteRename1)(int a, int b); //这个类型是指向函数的指针
typedef decltype(FuncPtrTest)* testFuncRename2;  //和auto一样可以类型推导，但是decltype不需要类型初始化就能推导，当推断函数指针时，需要在屁股后面加个*来指名这是指针

```

### cpp源

```c++
#include"TestHead.h"
#include<iostream>
using namespace std;
void FirstFunc(int data1, string data2)
{
	std::cout << data1 << "}}}}" << data2 << std::endl;
}

void Copydata(int& data1,string &str1,const string &str)
{
	string str2 = to_string(data1);
	str1 += str2;
	str1 += str;
}

void PrintArry(const int* head, const int* end)
{
	for (const int *a = head; a != end; a++)
	{
		cout << "数组内容是" << *a << endl;
	}
}
auto RetureTes3(int data) ->  int(*)[10]
{
	int aa[10] = {1,2,3,4,5,6,7,8,9,data};
	auto test = &aa;

	int* bb = *test;

	cout << *bb << endl;
	cout << *(bb + 1) << endl;

	cout << *(bb + 2) << endl;
	cout << *(bb + 3) << endl;
	cout << *(bb + 4) << endl;
	return &aa;
}

int FuncPtr(int bb, int aa)
{
	cout << "执行了指针指向的函数 " << aa + bb << endl;
	return 1;
}

void FuncPtrTest(int a, int b)
{
	int (*test)(int, int) = FuncPtr; //函数指针定义，第一个代表返回值，后面的代表第一个和第二个参数
	test(a, b);
	 
}

void FunPtrTest2(int a, void (*func)(int a, int b))
{
	int x = 100;
	func(50, x);
}


testFuncRename2 FuncPtrTest3(int a, int b)
{
	xxl aa = 100;
	return FuncPtrTest;
}



```

## 流

### h头

```c++

#ifndef fileTestt
#define fileTestt
#include<iostream>
#include<fstream>

void readFileTest(std::istream& in, int data); // 向输入写入数据
void printFileTest(std::ostream& out); //向输出写入数据
void ReadFileTesttt();
void WriteFileTesttt();


#endif // !fileTestt

```

### cpp源
```c++

#include<fstream>
#include<iostream>
#include<sstream>
#include"FileReadWrite.h"
using namespace std;

const string  &filePath = "TestFile/TestFile.txt"; //指向常量的引用

int constInt = 10;


void readTest(istream& in, int data) // 向输入写入数据
{
	in >> data;
}
void writeTest(ostream& out) //向输出写入数据
{

}
//类比于cin和cou，对应cin来说，键盘输入放到流中，再从流中读到对应的变量里面去，所以istream是从流中读数据
//对于cout来说，把对应的字符串输入到流中，缓冲区再从流中读取数据，所以ostream是向流中写入数据


//写文件
void WriteFileTesttt()
{
	ofstream outData;
	
	//outData.open(filePath); //默认为out模式，也就是trunc模式，打开会清空文件

	outData.open(filePath, fstream::app);//指定为appden模式，会在文件末尾添加
	string data = "33435465";
	outData << data;
	//这里函数结束了，文件流会被销毁，会自动调用close函数，但一般建议使用完文件流还是要自己关闭

	outData.close();
}

//读文件
void ReadFileTesttt()
{
	ifstream inData;
	inData.open(filePath); //默认为out模式，也就是trunc模式，打开会清空文件

	//inData.open(filePath, fstream::app);//指定为appden模式，会在文件末尾添加

	if (inData.is_open())
	{
		string data;
		stringstream sstream;
		cout << "打开成功:" << endl;

		while (getline(inData, data))  //按行读取文件数据
		{
			sstream << data;
			string singleData;
			while (sstream >> singleData) //按空格读取行数据
			{
				cout << "|";
				cout << singleData;
			}
			cout << endl;
			sstream.clear();
		}
	}
	else
	{
		cout << "打开失败" << endl;

	}
	inData.close();
}


```


## 容器

### h头

```c++

#ifndef VECTOR_LEARN

#define VECTOR_LEARN


//顺序容器总结
//vector是可变大小数组内存空间连续，，支持快速随机访问，尾插很快
//deque 双向队列，支持快速随机访问，头尾插很快
//list 双向链表，只能从头尾开始查找，随机访问很慢，但随机插入很快
//forward_list 单向链表，只能从头开始查找，随机访问很慢，但随机插入很快
//arry 固定大小数组
//string 类似与vector的专门用来处理字符串的容器

//关于容器的选择
//如果没有别的更好的选择，一般默认选择vector
//如果要选择别的容器，则从操作消耗时间和内存占用两个维度进行选择，必要时需要对不同的容器进行测试

void TestVec1();


//关联容器
//有序：
// map 存储键值对：关键字-值
//set 关键字就是值，即只保存关键字的容器
//multimap 关键字可以重复出现的map
//mutiset 关键字可以重复出现的set

//无序： 用哈希算法实现的容器
// unordered_map 
//unordered_set 
//unordered_multimap 
//unordered_mutiset 
void TestVec2();
#endif

```

### cpp源

```c++
#include<iostream>
#include<vector>
#include<string>
#include<queue>
#include<stack>
#include<iterator>
#include<map>
#include<set>
#include<unordered_map>
#include<unordered_set>
using namespace std;

class VecClsTest
{
	int a;
	int b;
	int c;
public:
	VecClsTest() = default;
	VecClsTest(int s,int d,int f)
	{
		a = s;
		b = d;
		c = f;
	}
};

//有序容器
void TestVec1()
{
	vector<int> tesVec = { 1,2,3,4,5 };
	//vector<int> tesVec(tesVec2); 拷贝初始化
	//vector<int> tesVec(a, b); 包含a个初始化为b的函数，只支持顺序容器
	//vec1.assign(b.cbegin(), b.cend()) :表示vec1中的元素替换为迭代器指定范围的b的元素的拷贝,只支持顺序容器
	//swap(vec1, vec2) //交换两个容器，只交换内部结构，所以会很快，而且指针不失效，但string会失效，对于arry来说，这个交换和普通拷贝一样耗时间

	//tesVec.size() 返回容器的元素个数
	//tesVec.max_size() 返回容器能容纳的最大元素个数
	//tesVec.capacity() 返回容器不扩容的话能容纳的最大元素个数


	vector<int> ::iterator tesVecIter = tesVec.begin();
	//tesVec.rbegin() 返回的是一个反向迭代器
	//tesVec.cbegin() 返回的是一个const迭代器
	for (; tesVecIter != tesVec.end(); tesVecIter++)
	{
		cout << "value is" << *tesVecIter << endl;
	}

	tesVec.push_back(1); //向容器尾部添加元素

	vector<VecClsTest> tesVec1;
	tesVec1.push_back(VecClsTest(1, 2, 3)); //外部构造然后塞进去
	tesVec1.emplace_back(1, 2, 3);//之间在容器内的空间构造元素

	string str1 = "str1";
	str1.append("11111");//向字符串后面追加
	str1.assign("2323");//对字符串覆盖

	string str2 = "13";
	int b = 89;
	int a = 12;
	b = stoi(str2);
	cout << "字符串转换：" << a + b << endl;
	string str3 = to_string(a);
	str2.append(str3);
	cout << "字符串转换：" << str2 << endl;


	//容器适配器 栈队列deng
	stack<int, vector<int>> stacktaest1;//定义一个基于vector的栈
	queue<int> queuetest1; //定义一个队列
	priority_queue<int> queuetest2; //定义一个优先级队列

	//特殊迭代器

	//插入迭代器

	auto iterInserterTest = inserter(tesVec, tesVec.begin());
	*iterInserterTest = 100;
	for (int i = 0; i < tesVec.size() - 1; i++)
	{
		cout << "插入迭代器:" << tesVec[i] << endl;
	}

	//iostream 迭代器
	istream_iterator<int> in_iter(cin), eof;
	vector<int> newIntVec(in_iter, eof);
	for (int i = 0; i < newIntVec.size() - 1; i++)
	{
		cout << "io迭代器:" << newIntVec[i] << endl;
	}

	ostream_iterator<int> outTest(cout, " wawaw");
	for (int i = 0; i < newIntVec.size() - 1; i++)
	{
		outTest = newIntVec[i];
		cout << endl;
	}

	//反向迭代器

	for (auto iter = newIntVec.crbegin(); iter != newIntVec.crend(); iter++)
	{
		cout << *iter << endl;
	}
};

class TempClass {
	int a = 0;
};
// 关联容器
void TestVec2()
{
	map<string, int> map1;
	map1["a"] = 156;
	map1["c"] = 18856;

	map1.insert({ "11", 1 });

	set<string> set1 = {"qq", "rr", "yy"};
	multiset<string> set2 = { "qq", "rr", "yy" };

	set1.insert("qq");
	set2.insert("qq");
	cout << set1.size() << endl;
	cout << set2.size() << endl;
	pair<string, int> pair1("qq", 1);

	auto isSucces = map1.insert(pair1); //插入函数返回一个pair，pair->first是个迭代器，指向给定关键字的元素，pair->second是一个bool，表示是否插入成功
	if (isSucces.second)
	{

	}

	cout << pair1.first << pair1.second << endl;

	multimap<string, TempClass> map2;

	multiset<TempClass> set3;

	multimap<string, int> map3 = { {"aa", 11}, {"bb", 22}, {"cc", 33}, {"cc", 44} };
	for (auto begin = map3.cbegin(); begin != map3.cend(); begin++)
	{
		cout << begin->first << ", " << begin->second << endl;
	}
	cout << "??" << endl;

	auto map1_iter = map1.find("111");  // 查找map中的元素
	if (map1_iter == map1.end())
	{
		cout << "没找到 " << endl;
	}
	else
	{
		cout << map1_iter->first << ", " << map1_iter->second << endl;

	}
	int count = map3.count("cc"); //查找map中的元素个数
	auto map3_iter = map3.find("cc");//对于可重复的关联容器，find找到的是第一个元素的迭代器，可以用上面获得 的count对迭代器遍历，就能拿到对应关键字的所有键值

	map3.upper_bound("cc"); //返回最后一个重复元素的下一个位置
	map3.lower_bound("cc"); //返回第一个元素

	auto pair = map3.equal_range("cc"); // 范围是从pair ->first 到second
	for (; pair.first != pair.second; pair.first++)
	{
		cout << pair.first->second << endl;
	}
	//cout << map3["bb"] << endl;
}

// 无序容器 使用哈希桶

class HashClass {
public:
	string name;
};
size_t GetClassHash(HashClass &cls) // size_t是一个机器相关的无符号整形
{
	return hash<string>()(cls.name);
}
bool HashCompare(HashClass& cls1, HashClass& cls2)
{
	return GetClassHash(cls1) == GetClassHash(cls2);
}
void Test3()
{
	unordered_map<int, string> map1;
	unordered_set <HashClass, decltype(GetClassHash)*, decltype(HashCompare)*> claSet; //定义一个自定义类的hash set
}


```

## 泛型算法

### h头

```c++

#ifndef GeneralizedAlgorithm
#define GeneralizedAlgorithm


void TestGeneralizedAlgorithm();


#endif 


```

### cpp源

```c++

#include "GeneralizedAlgorithm.h"
#include<iostream>
#include<vector>
#include<string>
#include<queue>
#include<stack>

#include<algorithm> //大部分算法定义在这个里面
#include<numeric> //这里面也有一些

#include "ClassLearn.h"
using namespace std;

bool NewCompare(ClassLearn1 &cls1, ClassLearn1& cls2)
{
	return cls1.data1 > cls2.data2;
}

void TestGeneralizedAlgorithm()
{
	vector<int> vec1 = { 1,2,3,4,5,67,445 };
	auto result = find(vec1.begin(), vec1.end(), 445);
	if (result != vec1.end())
	{
		cout << "找到了 ！   " << *result << endl;
	}
	else
	{
		cout << "找不到" << endl;
	}

	// 只读算法：
	find(vec1.begin(), vec1.end(), 445);
	int data1 = accumulate(vec1.begin(), vec1.end(), 0); //算和
	// 写算法
	fill(vec1.begin(), vec1.end(), 0); //将容器的值置为0
	fill_n(vec1.begin(), 3, 5);//向容器写入3个为5的元素，注意算法不改变容器大小，如果容器没有10个那么大会报错

	//拷贝算法
	vector<int> vec2 = { 1,2,3,4,5,1,445,3,4 };
	auto itercopytest = copy(vec1.begin(), vec1.end(), vec2.begin()); //将vec1的值拷贝到vec2，注意vec2类型和大小匹配，返回拷贝完vec2最后一个拷贝值后一个迭代器'
	replace(vec1.begin(), vec1.end(), 1, 2); //替换算法，将vec1中的1全都换成2
	//重排算法
	sort(vec1.begin(), vec1.end());

	//消除重复单词
	vector<string> vec3 = { "111", "222", "111" };

	sort(vec3.begin(), vec3.end()); //先排序
	auto iter_unique = unique(vec3.begin(), vec3.end()); //重新排列容器，使不重复的都放在前面，重复的都放在后面，返回一个指向不重复区域之后的一个位置的迭代器
	vec3.erase(iter_unique, vec3.end());// 再执行擦除工作，将后面的重复内容截断

	//给算法重新定义谓词来进行算法判断：(谓词就是新的比较方法，返回值是bool)
	vector<ClassLearn1> vecTest4;
	//sort(vecTest4.begin(), vecTest4.end(), NewCompare);

	//lamda表达式
	auto lamdaTest = [] {return 100; }; // 这是一个最简单的lambda表达式，表示返回一个100
	cout << lamdaTest() << endl;

	//一个接受参数的lambda
	auto lambdaTest2 = [](int x){ return 100 + x; };
	cout << lambdaTest2(50) << endl;

	//lambda的值捕获， 是在lambda创建的时候捕获的

	int data = 100;
	auto lambdaTestCatch = [data](int x) {return x + data; };

	//lambda的引用捕获
	auto lambdaTestCatch2 = [&data](int x) {return x + data; };



	//lambda表达式通过捕获列表(那个中方括号括起来的，捕获参数用逗号分割)来拿到上一级函数的值
	int lambdaInt = 85;
	auto lambdaTest3 = [lambdaInt](int x) {return 100 + x + lambdaInt; };
	cout << lambdaTest3(20) << endl;

	//使用尾置 自定义lambda返回类型
	auto lambdaTesst4 = [](int x) -> bool {return x == 100; };

}

```



## 类

### h头

```c++

#ifndef Classlearn
#define ClassLearn
#include<string>
#include<iostream>


class ClassLearn1 {
	friend void GetClassPrivateData(ClassLearn1&); //友元函数声明，好让这个函数使用私有变量
	friend class FriendClass; //声明这个类是友元类，那FriendClass可以直接调用ClassLearn里面的私有成员

public:
	int data1;
	int data2;
	const int www = 10;
	static const int aaaStatic = 100;
	std::string data3;
	ClassLearn1() = default; // 声明这个构造函数是默认的
	void SetPrivateData(int data)
	{
		data_privat = data;
	}
	ClassLearn1(int a): www(a), data1(a){}

	ClassLearn1(int a, int b, std::string c) : data1(a), data2(b), data3(c), www(a) { }//构造函数初始值列表初始化函数,如果没有给常量初始化，则必须在初始化列表中初始化常量
																						//常量也只能通过列表来初始化


	ClassLearn1& ThisPtrTest(int a, int b, std::string c);// this指针测试，返回对象本身
	void PrintSelf()
	{
		std::cout << data1 << std::endl;
		std::cout << data2 << std::endl;
		std::cout << data3 << std::endl;
		std::cout << std::endl << std::endl;
	}
private:
	int data_privat;
};
void GetClassPrivateData(ClassLearn1&); // 为了统一，一般把友元的声明和对应类的声明放在同一个头文件中

class FriendClass {
public: 
	void GetClsData(ClassLearn1& cls)
	{
		cls.data_privat;
	}
};
class FuncClass //这是一个可调用对象，lambda表达式就是临时构造这样一个可调用对象
{
public:
	int operator()(int a, int b)
	{
		return a + b;
	}
};

class BaseClass2
{
public:
	int data1;
	static int staticData; //静态类只会有一个实例
	std::string data2;
	BaseClass2(int _data1, std::string _data2)
	{
		data1 = _data1;
		data2 = _data2;
		//基类中调用子类的虚函数，该虚函数依赖与子类的成员，子类的成员这个时候还没初始化，会出问题
	}
	virtual void Test1() //当子类附加在父类指针上的时候，调用虚函数会调用子类的相关覆写函数，虚函数在运行时才会创建，虚函数指针会被存到虚函数表下
	{
		std::cout << "我是基类" << std::endl;


	}
	void Test2()
	{
		std::cout << "我是基类" << std::endl;

	}
	virtual void test3() final //final 表示这个虚函数不能再被覆盖了
	{

	}
	virtual ~BaseClass2() = default; // 对于继承的根节点基类，一般都定义一个虚析构函数，这样可以保证指针调用的是正确的析构函数
};

class FromClass1 : public BaseClass2 //派生访问符决定了实例化出来的用户对基类的访问权限，如果定义为非public则不能进行基类和派生类的类型转换
{
public:
	int subData1;
	std::string subData;
	FromClass1(int _data1, std::string _data2, int _subData1, std::string _subData2) : BaseClass2(_data1, _data2)
	{

	}
	using BaseClass2::BaseClass2; //直接继承基类的构造函数 
	int GetBaseData()
	{
		return BaseClass2::data1; //使用域运算符来获取基类的数据
	}
	void Test1() override
	{
		std::cout << "我是派生类" << std::endl;
	}
	void Test2()
	{
		std::cout << "我是派生类" << std::endl;

	}
	~FromClass1()
	{
		//派生类实现自己的析构函数
	}
};

class BaseClass3
{
public:
	virtual void Test1() = 0; //这是一个纯虚函数，含有纯虚函数的类被称为接口类，不允许单独实例化，继承于他的子类必须实现纯虚函数成员
};
class tesss1x final {}; //final关键词表示这个类不能被继承
class tesss1 {}; 
class tesss2 :public tesss1 {};
#endif // !Classlearn



```

### cpp源

```c++

#include"ClassLearn.h"
#include<iostream>
#include<functional>
using namespace std;

int addTest(int a, int b)
{
	return a + b;
}

ClassLearn1& ClassLearn1 :: ThisPtrTest(int a, int b, string c)
{
	this->data1 = a;
	this->data2 = b;
	this->data3 = c;

	function<int(int, int)> f1 = FuncClass(); //function模板存储一个可调用对象
	f1(1,3); //调用可调用对象
	function<int(int, int)> f2 = addTest; //function也可以存储一个函数指针 r

	return *this;
}

void GetClassPrivateData(ClassLearn1 & cls)
{
	int privatedata = cls.data_privat;
	cout << privatedata << endl;
}

```


## 动态内存

### h头

```c++

#ifndef RunTimeMemory
#define RunTimeMemory
#include<string>
#include<vector>
#include<memory>
#include<iostream>

class BaseClass //这是一个底层容器，用来测试自管理内存的类
{
	typedef std::vector<std::string>::size_type size_strtest_type;
private:
	std::vector<std::string> strList;
	std::string name;
public:
	BaseClass() = default;
	BaseClass(const std::initializer_list<std::string> &_str) :strList(_str) {
		name = *_str.begin();
	};
	~BaseClass()
	{
		std::cout << "销毁了一个基础容器:" << name << std::endl;
	}
	size_strtest_type size()
	{
		return strList.size();
	}
	bool empty()
	{
		return strList.empty();
	}

	void push_back(std::string str)
	{
		strList.push_back(str);
	}
	std::vector<std::string>::iterator begin()
	{
		auto aa = strList.begin();
		return aa;
	}
	std::vector<std::string>::iterator end()
	{
		auto aa = strList.end();
		return aa;
	}

};
class MemoryTestClass1
{
public:
	int aa = 2;
	int bb = 3;
	int cc = 7;
	std::string str = "11";
	int GetNum()
	{
		return aa + bb + cc;
	}
	MemoryTestClass1() = default;
	MemoryTestClass1(int data)
	{
		aa = data;
	}
	std::string DeleteTest()
	{
		return "我自己执行了删除";
	}
	~MemoryTestClass1()
	{
		std::cout << "调用MemoryTestClass1析构函数" << std::endl;
	}
};
class MemoryTestClass
{
public:
	int aa = 2;
	int bb = 3;
	int cc = 7;
	std :: string str = "11";
	int GetNum()
	{
		return aa + bb + cc;
	}
	MemoryTestClass() = default;
	MemoryTestClass(int data)
	{
		aa = data;
	}
	std::string DeleteTest()
	{
		return "我自己执行了删除";
	}
	~MemoryTestClass()
	{
		std::cout << "调用MemoryTestClass析构函数" << std::endl;
	}
};

class MemoryTestClass2  //这是一个自管理内存的类，总的来说就是这个类在构造的时候进行内存分配，并且将内存全都放到一个智能指针下面去，这样在传统拷贝的时候就可以自动做到对应内存的引用计数，当引用计数为0时，说明这个类也没有人在引用，就会调用相关容器的析构函数销毁内存
{
public:
	typedef std::vector<std :: string> ::size_type sizeData; //定义sizeData是一个string的Vector的size类型
	typedef std::shared_ptr<BaseClass> vector_shared_ptr;
	MemoryTestClass2() : str(std :: make_shared<BaseClass>()) {}//默认构造函数

	MemoryTestClass2(const std :: initializer_list<std :: string>  &_str):str(std :: make_shared<BaseClass>(_str))//该参数接收一个花括号列表
	{

	}

	sizeData size() const //后面加const是静态的成员函数，不能改变类的值
	{
		return str->size();
	}

	bool empty() const
	{
		return str->empty();
	}

	void pushback(const std :: string _str)
	{
		str->push_back(_str);
	}
	void LogStr()
	{
		if (!str)
		{
			std :: cout << "这个类的底层容器被销毁了" << std :: endl;
		}
		for (auto i = str->begin(); i != str->end(); i++)
		{
			std :: cout << "ptrCls is " << *i << std::endl;
		}
		std::cout << "_________________________"<< std::endl;
	}

private:
	//vector<std::string> str = { "aa" , "sdsd"};

	vector_shared_ptr str; // 声明一个智能指针，指向一个string的vector
	void check(sizeData i, std :: string &msg)
	{
		if (i >= str->size())
		{
			throw std :: out_of_range(msg);
		}
	}
};

void RunTimeMemoryTest();




#endif

```

### cpp源

```c++

#include"RunTimeMemory.h"
#include<iostream>
#include<memory>
#include<new>
using namespace std;



void CustomDelete(MemoryTestClass1* cls)
{
	cout << "我是自定义的删除函数" << endl;
	cout << cls->DeleteTest() << endl;
	cout << cls->aa << endl;

}

// 自定义删除方法的智能指针
void CustomSharedPtr()
{
	shared_ptr<MemoryTestClass1> p2(new MemoryTestClass1(2), CustomDelete);//自定义delete
	shared_ptr<MemoryTestClass1> p3(new MemoryTestClass1(2)); //默认的delete，自动执行析构函数
}

void RunTimeMemoryTest()
{
	cout << "runtimeTest!!" << endl;
	MemoryTestClass *cls = new MemoryTestClass(); // 动态内存，手动申请一块内存新建一个类

	//delete cls;									//动态内存，手动释放一个指针所指向的空间


	//智能指针, 类似于vector 智能指针也是模板
	shared_ptr<MemoryTestClass> sharePtr; // 允许多个指针指向同一个对象，尖括号里面的是允许指向的类型，内部会有引用计数，每新建一个指针，指针指向的对象的引用技计数就会+1，销毁一个指针，计数减一，计数为0时，自动调用对象的析构函数，并用delete释放空间

	if (sharePtr)					//判断是否有值
	{
		MemoryTestClass cls3 = *sharePtr; //解引用,注意解引用指针之前一定要判空
	}

	MemoryTestClass* classptr = sharePtr.get();//返回智能指针中保存的对象指针
	shared_ptr<MemoryTestClass> sharePtr2 = make_shared<MemoryTestClass>(5); //最安全的方式是用make_shared方法来生成一个智能指针
	cout << "runtimeTest!!" << endl;
	cout << "num1 is " << cls->GetNum() << endl;
	cout << "num2 is " << sharePtr2->GetNum() << endl;

	const string str1 = "我是容器4";
	const string str3 = "22";
	const string str2 = "33";

	MemoryTestClass2 cls4 = {str1, str2,str3};
	MemoryTestClass2 cls5{"我是容器5"};
	cls4.LogStr();
	cls5.LogStr();
	cls5 = cls4; //赋值的时候，cls5的底层容器引用为0，自动销毁了。
	cls5.LogStr();

	shared_ptr<MemoryTestClass> sharePtr4 = make_shared<MemoryTestClass>(4);
	if (!sharePtr4.unique()) //判断自己是不是唯一引用的指针
	{
		sharePtr4.reset(new MemoryTestClass(2)); //分配新的
	}
	CustomSharedPtr();
	
	//unique_ptr 不同于shareptr，他只能绑定一个对象
	unique_ptr<int> uniptr(new int(3));

	//weakptr是弱引用，用一个shareptr初始化，但是不会增加他的引用计数，因此在通过weak访问时，需要用lock函数确认对象还存在
	weak_ptr<int> wakeptr(make_shared<int>(5));

	//动态数组
	//申请一个58大小的数组内存
	int* arryPtr = new int[58];

	//换个方式
	typedef int arryPtrType[100];
	int* arryPtr2 = new arryPtrType;

	//删除一个数组
	delete[] arryPtr;

	//智能指针
	unique_ptr<int[]> uniptr_arry(new int[10] {1, 2, 3, 4, 5, 6, 0});
	cout << uniptr_arry[0] << endl; //使用下标访问

	uniptr_arry.release(); //手动销毁

	//share智能指针只能自己定义删除函数
	shared_ptr<int> sharedptr_arry(new int[20], [](int* arry) {delete[] arry; });
	*(sharedptr_arry.get() + 1) = 3; //share不定义下标运算jkl'

	//allocator 区别于new和delete，alloctor不会在分配的时候进行初始化，只会分配对应的空间大小
	allocator<int> alloc;
	auto allocSpace = alloc.allocate(10);//分配10个int空间
	auto allocSpaceCopy = allocSpace;
	alloc.construct(allocSpaceCopy); //构造内存里面的东西
	alloc.construct(allocSpaceCopy+1, 20);
	
	alloc.destroy(allocSpaceCopy); // 同样也能回收，回收了可以重新初始化
	alloc.destroy(allocSpaceCopy + 1);

	alloc.deallocate(allocSpace, 10);// 销毁归还内存


}

//不用智能指针直接管理内存
void DerictoryMemory()
{
	int* p = new int; //这是一个指向无名地址的p指针
	string* p2 = new string(20, 'a');

	vector<int>* p3 = new vector<int>{ 1,2,3,4,5,6,7 };

	int* p4 = new(nothrow) int; //如果内存不足会抛出bad_alloc异常，使用nothrow可以阻止抛出异常，返回一个空指针

	delete p; //p必须指向一个new分配的内存地址或者空指针、
	p = nullptr; // 空悬指针需要释放
}

shared_ptr<MemoryTestClass> SharedPtrFunc()
{
	return make_shared<MemoryTestClass>(3); //函数返回智能指针，并创建对象
}

void ExceptionSharePtr()
{
	int* p1 = new int;
	shared_ptr<int> p2 = make_shared<int>();
	//这里抛出一个未捕获的异常

	delete p1; //这里不会被销毁

}//执行完以后p2还是能销毁


void SharedPtrExample() //一个简单的智能指针使用用例
{
	auto ptr = SharedPtrFunc();
}
//当函数调用结束，ptr指针销毁，引用计数为0，对应的对象也会被销毁
//所以引用计数为0才会被销毁，所以记得把不需要的智能指针给销毁了，如果智能指针放在容器里，记得使用erase销毁



```


## 拷贝控制

### h头

```c++

#ifndef CopyControl
#define CopyControl

#include<utility>

class DataCalss
{
	public:
	int data;
};
class CopyClass {
public:
	int data;
	DataCalss* Dcls;
	void free()
	{
		//释放函数
	}
	CopyClass(); //默认构造函数
	CopyClass(const CopyClass& cpCls); //拷贝构造函数

	CopyClass& operator=(const CopyClass& rhs)//拷贝赋值运算符重载
	{
		data = rhs.data;
	}
	CopyClass& operator=(CopyClass&& rhs)noexcept//移动赋值运算符重载
	{
		if (this != &rhs)
		{
			free(); //释放自身的元素                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  
			data = rhs.data;
			Dcls = rhs.Dcls;
			rhs.Dcls = nullptr;
		}
		return *this;
	}
	
	~CopyClass(); //析构函数，用来回收生存期分配的资源hjk'

	CopyClass(CopyClass&& cls)noexcept //noexcept表示不抛出任何异常,因为移动是一个原子操作，不应该被打断
	{
		data = cls.data;
		Dcls = cls.Dcls;
		cls.Dcls = nullptr; //移动完以后记得把以前的指针设为null避免移动后的内存被原来的引用的析构函数销毁
	}

	void GetData(DataCalss &dcls) //这是一个普通的赋值函数
	{
		*Dcls = dcls;
	}

	void GetData(DataCalss&& dcls)//这是右值引用的移动函数
	{
		*Dcls = std::move(dcls);
	}
	//阻止拷贝
	//CopyClass(const CopyClass& cpCls) = delete; //拷贝构造函数
	//CopyClass& operator=(const CopyClass& rhs) = delete;//赋值运算符重载

	//左值和右值的成员函数，函数后面加的叫做引用限定符，决定了函数的返回值是只能用于左值还是右值
	CopyClass &GetCls() & //返回一个左值，是个引用
	{
		return *this;
	}
	int GetData()&& //返回一个右值
	{
		return data;
	}

};

void TestCopy();



#endif


```

### cpp源

```c++

#include "CopyControl.h"
#include<utility>
using namespace std;
void TestCopy()
{
	//右值引用,主要解决对象移动的问题，对象移动可以避免拷贝，省很多性能
	//右值的特性：生命周期短暂即将销毁，一般是字面常量或临时对象
	int i = 42;
	int& r = i; //左值放到左值引用上

	int&& t = 53; //将右值绑定在右值引用上
	
	int ii = 20;
	int&& tt = std::move(ii); //move方法可以返回绑定在左值上的右值引用,在ii调用move后，原则上就不应该再使用他了
	//注意移动操作以后原对象状态将不确定，代码中小心使用move

}

```


## 模板

### h头

```c++

#include"template.h"
using namespace std;

//定义一个模板函数
template <typename T>
void FirstFunc(const T &ls, const T &rs)
{
	return ls < rs;
}
extern template class TemplateClass<int>; //模板在使用时才会实例化，所以会出现同一个模板出现在不同的文件中的情况，为避免这种开销，需要在模板具体使用前用extern全局声明

void TestFunc()
{

	TemplateClass<int> templateClass(20, 50);
	//TemplateClass<int>::staticTest = 100; // 定义模板对应的静态成员
}



```

### cpp源

```c++

#ifndef TEMPLATE
#define TEMPLATE
#include<utility>
//定义一个模板类
template <typename T> class TemplateClass
{
public:
	int data;
	T templateData;
	TemplateClass(int _data, T _data2) : data(_data), templateData(_data2) {};
	static int staticTest; //静态成员在相同的T类型下共享、

	template<typename TT> void Func1(const T& val) // 成员模板
	{

	}
	//template<>
	//void Func1(const std :: string& val) //特例化的模板
	//{

	//}
	template <typename TTT> auto Func2(TTT& data1, TTT& data2) -> decltype(*data1) //当模板函数的返回值不好确定的时候可以尝试尾置返回类型
	{

	}
	template <typename T2> void teseFunc(T2 data)
	{

	}
	template <typename... T> void testFunc3(T&&... args)
	{

	}
	template <typename T2> void Func3(const T2& data)
	{
		teseFunc(std::forward<T2>(data)); // forward返回&&T2，可以保持参数是属性
	}

	//可变参数
	template <typename... T3>
	void Func3(const T3& ... args)
	{
		int TypeCount = sizeof...(T3); //得到类型的数量
		int ArgCount = sizeof...(args);//得到参数的数量
		testFunc3(std::forward(args)...); //参数包的转发
	}
};

#endif


```



## 标准库特殊设施

### h头

```c++

#ifndef StandardSpecial
#define StandardSpecial

namespace StandardSpecial_Space 
{
	void TestFuncStandardSpecial();

	//限定作用域枚举类
	enum class NumTest {
		first,
		second,
		third
	};
	//不限定作用域的枚举类
	enum NumTest2
	{
		first,
		second,
		third
	};
}
//union是一个节省空间的类，不能含有引用类型，同一时刻里面只能有一个数据生效
union UnionTest
{
	int a;
	int b;
	int c;
};

#endif


```

### cpp源

```c++

#include "StandardSpecial.h"
#include<iostream>
#include<tuple>
#include<bitset>
#include<regex>
#include<random>
using namespace std;
namespace StandardSpecial_Space
{
	void TestFuncStandardSpecial()
	{
		//tupl,类似于pair，但是成员不止两个，tupl可以用于一些简单的数据组合,一般用来打包函数返回结果
		//tuple<int, int> tuple1 = std::make_tuple();
		tuple<int, int> tuple1(1, 2);
		tuple<int, string> tuple2 = make_tuple(1, "22");
		//获取tuple的成员
		int data = get<0>(tuple1);
		string data2 = get<1>(tuple2);



		//bits类，方便进行位运算
		bitset<32> bitVec; //32位都是0；
		bitset<13> bitVec2(0xbeef); // 1111011101111,长度不够初始值会丢弃掉高位
		bitset<20> bitVec3(0xbeef); // 00001011111011101111, 长度超过初始值会把高位填充成0

		//bitset的操作
		bitVec2.any();//是否存在置位
		bitVec2[3];//访问某一位
		bitVec2.set();// 设置某一个位置
		bitVec2.flip(); //反转
		//.......


		//正则表达式,内容放在regex中



		//随机数
		default_random_engine e(20); //20是随机数种子
		unsigned randomData = e();//随机生成

		uniform_int_distribution<unsigned> u(0, 9);
		uniform_real_distribution<double> u2(0, 1);
		normal_distribution<> u3(4, 1.5);

		u(e); //生成0-9的随机数
		u2(e);//生产0-1的随机浮点数
		u3(e);//生成的随机数均值为4，标准差为1.5


		// io库
		cout << true << "  " << false << endl;
		cout << boolalpha << true << "  " << false << endl; //输出true和false为字符串而不是0或者1

		cout << showbase << endl;
		cout << "default: " << 1024 << endl;
		cout << "oct:  " << oct << 1024 << endl;
		cout << "hex: " << hex << 1024 << endl;
		cout << "decimal: " << dec << 1024 << endl;

		NumTest numtest = NumTest::third;
	}
}

```

