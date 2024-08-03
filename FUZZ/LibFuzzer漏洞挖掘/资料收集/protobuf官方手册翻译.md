# 概述
- 原文：https://developers.google.com/protocol-buffers/docs/overview
- 参考译文：https://colobu.com/2015/01/07/Protobuf-language-guide/
```cpp
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3;
}
```
## 以上结构表示一个数据结构类型
## 数据结构内部的字段类型可以分为标量类型（int32 string等），也可以为合成类型（枚举等）
## 数据结构内部的字段均有唯一的数字标识符，用于在消息的二进制格式中识别各个字段
- [1,15]之内的标识号在编码的时候会占用一个字节。包括字段编号和字段的类型，[16,2047]之内的标识号则占用2个字节。所以应该为那些频繁出现的消息元素保留 [1,15]之内的标识号。切记：要为将来有可能添加的、频繁出现的标识号预留一些标识号。
- 最小的标识号可以从1开始，最大到2^29 - 1, or 536,870,911。不可以使用其中的[19000－19999]的标识号(FieldDescriptor::kFirstReservedNumber through FieldDescriptor::kLastReservedNumber)， Protobuf协议实现中对这些进行了预留。如果非要在.proto文件中使用这些预留标识号，编译时就会报警。
## 指定字段规则
- required：一个格式良好的消息一定要含有1个这种字段。表示该值是必须要设置的；
- optional：消息格式中该字段可以有0个或1个值（不超过1个）。
- repeated：在一个格式良好的消息中，这种字段可以重复任意多次（包括0次）。重复的值的顺序会被保留。表示该值可以重复，相当于java中的List。
- 由于一些历史原因，基本数值类型的repeated的字段并没有被尽可能地高效编码。在新的代码中，用户应该使用特殊选项[packed=true]来保证更高效的编码。如：
  - repeated int32， samples = 4 [packed=true];
- required是永久性的：在将一个字段标识为required的时候，应该特别小心。如果在某些情况下不想写入或者发送一个required的字段，将原始该字段修饰符更改为optional可能会遇到问题——旧版本的使用者会认为不含该字段的消息是不完整的，从而可能会无目的的拒绝解析。在这种情况下，你应该考虑编写特别针对于应用程序的、自定义的消息校验函数。Google的一些工程师得出了一个结论：使用required弊多于利；他们更愿意使用optional和repeated而不是required。当然，这个观点并不具有普遍性。
## 在一个.proto文件中可以定义多个消息类型。在定义多个相关的消息的时候，这一点特别有用——例如，如果想定义与SearchResponse消息类型对应的回复消息格式的话，你可以将它添加到相同的.proto文件中，如：
```cpp
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3;
}
message SearchResponse {
 ...
}
```
- 合并消息会导致膨胀。虽然可以在单个.proto文件中定义多种消息类型（例如，消息，枚举和服务），但是当在单个文件中定义大量具有不同依赖性的消息时，也会导致依赖性膨胀。建议每个.proto文件尽可能少地包含消息类型
## 向.proto文件添加注释，可以使用C/C++/java风格的双斜杠（//）以及/*...*/ 语法格式，如：
```cpp
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;  // Which page number do we want?
  optional int32 result_per_page = 3;  // Number of results to return per page.
}
```
## 针对消息的保留字段
- 如果您通过完全删除某个字段或将其注释掉来更新消息类型，则将来的用户可以在对该类型进行自己的更新时重用该字段号。如果他们以后加载同一.proto的旧版本，可能会导致严重的问题，包括数据损坏，隐私错误等。确保不会发生这种情况的一种方法是将已删除字段的字段编号（和/或名称，这也可能导致JSON序列化问题）指定为reserved。 如果将来有任何用户尝试使用这些字段标识符，则协议缓冲区编译器将抱怨。
- 请注意，您不能在同一reserved语句中混合使用字段名和字段号。
```cpp
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```
## 当用protocolbuffer编译器来运行.proto文件时，编译器将生成所选择语言的代码，这些代码可以操作在.proto文件中定义的消息类型，包括获取、设置字段值，将消息序列化到一个输出流中，以及从一个输入流中解析消息。
- 操作protobuf所使用的api：https://developers.google.com/protocol-buffers/docs/reference/overview
- 对C++，编译器会为每个.proto文件生成一个.h文件和一个.cc文件，.proto文件中的每一个消息有一个对应的类。
- 对Java，编译器为每一个消息类型生成了一个.java文件，以及一个特殊的Builder类（该类是用来创建消息类接口的）。
- 对Python，有点不太一样——Python编译器为.proto文件中的每个消息类型生成一个含有静态描述符的模块，，该模块与一个元类（metaclass）在运行时（runtime）被用来创建所需的Python数据访问类。
- 对于Go，编译器会生成一个.pb.go文件，其中包含文件中每种消息类型的类型。
## 标量数据类型
- 一个标量消息字段可以含有一个如下的类型——该表格展示了定义于.proto文件中的类型，以及与之对应的、在自动生成的访问类中定义的类型：
![](pic/2021-05-15-10-35-26.png)
- 当使用protobuf encoding对protobuf消息进行序列化时，可以了解有关这些类型的编码方式的更多信息:https://developers.google.com/protocol-buffers/docs/encoding
## Optional字段和默认值
- 如上所述，消息描述中的一个元素可以被标记为“可选的”（optional）。一个格式良好的消息可以包含0个或一个optional的元素。当解析消息时，如果它不包含optional的元素值，那么解析出来的对象中的对应字段就被置为默认值。默认值可以在消息描述文件中指定。例如，要为 SearchRequest消息的result_per_page字段指定默认值10，在定义消息格式时如下所示：
```cpp
optional int32 result_per_page = 3 [default = 10];
```
- 如果没有为optional的元素指定默认值，就会使用与特定类型相关的默认值：对string来说，默认值是空字符串。对bool来说，默认值是false。对数值类型来说，默认值是0。对于字节，默认值为空字节字符串。对枚举来说，默认值是枚举类型定义中的第一个值。这意味着在将值添加到枚举值列表的开头时必须格外小心。有关如何安全地更改定义的准则，请参阅“更新消息类型”部分。
## 枚举类型
- 当需要定义一个消息类型的时候，可能想为一个字段指定某“预定义值序列”中的一个值。例如，假设要为每一个SearchRequest消息添加一个 corpus字段，而corpus的值可能是UNIVERSAL，WEB，IMAGES，LOCAL，NEWS，PRODUCTS或VIDEO中的一个。 其实可以很容易地实现这一点：通过向消息定义中添加一个枚举（enum）就可以了。一个enum类型的字段只能用指定的常量集中的一个值作为其值（如果尝 试指定不同的值，解析器就会把它当作一个未知的字段来对待）。在下面的例子中，在消息格式中添加了一个叫做Corpus的枚举类型——它含有所有可能的值 ——以及一个类型为Corpus的字段：
```cpp
message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;
  optional int32 result_per_page = 3 [default = 10];
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  optional Corpus corpus = 4 [default = UNIVERSAL];
}
```
- 您可以通过将相同的值分配给不同的枚举常量来定义别名。为此，您需要将allow_alias选项设置为true，否则协议编译器将在找到别名时生成错误消息。
```cpp
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;//这里STARTED RUNNIN两者值相同，两者互为彼此的别名
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;//因为未设置option allow_alias = true;，故不允许使用别名
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```
- 枚举常量必须在32位整型值的范围内。因为enum值是使用可变编码方式的，对负数不够高效，因此不推荐在enum中使用负数。如上例所示，可以在 一个消息定义的内部或外部定义枚举——这些枚举可以在.proto文件中的任何消息定义里重用。当然也可以在一个消息中声明一个枚举类型，而在另一个不同 的消息中使用它——采用MessageType.EnumType的语法格式。
- 当对一个使用了枚举的.proto文件运行protocol buffer编译器的时候，生成的代码中将有一个对应的enum（对Java或C++来说），或者一个特殊的EnumDescriptor类（对 Python来说），它被用来在运行时生成的类中创建一系列的整型值符号常量（symbolic constants）。
- 有关如何在应用程序中使用消息枚举的更多信息，请参见针对所选语言生成的代码指南。
- https://developers.google.com/protocol-buffers/docs/reference/overview
## 针对枚举的保留字段
- 如果您通过完全删除枚举条目或将其注释掉来更新枚举类型，则将来的用户在自己对类型进行更新时可以重用数值。如果他们以后加载同一.proto的旧版本，可能会导致严重的问题，包括数据损坏，隐私错误等。 确保不会发生这种情况的一种方法是指定保留已删除条目的数字值（和/或名称，这也可能导致JSON序列化问题）。 如果将来有任何用户尝试使用这些标识符，则协议缓冲区编译器会抱怨。 您可以使用max关键字指定保留的数值范围达到最大可能值。
```cpp
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```
- 请注意，您不能在同一reserved语句中混合使用字段名和数字值。
## 使用其他消息类型
- 你可以将其他消息类型用作字段类型。例如，假设在每一个SearchResponse消息中包含Result消息，此时可以在相同的.proto文件中定义一个Result消息类型，然后在SearchResponse消息中指定一个Result类型的字段，如：
```cpp
message SearchResponse {
  repeated Result result = 1;
}
message Result {
  required string url = 1;
  optional string title = 2;
  repeated string snippets = 3;
}
```
## 导入定义
- 在上面的例子中，Result消息类型与SearchResponse是定义在同一文件中的。如果想要使用的消息类型已经在其他.proto文件中已经定义过了呢？你可以通过导入（importing）其他.proto文件中的定义来使用它们。要导入其他.proto文件的定义，你需要在你的文件的顶部添加一个导入声明，如：
```cpp
import "myproject/other_protos.proto";
```
- 默认情况下，您只能使用直接导入的.proto文件中的定义。 但是，有时您可能需要将.proto文件移动到新位置,为了避免直接移动.proto文件所导致的修改所有程序调用，可以在原.proto文件位置设置虚拟的.proto文件，该虚拟的.proto文件终导入的了位于新位置的.proto文件，（通过import public关键字）所以当代码将该虚拟.proto文件导入时，就导入了新位置的.proto文件
- 新位置的.proto文件
```cpp
// new.proto
// All definitions are moved here
```
- 原来位置的虚拟.proto文件，用import public关键字引入了新位置的.proto文件
```cpp
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```
- 实际使用的.proto文件，仅需引入原位置的虚拟.proto文件，即可使用new.proto中的定义
```cpp
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```
- 协议编译器使用-I /-proto_path标志在协议编译器命令行中指定的一组目录中搜索导入的文件。 如果未给出标志，它将在调用编译器的目录中查找。 通常，应将--proto_path标志设置为项目的根目录，并对所有导入使用完全限定的名称。
## 使用proto3消息类型
可以导入proto3消息类型并在proto2消息中使用它们，反之亦然。 但是，不能在proto3语法中使用proto2枚举。
## 嵌套类型
- 你可以在其他消息类型中定义、使用消息类型，在下面的例子中，Result消息就定义在SearchResponse消息内，如：
```cpp
message SearchResponse {
  message Result {
    required string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
  repeated Result result = 1;
}
```
- 如果你想在它的父消息类型的外部重用这个消息类型，你需要以Parent.Type的形式使用它，如
```cpp
message SomeOtherMessage {
  optional SearchResponse.Result result = 1;
}
```
当然，你也可以将消息嵌套任意多层，如：
```cpp
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      required int64 ival = 1;
      optional bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      required int32 ival = 1;
      optional bool  booly = 2;
    }
  }
}
```
## 组
- 该特性已经被弃用，使用嵌套消息代替该特性
## 更新一个消息类型
- 如果一个已有的消息格式已无法满足新的需求——如，要在消息中添加一个额外的字段——但是同时旧版本写的代码仍然可用。不用担心！更新消息而不破坏已有代码是非常简单的。在更新时只要记住以下的规则即可。
- 不要更改任何已有的字段的数值标识。
- 您添加的任何新字段都应该是optional或repeated的。 这意味着任何使用“旧”消息格式通过代码序列化的消息都可以由新生成的代码解析，因为它们不会缺少任何required的元素。 您应该为这些元素设置合理的默认值，以便新代码可以与旧代码生成的消息正确交互。 同样，由新代码创建的消息可以由旧代码解析：旧的二进制文件在解析时只会忽略新字段。 但是，未知字段不会被丢弃，并且如果消息随后被序列化，则未知字段也会与之一起进行序列化–因此，如果消息传递给新代码，则新字段仍然可用
- 可以删除非required的字段，前提是只要在更新后的消息类型中不再使用该字段号。 您可能想要重命名该字段，或者添加前缀“ OBSOLETE_”，或者使用reserved机制保留该字段编号，以使.proto的未来用户不会意外重用该编号。
- 一个非required的字段可以转换为一个扩展，反之亦然——只要它的类型和标识号保持不变。（扩展的详细内容之后介绍）
- int32, uint32, int64, uint64,和bool是全部兼容的，这意味着可以将这些类型中的一个转换为另外一个，而不会破坏向前、 向后的兼容性。如果解析出来的数字与对应的类型不相符，那么结果就像在C++中对它进行了强制类型转换一样（例如，如果把一个64位数字当作int32来 读取，那么它就会被截断为32位的数字）。
- sint32和sint64彼此兼容，但与其他整数类型不兼容。
- string和bytes是兼容的——只要bytes是有效的UTF-8编码。
- 嵌套消息与bytes是兼容的——只要bytes包含该消息的一个编码过的版本。
- fixed32与sfixed32是兼容的，fixed64与sfixed64是兼容的。
- 对于string，bytes和消息字段，optional与repeated兼容，给定repeated字段的序列化数据作为输入，如果客户端期望此字段是optional，则如果它是原始类型字段，则客户端将采用最后一个输入值；如果是消息类型字段，则将合并所有输入元素。请注意，这对于数字类型（包括布尔值和枚举）通常并不安全。重复的数字类型字段可以以打包格式序列化，当期望使用可选字段时，将无法正确解析该格式。
- 只要您记得从未通过网络发送默认值，就可以更改默认值。 因此，如果程序收到未设置特定字段的消息，则该程序将看到该程序的协议版本中定义的默认值。 它不会看到在发送者的代码中定义的默认值。
- enum在有线格式方面与int32，uint32，int64和uint64兼容（请注意，如果值不合适，它们将被截断），但是请注意，客户端代码在反序列化消息时可能会以不同的方式对待它们。值得注意的是，对消息进行反序列化时，无法识别的枚举值将被丢弃，这会使字段的has..accessor返回false，并且其getter返回枚举定义中列出的第一个值；如果指定了默认值，则返回默认值。对于重复的枚举字段，所有无法识别的值将从列表中删除。 但是，整数字段将始终保留其值。 因此，在将整数升级为枚举时，您需要非常小心，因为它会在线路上接收超出范围的枚举值。
- 在当前的Java和C ++实现中，当去除了无法识别的枚举值时，它们会与其他未知字段一起存储。请注意，如果将此数据序列化然后由识别这些值的客户端重新解析，则可能导致奇怪的行为。对于可选字段，即使在反序列化原始消息后写入了新值，识别该值的客户端仍会读取旧值。在重复字段的情况下，旧值将出现在任何已识别和新添加的值之后，这意味着将不保留顺序。
- 将单个可选值更改为新的oneof（oneof字段后面介绍）的成员是安全且二进制兼容的。 如果您确定没有代码一次设置多个，则将多个可选字段移动到一个新的字段中可能是安全的。 将任何字段移动到现有的字段中都是不安全的。
- 在map <K，V>和相应的重复消息字段之间更改字段是二进制兼容的（有关消息布局和其他限制，请参见下面的Maps）。 但是，更改的安全性取决于应用程序：在反序列化和重新序列化消息时，使用重复字段定义的客户端将产生语义上相同的结果； 但是，使用map字段定义的客户端可以对条目进行重新排序，并删除具有重复键的条目。
## 扩展
- 通过扩展，可以将一个范围内的字段标识号声明为可被第三方扩展所用。然后，其他人就可以在他们自己的.proto文件中为该消息类型声明新的字段，而不必去编辑原始文件了。看个具体例子：
```cpp
message Foo {
  // ...
  extensions 100 to 199;
}
```
- 这表示Foo中的字段号[100，199]的范围是为扩展保留的。 现在，其他用户可以使用您指定范围内的字段编号在自己的.proto文件(该文件需要import message Foo所在的proto文件)中将新字段添加到Foo中，以导入.proto文件，例如：
```cpp
extend Foo {
  optional int32 bar = 126;
}
```
- 这会在Foo的原始定义中添加一个名为bar的字段，其字段号为126。
- 当用户的Foo消息被编码的时候，数据的传输格式与用户在Foo里定义新字段的效果是完全一样的。
然而，要在程序代码中访问扩展字段的方法与访问普通的字段稍有不同——生成的数据访问代码为扩展准备了特殊的访问函数来访问它。例如，下面是如何在C++中设置bar的值：
```cpp
Foo foo;
foo.SetExtension(bar, 15);
```
- 类似地，Foo类也定义了模板函数 HasExtension()，ClearExtension()，GetExtension()，MutableExtension()，以及 AddExtension()。这些函数的语义都与对应的普通字段的访问函数相符。要查看更多使用扩展的信息，请参考相应语言的代码生成指南。注：扩展可 以是任何字段类型，包括消息类型。
- 请注意，扩展名可以是任何字段类型，包括消息类型，但不能是oneofs或maps。
## 嵌套的扩展
- 可以在另一个类型的范围内声明扩展，如：
```cpp
message Baz {
  extend Foo {
    optional int32 bar = 126;
  }
  ...
}
```
- 在此例中，访问此扩展的C++代码如下:
```cpp
Foo foo;
foo.SetExtension(Baz::bar, 15);
```
- 换句话说，唯一的效果是在Baz的范围内定义了bar。这是造成混淆的常见原因：声明嵌套在消息类型内的扩展块并不意味着外部类型与扩展类型之间没有任何关系。 特别地，以上示例并不意味着Baz是Foo的任何子类。 这意味着bar符号已在Baz范围内声明； 它只是一个静态成员。
- 一个通常的设计模式就是：在扩展的字段类型的范围内定义该扩展——例如，下面是一个Foo的扩展（该扩展是Baz类型的），其中，扩展被定义为了Baz的一部分：
```cpp
message Baz {
  extend Foo {
    optional Baz foo_ext = 127;
  }
  ...
}
```
- 然而，并没有强制要求一个消息类型的扩展一定要定义在那个消息中。也可以这样做：
```cpp
message Baz {
  ...
}
// This can even be in a different file.
extend Foo {
  optional Baz foo_baz_ext = 127;
}
```
- 事实上，这种语法格式更能防止引起混淆。正如上面所提到的，嵌套的语法通常被错误地认为有子类化的关系——尤其是对那些还不熟悉扩展的用户来说。
## 选择可扩展的标量符号
- 在同一个消息类型中一定要确保两个用户不会扩展新增相同的标识号，否则可能会导致数据的不一致。可以通过为新项目定义一个可扩展标识号规则来防止该情况的发生。
- 如果标识号需要很大的数量时，可以将该可扩展标符号的范围扩大至max，其中max是229 - 1, 或536,870,911。如下所示：
```cpp
message Foo {
  extensions 1000 to max;
}
```
- max 是 2^29 - 1, 或者 536,870,911.
- 与通常选择字段编号时一样，您的编号约定也需要避免使用字段编号19000到19999（FieldDescriptor :: kFirstReservedNumber到FieldDescriptor :: kLastReservedNumber），因为它们是为protobuf实现保留的。 您可以定义包括该范围的扩展名范围，但是协议编译器将不允许您使用这些数字定义实际的扩展名。
## Oneof
- 如果你的消息中有很多可选字段， 并且同时至多一个字段会被设置， 你可以加强这个行为，使用oneof特性节省内存.
- Oneof字段就像可选字段， 除了它们会共享内存， 至多一个字段会被设置。 设置其中一个字段会清除其它oneof字段。 你可以使用case()或者WhichOneof() 方法检查哪个oneof字段被设置， 看你使用什么语言了.
### 使用Oneof
- 为了在.proto定义Oneof字段， 你需要在名字前面加上oneof关键字, 比如下面例子的test_oneof:
```cpp
message SampleMessage {
  oneof test_oneof {
     string name = 4;
     SubMessage sub_message = 9;
  }
}
```
- 然后，将您的oneof字段添加到oneof定义。 您可以添加任何类型的字段，但不能使用required, optional, repeated的关键字。 如果需要将repeated字段添加到oneof字段中，则可以使用包含重复字段的消息。
- 在产生的代码中, oneof字段拥有同样的 getters 和setters， 就像正常的可选字段一样. 也有一个特殊的方法来检查到底那个字段被设置. 你可以在相应的语言API中找到oneof API介绍.
- https://developers.google.com/protocol-buffers/docs/reference/overview
### Oneof 特性
- 设置oneof会自动清除其它oneof字段的值. 所以设置多次后，只有最后一次设置的字段有值.
```cpp
SampleMessage message;
message.set_name(“name”);
CHECK(message.has_name());
message.mutable_sub_message();   // Will clear name field.
CHECK(!message.has_name());
```
- 如果解析器在传输过程中上遇到同一个oneof的多个成员，则在解析的消息中仅使用最后看到的成员。
- oneof不支持扩展.
- oneof不能 repeated.
- 反射API对oneof字段有效.
- 如果将oneof字段设置为默认值（例如将int32 oneof字段设置为0），则将设置该oneof字段的“大小写”，并且该值将在传输过程中序列化。
- 如果使用C++,需确保代码不会导致内存泄漏. 下面的代码会崩溃， 因为sub_message 已经通过set_name()删除了.
```cpp
SampleMessage message;
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");      // Will delete sub_message
sub_message->set_...            // Crashes here
```
- 同样，在C ++中，如果您用一个awap（）交换两条message（两消息中均有oneof），则每条message都将以另一种形式的oneof结尾：在下面的示例中，msg1将具有sub_message，而msg2将具有name
```cpp
SampleMessage msg1;
msg1.set_name(“name”);
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```
### 向后兼容性问题
- 添加或删除oneof字段时要小心。 如果检查 oneof 的值返回 None/NOT_SET，它意味着oneof字段没有被赋值或者在一个不同的版本中赋值了，无法区分这两种情况，因为无法知道传输过程中的未知字段是否是 oneof 的成员。
- 重用问题
  - 将optional字段移入或移出 oneof：在消息序列化和解析后，您可能会丢失某些信息（某些字段将被清除）。 但是，您可以安全地将单个字段移动到新的 oneof 中，并且如果知道oneof中只设置了一个字段，则可能能够移动多个字段到oneof中。
  - 删除 oneof 字段并将其添加回来：这可能会在消息被序列化和解析后清除您当前设置的 oneof 字段。
  - 拆分或合并oneof：这与移动常规optional字段有类似的问题。
## 映射
- 如果你想创建一个关联映射作为数据定义的一部分，protocol buffers 提供了一个方便的快捷语法：
```cpp
map<key_type, value_type> map_field = N;
```
- 其中 key_type 可以是任何整数或字符串类型（除了浮点类型和字节之外的任何标量类型）。 请注意，枚举不是有效的 key_type。  value_type可以是除另一个映射以外的任何类型。
- 因此，例如，如果您想创建一个项目映射，其中每个项目消息都与一个字符串键相关联，您可以像这样定义它：
```cpp
map<string, Project> projects = 3;
```
- 生成的映射 API 目前可用于所有 proto2 支持的语言。 您可以在相关 API 参考中找到有关所选语言的映射 API 的更多信息。
- https://developers.google.com/protocol-buffers/docs/reference/overview
### 映射特性
- 映射不支持扩展。
- 映射不能是 repeated, optional, 或 required.
- 映射值的传输格式排序和映射迭代排序未定义，因此您不能依赖于特定顺序的映射项。
- 为.proto生成文本格式时，映射按键排序。 数字键按数字排序。
- 当传输解析或合并时，如果有重复的映射键，则使用看到的最后一个键。从文本格式解析映射时，如果有重复的键，解析可能会失败。
### 向后兼容性问题
- map 语法等效于以下内容，因此不支持 map 的protobuf的实现仍然可以处理您的数据：
```cpp
message MapFieldEntry {
  optional key_type key = 1;
  optional value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```
- 任何支持映射的protobuf实现都必须生成和接受上述定义可以接受的数据。
## 包
- 您可以向 .proto 文件添加可选的包说明符，以防止协议消息类型之间的名称冲突。
```cpp
package foo.bar;
message Open { ... }
```
- 在其他的消息格式定义中可以使用包名+消息名的方式来定义域的类型，如：
```cpp
message Foo {
  ...
  required foo.bar.Open open = 1;
  ...
}
```
- 包的声明符会根据使用语言的不同影响生成的代码。
  - 在C++中，生成的类被包装在C++命名空间中。例如，Open将位于名称空间foo::bar中。
  - 在Java中，除非您在.proto文件中明确提供选项java_package，否则该包将用作Java包。
  - 在Python中，package指令被忽略，因为Python模块是根据它们在文件系统中的位置组织的。
  - 在Go中，package指令被忽略，生成的 .pb.go 文件在以对应的 go_proto_library 规则命名的包中。
- 请注意，即使 package 指令不直接影响生成的代码，例如在 Python 中，仍然强烈建议为 .proto 文件指定包，否则可能会导致描述符中的命名冲突并使 proto 不可移植到其他语言。
### 包和名称解析
- protobuf语言中的类型名称解析的工作方式与C++类似：首先搜索最内部的范围，然后搜索下一个最内部的范围，依此类推，每个包都被认为是其父包的“内部”。 一个领先的'.'  （例如，.foo.bar.Baz）表示从最外层作用域开始。
- 协议缓冲区编译器通过解析导入的 .proto 文件来解析所有类型名称。 每种语言的代码生成器都知道如何引用该语言中的每种类型，即使它具有不同的范围规则。
## 定义服务
- 如果想要将消息类型用在RPC(远程方法调用)系统中，可以在.proto文件中定义一个RPC服务接口，protocol buffer编译器将会根据所选择的不同语言生成服务接口代码及存根。如，想要定义一个RPC服务并具有一个方法，该方法能够接收 SearchRequest并返回一个SearchResponse，此时可以在.proto文件中进行如下定义：
```cpp
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```
- protocol编译器将产生一个抽象接口SearchService以及一个相应的存根实现。存根将所有的调用指向RpcChannel，它是一 个抽象接口，必须在RPC系统中对该接口进行实现。如，可以实现RpcChannel以完成序列化消息并通过HTTP方式来发送到一个服务器。换句话说， 产生的存根提供了一个类型安全的接口用来完成基于protocolbuffer的RPC调用，而不是将你限定在一个特定的RPC的实现中。C++中的代码 如下所示：
```cpp
using google::protobuf;

protobuf::RpcChannel* channel;
protobuf::RpcController* controller;
SearchService* service;
SearchRequest request;
SearchResponse response;

void DoSearch() {
  // You provide classes MyRpcChannel and MyRpcController, which implement
  // the abstract interfaces protobuf::RpcChannel and protobuf::RpcController.
  channel = new MyRpcChannel("somehost.example.com:1234");
  controller = new MyRpcController;

  // The protocol compiler generates the SearchService class based on the
  // definition given above.
  service = new SearchService::Stub(channel);

  // Set up the request.
  request.set_query("protocol buffers");

  // Execute the RPC.
  service->Search(controller, request, response, protobuf::NewCallback(&Done));
}

void Done() {
  delete service;
  delete channel;
  delete controller;
}
```
- 所有服务类还实现了 Service 接口，该接口提供了一种在编译时无需知道方法名称或其输入和输出类型即可调用特定方法的方法。 在服务器端，这可用于实现您可以注册服务的 RPC 服务器。
```cpp
using google::protobuf;

class ExampleSearchService : public SearchService {
 public:
  void Search(protobuf::RpcController* controller,
              const SearchRequest* request,
              SearchResponse* response,
              protobuf::Closure* done) {
    if (request->query() == "google") {
      response->add_result()->set_url("http://www.google.com");
    } else if (request->query() == "protocol buffers") {
      response->add_result()->set_url("http://protobuf.googlecode.com");
    }
    done->Run();
  }
};

int main() {
  // You provide class MyRpcServer.  It does not have to implement any
  // particular interface; this is just an example.
  MyRpcServer server;

  protobuf::Service* service = new ExampleSearchService;
  server.ExportOnPort(1234, service);
  server.Run();

  delete service;
  return 0;
}
```
- 如果您不想插入自己现有的 RPC 系统，现在可以使用 gRPC：一种由 Google 开发的与语言和平台无关的开源 RPC 系统。  gRPC 与协议缓冲区配合得特别好，并允许您使用特殊的协议缓冲区编译器插件直接从 .proto 文件生成相关的 RPC 代码。
- 但是，由于使用 proto2 和 proto3 生成的客户端和服务器之间存在潜在的兼容性问题，我们建议您使用 proto3 来定义 gRPC 服务。 您可以在 Proto3 语言指南中找到有关 proto3 语法的更多信息。 如果您确实想将 proto2 与 gRPC 一起使用，则需要使用 3.0.0 或更高版本的协议缓冲区编译器和库。
- proto3语法：https://developers.google.com/protocol-buffers/docs/proto3
- 除了 gRPC，还有一些正在进行的第三方项目为 Protocol Buffers 开发 RPC 实现。 有关我们了解的项目的链接列表，请参阅第三方附加组件 wiki 页面。https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md
## 选项（Options）
- 在定义.proto文件时能够标注一系列的options。Options并不改变整个文件声明的含义，但却能够影响特定环境下处理方式。完整的可用选项可以在google/protobuf/descriptor.proto找到。
- 有些选项是文件级选项，这意味着它们应该写在顶级范围内，而不是在任何消息、枚举或服务定义中。 一些选项是消息级别的选项，这意味着它们应该写在消息定义中。 有些选项是字段级选项，这意味着它们应该写在字段定义中。 选项也可以写在枚举类型、枚举值、oneof字段、服务类型和服务方法上； 然而，目前没有任何有用的选择。
- 以下是一些最常用的选项：
  - java_package（文件级选项）：要用于指定生成的Java类的包。 如果.proto文件中未提供显式的java_package选项，则默认情况下将使用proto软件包（在.proto文件中使用“ package”关键字指定）。 然而，proto 包通常不会成为好的 Java 包，因为 proto 包不会以反向域名开头。如果在.proto文件中没有明确的声明java_package，就采用默认的包名。当然了，默认方式产生的 java包名并不是最好的方式，按照应用名称倒序方式进行排序的。 如果不生成 Java 代码，则此选项无效。
```bash
option java_package = "com.example.foo";
 ```
   - java_outer_classname (文件级选项): 该选项表明想要生成Java类的名称。如果在.proto文件中没有明确的java_outer_classname定义，生成的class名称将会根据.proto文件的名称采用驼峰式的命名方式进行生成。如（foo_bar.proto生成的java类名为FooBar.java）如果 java_multiple_files 选项被禁用，则所有其他类/枚举/等。 为 .proto 文件生成的文件将在此外部包装器 Java 类中生成为嵌套类/枚举/等。 如果不生成 Java 代码，则此选项无效。
```bash
option java_outer_classname = "Ponycopter";
```
  - java_multiple_files（文件选项）：如果为 false，则只会为这个 .proto 文件生成一个 .java 文件，以及所有的 Java 类/枚举/等。 为顶级消息、服务和枚举生成的信息将嵌套在外部类中（请参阅 java_outer_classname）。 如果为 true，将为每个 Java 类/枚举/等生成单独的 .java 文件。 为顶级消息、服务和枚举生成，并且为此 .proto 文件生成的包装器 Java 类将不包含任何嵌套类/枚举/等。 这是一个布尔选项，默认为 false。 如果不生成 Java 代码，则此选项无效。
```bash
option java_multiple_files = true;
```
  - optimize_for (fileoption): 可以被设置为 SPEED, CODE_SIZE,or LITE_RUNTIME。这些值将通过如下的方式影响C++及java代码的生成：
    - SPEED (default): protocol buffer编译器将通过在消息类型上执行序列化、语法分析及其他通用的操作。这种代码是最优的。
    - CODE_SIZE: protocol buffer编译器将会产生最少量的类，通过共享或基于反射的代码来实现序列化、语法分析及各种其它操作。采用该方式产生的代码将比SPEED要少得多， 但是操作要相对慢些。当然实现的类及其对外的API与SPEED模式都是一样的。这种方式经常用在一些包含大量的.proto文件而且并不盲目追求速度的应用中。
    - LITE_RUNTIME：协议缓冲区编译器将生成仅依赖于“lite”运行时库（libprotobuf-lite 而不是 libprotobuf）的类。  lite 运行时比完整库小得多（大约小一个数量级），但省略了某些功能，如描述符和反射。 这对于在手机等受限平台上运行的应用程序特别有用。 编译器仍然会像在 SPEED 模式下一样生成所有方法的快速实现。 生成的类将仅以每种语言实现MessageLite接口，该接口仅提供完整Message接口的方法的子集。
```bash
option optimize_for = CODE_SIZE;
```
  - cc_generic_services、java_generic_services、py_generic_services（文件选项）：protocol buffer 编译器是否应该分别根据 C++、Java 和 Python 中的服务定义(即前文所述的服务)生成抽象服务代码。出于遗留原因，这些默认为 true。 但是，从2.3.0版（2010年1月）开始，它被认为通过提供代码生成器插件来对RPC实现更可取，而不是依赖于“抽象”服务。
  - 代码生成器插件:https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.compiler.plugin.pb
```bash
// This file relies on plugins to generate service code.
option cc_generic_services = false;
option java_generic_services = false;
option py_generic_services = false;
```
  - cc_enable_arenas（文件选项）：启用C++生成代码的arena分配。
    - c++代码arena分配：https://developers.google.com/protocol-buffers/docs/reference/arenas
  - message_set_wire_format（消息选项）：如果设置为 true，则消息使用不同的二进制格式，旨在与 Google 内部使用的名为 MessageSet 的旧格式兼容。Google外部的用户可能永远不需要使用此选项。 消息必须严格按照如下方式声明：
```cpp
message Foo {
  option message_set_wire_format = true;
  extensions 4 to max;
}
```
  - packed (field option): 如果在基本数字类型的重复字段上设置为 true，则使用更紧凑的编码。 使用此选项没有任何缺点。 但是，请注意，在 2.3.0 版本之前，收到非预期打包数据的解析器将忽略它。因此，它不可能在不破坏现有框架的兼容性上而改变压缩格式。在2.3.0及更高版本中，此更改是安全的，因为packed字段的解析器将始终接受这两种格式，但如果您必须处理使用旧 protobuf 版本的旧程序，请小心。
```bash
repeated int32 samples = 4 [packed=true];
```
  - deprecated（字段选项）：如果设置为 true，则表示该字段已弃用，不应由新代码使用。在大多数语言中，这没有实际效果。在 Java 中，这变成了 @Deprecated 注释。 将来，其他特定于语言的代码生成器可能会在字段的访问器上生成弃用注释，这反过来会导致在编译尝试使用该字段的代码时发出警告。如果该字段未被任何人使用并且您希望阻止新用户使用它，请考虑使用reserved语句(即上文中提到的)替换该字段声明。
```bash
optional int32 old_field = 6 [deprecated=true];
```
## 自定义选项
- Protocol Buffers甚至允许您定义和使用您自己的选项。 请注意，这是大多数人不需要的高级功能。由于选项是由google/protobuf/descriptor.proto（如 FileOptions 或 FieldOptions）中定义的消息定义的，因此定义您自己的选项只是扩展这些消息的问题。 例如：
```cpp
import "google/protobuf/descriptor.proto";

extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}

message MyMessage {
  option (my_option) = "Hello world!";
}
```
- 在这里，我们通过扩展 MessageOptions 定义了一个新的消息级选项。 当我们然后使用选项时，选项名称必须括在括号中以表明它是一个扩展。 我们现在可以像这样在 C++ 中读取 my_option 的值：
```cpp
string value = MyMessage::descriptor()->options().GetExtension(my_option);
```
- 这里，MyMessage::descriptor()->options() 返回 MyMessage 的 MessageOptions 协议消息。 从中读取自定义选项就像读取任何其他扩展一样。
- 同样，在 Java 中，我们会这样写：
```java
String value = MyProtoFile.MyMessage.getDescriptor().getOptions()
  .getExtension(MyProtoFile.myOption);
```
- 在 Python 中，它将是：
```python
value = my_proto_file_pb2.MyMessage.DESCRIPTOR.GetOptions()
  .Extensions[my_proto_file_pb2.my_option]
```
- 可以为 Protocol Buffers 语言中的每种构造定义自定义选项。 这是一个使用各种选项的示例：
```bash
import "google/protobuf/descriptor.proto";

extend google.protobuf.FileOptions {
  optional string my_file_option = 50000;
}
extend google.protobuf.MessageOptions {
  optional int32 my_message_option = 50001;
}
extend google.protobuf.FieldOptions {
  optional float my_field_option = 50002;
}
extend google.protobuf.OneofOptions {
  optional int64 my_oneof_option = 50003;
}
extend google.protobuf.EnumOptions {
  optional bool my_enum_option = 50004;
}
extend google.protobuf.EnumValueOptions {
  optional uint32 my_enum_value_option = 50005;
}
extend google.protobuf.ServiceOptions {
  optional MyEnum my_service_option = 50006;
}
extend google.protobuf.MethodOptions {
  optional MyMessage my_method_option = 50007;
}

option (my_file_option) = "Hello world!";

message MyMessage {
  option (my_message_option) = 1234;

  optional int32 foo = 1 [(my_field_option) = 4.5];
  optional string bar = 2;
  oneof qux {
    option (my_oneof_option) = 42;

    string quux = 3;
  }
}

enum MyEnum {
  option (my_enum_option) = true;

  FOO = 1 [(my_enum_value_option) = 321];
  BAR = 2;
}

message RequestType {}
message ResponseType {}

service MyService {
  option (my_service_option) = FOO;

  rpc MyMethod(RequestType) returns(ResponseType) {
    // Note:  my_method_option has type MyMessage.  We can set each field
    //   within it using a separate "option" line.
    option (my_method_option).foo = 567;
    option (my_method_option).bar = "Some string";
  }
}
```
- 请注意，如果您想在包中使用自定义选项（该包不是定义该自定义选项的包），则必须在选项名称前加上包名称，就像您对类型名称所做的那样。 例如：
```bash
// foo.proto
import "google/protobuf/descriptor.proto";
package foo;
extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}
```
```bash
// bar.proto
import "foo.proto";
package bar;
message MyMessage {
  option (foo.my_option) = "Hello world!";
}
```
- 最后一件事：由于自定义选项是扩展，因此必须像任何其他字段或扩展一样为它们分配字段编号。 在上面的示例中，我们使用了 50000-99999 范围内的字段编号。 此范围保留供个别组织内部使用，因此您可以在内部应用程序中自由使用此范围内的数字。 但是，如果您打算在公共应用程序中使用自定义选项，那么确保您的字段编号是全局唯一的就很重要.可以通过protobuf-global-extension-registry@google.com来获取全局唯一标识号。 只需提供你的项目名和项目网站. 通常你只需要一个扩展号。 你可以使用一个扩展号声明多个选项:
```bash
message FooOptions {
  optional int32 opt1 = 1;
  optional string opt2 = 2;
}

extend google.protobuf.FieldOptions {
  optional FooOptions foo_options = 1234;
}

// usage:
message Bar {
  optional int32 a = 1 [(foo_options).opt1 = 123, (foo_options).opt2 = "baz"];
  // alternative aggregate syntax (uses TextFormat):
  optional int32 b = 2 [(foo_options) = { opt1: 123 opt2: "baz" }];
}
```
- 另请注意，每个选项类型（文件级、消息级、字段级等）都有自己的编号空间，例如 您可以使用相同的数字声明 FieldOptions 和 MessageOptions 的扩展。
## 生成对应类
- 要生成Java，Python或C ++代码，您需要使用.proto文件中定义的消息类型，需要在.proto上运行protobuf编译器协议。 如果您尚未安装编译器，请下载该软件包并按照自述文件中的说明进行操作。
  - 软件包下载：https://developers.google.com/protocol-buffers/docs/downloads
- 协议编译器的调用如下：
```bash
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR path/to/file.proto
```
- IMPORT_PATH 指定在解析导入指令时查找 .proto 文件的目录。 如果省略，则使用当前目录。 可以通过多次传递 --proto_path 选项来指定多个导入目录； 将按顺序搜索它们。  -IIMPORT_PATH 可以用作 --proto_path 的缩写形式。
- 当然也可以提供一个或多个输出路径：
  - --cpp_out 在目标目录DST_DIR中产生C++代码，可以在 http://code.google.com/intl/zh-CN/apis/protocolbuffers/docs/reference /cpp-generated.html中查看更多
  - --java_out 在目标目录DST_DIR中产生Java代码，可以在 http://code.google.com/intl/zh-CN/apis/protocolbuffers/docs/reference /java-generated.html中查看更多。
  - --python_out 在目标目录 DST_DIR 中产生Python代码，可以在http://code.google.com/intl/zh-CN/apis/protocolbuffers /docs/reference/python-generated.html中查看更多。
  - 作为一种额外的约定，如果DST_DIR 是以.zip或.jar结尾的，编译器将输出结果打包成一个zip格式的归档文件。.jar将会输出一个 Java JAR声明必须的manifest文件。注：如果该输出归档文件已经存在，它将会被重写，编译器并没有做到足够的智能来为已经存在的归档文件添加新的文件。
  - 你必须提供一个或多个.proto文件作为输入。多个.proto文件能够一次全部声明。虽然这些文件是相对于当前目录来命名的，每个文件必须在一个IMPORT_PATH中，只有如此编译器才可以决定它的标准名称。



# 针对c++的教程
https://developers.google.com/protocol-buffers/docs/cpptutorial

# proto2语法
https://developers.google.com/protocol-buffers/docs/proto

# proto3语法
https://developers.google.com/protocol-buffers/docs/proto3