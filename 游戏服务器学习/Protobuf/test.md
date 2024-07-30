# Protobuf研究

## 简介

​		Protobuf，全称为Protocol Buffers，是Google开发的一种轻量、高效的结构化数据存储格式，主要目标是实现高效的数据序列化和反序列化。它提供了一种简洁明了的结构化描述语言，来定义结构数据，同时提供对应的工具，生成多种编程语言的代码，从而实现跨平台，多语言支持。

## 特性

- 丰富的数据类型

  支持基本数据类型（如整数、浮点数、布尔值等）、枚举、结构体、嵌套消息、数组和map等复杂数据类型

  使用repeated字段规则实现数组，可以不必指定数组大小

- 支持多种编程语言

  C++、C#、Go、Ruby、Python、Java、PHP、Objective-C等

- 数据压缩

  以int类型的变量为例，当其值为1时，只需要占用一个字节，而不是4个字节

- 语法简洁

- 方便扩展

- 兼容性好

  当数据结构定义发生变化时，在解析老数据时，可以正常解析

## 版本

Protobuf最初发布时，支持的版本为proto2，从Protobuf 3.0开始，推出proto3，默认为proto2，如果要使用proto3的语法，需要在.proto文件开头指定syntax = "proto3"。

proto3相对proto2的主要修改如下：

- proto2的字段规则有三种：required、optional、repeated，每个字段都必须指定一种字段规则，在proto3中移除了required，只支持optional和repeated，如果没为字段指定规则，则字段可以不赋值
- 对于不指定字段规则的字段，不能使用has_xxx()函数是否有值，因为总会有一个默认值
- 如果在proto3中指定了optional规则，可以继续使用has_xxx()函数，来判断字段是否有值
- 移除了default关键字，不能为字段指定默认值，如果不给字段赋值，则序列化时跳过此变量，使用系统默认值
- 枚举类型，第一个枚举值必须为0，而proto2可为任意数值
- 对于数值型的repeated字段，在proto2中需要手动指定[packed=true]，在proto3中默认就指定了[packed=true]，此限定可大大降低序列化后的包大小
- 在proto2中，如果解析时，遇到未知字段，会跳过，继续解析后面的数据，proto3会直接报错
- 移除proto2中的extension
- 新增Any类型，可以存储任意类型数据
- 支持更多语言

原生Protobuf不支持lua，如果要生成Lua代码，需要protoc-gen-lua插件，但是这个插件支持的protobuf比较老，对Protobuf3的支持程序未知，需要再研究

在最新的27.x版本中，改为使用edition取代syntax = "proto2"和syntax = "proto3"，可以使用官方提供的prototiller工具将旧的proto文件转成edition版本，详细的说明见https://protobuf.com.cn/editions/

## 编译

### 下载源码

```shell
git clone https://github.com/protocolbuffers/protobuf
cd protobuf
git submodule update --init --recursive
```

使用cmake工具编译源码，不同版本的protobuf，CMakeLists.txt文件所在位置不同，以3.18版本为例，CMakeLists.txt放在cmake目录下

### 准备编译环境

使用cmake工具来编译

#### windows

cmake、visual studio

#### linux

cmake、g++

### 编译

```shell
mkdir builddir
cd builddir
cmake ../cmake  -DCMAKE_INSTALL_PREFIX=./install # cmake自己侦测编译器
//如果要编译动态库版本，使用下面命令
//cmake ../cmake -DBUILD_SHARED_LIBS=1 -DCMAKE_INSTALL_PREFIX=./install
cmake --build . --config release --target install #编译release版本
cmake --build . --config debug --target install #编译debug版本
```

-DCMAKE_INSTALL_PREFIX用于指定安装目录

编译安装完成后，会生成以下文件：

- install/lib/libprotobuf.lib 

  完整功能的库文件

- install/lib/libprotobuf-lite.lib 

  保留基础的序列化、反序列化功能，在libprotobuf.lib的基础上，精简了反射等功能，因此库文件会更小，大约是libprotobuf.lib的1/7左右。

  若要使用libprotobuf-lite.lib，需要在.proto文件中，指定option optimize_for = LITE_RUNTIME;;

- install/bin/protoc.exe

  用于解析.proto文件，生成指定语言的代码文件
  
- install/include

  头文件目录

## 使用

在.proto文件中定义消息，定义消息时，需要为每一个字段分配一个序号，序号为正数，不可重复，最大支持2 ^ 29 - 1个字段，19000 - 19999之间是Protobuf的保留字段，不要使用，protobuf虽然支持很大的字段序号，但是字段越大，需要的存储空间越大，编码效率也越低，所以，建议从1开始，顺序递增。

```protobuf
syntax = "proto3";//指定使用proto3语法

// package Test;//命名空间，或者包，可有可无，看需要
// import "other_proto.proto"; //导入另一个proto文件

message TestMessage
{
	int32	i32 = 1;
	sint32	si32 = 2;
	....
};
```

定义好协议之后，使用protoc.exe工具，生成指定语言的代码：

```shell
//生成C++代码
protoc.exe --cpp_out=out_dir --proto_path=proto_path *.proto	//--proto_path指定proto文件搜索路径

//生成C#代码
protoc.exe --csharp_out=out_dir --proto_path=proto_path*.proto
```

## 数据类型

protobuf支持以下数据类型:

| protobuf | C++    | C#         |                      |
| -------- | ------ | ---------- | -------------------- |
| bool     | bool   | bool       | 默认值false          |
| int32    | int32  | int        | varint变长编码       |
| int64    | int64  | long       | varint变长编码       |
| uinit32  | uint32 | uint       | varint变长编码       |
| uint64   | uint64 | ulong      | varint变长编码       |
| sint32   | int32  | int        | varint变长编码       |
| sint64   | int64  | long       | varint变长编码       |
| fixed32  | uint32 | uint       | 固定4字节            |
| fixed64  | uint64 | ulong      | 固定8字节            |
| sfixed32 | int32  | int        | 固定4字节            |
| sfixed64 | int64  | long       | 固定8字节            |
| string   | string | string     | UTF8编码格式的字符串 |
| bytes    | string | ByteString | 字节序列             |
| float    | float  | float      | 4字节                |
| double   | double | double     | 8字节                |
| enum     | enum   | enum       | varint变长编码       |

## 编码

protobuf对不同的数值型字段，使用两种编码方式：

- varint变长编码

  适应于int32、int64、uint32、uint64、sint32、sint64、enum、bool等

  越小的数，占用的字节数越少，规则为：

  1. 每个字节的最高位为标志位，为0表示当前是最后一个字节，为1表示后面的BYTE还是当前字段的数据
  2. 每个字节的后7位，表示实际的数据
  3. 采用小端字节序

  具体表示方式如下：

  ```c++
  [0, 2^7 -1]   		 0xxxxxxx
  [2^7，2^14 -1]		1xxxxxxx 0xxxxxxx
  [2^14, 2^21 -1]		 1xxxxxxx 1xxxxxxx 0xxxxxxx
  [2^21, 2^28 -1]		 1xxxxxxx 1xxxxxxx 1xxxxxxx 0xxxxxxx
  [2^28, 2^35 -1]		 1xxxxxxx 1xxxxxxx 1xxxxxxx 1xxxxxxx 0xxxxxxx
  ```

  因为每个字节只有7位用来表示具体数值，所以，要完整表示一个32位数值，最多需要用到5个字节，同理，要完整表示一个64位数值，最多需要用到10个字节。

  在数据类型部分，可以看到int32和sint32对应的C++类型都是int，这两者之间有什么区别呢？测试proto如下：

  ```c++
  message Int32Message
  {
      int32 value = 1;
  }
  
  message SInt32Message
  {
      sint32 value = 1;
  }
  ```

  设置value = -1，然后查看序列化后的值，输出如下：

  ```c++
  int32 msg size 11, sint32 msg size 2
  ```

  int32类型的-1，占用了10个字节（11个字节中的第一个字节，存储字段序号），在protobuf的源码中，针对负数的序列化代码如下：

  ```c++
  inline void CodedOutputStream::WriteVarint32SignExtended(int32 value) {
    if (value < 0) {
      WriteVarint64(static_cast<uint64>(value));
    } else {
      WriteVarint32(static_cast<uint32>(value));
    }
  }
  ```

  负的int32字段，会调用WriteVarint64，当成一个64位int的最大值来处理，刚好是10个字节

  而sint32字段，-1只占用一个字节的大小，这是因为sint32采用了zigzag编码

- zigzag编码

  sint32、sint64类型

  zigzag将有符号数统一映射到无符号数，详细映射规则如下表：

  | Signed Original | 映射为     |
  | --------------- | ---------- |
  | 0               | 0          |
  | -1              | 1          |
  | 1               | 2          |
  | -2              | 3          |
  | 2               | 4          |
  | 2147483647      | 4294967294 |
  | -2147483647     | 4294967295 |

  然后，对映射后的数值，采用varint编码，所以sint32类型的-1只需要一个字节来表示。

- 固定长度编码

  fixed32、sfixed32、float 固定占用4个字节
  
  fixed64、sfixed64、double固定占用8个字节
  
- Length-delimited

  对于string、bytes、内嵌message、repeated字段，使用这种方式，先写data数据长度，再写实际数据，数据长度用varint编码方式

## 序列化

protobuf在不同的字段类型，进行了分类，每一类都对应一个TypeID，如下：

| Type ID | Name   | Used For                                                 |
| ------- | ------ | -------------------------------------------------------- |
| 0       | varint | int32、int64、uint32、uint64、sint32、sint64、enum、bool |
| 1       | i64    | fixed64、sfixed64、double                                |
| 2       | LEN    | string、bytes、内嵌消息、packed repeated字段             |
| 3       | SGROUP | 废弃                                                     |
| 4       | EGROUP | 废弃                                                     |
| 5       | i32    | fixed32、sfixed32、float                                 |

序列化一个字段时，需要写入

- 字段序号

- 字段编码类型TypeID

- 实际数据

为了节省空间，Protobuf将字段序号和TypeID组合到一起，有一个专门的名词叫做Tag

对于TypeID为0、1、5的字段，写入格式为Tag | Data

对于TypeID为2的字段，写入格式为Tag | DataLen | Data

以string为例，序列化步骤如下：

1. 计算Tag = 字段序号 << 3 + TypeID
2. 计算Tag的varint编码
3. 将Tag写入输出缓冲区(小端字节序)
4. 计算字符串长度的varint编码StrLen
5. 将StrLen写入输出缓冲区(小端字节序)
6. 将string的实际数据写入输出缓冲区

在源码中，对字段的序列化处理都在类WireFormatLite中，如：

```c++
inline uint8_t* WireFormatLite::WriteInt32ToArray(int field_number,
                                                  int32_t value,
                                                  uint8_t* target) {
  target = WriteTagToArray(field_number, WIRETYPE_VARINT, target);//写Tag
  return WriteInt32NoTagToArray(value, target);//写实际数据
}
```

## 反序列化

在反序化时，调用Message::ParseFromArray函数，经过一系列处理后，会调用虚函数_InternalParse进行实际的数据读取和解析，伪代码如下：

```c++
const char *MyMessage::_InternalParse(const char *ptr, ::pbi::ParseContex* ctx)
{
    while (!ctx->Done(&ptr))
    {
        ::pbi::ReadTag(ptr, &tag);//读取Tag
        解析字段序号FieldNum
        switch(FieldNum)
        {
            case XXX:
                读取并解析实际数据
        }
    }
}
```

## 性能

### 数值字段性能

protobuff中，不同的字段类型，其编码形式不一样，不同的编码方式对性能有什么样的影响呢？

以int32、sint32、fixed32、sfixed32为例，测试数值型字段的序列化性能（64位可以参考32位字段），循环100W次的输出如下：

```c++
//负值
value = -1, fixed32 serialize cost 8938 微秒
value = -1, sfixed32 serialize cost 9092 微秒
value = -1, int32 serialize cost 13402 微秒
value = -1, sint32 serialize cost 9843 微秒
//小整数    
value = 1, fixed32 serialize cost 8956 微秒
value = 1, sfixed32 serialize cost 9125 微秒
value = 1, int32 serialize cost 9931 微秒
value = 1, sint32 serialize cost 9943 微秒
//大整数    
value = 2147483647, fixed32 serialize cost 8913 微秒
value = 2147483647, sfixed32 serialize cost 9140 微秒
value = 2147483647, int32 serialize cost 10742 微秒
value = 2147483647, sint32 serialize cost 10980 微秒
```

当值为负数时，fixed32、sfixed32性能基本相同，sint32耗时比前两者高出将近10%，int32因为把负数当成了64位int来处理，需要编码和序列化更多的字节，所以性能表现最差，比fixed32和sfixed32耗时高出40%多。

当值为小的正数时，sint32、int32比fixed32、sfixed32耗时高出接近10%，值越大，耗时越多。

### 综合性能对比

以游戏中的LCRetCharList消息为模板做测试，对比下Protobuf和packgenerator的性能

具体的消息定义可见测试工程源码，两种消息，都填充相同的内容

测试结果如下：

```c++
//protobuf创建消息并序列化
serialize msg LCRetCharList, loop count 1000000, protobuf time used 2934963 微秒
//protobuf反序列化
deserialize msg LCRetCharList, loop count 1000000, proto time used 2950556 微秒

//packgenerator序列化
serialize msg LCRetCharList, loop count 1000000, packgenerator time used 1094577 微秒
//packgenerator反序列化
deserialize msg LCRetCharList, loop count 1000000, packgenerator time used 725070 微秒
```

packgenerator无论序列化还是反序列化，性能都比protobuf高出好几倍。

分析后，发现protobuf在对repeated字段，调用add_xxx时都会触发内存分配操作，这个是非常耗时的。

针对内存操作带来的性能开销，protobuf也提供了一种方案，即Arena机制。

## Arena分配机制

旨在降低C++版本中内存分配和回收的性能开销

### 原理

​		Arena维护一个内存块链表BlockList，在创建Arena对象时，预先分配一个内存块，并将其加入BlockList。创建Message和内部对象的时候，都是在分配好的内存块上执行placement new，从而避免触发new操作。Message对象内部会有一个变量，用来记录此Message所属的Arena，在对消息字段进行操作时，如果需要new，都在所属Arena上执行placement new。

​		Arena在创建时允许指定初始的内存块，如果不指定，则默认分配一个256Byte的内存块，内存块耗尽时，会触发扩容，创建一个新的内存块，并加入BlockList，新内存块的大小为上一个内存块的大小 * 2，最大不能超过设定的最大内存值。

​		使用Arena机制创建的message，不能手动delete，其生命周期与Arena对象相同，在Arena对象释放时，会统一释放在此Arena上创建出来的Message。

### 使用

Arena提供了三个构造函数：

```c++
Arena();//默认构造函数
Arena(char *block, size_t block_size);//指定初始内存块和内存块大小
Arena(ArenaOptions ao);//可以指定初始内存块、初始内存块大小、Arena最大内存块大小
```

引入Arena机制后，修改LCRetCharList消息的创建如下：

```c++
#include <google/protobuf/arena.h>

//预分配1M空间
const size_t block_size = 1 * 1024 * 1024;
char *pArenaBlock = new char[block_size];
Assert(nullptr != pArenaBlock);

//循环测试性能
for (int i = 0; i < 1000000; ++i)
{
    //创建Arena对象
    google::protobuf::Arena arena(pArenaBlock, block_size);
    //创建消息
    auto msg = google::protobuf::Arena::CreateMessage<LCRetCharList>(&arena);
    Assert(nullptr != msg);
    //消息创建后，其它操作跟之前保持一致
}
```

再次运行性能测试程序，记录输出结果如下：

```c++
serialize msg LCRetCharList, loop count 1000000, protobuf time used 1392924 微秒
serialize msg LCRetCharList, loop count 1000000, packgenerator time used 951083 微秒
deserialize msg LCRetCharList, loop count 1000000, proto time used 1529971 微秒
deserialize msg LCRetCharList, loop count 1000000, packgenerator time used 722319 微秒
```

protobuf在创建消息，并序列化操作上，耗时从293万微秒，减少到了139万微秒

在反序列化操作上，耗时从295万微秒，减少到了153万微秒

性能提升了50%左右。

### 注意事项

- 在老版本的proto中，如果要使用arena分配，需要在proto文件中，显式开启 option cc_enable_arenas = true;(3.18版本无需显式指定，之前的版本需要在使用时进行测试是否需要显式开启)
- 由于Arena分配的对象生命周期跟arena一样，不建议使用全局的Arena对象，这会导致进程的内存占用持续增长，建议每个message对应一个Arena，message使用完就即时释放。
- 假如消息中，没有repeated字段时，可以不使用Arena机制来创建message
- 字符串字段的内存分配操作不受Arena影响，始终是在堆上分配和释放
- Arena线程安全
- Message的移动构造、移动赋值、Swap函数：如果两个message都在堆上分配，或者在相同的arena分配，执行浅拷贝，否则执行深拷贝

## 空间占用

在字符串和二进制数据的处理上，同常规思想一样，先写数据长度、再写实际数据，所以这两种数据的空间占用不是protobuf优化的重点。Protobuf区别于常规序列化思想的一点，就是数值字段的编码机制，采用varint编码，设计者不用操心一个字段应该是int8、int16、还是int32，统一使用int32，Protobuf来保证小的数值占用较少的存储空间，不仅优化了空间占用，也不用担心int8不够存了，需要将字段改为int16等，扩展性更强。

设计者需要关心的是：

1. 当有负值时，尽量使用sint，而不是int，无论在空间占用，还是性能上sint性能都更优
2. 字段值如果是大数值（超过2^21），可以考虑使用fixed32，固定宽度的字段类型

## option

分为三个级别：FileOptions、MessageOptions、FieldOptions，所有的option可以在descriptor.proto中找到

常用的option说明

- optimize_for

  编译优化选项，FileOptions

  1. SPEED

     默认选项，生成的代码拥有更快的运行速度，缺点是需要链接的库文件libprotobuf.lib比较大

  2. CODE_SIZE

     只重载Message中必要的函数，因此生成的代码行数较少，编译速度快一些，需要链接libprotobuf.lib库

  3. LITE_RUNTIME

     舍弃Descriptors和反射功能，链接libprotobuf-lite.lib，生成的可执行文件体积更小，适用于对资源要求苛刻的场景

  以LCRetCharList消息为测试模板，三种优化选项的性能对比如下图所示：

  ![./optimize.png](.\optimize.png)

  CODE_SIZE选项，性能极低，几乎是SPEED选项下的10倍耗时

  LITE_RUNTIME选项的反序列化性能较低，是SPEED的两倍耗时

  另外，Arena机制在CODE_SIZE优化选项下几乎没什么效果，性能只提升了5%左右

  下表展示生成代码行数、可执行程序大小的对比：

  | 选项         | 代码行数 | 可执行文件大小 |
  | ------------ | -------- | -------------- |
  | SPEED        | 1400     | 2M             |
  | CODE_SIZE    | 426      | 1.35M          |
  | LITE_RUNTIME | 1296     | 500K           |

- cc_enable_arenas

  FileOptions

  是否开启Arena分配，true为开启，提升生成C++代码的性能

- deprecated

  如果某个proto文件废弃了，不想生成对应的代码，可以加这个选项 option deprecated = true;

  这个选项同样适用于字段、枚举定义等

## Attention

- 游戏项目中，优化选项保持默认的SPEED，追求运行效率
- Arena机制对提升C++代码的性能，有显著效果，但是使用时，一定要做好封装，否则很容易出现内存相关的问题
- 不用的字段，最好使用reserved关键字声明一下，避免重用后导致的老数据解析问题
- 定义数值字段类型时，对有可能为负值的字段，使用sint32/sint64类型
- 对于string字段类型，会做UTF8格式检查，如果不必做UTF8检查，建议使用bytes
- 字段序号从1开始，递增，尽量避免大序号
- 为兼容proto2和proto3语法，枚举类型的第一值建议用0

## 项目应用

若要在项目中应用protobuf，需要修改：

- 设计服务器/客户端的胶水代码，适应当前项目框架

- 重新设计packgenerator生成的协议，由protoc.exe生成具体的协议代码，而packgenerator只负责生成胶水代码
- protobuf不支持生成lua代码，需要引入第三方插件，或者自行实现
- 修改服务器和客户端底层的协议处理

服务器的胶水代码大致设计：

```c++
class LCRetCharListPak : public Packet
{
public:
    LCRetCharList();
    virtual ~LCRetCharList();
    
    virtual UINT	Execute(Player* pPlayer);
    virtual PacketID_t GetPacketID() const { return PACKET_LC_RETCHARLIST; }
    virtual UINT	GetPacketSize() const { return (UINT)m_Message.ByteSizeLong(); }
public:
    LCRetCharList	m_Message;
};

class LCRetCharListPakFactory ：public PacketFactoryBaseWithDestoryPacket
{
public:
    Packet*	CreatePacket() { return new LCRetCharList(); }
    PacketID_t GetPacketID() const { return PACKET_LC_RETCHARLIST; }
};

class LCRetCharListHandler
{
public:
    static UINT Execute(LCRetCharListPak* pPacket, Player *ppLayer)
};
```

具体的Read/Write，可以封装到底层，调用Message::SerializeToArray/Message::ParseFromArray即可。

## 总结

protobuf本身具有多语言支持、跨平台、语法简洁等优势，非常适用于游戏中的前后端通信，同一个协议，程序员只需要在.proto文件中定义一份，然后使用工具生成不同语言的版本，可以避免使用多种语言重复定义同一个协议，最主要的是，可以保证前后端的数据在解析方式和解析顺序上的强一致，不会出现数据错乱的问题。

支持丰富的数据类型和字段规则，其repeated字段，对消息中数组类型支持更好，提供了丰富的访问接口，使编码时更加易于使用，而且程序员在定义消息时，无需关心数组上限。

是远程过程调用库gRPC的依赖，如果项目中用到了gRPC，那protobuf就是必须引进的库。

与项目中当前正在使用的pack generator工具相比：

- 功能更丰富，支持optional、repeated、map等
- 支持import其它proto文件

- 扩展性更强
- 当消息中有repeated字段时，序列化和反序列化性能较差，即使使用Arena机制优化，开销还是比packgenerator高将近40%。
- 使用时，需要做进一步封装，以适用现有服务器的消息分发框架

