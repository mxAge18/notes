# proto buffer

## 说明

- 说明文档online：<https://developers.google.com/protocol-buffers/docs/proto3>

## 简单例子

```protobuf
syntax = "proto3";      //指定版本信息，不指定会报错
package pb;      //后期生成go文件的包名
//message为关键字，作用为定义一种消息类型
message CustomerMsg{
    //    名字
    string name = 1;
    //    年龄
    int32  age = 2 ;
    //    公司
    string company = 3;
}

enum test{
    int32 age = 0;
}
```

> - protobuf消息的定义（或者称为描述）通常都写在一个以 .proto 结尾的文件中。
> - 该文件的第一行指定正在使用`proto3`语法：如果不这样做，协议缓冲区编译器将假定正在使用proto2。这也必须是文件的第一个非空的非注释行。
>
> - 第二行package指明当前是pb包（生成go文件之后和Go的包名保持一致）
>
> - 最后message关键字定义一个CustomerMsg消息体，类似于go语言中的结构体，是包含一系列类型数据的集合。许多标准的简单数据类型都可以作为字段类型，包括`bool`，`int32`， `float`，`double`，和`string`。也可以使用其他message类型作为字段类型。
> - 在message中有一个字符串类型的value成员，该成员编码时用1代替名字。我们知道，在json中是通过成员的名字来绑定对应的数据，但是Protobuf编码却是通过成员的唯一编号来绑定对应的数据，因此Protobuf编码后数据的体积会比较小，能够快速传输，缺点是不利于阅读。

### **message的格式说明**

消息由至少一个字段组合而成，类似于Go语言中的结构体，每个字段都有一定的格式：

```message格式说明
//注释格式 注释尽量也写在内容上方
（字段修饰符）数据类型 字段名称 = 唯一的编号标签值;
```

> - 唯一的编号标签：代表每个字段的一个唯一的编号标签，在同一个消息里不可以重复。这些编号标签用与在消息二进制格式中标识你的字段，并且消息一旦定义就不能更改。需要说明的是标签在1到15范围的采用一个字节进行编码，所以通常将标签1到15用于频繁发生的消息字段。编号标签大小的范围是1到2的29次。19000-19999是官方预留的值，不能使用。
> - 注释格式：向.proto文件添加注释，可以使用C/C++/java/Go风格的双斜杠（//） 语法格式或者`/*.....*/`

### message常见的数据类型与go中类型对比

| .proto类型 | Go类型  | 介绍                                                              |
| ---------- | ------- | ------------------------------------------------------------    |
| double     | float64 | 64位浮点数                                                        |
| float      | float32 | 32位浮点数                                                        |
| int32      | int32   | 使用可变长度编码。编码负数效率低下——如果你的字段可能有负值，请改用sint32。 |
| int64      | int64   | 使用可变长度编码。编码负数效率低下——如果你的字段可能有负值，请改用sint64。 |
| uint32     | uint32  | 使用可变长度编码。                                                 |
| uint64     | uint64  | 使用可变长度编码。                                                 |
| sint32     | int32   | 使用可变长度编码。符号整型值。这些比常规int32s编码负数更有效             |
| sint64     | int64   | 使用可变长度编码。符号整型值。这些比常规int64s编码负数更有效             |
| fixed32    | uint32  | 总是四字节。如果值通常大于228则比uint 32更有效                        |
| fixed64    | uint64  | 总是八字节。如果值通常大于256则比uint64更有效                         |
| sfixed32   | int32   | 总是四字节。                                                      |
| sfixed64   | int64   | 总是八字节。                                                      |
| bool       | bool    | 布尔类型。                                                        |
| string     | string  | 字符串必须始终包含UTF - 8编码或7位ASCII文本                          |
| bytes      | []byte  | 可以包含任意字节序列                                                |

### enum关键字

在定义消息类型时，可能会希望其中一个字段有一个预定义的值列表。比如说，电话号码字段有个类型，这个类型可以是，home,work,mobile。我们可以通过enum在消息定义中添加每个可能值的常量来非常简单的执行此操作。

- enum的第一个常量映射为0，每个枚举定义**必须**包含一个映射到零的常量作为其第一个元素。这是因为：
- 必须有一个零值，以便我们可以使用0作为数字默认值。
- 零值必须是第一个元素，以便与proto2语义兼容，其中第一个枚举值始终是默认值。

### repeat关键字

repeadted关键字类似与go中的切片，编译之后对应的也是go的切片

### oneof关键字

如果有一个包含许多字段的消息，并且最多只能同时设置其中的一个字段，则可以使用oneof功能(union联合)

```protobuf
syntax = "proto3";  //指定版本信息，不指定会报错
package pb; //后期生成go文件的包名
//message为关键字，作用为定义一种消息类型
message Person{
    string name = 1;
    int32  age = 2 ;
    //定义一个message
    message PhoneNumber {
        string number = 1;
        PhoneType type = 2;
    }
    repeated PhoneNumber phone = 3;
    
    oneof data{
        string school = 5;
        int32 score = 6;
    }
}

//enum为关键字，作用为定义一种枚举类型
enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
}
```

## 定义RPC service

如果需要将message与RPC一起使用，则可以在`.proto`文件中定义RPC服务接口，protobuf编译器将根据你选择的语言生成RPC接口代码。示例如下：

```protobuf
//定义RPC服务
service HelloService {
    rpc Hello (Person)returns (Person);
}
```
