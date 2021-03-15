# 第十二章  C/C++ 接口

在Verilog中，您可以使用编程语言接口与C例程通信。随着三代的PLI: TF(任务/功能)，ACC(访问)，和VPI(验证程序接口)，您可以创建延迟计算器，连接和同步多个模拟器，并添加调试工具，如波形显示。然而，PLI最大的优点也是它最大的缺点。如果您只是想使用PLI连接一个简单的C例程，您需要编写几十行代码，并理解许多不同的概念，如与多个模拟阶段同步、调用帧和实例指针。此外，为了保护Verilog数据结构不受破坏，PLI在Verilog和C域之间复制数据时给模拟增加了开销。

SystemVerilog引入了直接编程接口(DPI)，这是一种与C、c++或任何其他外语进行接口的更简单的方法。使用import语句声明或“导入”C例程之后，就可以像调用任何SystemVerilog例程一样调用它。此外，您的C代码可以调用SystemVerilog例程。通过DPI，您可以连接读取刺激、包含参考模型或仅用新功能扩展SystemVerilog的C代码。目前SystemVerilog只支持到C语言的接口。c++代码必须包装成C语言的样子。

如果您有一个不消耗时间的SystemC模型，并且希望连接到SystemVerilog，那么您可以使用DPI。使用耗时方法的SystemC模型最好与内置在您最喜欢的模拟器中的实用程序连接。

上半年,本章以数据为中心,展示了如何通过不同的数据类型SystemVerilog和C .下半年之间控制中心,展示——荷兰国际集团(ing)如何通过控制之间来回SystemVerilog和C,而实际的C代码很简单,阶乘函数,斐波纳契数列,计数器,他们很容易理解,这样你就可以很快替代自己的代码。

## 12.1. 通过简单的值

本章的前几个例子向您展示了如何在SystemVerilog和C之间传递整型值，以及如何声明例程及其参数的机制。后面的部分将展示如何传递数组和结构。

### 12.1.1 传递整数和实数值

SystemVerilog和C之间可以传递的最基本数据类型是int，即2状态32位类型。示例12.1展示了SystemVerilog代码，该代码调用了一个C阶乘例程，如示例12.2所示。

**示例12.1 SystemVerilog代码调用C阶乘例程**

<img src="..\media\ch12_image1.png" style="width:3.79603in;height:1.17333in" />

import语句声明SystemVerilog例程的阶乘是用外国语言(如c)实现的。修饰符“DPI-C”指定这是一个直接编程接口例程，语句的其余部分描述例程参数。

示例12.1使用直接映射到C int类型的SystemVerilog int数据类型传递32位带符号的值。SystemVerilog int总是32位，而C中int的宽度取决于操作系统。示例12.2中的C函数接受一个整数作为输入，因此DPI按值传递参数。

**示例12.2 C factorial函数**

<img src="..\media\ch12_image2.png" style="width:2.6638in;height:0.58333in" />

### 12.1.2 进口申报

import声明定义了C任务或函数的原型，但使用的是SystemVerilog类型。带返回值的C函数被映射到SystemVerilog

函数。void C函数可以映射到SystemVerilog任务或void函数。如果导入的C函数的名称与SystemVerilog名称冲突，则可以使用新名称导入该函数。在示例12.3中，C函数expect被映射到SystemVerilog名称fexpect，因为名称expect是SystemVerilog中的保留关键字。名称expect成为一个全局符号，用于与C代码链接，而fexpect是一个本地SystemVerilog符号。在本例的后半部分中，C函数stat在SystemVerilog中被赋予一个新名称file\_exists。SystemVerilog不支持重载例程，例如，一次使用真实参数导入expect，另一次使用int。

**样例12.3修改导入函数的名称**

<img src="..\media\ch12_image3.png" style="width:4.50917in;height:2.56in" />

可以在SystemVerilog代码中的任何地方导入例程，在这些地方可以声明例程，包括程序内部、模块、接口、包和$unit(编译单元空间)。导入的例程将是声明它的声明空间的本地例程。如果您需要在代码中的几个位置调用导入例程，请将import语句放在需要导入的包中。对import语句的任何更改都本地化到包中。

### 12.1.3 参数的方向

导入的C例程可以有零个或多个参数。默认情况下，参数方向是输入(数据从SystemVerilog到C)，但也可以是输出和inout。不支持方向ref。函数可以返回一个简单的值，比如一个整数或实数，也可以不返回任何值。样例12.4展示了如何指定参数方向。

**示例12.4参数指示**

<img src="..\media\ch12_image4.png" style="width:3.88354in;height:0.42in" />

您可以通过将任何输入参数声明为const(如示例12.5中所示)来减少在C代码中出现错误的机会，这样C编译器就会为任何对输入的写操作给出一个错误。

**带有const参数的阶乘例程**

<img src="..\media\ch12_image5.png" style="width:2.67167in;height:0.585in" />

### 12.1.4 参数类型

通过DPI传递的每个变量都有两个匹配的定义:一个用于SystemVerilog端，一个用于C端。使用兼容类型是您的责任。SystemVerilog模拟器无法比较类型，因为它无法读取C代码。(VCS编译器会生成vc\_hdrs.h，而Questa会为你导入的任何例程创建包含C头的include .h。你可以使用这个文件作为匹配类型的指南。)

表12.1显示了SystemVerilog与C例程的输入和输出之间的数据类型映射。C结构在包含文件svdpi.h中定义。数组映射将在第12.4节和第12.5节讨论，结构将在第12.6节讨论。

**表12.1 SystemVerilog和C的数据类型映射**

<table><thead><tr class="header"><th><em>SystemVerilog</em></th><th><blockquote><p><em>C(输入)</em></p></blockquote></th><th><blockquote><p><em>C(输出)</em></p></blockquote></th></tr></thead><tbody><tr class="odd"><td>字节</td><td><blockquote><p>字符</p></blockquote></td><td><blockquote><p>char *</p></blockquote></td></tr><tr class="even"><td>shortint</td><td><blockquote><p>短整型</p></blockquote></td><td><blockquote><p>短int *</p></blockquote></td></tr><tr class="odd"><td>int</td><td><blockquote><p>int</p></blockquote></td><td><blockquote><p>int *</p></blockquote></td></tr><tr class="even"><td>longint</td><td><blockquote><p>长长的int</p></blockquote></td><td><blockquote><p>长int *</p></blockquote></td></tr><tr class="odd"><td>shortreal</td><td><blockquote><p>浮动</p></blockquote></td><td><blockquote><p>浮*</p></blockquote></td></tr><tr class="even"><td>真正的</td><td><blockquote><p>双</p></blockquote></td><td><blockquote><p>双*</p></blockquote></td></tr><tr class="odd"><td>字符串</td><td><blockquote><p>const char *</p></blockquote></td><td><blockquote><p>char * *</p></blockquote></td></tr><tr class="even"><td>string [N]</td><td><blockquote><p>const char * *</p></blockquote></td><td><blockquote><p>char * *</p></blockquote></td></tr><tr class="odd"><td>位</td><td><blockquote><p>svBit或unsigned char</p></blockquote></td><td><blockquote><p>svBit*或unsigned char*</p></blockquote></td></tr><tr class="even"><td>逻辑,注册</td><td><blockquote><p>svLogic或unsigned char</p></blockquote></td><td><blockquote><p>svLogic*或unsigned char*</p></blockquote></td></tr><tr class="odd"><td>位(N: 0)</td><td><blockquote><p>const svBitVecVal *</p></blockquote></td><td><blockquote><p>svBitVecVal *</p></blockquote></td></tr><tr class="even"><td>注册(N: 0)逻辑(N: 0)</td><td><blockquote><p>const svLogicVecVal *</p></blockquote></td><td><blockquote><p>svLogicVecVal *</p></blockquote></td></tr><tr class="odd"><td>无浆[]数组</td><td><blockquote><p>const svOpenArrayHandle</p></blockquote></td><td><blockquote><p>svOpenArrayHandle</p></blockquote></td></tr><tr class="even"><td>chandle</td><td><blockquote><p>const void *</p></blockquote></td><td><blockquote><p>void *</p></blockquote></td></tr></tbody></table>

<img src="..\media\ch12_image6.png" style="width:0.89333in;height:0.53167in" />注意，有些映射是不精确的。例如，SystemVerilog中的一个位映射到C中的svBit，它最终映射到svdpi.h包含文件中的unsigned char。因此，您可能会将非法的值写入上位。

LRM限制导入函数的结果为“小值”，包括:void、byte、shortint、int、longint、real、shortreal、chandle和string，以及bit和logic类型的单个位值。函数不能返回像bit\[6:0\]这样的向量，因为这需要返回指向svBitVecVal结构体的指针。

### 12.1.5 导入数学库例程

样例12.6展示了如何在不使用C包装器的情况下直接调用C math库中的许多函数，从而减少了需要编写的代码量。Verilog真实类型映射到C double类型。

**样例12.6导入C数学函数**

<img src="..\media\ch12_image7.png" style="width:3.575in;height:0.41in" />

## 12.2. 连接到一个简单的C例程

您的C代码可能包含一个模拟模型，比如一个处理器，它与Verilog模型一起实例化。或者，您的代码可以是一个参考模型，在事务或周期级别与Verilog模型进行比较。本章的许多考试展示了用C或c++编写的7位计数器。尽管计数器非常简单，但它具有与复杂模型相同的部分，包括输入、输出、调用之间的内部值存储，以及需要支持多个实例。计数器为7位，用于显示硬件类型与C类型不匹配时发生的情况。

### 12.2.1 具有静态存储器的计数器

示例12.7是一个7位计数器的C代码。计数存储在一个静态变量中，就像您在考虑模拟之前编写模型时所做的那样。

**示例12.7计数器例程使用静态变量**

<img src="..\media\ch12_image8.png" style="width:4.26408in;height:2.24833in" />

reset和load信号是2状态的单比特信号，所以它们作为svBit被传递，减少为unsigned char。您的代码可以用两种方式声明值，但要确保安全，请使用SystemVerilog DPI类型。输入i是2状态的，7位宽，作为svBitVecVal传递。注意，它是作为const指针传递的，这意味着底层的值可以更改，但不能更改指针的值，例如使其指向另一个值。同样，reset和load输入也被标记为const。在本例中，7位计数器值存储在一个char中，因此必须屏蔽上面的位。

h文件包含SystemVerilog DPI结构和方法的定义。本章剩下的C代码示例没有使用\#include语句，除非它们对讨论很重要。

示例12.8显示了SystemVerilog程序，该程序为7位计数器导入并调用C函数。

**示例12.8带有静态存储的7位计数器的Testbench**

<img src="..\media\ch12_image9.png" style="width:4.58347in;height:3.94333in" />

### 12.2.2 Chandle数据类型

chandle数据类型允许您在SystemVerilog代码中存储C或c++指针。chandle变量的宽度足以在编译代码的机器上保存指针，即32位或64位。样品12.7中的计数器只要是设计中唯一的计数器就可以正常工作。您可以将示例12.8中的counter7调用包装在一个模块中，并在设计中实例化多个副本。然而，由于计数器的值存储在C静态中，因此每个实例共享单个值。如果需要调用C代码的模块的多个实例，C代码需要将其变量存储在静态变量之外的其他地方。更好的方法是分配存储，并向其传递一个句柄，以及输入和输出信号值。示例12.9显示了一个在结构c7中存储7位计数的计数器。对于一个简单的计数器来说，这有点小题大做了，但是如果您正在为一个更大的设备创建一个模型，您可以从这个示例进行构建。

**示例12.9使用实例存储的计数器例程**

<img src="..\media\ch12_image10.png" style="width:4.60667in;height:4.67in" />

例程counter7\_new构造了计数器实例。这将返回一个chandle，它必须被传递到对counter7的后续调用中。计数器的值存储在c7类型的结构体中。函数counter7\_new调用malloc分配该结构体，并将结果转换为一个本地指针c。

C代码使用PLI任务io\_printf来显示调试消息。当您同时调试C和SystemVerilog代码时，这个例程很有帮助，因为它写入相同的输出(包括日志文件)，就像$display一样，包括

模拟器的日志文件。例程在veriuser.h中定义。

示例12.10中此计数器的测试工作台与静态计数器在几个方面有所不同。首先，计数器必须在使用之前构造好。接下来，计数器在时钟边缘被调用，而不是与刺激同步。为简单起见，在时钟走高时调用计数器，在时钟走低时应用刺激，以避免任何竞争条件。

**示例12.10 Testbench，用于每个实例存储的7位计数器**

<img src="..\media\ch12_image11.png" style="width:4.11648in;height:5.29833in" />

### 12.2.3 包装值表示

字符串"DPI-C"1指定您正在使用打包值的规范表示。这种表示将SystemVerilog变量存储为一个或多个元素的C数组。2状态变量使用svBitVecVal类型存储。2状态数组与该类型的多个元素一起存储。

早期版本的LRM使用“DPI”，但现在已经过时，不应该使用。

<table><thead><tr class="header"><th><blockquote><p>31:0</p></blockquote></th><th></th></tr></thead><tbody><tr class="odd"><td><blockquote><p>未使用的</p></blockquote></td><td><blockquote><p>39:32</p></blockquote></td></tr></tbody></table>

**图12.1 40位2状态变量的存储**

由于性能的原因。SystemVerilog模拟器在调用DPI例程后可能不会屏蔽较高的位，因此SystemVerilog变量可能会损坏。确保你的C代码正确处理这些值。

如果需要在位和字之间进行转换，请使用SV\_PACKED\_ DATA\_NELEMS宏。例如，要将40位转换为两个32位的单词(如图12.1所示)，请使用SV\_PACKED\_DATA\_NELEMS(40)。

### 12.2.4 4-State值

SystemVerilog中的每个4状态位存储在模拟器中，使用两个称为

aval和bval，见表12.2

**表12.2 4状态位编码**

<table><thead><tr class="header"><th><em>4-state价值</em></th><th><blockquote><p><em>bval</em></p></blockquote></th><th><blockquote><p><em>保兑</em></p></blockquote></th></tr></thead><tbody><tr class="odd"><td>0</td><td><blockquote><p>0</p></blockquote></td><td><blockquote><p>0</p></blockquote></td></tr><tr class="even"><td>1</td><td><blockquote><p>0</p></blockquote></td><td><blockquote><p>1</p></blockquote></td></tr><tr class="odd"><td>Z</td><td><blockquote><p>1</p></blockquote></td><td><blockquote><p>0</p></blockquote></td></tr><tr class="even"><td>X</td><td><blockquote><p>1</p></blockquote></td><td><blockquote><p>1</p></blockquote></td></tr></tbody></table>

单个位的4状态变量，如逻辑f，存储在无符号字节中，aval位存储在最低有效位中，bval位存储在更高的位中。因此，值1'b0在C中被视为0x0, 1'b1是0x1, 1'bz是0x2, 1'bx是0x3。像logic \[31:0\] lword这样的4状态向量使用32位对svLogicVecVal存储，如图12.2所示，其中包含aval和bval位。32位变量lword存储在单个svLogicVecVal中。大于32位的变量存储在多个svLogicVecVal元素中，第一个元素包含32个最低有效位，下一个元素包含下一个32位，直到最高有效位。一个40位的逻辑变量被存储为最低32位的svLogicVecVal，最高8位的svLogicVecVal(图12.2)。这个最大值中未使用的24位是未确定的，您需要负责屏蔽或扩展符号位。svLogicVecVal类型等价于s\_vpi\_vecval，它用于表示诸如此类的4种状态类型

作为VPI中的逻辑。

<table><thead><tr class="header"><th><blockquote><p>保兑31:0</p></blockquote></th><th></th></tr></thead><tbody><tr class="odd"><td><blockquote><p>bval 31:0</p></blockquote></td><td></td></tr><tr class="even"><td><blockquote><p>未使用的</p></blockquote></td><td>保兑39:32</td></tr><tr class="odd"><td><blockquote><p>未使用的</p></blockquote></td><td>bval 39:32</td></tr></tbody></table>

**图12.2 40位四态变量的存储**

<img src="..\media\ch12_image6.png" style="width:0.89333in;height:0.53167in" />注意那些没有使用位下标或使用单个位声明的参数。声明为输入逻辑a的参数存储在unsigned char中。参数输入逻辑\[0:0\]b是svLogicVecVal，即使它只包含一个比特。

样例12.11显示了一个4状态计数器的import语句。与示例12.10的唯一区别是位类型现在是逻辑类型。

**样例12.11用于检查Z或X值的计数器的Testbench**

<img src="..\media\ch12_image12.png" style="width:3.51835in;height:0.875in" />

前面示例12.9中显示的计数器假设所有输入都是两种状态。示例12.12扩展了这段代码，在reset、load和i时检查Z和X的值。实际的计数仍然保持为2状态值。

**示例12.12检查Z和X值的计数器例程**

<img src="..\media\ch12_image13.png" style="width:4.60668in;height:4.11667in" />

如果你想因为在导入的例程中发现一个条件而强制终止模拟，你可以调用VPI例程vpi\_ control(vpiFinish, 0).这个例程和常量在包含文件vpi\_user.h中定义。值vpiFinish告诉模拟器在导入的例程返回后执行$finish。

### 12.2.5 从2状态转换为4状态

如果您有一个使用2种状态类型的DPI应用程序，并且希望将其转换为使用4种状态类型，请遵循以下指导原则。

在SystemVerilog端，将导入声明从使用2状态类型(如bit和int)更改为4状态类型(如logic和integer)。确保在函数调用中使用了4个状态变量。

在C端，将参数声明从svBitVecVal切换到svLog- icVecVal。对参数的任何引用都必须使用.aval后缀to

正确访问数据。当您从一个4状态变量中读取数据时，检查bval位，看看是否有Z或X值。当写入4状态变量时，除非需要写入Z或X值，否则清除bval位。

## 12.3. 连接到c++

您可以使用DPI将用C或c++编写的例程连接到SystemVerilog。有几种使用DPI的c++代码通信的方式，这取决于您的模型的抽象级别。

### 12.3.1 c++中的计数器

示例12.13显示了一个7位计数器的c++类，具有2状态输入。它连接到示例12.10中的SystemVerilog testbench和示例12.14中的c++包装器代码。

**示例12.13计数器类**

<img src="..\media\ch12_image14.png" style="width:3.875in;height:3.79833in" />

### 12.3.2 静态方法

DPI只能调用在链接时已知的C或c++函数。因此，SystemVerilog代码不能在对象中调用c++例程，因为链接器运行时对象不存在。

那么，如果需要调用c++类中的方法，该怎么办呢?解决方案，如示例12.14所示，是创建一个具有固定地址的函数，然后可以与c++动态对象和方法通信。第一个例程counter7\_new为该计数器构造一个对象，并返回该对象的句柄。第二个静态例程counter7使用对象句柄调用执行计数器逻辑的c++方法。

**样例12.14静态方法和链接**

<img src="..\media\ch12_image15.png" style="width:3.9723in;height:2.26in" />

extern " C "代码告诉c++编译器，发送给连接器的外部信息应该使用C调用约定，而不是执行名称mangling。你可以把它放在SystemVerilog调用的每个例程之前，或者把extern "C"{…}围绕一组方法。

从测试工作台的角度来看，c++计数器看起来与在每个实例存储中存储值的计数器是一样的，如示例12.9所示，因此您可以使用相同的测试工作台(示例12.10)。

### 12.3.3 与事务级c++模型通信

前面的C / c++代码示例是在信号级别与SystemVerilog通信的低级模型。这是没有效率的;例如，即使数据或控制输入没有改变，计数器也会在每个时钟周期被调用。当您为处理器和网络设备等复杂设备创建模型时，请在事务级别与它们通信以获得更快的模拟。

示例12.15中的c++计数器模型有一个事务级接口，与方法通信，而不是信号和时钟。

**示例12.15 c++计数器与方法通信**

<img src="..\media\ch12_image16.png" style="width:4.07in;height:5.05833in" />

诸如reset、load和count之类的动态c++方法被包装在使用从SystemVerilog传递的对象句柄的静态方法中，如示例12.16所示。

**示例12.16 c++事务级计数器的静态包装器**

<img src="..\media\ch12_image17.png" style="width:4.0475in;height:4.69333in" />

事务级计数器的OOP接口被携带到测试台中。样例12.17包含SystemVerilog导入语句和包装c++对象的类。这允许您将c++句柄隐藏在类内部。

请注意，counter7\_get()函数返回的是int(32位，带符号)，而不是bit\[6:0\]，因为后者需要返回一个指向svBitVecVal的指针，如表12.1所示。导入的函数不能返回指针。它只能返回void、byte、shortint、int、longint、real、shortreal、chandle和string类型的值，以及bit和logic类型的单个位值。

**示例12.17 c++模型使用方法的Testbench**

<img src="..\media\ch12_image18.png" style="width:4.60667in;height:6.95667in" />

<img src="..\media\ch12_image19.png" style="width:4.6085in;height:2.095in" />

## 12.4. 简单的数组共享

到目前为止，您已经看到了在SystemVerilog和C之间传递标量和向量的示例。典型的C模型可能读取一个值数组，执行一些计算，并返回另一个带有结果的数组。

### 12.4.1 一维数组- 2状态

示例12.18展示了一个例程，它计算斐波那契数列中的前20个值。它由示例12.19中的SystemVerilog代码调用。

**例程12.18 C计算斐波那契数列**

<img src="..\media\ch12_image20.png" style="width:2.67124in;height:1.04167in" />

注意，在C语言中，您也可以将参数声明为指针，

\*数据或数组，数据\[20\]。在这个例子中，它们是可以互换的。

**示例12.19 Fibonacci例程的Testbench**

<img src="..\media\ch12_image21.png" style="width:4.5459in;height:1.47833in" />

注意，斐波那契值数组是在SystemVerilog中分配和存储的，即使它是在C中计算的。没有办法在C中分配数组并在SystemVerilog中引用它。

### 12.4.2 一维数组- 4种状态

示例12.20展示了示例12.21中带有testbench的4状态数组的斐波那契C例程。

**示例12.20 C例程用4状态数组计算斐波那契数列**

<img src="..\media\ch12_image22.png" style="width:3.9025in;height:1.67in" />

**示例12.21带有4状态数组的斐波那契例程的Testbench**

<img src="..\media\ch12_image23.png" style="width:4.60931in;height:1.45167in" />

第12.2.5节介绍如何将一个2状态应用程序转换为4状态应用程序。

## 12.5. 开放的数组

在SystemVerilog和C之间共享数组时，有两个选项。对于最快的模拟，您可以逆向工程System- Verilog中的元素布局，并编写您的C代码来使用这个映射。这种方法是脆弱的，这意味着如果任何数组大小发生变化，就必须重写和调试C代码。一种更健壮的方法是使用“开放数组”及其相关的SystemVerilog例程来操作它们。这允许您编写可以操作任意大小数组的通用C例程。

### 12.5.1 基本开放数组

示例12.22和12.23展示了如何在SystemVerilog和C之间传递一个带有打开数组的简单数组。在SystemVerilog import语句中使用空方括号\[\]来指定传递的是一个开放数组。

**示例12.22 Testbench代码使用一个开放数组调用一个C例程**

<img src="..\media\ch12_image24.png" style="width:4.60883in;height:1.77833in" />

你的C代码用svOpenArray- handle类型的句柄引用这个开放数组。这指向一个结构，其中包含关于数组的信息，例如声明的字范围。您可以通过调用svGetArrayPtr来定位实际的数组元素。请注意，svSize()是一个开放数组查询方法，将在下一节中描述。

**示例12.23 C代码，使用基本的开放数组**

<img src="..\media\ch12_image25.png" style="width:3.28667in;height:1.12031in" />

### 12.5.2 开放数组的方法

正如svdpi.h中定义的那样，有许多DPI方法可以访问它们的内容和范围。这些操作只适用于声明为svOpenArray- Handle的开放数组句柄，而不适用于svBitVecVal或svLogicVecVal之类的指针。表12.3中的方法提供了有关开放数组大小的信息。

**表12.3 Open array查询功能**

*函数 描述*

int svLeft (h, d) 维数d的左界

int svRight (h, d) 维度d的右界

int svLow (h, d) 维数d的下限

int svHigh (h, d) 维数d的高界

int svIncrement (h, d) If left &gt;= right, 1, else−1

int svSize (h, d) d维元素个数:svHigh−svLow+1

int svDimensions (h) 开放数组的维度数

int svSizeOfArray (h) 以字节为单位的数组总大小

在表12.3中，变量h是一个svOpenArrayHandle, d是一个int，维度从d=1开始编号。

表12.4中的函数返回整个数组或单个元素的C存储位置。

**表12.4打开阵列定位器功能**

*函数 返回指针:*

void \* svGetArrayPtr (h) 存储整个数组void 数组中的一个元素 1维数组中的一个元素 2维数组中的一个元素 三维阵列中的一个元素

### 12.5.3 传递未设置大小的开放数组

示例12.24使用一个二维数组调用C代码。C代码使用svLow和svHigh方法来查找数组范围，在本例中，它不遵循通常的0..size-1。

**示例12.24 Testbench使用多维开放数组调用C代码**

<img src="..\media\ch12_image26.png" style="width:4.22635in;height:1.64333in" />

这调用了示例12.25中的C代码，该代码使用open array方法读取数组。例程svLow(handle, dimension)返回指定维度的最低索引号。所以svLow(h,1)对于使用范围\[6:1\]声明的数组返回1。同样，svHigh(h, 1)返回6。你应该使用svLow和svHigh与C for循环。

方法svLeft和svRight返回数组声明的左下标和右下标，范围\[6:1\]分别为6和1。在示例12.25的中心，调用svGetArrElemPtr2返回一个指向二维数组中的元素的指针。

**示例12.25 C代码，带有多维开放数组**

<img src="..\media\ch12_image27.png" style="width:3.66977in;height:2.125in" />

### 12.5.4 在DPI中打包开放数组

DPI中的开放数组被视为具有单个打包维度和一个或多个未打包维度。您可以传递具有多个打包维度的数组，只要这些维度打包到与单个元素大小相同的元素中即可

正式的论证。例如，如果你在import语句中有正式的参数bit\[63:0\] b64\[\]，你可以传入实际的参数bit\[1:0\]\[0:3\]\[6:−1\]bpack\[9:1\]。样例12.26展示了带有打开数组的SystemVerilog代码。

**示例12.26用于打包开放数组的Testbench**

<img src="..\media\ch12_image28.png" style="width:4.60667in;height:2.32833in" />

**示例12.27 C代码使用了打包的开放数组**

<img src="..\media\ch12_image29.png" style="width:4.47108in;height:1.04167in" />

注意，示例12.27中的C代码使用%llx打印64位值，并将svGetArrayElemPtr1的结果转换为long long int。

## 12.6. 分享复合类型

此时，您可能想知道如何在SystemVerilog和c之间传递对象。类属性的布局在两种语言之间不匹配，因此不能直接共享对象。相反，您必须在每个节点上创建类似的结构

边，加上pack和unpack方法来在两种格式之间进行转换。所有这些就绪之后，就可以共享复合类型了。

### 12.6.1 在SystemVerilog和C之间传递结构

下面的示例共享一个简单的像素结构，该像素由三个字节打包成一个单词。示例12.28显示了C结构。注意，C将char作为有符号变量处理，这可能会给您带来意想不到的结果，因此该结构将char标记为unsigned。字节的顺序与SystemVerilog相反，因为这段代码是为Intel x86处理器编写的，它是小端的，这意味着最低有效字节存储在比最高有效字节更低的地址中。Sun SPARC是大端，因此字节的存储顺序与SystemVerilog相同:r、g、b。

**示例12.28 C代码共享一个结构**

<img src="..\media\ch12_image30.png" style="width:3.75422in;height:1.81333in" />

样例12.29中的SystemVerilog testbench有一个包含单个像素的打包结构，以及封装像素操作的类。RGB\_T结构被打包，因此SystemVerilog将把字节存储在连续的位置。如果没有包装修饰符，每个8位值将存储在单独的单词中。

**示例12.29用于共享结构的Testbench**

<img src="..\media\ch12_image31.png" style="width:4.60668in;height:5.495in" />

### 12.6.2 在SystemVerilog和C之间传递字符串

使用DPI，您可以将字符串从C传递回SystemVerilog。您可能需要传递一个字符串作为结构的符号值，或者获取表示C代码内部状态的字符串以进行调试。

将字符串从C传递给SystemVerilog的最简单方法是让C函数返回一个指向静态字符串的指针，如示例12.30所示。字符串必须是

在C中声明为静态，而不是本地字符串。非静态变量存储在堆栈中，并在函数返回时回收。

**示例12.30从C语言返回一个字符串**

<img src="..\media\ch12_image32.png" style="width:4.0913in;height:0.735in" />

静态存储的一个危险是多个并发调用可能最终共享存储。例如，打印多个像素的SystemVerilog $display语句可能多次调用上述打印例程。根据SystemVerilog编译器对这些调用的排序，以后对print()的调用可能会覆盖以前调用的结果，除非SystemVerilog编译器复制了该字符串。注意，SystemVerilog调度程序永远不会中断对导入例程的调用。样例12.31将字符串存储在堆中以支持并发调用。

**示例12.31在C语言中从堆返回一个字符串**

<img src="..\media\ch12_image33.png" style="width:4.08792in;height:2.70833in" />

## 12.7. 纯和上下文导入方法

导入的方法分为纯方法、上下文方法和泛型方法。纯函数严格根据输入计算输出，没有外部交互。具体来说，纯函数不访问任何全局变量或静态变量，不执行任何文件操作，也不与函数外部的任何东西交互，比如操作

系统，进程，共享内存，套接字，等等。如果结果不需要，SystemVerilog编译器可能会优化对纯函数的调用，或者用使用相同参数的前一次调用的结果替换调用。样本12.5中的阶乘函数和样本12.6中的sin函数都是纯函数，因为它们的结果只基于它们的输入。样例12.32展示了如何导入纯函数。

**示例12.32导入纯函数**

<img src="..\media\ch12_image34.png" style="width:4.16333in;height:0.255in" />

导入的例程可能需要知道调用它的上下文，以便它可以调用PLI TF、ACC或VPI方法，或已导出的SystemVerilog任务。使用这些方法的context属性，如示例12.33所示。

**示例12.33导入的上下文任务**

<img src="..\media\ch12_image35.png" style="width:3.96751in;height:0.11333in" />

导入的例程可能使用全局存储，所以它不是纯的，但可能没有任何PLI引用，所以它不需要上下文例程的开销。Sutherland(2004)对这些方法使用术语“通用”，因为SystemVerilog LRM没有特定的名称。默认情况下，导入的例程是通用的，就像本章中的许多例子一样。

由于模拟器需要记录调用上下文，因此调用上下文导入例程会有开销，所以只有在需要时才将例程声明为上下文。另一方面，如果一个通用导入例程调用导出的任务或访问SystemVerilog数据对象的PLI例程，那么模拟器可能会崩溃。

上下文感知的PLI例程需要知道它是从哪里被调用的，这样它才能访问与该位置相关的信息。

## 12.8. 从C通信到SystemVerilog

到目前为止的示例已经向您展示了如何从SystemVerilog模型调用C代码。DPI还允许您从C代码调用SystemVerilog例程。SystemVerilog例程可以是用C语言记录操作结果的简单任务，也可以是表示部分硬件模型的耗时任务。

### 12.8.1 一个简单的导出函数

样例12.34显示了一个导入上下文函数和导出System- Verilog函数的模块。

**示例12.34导出SystemVerilog功能**

<img src="..\media\ch12_image36.png" style="width:4.28748in;height:1.46167in" />

<img src="..\media\ch12_image37.png" style="width:0.89333in;height:0.53333in" />样例12.34中的export声明看起来很直白，因为LRM禁止放入返回值声明或任何参数。你甚至不能给出通常的空括号。出口申报单中的这一信息与出口申报单中的

在模块末尾的函数声明中的信息，因此，如果您更改了函数，可能会失去同步。

样例12.35显示了调用导出函数的C代码。

**示例12.35从C调用导出的SystemVerilog函数**

<img src="..\media\ch12_image38.png" style="width:2.53167in;height:0.87667in" />

本例打印C代码中的行，然后是SystemVerilog的$display输出，如示例12.36所示。

**示例12.36简单导出的输出**

<img src="..\media\ch12_image39.png" style="width:1.27167in;height:0.26667in" />

### 12.8.2 函数调用SystemVerilog函数

虽然您的大部分测试台架应该在SystemVerilog中，但您可能有C或其他语言的遗留测试台架，或者您想要重用的应用程序。本节创建一个SystemVerilog内存模型，该模型由从外部文件读取事务的C代码激发。

内存模型的第一个版本(如示例12.38和12.37所示)只使用函数进行编码，因此一切都以零时间运行。中的C代码

示例12.37打开文件，读取命令，并调用导出的函数。为了简洁，错误检查已经被移除。

**示例12.37 C代码读取简单的命令文件并调用导出函数**

<img src="..\media\ch12_image40.png" style="width:2.3925in;height:3.50333in" />

SystemVerilog代码调用打开文件的C任务read\_file。文件中唯一的命令设置内存大小，因此C代码调用一个导出函数。

**示例12.38 SystemVerilog模块，用于简单的内存模型**

<img src="..\media\ch12_image41.png" style="width:4.60667in;height:2.045in" />

注意，在示例12.38中，export语句没有任何参数，因为该信息已经在函数声明中了。

命令文件很简单，只需要一个命令就可以构造一个包含100个元素的内存，如示例12.39所示。

**示例12.39用于简单内存模型的命令文件**

<img src="..\media\ch12_image42.png" style="width:0.36884in" />

### 12.8.3 C任务调用SystemVerilog任务

真正的内存模型有一些操作，比如读和写，这些操作会消耗时间，因此必须用任务来建模。

样例12.40显示了内存模型的第二个版本的SystemVerilog代码。与示例12.38相比，它有几个改进。有两个新任务，mem\_read和mem\_write，它们分别需要20ns和10ns来完成。导入的例程read\_file现在是SystemVerilog任务，因为它正在调用其他消耗时间的任务。import语句现在指定read\_file是一个上下文任务，因为在调用它时，模拟器需要创建一个单独的堆栈。

**示例12.40 SystemVerilog模块的内存模型导出的任务**

<img src="..\media\ch12_image43.png" style="width:3.79939in;height:3.125in" />

样例12.41中的C代码主要扩展了case语句，该语句对命令进行解码并调用导出的任务，根据lrm2，这些任务声明为extern int

**示例12.41 C代码读取命令文件并调用导出函数**

<img src="..\media\ch12_image44.png" style="width:4.11726in;height:5.79667in" />

示例12.42中的命令文件有新命令，这些命令写两个位置，然后读回其中一个位置，并包含预期的值。

VCS在C语言中将导出的任务声明为void函数。

**示例12.42用于简单内存模型的命令文件**

<img src="..\media\ch12_image45.png" style="width:0.51501in;height:0.54in" />

### 12.8.4 调用对象中的方法

您可以导出SystemVerilog方法，但类中定义的方法除外。这个限制类似于导入静态C方法的限制，如12.3.2节所示，因为当SystemVerilog详细阐述代码时对象不存在。解决方案是在SystemVerilog和C代码之间传递对对象的引用。然而，与C指针不同，SystemVerilog句柄不能通过DPI传递。您可以使用句柄数组，并在两种语言之间传递数组索引。

下面的示例建立在先前版本的内存之上。示例12.44中的SystemVerilog代码有一个封装内存的类。现在您可以拥有多个内存，每个内存在一个单独的对象中。示例12.43中的命令文件创建了两个内存，M0和M1。然后，它对两个内存中的初始化位置执行几次写操作，最后尝试读回这些值。请注意，位置12用于这两个内存。

**示例12.43带有OOP内存的导出方法的命令文件**

<img src="..\media\ch12_image46.png" style="width:0.59126in;height:1.155in" />

示例12.44中的SystemVerilog代码为文件中的每个M命令构造一个新对象。导出的函数mem\_build调用内存构造函数。然后，它将内存对象的句柄存储在SystemVerilog队列中，并将队列索引idx返回给C代码，如示例12.45所示。句柄存储在队列中，这样您就可以动态地添加新的内存。导出的任务mem\_read和mem\_write现在有了一个额外的参数，即内存句柄在队列中的索引。

**示例12.44 SystemVerilog模块与内存模型类**

<img src="..\media\ch12_image47.png" style="width:4.25365in;height:6.36875in" />

**使用OOP内存调用导出任务的示例12.45 C代码**

<img src="..\media\ch12_image48.png" style="width:4.29063in;height:5.81562in" />

### 12.8.5 语境的意义

导入例程的上下文是定义它的位置，例如

$单元、模块、程序或包作用域，就像普通的SystemVerilog例程一样。如果在两个不同的作用域中导入例程，相应的C代码将在发生import语句的上下文中执行。这类似于在两个单独的模块中分别定义SystemVerilog run()任务。每个任务访问自己模块中的变量，没有歧义。

样例12.46表明，如果向样例12.34添加第二个模块，该模块导入相同的C代码并导出自己的函数，C例程将调用不同的SystemVerilog方法，这取决于导入和导出语句的上下文。

**示例12.46第二个模块，用于简单的导出示例**

<img src="..\media\ch12_image49.png" style="width:3.81759in;height:3.46in" />

示例12.47中的输出显示，一个C例程调用两个单独的SystemVerilog方法，这取决于调用C例程的位置。

**带有两个模块的简单示例的输出**

<img src="..\media\ch12_image50.png" style="width:1.8in;height:0.57333in" />

### 12.8.6 为导入的例程设置范围

正如SystemVerilog代码可以调用本地作用域中的例程一样，导入的C例程也可以调用其默认上下文之外的例程。使用svGetScope例程获得当前作用域的句柄，然后在调用svGetScope时使用该句柄，使C代码认为它位于另一个上下文中。示例12.48展示了两个方法的C代码。第一个方法save\_my\_scope()保存从SystemVerilog端调用它的范围。第二个例程c\_display()将其作用域设置为保存的作用域，打印一条消息，然后调用函数sv\_display()。

**示例12.48 C代码获取和设置上下文**

<img src="..\media\ch12_image51.png" style="width:3.88786in;height:2.72in" />

C代码调用svGetNameFromScope()，它返回当前作用域的字符串。这个作用域被打印两次，一次是第一次调用C代码的作用域，另一次是先前保存的作用域。例程svGetScopeFromName()接受一个带有SystemVerilog作用域的字符串，并返回一个指向svScope句柄的指针，该句柄可以与svSetScope()一起使用。

在示例12.49中的SystemVerilog代码中，第一个模块block调用一个保存上下文的C例程。当模块top调用c\_display()时，例程将scope设为block，因此它调用block模块中的sv\_display()例程，而不是top模块。

**样例12.49模块调用获取和设置上下文的方法**

<img src="..\media\ch12_image52.png" style="width:4.1366in;height:4.57667in" />

这将产生示例12.50中所示的输出。

**示例12.50 svSetScope代码的输出**

<img src="..\media\ch12_image53.png" style="width:2.3294in;height:1.03833in" />

您可以使用这个范围的概念来允许C模型知道它是从哪里实例化的，并区分每个实例。例如，一个内存模型可能被实例化多次，并且需要为每个实例分配唯一的存储空间。

## 12.9. 连接其他语言

本章已经展示了使用C和c++的DPI。只需一点工作，你就可以连接其他语言。最简单的方法是调用Verilog $system()任务。如果需要命令的返回值，可以使用Unix system()函数和WEXITSTATUS宏。示例12.51中的SystemVerilog代码为system()调用了一个C包装器。

**示例12.51 SystemVerilog代码为Perl调用C包装器**

<img src="..\media\ch12_image54.png" style="width:4.10009in;height:1.94833in" />

样例12.52是调用system()并转换返回值的C包装器。

**示例12.52 Perl脚本的C包装器**

<img src="..\media\ch12_image55.png" style="width:2.67856in;height:1.18in" />

示例12.53是一个Perl脚本，它打印一条消息并返回一个值。

**示例12.53从C和SystemVerilog调用的Perl脚本**

<img src="..\media\ch12_image56.png" style="width:2.22961in;height:0.40333in" />

现在可以运行示例12.54中的Unix命令来运行模拟并调用hello.pl脚本。

12.11运动 453

**示例12.54 VCS命令行运行Perl脚本**

<img src="..\media\ch12_image57.png" style="width:2.23583in;height:0.10333in" />

## 12.10. 结论

直接编程接口允许您调用C例程,好像他们只是另一个SystemVerilog常规,通过SystemVerilog类型直接进入这开销低于PLI,构建参数列表,并一直跟踪调用上下文,更不用说有4 C例程的复杂性为每个系统的任务。

此外，使用DPI，您的C代码可以调用SystemVerilog例程，允许外部应用程序控制模拟。使用PLI，您将需要触发器变量和更多的参数列表，并且您必须担心多个调用和耗时任务带来的细微错误。

DPI最困难的部分是将SystemVerilog类型映射到C语言，特别是当您有两种语言之间共享的结构和类时。如果您能够掌握这个问题，那么几乎可以将任何应用程序连接到SystemVerilog。

## 12.11. 练习

1.  创建一个C函数，shift\_c，它有两个输入参数:一个32位的无符号输入值i和一个移动量n的整数。输入i移动了n位。当n为正时，值向左移动，当n为负时，值向右移动，当n为0时，不进行移动。函数返回移位后的值。创建一个SystemVerilog模块，调用C函数并测试每个特性。提供输出。

2.  展开练习1，向shift\_c添加第三个参数，一个加载标志ld。当ld为真时，i被移动n位，然后加载到内部32位寄存器中。当ld为假时，寄存器移位n位。在这些操作之后，函数返回寄存器的值。创建一个SystemVerilog模块，调用C函数并测试每个特性。提供输出。

3.  展开练习2，创建shift\_c函数的多个实例。C中的每个实例都需要一个唯一的标识符，所以请使用存储内部寄存器的地址。当函数shift\_c被调用时，打印这个地址和参数。实例化函数两次，并调用每个实例两次。提供输出。

4.  展开练习3中的C代码，显示shift\_c函数被调用的总次数，即使该函数被实例化了不止一次。

5.  展开练习4以提供在实例化时初始化存储值的能力。

6.  展开练习5，将shift\_c函数封装在一个类中。

7.  对于示例12.24和12.25中的代码，以下开放数组方法返回什么?

<img src="..\media\ch12_image58.png" style="width:1.4125in;height:1.50667in" />

8.  修改练习1，这样函数调用导出的SystemVerilog void函数shift\_sv来代替在C中移动值。

9.  展开练习8，为文本12.8.4节中演示的两个不同的SystemVerilog对象调用SystemVerilog函数shift\_sv。假设SystemVerilog函数shift\_build已经导出到C代码中。

10. 将练习8扩展到:

   1.  创建一个SystemVerilog类Shift，其中包含函数shift\_sv(将结果存储在类级变量中)和一个shift\_print函数(显示存储的结果)。

   2.  定义并导出SystemVerilog函数shift\_build。

   3.  支持创建多个Shift对象，这些对象的句柄存储在一个队列中。

   4.  创建一个构建多个Shift对象的testbench。演示每个对象在执行计算后持有一个单独的结果。
