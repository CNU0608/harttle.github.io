---
layout: blog
title: PLC 程序组织单元
tags: PLC 常量 引用 接口 数组 字符串 全局变量 类型转换
---

IEC 61131-3 定义的 **程序组织单元** 包括：功能、功能块、程序。这些程序组织单元不应是 *递归* 的，即对一个程序组织单元的调用不会再次引发对该类型程序组织单元的调用。

## 功能

**功能** ：执行后产生一个结果数据元素，和任意数量的输出元素的程序组织单元。

> 功能结果可以是多值的，即数组或结构。功能不能包含状态信息，即每次同样参数的调用应得到同样的结果。已声明的功能可以在其他程序组织单元中调用。

功能使用示例-ST语言：

```
VAR X,Y,Z,RES1,RES2 : REAL; EN1,V : BOOL; END_VAR

RES1 := DIV(IN1 := COS(X), IN2 := SIN(Y), ENO => EN1); 
RES2 := MUL (SIN(X), COS(Y));

Z: = ADD(EN := EN1, IN1 := RES1, IN2 := RES2, ENO => V);
```

<!--more-->

功能使用示例-FBD语言：

```
       +-----+      +------+     +------+
X ---+-| COS |--+  -|EN ENO|-----|EN ENO|--- V
     | |     |  |   |      |     |      |
     | +-----+  +---| DIV  |-----| ADD  |--- Z
     |              |      |     |      |
     | +-----+      |      |   +-|      |
Y -+---| SIN |------|      |   | +------+
   | | |     |      +------+   |
   | | +-----+                 |
   | |                         | 
   | | +-----+      +------+   |
   | +-| SIN |--+  -|EN ENO|-  | 
   |   |     |  |   |      |   |
   |   +-----+  +- -| MUL  |---+
   |                |      |
   |   +-----+      |      | 
   +---| COS |------|      | 
       |     |      +------+
       +-----+
```

### 表示法

* 对 `VAR_OUTPUT` 的赋值可以空白，也可以用变量。
* 对 `VAR_IN_OUT` 参数的赋值应该是变量。
* 对 `VAR_INPUT` 的赋值可以为空白、常量、变量和功能调用。对于后者，功能的结果将被作为实参。
* 在图形语言中，功能应表示为矩形块，大小取决于输入参数等信息的多少。功能名应卸载矩形块内部。每个输入输出都用一根线来表示。
* 对于输入输出变量名的规定：
    * 没有指定输入变量名时，默认名为：`IN1`, `IN2`（从上到下顺序）
    * 只有一个未命名输入时，默认名为：`IN`
    * 上述默认名可以出现在功能表示的左边，也可以不出现。
* 还可以使用附加输入 `EN` 和附加输出 `ENO`，它们分别位于左右的最上方。
* 应使用信号线进行参数的连接（包括功能结果）。
* 布尔类型的输入输出可用 `O` 进行翻转。
* `VAR_IN_OUT` 应被适当地连接。
* 参数列表是一系列实参到形参的赋值组成的集合。
    * `VAR_IN` `VAR_IN_OUT` 赋值使用 `:=`
    * `VAR_OUT` 赋值使用 `=>`
* 除`EN` `ENO`外，参数列表中参数的数目、类型和顺序都应与功能定义相符。

### 执行控制

附加的 `EN`(Enable) 输入、`ENO`(Enable Out) 输出可用于控制功能的执行，规则如下：

1. 如果 `EN` 值为 `FALSE`，功能被调用时不应执行，且 `ENO` 应被PC系统重置为 `FALSE`。否则功能应被执行，且 `ENO` 应被PC系统置为 `TRUE`，功能操作中也可以对`ENO`进行赋值。
2. 当功能执行中发生错误时，`ENO` 应被PC系统置为 `FALSE`，否则制造商应提供的其他的错误处理方式。
3. 当`ENO`值为`FALSE`时，功能所有输出的值均 **实现相关** 。

`EN` `ENO` 使用示例：

```
(* LD语言 *)
         +-------+         |
| ADD_EN |   +   | ADD_OK  |
+---||---|EN  ENO|---( )---+ 
|        |       |         |
|    A---|       |---C     |
|    B---|       |         | 
         +-------+         |

(* FBD语言 *)
         +-----+ 
ADD_EN---|EN   |
     A---|  +  |---C 
     B---|     |
         +-----+

```

### 声明

文本语言中，功能声明由以下几部分组成：

1. 关键字 `FUNCTION`；
2. `VAR_INPUT...END_VAR` 构造，规定输入变量；
4. `VAR_IN_OUT...END_VAR`构造和`VAR_OUTPUT...END_VAR`构造，分别规定输入-输出和输出变量。
3. `VAR...END_VAR` 构造，规定内部变量；
5. 功能体。规定功能对这些变量的操作，通过对与功能同名的变量赋值来设置返回值。
6. 终止关键字 `END_FUNCTION`

图形语言中，功能声明由以下几部分组成：

1. 关键字 `FUNCTION` `END_FUNCTION`，或等效的图形元素；
2. 功能名，功能结果和功能变量的名称、类型、初始值的图形说明；
3. 内部变量的名称、类型、初始值的图形说明。
4. 功能体，同上。

> 每个 *资源* 中的功能声明数目是一个 **实现相关** 的参数。

功能声明示例-ST语言：

```
FUNCTION SIMPLE_FUN : REAL 
    (* External interface specification *) 
    VAR_INPUT
        A,B : REAL ; 
        C   : REAL := 1.0;
    END_VAR
    VAR_IN_OUT COUNT : INT ; END_VAR 
    VAR COUNTP1 : INT ; END_VAR

    (*Function body specification *) 
    COUNTP1 := ADD(COUNT,1); 
    COUNT   := COUNTP1 ;
        SIMPLE_FUN := A*B/C; 
END_FUNCTION
```

功能声明示例-FBD语言：

```
FUNCTION
        +-------------+ (* External interface specification *)
        | SIMPLE_FUN  | 
REAL----|A            |----REAL
REAL----|B            | 
REAL----|C            | 
INT-----|COUNT---COUNT|----INT
        +-------------+
(* Function body specification *)
       +---+
       |ADD|---         +----+ 
COUNT--|   |---COUNTP1--| := |---COUNT
    1--|   |            +----+
       +---+    +---+
            A---| * |   +---+
            B---|   |---| / |---SIMPLE_FUN
                +---+   |   | 
            C-----------|   | 
                        +---+
END_FUNCTION
```

### 重载、类型化、类型转换

**重载** ：标准功能、功能块、操作或指令可以通过 *一般数据类型* 标识符操作多种类型的输入。可编程控制器系统支持某个标准功能、功能块、操作符、指令的重载时，应对其给定的一般数据类型可接受的基本数据类型都给与支持。

**类型化** ：即在重载功能名后添加`_`和类型名。此时，该功能的输入/输出将被限制为该类型。

> 例如，采用一般数据类型`ANY_NUM`的功能可操作的数据类型包括：`LREAL`,`REAL`,`DINT`,`INT`,`SINT`。
> 
> 在功能重载中，同样的一般数据类型参数应当具有相同的实际类型。如果需要，类型转换会被执行以满足该要求。

重载功能

```
            +-----+
            | ADD | 
ANY_NUM-----|     |----ANY_NUM
ANY_NUM-----|     |
     . -----|     |
     . -----|     |
ANY_NUM-----|     | 
            +-----+
```

类型化功能

```
        +---------+
        | ADD_INT | 
INT-----|         |----INT
INT-----|         | 
 . -----|         | 
 . -----|         |
INT-----|         |
        +---------+
```

类型转换

```
    +-----------+   +---+ 
A---|INT_TO_REAL|---| + |---C 
    +-----------+   |   |
B-------------------|   | 
                    +---+
C := INT_TO_REAL(A)+B;
```

### 标准功能

这一章对所有可编程控制器编程语言通用的功能进行了定义。包括一些 *可扩展* 的标准功能，它们可以对两个或者更多的输入进行指定的操作。

> 最大的输入数目则是 **实现相关** 的。对于指定形参方式的功能调用，最后位置的形参名将决定实际的输入数目。

示例：

```
X := ADD(Y1,Y2,Y3);
(* 相当于： *)
X := ADD(IN1 := Y1, IN2 := Y2, IN3 := Y3);

I := MUX_INT(K:=3,IN0 := 1, IN2 := 2, IN4 := 3);
(* 相当于： *)
I := 0;
```

标准功能包括：

1. 类型转换功能。`*_TO_**`,`TRUNC`(截断),`*_BCD_TO_**`,`**_TO_BCD_*`，`*`可以是字符类型，`**`可以是整型。
2. 数值功能。`ABS`,`SQRT`,`LN`,`LOG`,`EXP`,`SIN`,`COS`,`TAN`,`ASIN`,`ACOS`,`ATAN`,`+`,`*`,`-`,`/`,`modulo`,`**`,`:=`
3. 位串功能。`SHL`,`SHR`左右移位；`ROR`,`ROL`左右旋转。
4. 选择和比较功能。
    * 选择功能：`SEL(G:=0,IN0:=X,IN1:=5)`,`MAX`,`MIN`,`MUX`.
    * 比较功能：`>`,`<`,`>=`,`<=`,`=`,`<>`.
    * 布尔运算：`&`,`XOR`,`OR`,`NOT`.
5. 字符串功能。`LEN`,`LEFT`,`RIGHT`,`MID`,`CONCAT`,`INSERT`,`DELETE`,`REPLACE`,`FIND`.
6. 时间功能。`+`,`-`,`*`,`/`,`DT_TO_TOD`,`DT_TO_DATE`.
7. 枚举功能。`SEL`,`MUX`,`=`,`<>`

## 功能块

**功能块** ：执行后产生一个或多个值的程序组织单元。 *功能块* 可创建多个 *实例* ，每个实例拥有对应的标识符（即实例名）与相应的数据结构来保存输出变量、内部变量、输入变量（在某些实现中，只保存输入变量的引用）。

> 输出变量和必要的内部变量在执行后将被保存，于是同样参数的调用可能产生不同的结果。在功能块实例外部，只有输入变量和输出变量是可访问的。

### 表示法

在文本语言中， *功能块的实例* 可使用`VAR...END_VAR`声明，与 *结构体* 实例的声明相同。

在图形语言中， *功能块的实例* 采用矩形块表示，功能块类型名位于块内部，实例名位于块上方，其他规则同 *功能的图形表示* 。

功能块实例使用示例：

```
(* 图形方式，FBD语言 *)
         FF75 
       +------+
       |  SR  | 
%IX1---|S1  Q1|---%QX3
%IX2---|R     | 
       +------+

(* 文本方式，ST语言 *)
VAR FF75: SR; END_VAR (* 声明 *) 
FF75(S1:=%IX1, R:=%IX2); (* 调用 *) 
%QX3 := FF75.Q1 ; (* 输出赋值 *)
```

功能块变量的读写属性：

* 输入变量在功能块内部是只读的，外部是只写的（或通过通信功能等进行读写，见 IEC 61131-1）。
* 输出变量在功能块内部是读写的，外部是只读的。
* 输入-输出变量在内外都是可读写的。

> 未赋值（或未连接）的输入变量将保持他们的初始值或上次调用（假如曾被调用过）时的值。

对于`EN`,`ENO`在功能块中的用法有如下规则：

1. 当`EN`为`FALSE`时，输入变量是否进行赋值是 **实现相关** 的，同时功能块体不应被执行，且`ENO`应被PC系统设为`FALSE`；否则，功能块将被执行且`ENO`应被PC系统赋为`TRUE`，功能块操作中也可以对`ENO`进行赋值。
2. 当`ENO`为`FALSE`时，功能块的输出变量`VAR_OUTPUT`应保持上次调用的值。


### 声明

如同 *功能声明* ， *功能块声明* 也可采用文本方式和图形方式，他们之间的区别如下：

1. 功能块的界定关键字为：`FUNCTION_BLOCK`,`END_FUNCTION_BLOCK`；
2. `RETAIN`可用作内部变量和输出变量的限定符；
3. 以`VAR_EXTERNAL`构造传入的变量是可修改的；
4. 以`VAR_INPUT`,`VAR_IN_OUT`,`VAR_EXTERNAL`传入的功能块实例的输出变量是只读的。
5. 以`VAR_IN_OUT`,`VAR_EXTERNAL`传入的功能块实例可以被调用；
6. 在文本方式的声明中，`R_EDGE`,`F_EDGE`限定符将引发对`R_TRIG`,`F_TRIG`功能块实例的隐式声明，该限定符提供边沿检测功能；
7. 边沿检测的图形表示中，用`>`,`<`作为输入变量与功能块的交点；
8. 一般数据类型的使用与实际类型的推导与 *功能声明* 相同，对于这类功能块在调用时应显式地进行输出赋值（`=>`）；
9. `*`可用于内部变量声明。
10. 下列情况中的未赋值将产生 **错误** ：
    * 输入-输出变量
    * 功能块类型的输入变量

> 变量和功能块实例可通过`VAR_IN_OUT`传入功能块，而功能/功能块的输出却不可以。这是为了避免不小心对他们进行的修改。

功能块声明-ST语言

```
FUNCTION_BLOCK DEBOUNCE 
(* 外部接口 *) 
VAR_INPUT IN : BOOL ;   (* 初始值 = 0 *)
    DB_TIME : TIME := t#10ms ; (* 初始值 = t#10ms *)
END_VAR
VAR_OUTPUT OUT : BOOL ; (* 初始值 = 0 *) 
    ET_OFF : TIME ;     (* 初始值 = t#0s *)
END_VAR
VAR DB_ON : TON ;       (* 内部变量 *) 
    DB_OFF : TON        (* 功能块实例 *);
    DB_FF : SR ; 
END_VAR

(* 功能块体 *) 
DB_ON(IN := IN, PT := DB_TIME) ;
DB_OFF(IN := NOT IN, PT:=DB_TIME) ; 
DB_FF(S1 :=DB_ON.Q, R := DB_OFF.Q) ; 
OUT := DB_FF.Q ;
ET_OFF := DB_OFF.ET ;

END_FUNCTION_BLOCK
```

功能块声明-FBD语言

```
FUNCTION_BLOCK
(* 外部接口 *)
       +---------------+
       |    DEBOUNCE   | 
BOOL---|IN          OUT|---BOOL 
TIME---|DB_TIME  ET_OFF|---TIME 
       +---------------+

(* 功能体 *)
              DB_ON       DB_FF 
             +-----+     +----+ 
             |TON  |     | SR |
IN----+------|IN  Q|-----|S1 Q|---OUT 
      |  +---|PT ET|  +--|R   |
      |  |   +-----+  |  +----+
      |  |            |
      |  |    DB_OFF  |
      |  |   +-----+  | 
      |  |   |TON  |  |
      +--|--O|IN  Q|--+
DB_TIME--+---|PT ET|--------------ET_OFF 
             +-----+
END_FUNCTION_BLOCK
```

### 标准功能块

这一章对所有可编程控制器编程语言通用的功能块进行了定义。有些是重载过的，有些具有 *可扩展* 的输入输出。

标准功能块包括：

1. 双稳态。包括置位主导的和复位主导的两种。
2. 边沿检测。包括上升沿检测和下降沿检测两种。
3. 计数器。包括升计数器、降计数器、升降计数器。
4. 定时器。包括脉冲定时器、接通延时定时器、断开延时定时器。
5. 通信功能块。标准通信功能块在 IEC 61131-5 定义，包括：设备检验、轮询数据获取、程控数据获取、参数控制、互锁控制、程控报警、连接管理和保护。

## 程序

**程序** ：为实现机械控制或PC系统过程需要的信号处理，必要的编程语言元素和构造组成的逻辑组合体。

程序的声明和使用与功能块基本相同，附加的特性和区别如下：

1. 分隔关键字为`PROGRAM...END_PROGRAM`
2. 程序可以包含访问路径的定义：`VAR_ACCESS...END_VAR`，提供了命名变量用于通信服务（IEC 61131-5）。
2. 可以包含全局变量定义：`VAR_GLOBAL...END_VAR`。
2. 可以包含外部变量声明：`VAR_EXTERNAL`。
2. 可以包含临时变量定义：`VAR_TEMP`。
3. *程序* 只能在 *资源* 中实例化，而 *功能块* 只能在 *程序* 或其他 *功能块* 中实例化。
4. 程序的全局和内部变量声明中可以包含地址赋值，而不完整的直接表示的地址赋值只能用于内部变量。



