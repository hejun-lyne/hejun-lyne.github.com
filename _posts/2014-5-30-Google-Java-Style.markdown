---
layout: post
title: Google Java Style 中文（自翻）
category: java
url: Google-Java-Style.html
summary: 谷歌Java语言编程规范，中文翻译（自翻）
---

该文档翻译自<http://google-styleguide.googlecode.com/svn/trunk/javaguide.html>
###1 介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本文档为使用Java编程语言的源代码定义了Google编码规范。一个Java源文件只有在遵守了文档所定义的规则前提下才能称之为使用了*Google Style*。
和其他编程风格指南一样，在这个方面（编码规范）不仅仅只包括格式上的美观问题，还涵盖了其他的约定俗成和编码标准。但是本文档主要关注那些可以被我们统一遵循的明确易懂的规则，并且尽量避免那些实施（人或者工具）可能性不大的规则。
####1.1 名词解析
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本文档中如无特别说明：
1. 名词 *类(class)* 用于表示：“普通” class、enum class、 interface 或 annotation 类型（@interface）。
2. 名词 *评论(comment)* 始终表示 *实现(implementation)* 评论。我们不使用“documentation comments”，而是使用常用词“Javadoc”。

其他的“名词解析”会在文档中偶然出现。
####1.2 指南解析
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;文档中的示例代码是非标准的，这是指：虽然示例使用了Google Style，但并不是用来说明这是代表该段代码唯一的正确方式。在示例中使用的其他可选的格式化方式不应该认为是强制规则。
###2 源文件基础
####2.1 文件名
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;源文件的名字由其包含的最外层（to-level）的类的大小写敏感（case-sensitive）的名字，加上`.java`扩展名组成。
####2.2 文件编码：UTF-8
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;源文件使用UTF-8编码
####2.3 特殊字符
#####2.3.1 空白字符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了行结束符（line terminator sequence？）之外，只有ASCII水平空白字符（0x20）可以出现在源文件的任何地方，这表示：
1. 所有在字符串或字符值中的空白字符应该进行转义（escaped）。
2. Tab字符不应该用来进行缩进。

#####2.3.2 特殊转义符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于那些有特殊转义符（`\b`,`\t`,`\n`,`\f`,`\r`,`\"`,`\'`,`\\`），应该使用对应的转义符而不是对应的十六进制(例如`\012`)或者Unicode(例如`\u00a`)转义。
#####2.3.3 非ASCII字符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于剩下的非ASCII字符，可以使用真实的Unicode字符（例如`∞`）或者等价的Unicode转义（例如`\u221e`），这取决于哪种方式代码的可读性更好。
**注：**在使用Unicode转义，甚至有时候使用真实Unicode字符的情况下，一天解释的注释会非常有帮助。

| 示例      |     评论 |
| :-------- | --------:|
| `String unitAbbrev = "μs";`    |   最好：不需要评论也很清楚了 |
| `String unitAbbrev = "\u03bcs"; // "μs"` | 可以接受：但没理由这样做 |
| `String unitAbbrev = "\u03bcs"; // Greek letter mu, "s"` | 可以接受：但有点笨拙且容易弄错 |
| `String unitAbbrev = "\u03bcs";` | 差劲：读者完全不知道这是什么 |
| `return '\ufeff' + content; // byte order mark` | 良好：使用转义代替不可打印字符，同时又加了解释 |

**注：**为了防止某些程序可能没有正确处理不可打印ASCII字符，你应该尽量增加代码的可读性。如果发生了这种事情，那些程序就会出错，这些问题应该得到修复。

###3 源文件结构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个源文件由以下组成（按顺序）：
1. 授权（license) 或者版权（copyright）信息，如果有的话
2. package 语句
3. import 语句
4. 唯一的最外层（top-level）class

各个区块之间使用一个空白行来分隔。

####3.1 授权（license) 或者版权（copyright）信息，如果有的话
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果一个文件包括授权或版权信息，应该放在这个位置。
####3.2 package 语句
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;package语句不是用行封装的，不受列数的限制（4.4节：列数限制：80或100）。
####3.3 import 语句
#####3.3.1 不要在imports语句使用通配符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;包含通配符的import语句，不管是静态还是其他，都不应该被使用。
#####3.3.2 不要使用行封装（line-wrapping）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;import语句不应该使用行封装，其不适用列限制（4.4 列限制：80或100）。
#####3.3.3 顺序及留空
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Import语句分为以下几组（group），组之间使用一个空行分隔：
1. 所有静态（static）import在一个组
2. `com.google` import（只有该源文件在`com.google` package空间的时候）
3. 第三方import，每个最顶级（top-level）的package一个组，使用ASCII字母排序，例如`android`,`com`,`org`,`sun`
4. `java` import
5. `javax` import

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;组内不应该出现空行，并且引入的名字也应该按照ASCII进行排序。（注：这个与import语句按照ASCII排序不同，因为要考虑分号的存在）。
####3.4 类声明
####3.4.1 只声明一个最顶层(top-level)类
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个源文件只能有一个自己的最顶层类。
####3.4.2 类成员顺序
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类成员的顺序对可读性有非常大影响，但是关于这方面没有所谓的唯一的良方。不同的类在成员的排序上也会不同。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;重要的是每一个类应该按照一定的逻辑顺序来安排其成员，该顺序在被问及的时候维护它的人应该能够做出相应的解析。例如，新方法不是习惯性的添加到类的最后，如果这样做的话就是按照“添加的时间先后”排序，这个排序就不是一个符合逻辑的排序。
#####3.4.2.1 重载(Overloads)：绝对不要分开
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果一个类有多个构造方法，或者是同名方法，则它们应该按顺序先后出现，不应该在中间插入其他成员。
###4 格式化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**术语解析** *类块结构(block-like construct)* 指的是一个类的body、方法或者构造方法。需要注意，在4.8.3.1中关于数组初始化中，任何的数组初始化都可能有意的认为是类块结构。
####4.1 大括号(Braces)
#####4.1.1 大括号应该有选择的使用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;大括号在这些语句（`if`,`else`,`for`,`do`,`while`）中使用，即使主体为空或只有一个语句的时候也应该使用。
#####4.1.2 非空块(Nonempty blocks)：K & R 风格
* 在`{`之前无换行
* 在`{`之后换行
* 在`}`之前换行
* 如果`}`是一个语句或方法、构造方法体或命名类的结束，应该使用换行；例如，如果`}`后面跟着`else`或者`,`就不应该换行。

例子：
{% highlight java %}
return new MyClass() {
  @Override public void method() {
    if (condition()) {
      try {
        something();
      } catch (ProblemException e) {
        recover();
      }
    }
  }
};
{% endhighlight %}
在4.8.1中会提到一些与enum有关的额外情况。
#####4.1.3 空块：可以更加简明
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个空块意味着`{}`，中间无任何字符。在多块语句(multi-block statement)中出现的不称之为空块（直接包括多块的语句：`if/else-if/else` 或`try/catch/finnaly`）。
例子：
{% highlight java %}
void doNothing() {}
{% endhighlight %}
####4.2 块缩进：+2 空格
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每次开始一个新的块或类块结构的时候，应该增加2个空格缩进。块结束之后，缩进应该回复上一层缩进的情况。缩进层级对整个块内的代码和评论均有效。（具体请查看4.1.2中的例子）。
####4.3 一行一条语句
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一条语句后面应该跟着换行。
####4.4 列限制：80 或 100
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;工程可以自由选择80或100字符的列限制。任何超过这个限制的行需要使用行封装（line-wrapped），这个会在4.5中解释。例外的情况如下：
1. 不可能遵守该规则的行（例如，在Javadoc中的长URL，或者一个较长的JSNI方法引用）
2. `package`和`import`语句（查看3.2和3.3）
3. 评论中的命令行，该命令可能被复制粘贴到shell中执行

####4.5 行封装（Line-wrapping）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**术语解析** *行封装(line-wrapping)* 指：为了不超出列限制，代码可以合理占用一行的时候被分割为多行。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;没有广泛可以确定的公式告诉我们在各种情况下如何进行行封装。很多时候，可以有很多中方式来对同一段代码进行行封装。
**注：**从代码中提取出方法或本地变量可能不需要进行行封装就可以解决问题。
#####4.5.1 在哪里换行
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;关于行封装最基本的指导是：尽量在更高的语法层次换行。还有：
1. 当行在一个*非赋值(non-assignment)*操作符打断的时候，应该在该符号之前换行。（注意这个与其他语言的Google Style不同）。同样也适用以下的类操作符(operator-like)符号：点分割符`.`、概括类型的尖括号`<T extends Foo & Bar>`、catch中`catch (FooException | BarException e)`
2. 当行在*赋值(assignment)*)操作符中断的时候一般在符号之后换行，但在之前换行也是可以接受的，这个对于*类赋值操作符(assignment-operator-like)*符号也是适用的，如`for`("foreach")语句中的冒号
3. 方法或构造方法（constructor）的名字应该紧接着`(`符号
4. 一个逗号(`,`)应该紧跟着它前面的字符。

#####4.5.2 续行的缩进至少+4空格
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在进行行封装的时候，第一行之后的所有行至少要比原始行增加4个空格的缩进。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有多个续行的时候，根据需要缩进可能会超过+4空格。一般情况下2个续航如果在语义上是并行的话只需要使用+4空格缩进。
4.6.3关于*水平对齐*指出了最好不要使用可变数量的空格来对齐前面行的特定元素。
####4.6 空白
#####4.6.1 垂直方向空白
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;什么时候使用空白行：
1. 在连续的类成员（或构造方法）之间，包括：成员变量（值，fields）、构造方法（constructors）、成员方法（methods）、内部类（nested classes）、静态初始化器（static initializers）、实例初始化器（instance initializers）。
    * 例外：两个连续成员变量之间的空行是可选的。这种情况下的空行一般是用于创建逻辑分组。
2. 方法体内部如果需要为语句创建逻辑分组的时候可以使用。
3. 在第一成员之前或者最后一个成员之后是可选的（不鼓励也不否定）。
4. 文档中其他部分有要求的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;多个连续空行是允许使用的，但绝不会说要求（或鼓励）。
#####4.6.2 水平空白
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了因为语言或其他的风格规则，或者纯文本、评论和Javadoc以外，单个的ASCII空白字符也只能在以下几种地方出现：
1. 用于分隔保留字（例如`if`,`for`,`catch`）和在同一行紧跟其后的`(`。
2. 用于分隔保留字（例如`else`,`catch`）和在同一行在其之前的`{`。
3. 在`{`之前使用，有两个例外情况：
    * `@SomeAnnotation({a, b})`不使用任何空格
    * `String[][] x = {{"foo"}}`（按照下面第8条不使用空格）
4. 在任何二元或三元操作符的两边。同样也适用下面的“类操作”符号
    * 连接类型边界的尖括号`<T extends Foo & Bar>`
    * catch块中处理多个异常的竖线符号`catch (FooException | BarException e)`
    * 加强的`for`("foreach")语句中的冒号`:`
5. 在`,:;`之后，或者是强制类型转换的`)`之后
6. 在用于开始一个单行评论的双斜线`//`的两边，在这里，多个空格是可以使用的，但不是必须的。
7. 在声明类型或变量语句的中间：`List<String> list`
8. 在数组初始化的括号内部是*可选的*
    * `new int[] {5, 6}`或者`new int[] { 5, 6 }`都可以

**注：**该规则不要求或禁止在行开始或结束的地方使用额外的空白，只是对内部空间的要求。
#####4.6.3 水平对齐：不要求
**术语解释**：*水平对齐*指为了达到让特定字符直接位于上一行的特定字符之下，而在代码中添加额外的若干空白。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;你可以这样做，但这不是Google Style所要求的。我们甚至对那些已经这么做的地方也没有要求保持水平对齐。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面是没有使用和使用了的两种情况对比：
{% highlight java %}
private int x; // this is fine
private Color color; // this too

private int   x;       // permitted, but future edits
private Color color;   // may leave it unaligned
{% endhighlight %}
**贴士**：对齐可以带来可读性，但是会为将来的维护制造了麻烦。如果我们将来需要改懂其中的某一行。这个改动可能会破坏了原来很整齐的格式，这是可以接受的。更多时候会建议程序员（可能就是你哦）也调整一下附近行的空白处，这很有可能会触发一连串的重新格式化。那一行的改动现在有了“爆炸性范围”的影响，这会导致毫无意义的大量工作，但最糟糕的是的是它还破坏了版本历史信息，降低审查者们（reviewers）的工作效率以及增加了合并冲突的难度。

####4.7使用括号分组：推荐
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可用可不用的分组括号只有在作者和审查者都认为缺少括号的情况下不会对代码的解释产生歧义或者对代码可读性没有帮助的情况下可以省略掉。假设每个代码阅读者都完全记住了Java操作符优先顺序表是不合理的。
####4.8 具体的结构体们
#####4.8.1 枚举类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个枚举常量后面的逗号是否使用换行，是可选的。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个没有方法或常量解释文档的枚举类型可以用类似数组初始化的格式（看4.8.3.1关于数组初始化的内容）。
{% highlight java %}
private enum Suit { CLUBS, HEARTS, SPADES, DIAMONDS }
{% endhighlight %}
因为枚举类型也是类的一种，所以其他关于格式化类的规则都适用。
#####4.8.2 变量声明
######4.8.2.1 一条生命语句一个变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个变量声明（成员或局部）只声明一个变量：像`int a, b;`这样的声明不应该使用。
######4.8.2.2 按需声明，尽快初始化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;局部变量不应该按照通常情况在其包含块或类块结构的开始处声明。而应该在最接近它们被首次使用（合理地）的地方声明，这样可以最小化其作用范围。局部变量声明一般包括初始化，或者在声明之后马上初始化。
#####4.8.3 数组
######4.8.3.1 数组初始化：可以是“类块”的
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;任何数组初始化可以选择按照“类块”结构进行格式化，例如下面这些都是可以的（不是全部）：
{% highlight java %}
new int[] {
  0, 1, 2, 3
}
new int[] {
  0,
  1,
  2,
  3,
}
new int[] {
  0, 1, 
  2, 3,
}
new int[]
    {0, 1, 2, 3}
{% endhighlight %}
######4.8.3.2 不要使用C风格的数组声明
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在类型后面使用`[]`，而不是变量名后面：`Sring[] args`，不是`String args[]`。
#####Switch 语句
**术语解释:**在*switch块*的括号内部是1或多个语句组，每一个语句组由1或多个`switch 标签`（`case FOO:`或`default:`）组成，紧跟着是1或多条语句。
######缩进
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;和任何其他块一样，switch块的内容也是使用+2缩进。
在switch标签后面，应该使用新行，并且缩进级别+2，就好像新开始了一个块。后面的switch标签恢复到之前的缩进级别，就好像一个块结束了。
######4.8.4.2 落下（Fall-through，指不使用break,return）：须用评论
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在一个switch块内，每一个语句组或是突然终止（使用`break`,`continue`,`return`或抛出异常），或是使用一条评论表明执行有可能会继续到下一个语句组。任何评论只要能够表达这个意思即可（一般`// fall through`）。在最后一个语句组并不要求这个特定的评论。例如：
{% highlight java %}
switch (input) {
  case 1:
  case 2:
    prepareOneOrTwo();
    // fall through
  case 3:
    handleOneTwoOrThree();
    break;
  default:
    handleLargeNumber(input);
}
{% endhighlight %}
######4.8.4.3 必须使用default case
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个switch语句都包含一个`default`语句组，即使不包含任何代码。
#####4.8.5 注释（Annotations）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注释用于一个类、方法或构造方法的文档块后面，并且每一个注释占据一行。换行不构成行封装（4.5），所以缩进级别不会增加。例如：
{% highlight java %}
@Override
@Nullable
public String getNameIfPresent() { ... }
{% endhighlight %}
**例外:** 一个无参数的注释可能会和被注释的代码出现在同一行。例如：
{% highlight java %}
@Override public int hashCode() { ... }
{% endhighlight %}
对成员变量的注释也是在文档块之后出现，但在这种情况下，多个注释（可能带参数）也可能出现在同一行；例如：
{% highlight java %}
@Partial @Mock Dataloader loader;
{% endhighlight %}
关于格式化参数和局部变量注释没有特别的规则。
#####4.8.6 评论
######4.8.6.1 块评论风格
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;块评论和周围的代码使用同样的缩进。可以使用`/* ... */`或`// ...`。对于多行的 `/* ... */` 评论，后一行必须使用`*`和前一行对齐。
{% highlight java %}
/*
 * This is
 * okay.
 */
 
// And so
// is this.

/* Or you can
 * even do this. */
{% endhighlight %}
评论不使用由星号或其他符号构成的盒状包含
**贴士：**当写多行评论的时候，使用`/* ... */`
#####4.8.7 修饰词
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类和成员的修饰词应该按照Java语言推荐的顺序排列：
{% highlight java %}
public protected private abstract static final transient volatile synchronized native strictfp
{% endhighlight %}
#####4.8.8 数字常量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`long`类型常量使用大写`L`后缀，绝对不要使用小写（免得和数字`1`混淆），使用`30000000L`而非`30000000l`。

###5 命名
####5.1 对所有标识符适用的规则
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;标识符只使用ASCII字符和数字，只有在下面说明的两种情况可以使用下划线。因此每一个有效的名字都能使用正则表达式`\w+`匹配。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Google Style中特别的前缀或后缀不应该使用，例如`name_`,`mName`,`s_name` 和 `kName`。
####5.2 针对不同标识符类型的规则
#####5.2.1 包（Package）名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;包名字都应该使用小写，连着的单词不要使用下划线。例如，`com.example.deepspace`，而不是`com.example.deepSpace`或`com.example.deep_space`。
#####5.2.2 类（Classes）名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类名字使用单词字母大写的规则（UpperCamelCase）。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类名字一般是名词或名词组合。例如，`Character`或`ImmutableList`。接口名字也一般是名词或名词组合（例如，`List`），但有时候也使用形容词或形容词组合（例如，`Readable`）。
对于注释类型，还没有特别的或已经建立的很好的规则。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Test*类型以测试的类名开头，以`Test`结束。例如，`HashTest`或`HashIntegrationTest`。
#####5.2.3 方法名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;方法名字以小写字母开头，内部单词首字母大写（lowerCamelCase）。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;方法名字一般使用动词或动词组合。例如，`sendMessage`或`stop`。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在JUnit测试方法中可能使用下划线来分隔名字的各个逻辑部分。一个典型的模版是`test<MethodUnderTest>_<state>`，例如`testPop_emptyStack`。关于如何命名测试方法不存在说一个正确的方式。
#####5.2.4 常量名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;常量名字使用`CONSTANT_CASE`：使用下划线分隔的大写字母。首先我们需要了解下究竟什么是常量？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个常量都是一个static final的成员，但并不是所有的static final成员都是常量。在考虑使用常量之前，先考虑该成员是否真的感觉像一个常量。例如，如果那个实例的任何可观察的状态都是可变的，那么显然着不是一个常量。仅仅想要从不改变对象是通常不够的。例如：
{% highlight java %}
// Constants
static final int NUMBER = 5;
static final ImmutableList<String> NAMES = ImmutableList.of("Ed", "Ann");
static final Joiner COMMA_JOINER = Joiner.on(',');  // because Joiner is immutable
static final SomeMutableType[] EMPTY_ARRAY = {};
enum SomeEnum { ENUM_CONSTANT }

// Not constants
static String nonFinal = "non-final";
final String nonStatic = "non-static";
static final Set<String> mutableCollection = new HashSet<String>();
static final ImmutableSet<SomeMutableType> mutableElements = ImmutableSet.of(mutable);
static final Logger logger = Logger.getLogger(MyClass.getName());
static final String[] nonEmptyArray = {"these", "can", "change"};
{% endhighlight %}
这些名字一般都是名词或名词组合。
#####5.2.5 非常量（non-constant）成员名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;非常量成员名字（static 或 非static）使用首字母小写，后续单词首字母大写（lowerCamelCase）。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一般使用名词或名词组合。例如，`computedValues`或`index`。
#####5.2.6 参数名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参数名字也采用首字母小写，后续单词首字母大写（lowerCamelCase）。最好不要使用只有一个字母的变量名字。
#####5.2.7 局部变量名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;局部变量名字也采用`lowerCamelCase`，相比其他类型的名字可以使用更加自由的缩写。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但除了临时变量和循环的变量以外，也不应该使用1个字母的名字，
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;即使局部变量是最终不可修改的，也不被认为是常量，所以也不应该按照常量的风格来书写。
#####5.2.8 类型变量名字
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个类型变量都是使用以下两种风格之一来命名：
* 一个单独的大写字母，可以跟一个单独的数字（例如`E`,`T`,`X`,`T2`）
* 像类名一样的名字（5.2.2，类名字），跟着一个大写的字母`T`（例如：`RequestT`,`FooBarT`）。

####5.3 Camel case: 定义
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将一个英语单词转换成Camel case，有时候有多种可行的方式，例如首字母缩写和不常使用的结构如"IPv6"或"iOS"。为了改进可预测性，Google Style定义了下面的确定的格式。
以名字的简单形式开始：
1. 将单词转换为单纯的ASCII，去掉任何的撇号。例如，"Müller's algorithm"可能会变为"Muellers algorithm"。
2. 将结果划分为单词，从空白或标点符号（一般为连字符）处分开。
    * *推荐：* 如果一个单词在平常使用中已经有一个约定俗成的Camel-case形式，那么将其分割为连着的多个部分（例如，"AdWords"使用"ad words"代替）。需要注意像"iOS"这样的单词其实不是真的使用了Camel-case，它违背了任何的常理，所以不适用这里推荐的方法。
3. 现在全部改成小写（包括那些首字母缩写词），然后对下面的单词的首字母大写：
    * ... 每一个单词，使其成为upper camel case，或
    * ... 每一个单词除了第一个，使其成为lower camel case
4. 最后，把所有单词连接在一块组成单一的标识符。

注意所有原始单词的大小写都应该被忽略掉。例子：

| 简单形式      |     正确 |   错误   |
| :-------- | --------:| :------: |
| "XML HTTP request"    |   XmlHttpRequest |  XMLHTTPRequest  |
| "new customer ID" | newCustomerId | newCustomerID|
| "inner stopwatch" | innerStopwatch | innerStopWatch|
| "supports IPv6 ON iOS?" | supportsIpv6OnIos | supportsIPv6OnIOS |
| "YouTube importer" | YouTubeImporter YoutubeImporter *|  |
\*表示可以接受，但不推荐。
**注：**在英语中有些单词组在连字符的使用上面是多变的。例如`nonempty`和`non-empty`都是正确的，所有方法名`checkNonempty`和`checkNonEmpty`看起来都是正确的。

###6 编程实践
####6.1 @Override: 始终使用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;任何合理的时候一个方法都会使用`@Override`注释。这些情况包括：一个类方法重写了其父类的方法；一个类方法实现了一个接口方法；一个接口方法重新定义了其父接口的方法。
例外：如果父方法属于`@Deprecated`的时候`@Override`可能会被省略掉。
####6.2 捕获异常：不应该忽略
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了下面特意指出的情况外，在捕获异常的响应内部不做任何事情是不正确的。（一般的响应是记录这个异常，或者如果该异常被认为是"impossible"，那么重新将其作为`AssertionError`抛出。）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果在一个catch块内部无论如何都不采取任何动作是正确的话，那么其原因应该在评论中解释。
{% highlight java %}
try {
  int i = Integer.parseInt(response);
  return handleNumericResponse(i);
} catch (NumberFormatException ok) {
  // it's not numeric; that's fine, just continue
}
return handleTextResponse(response);
{% endhighlight %}
**例外：**在测试中，如果异常是命名为`expected`的话那么可能会被忽略掉（无须评论），下面是保证一个测试确实会抛出特定类型的异常的通常做法，所以是不需要在这里添加评论的。
{% highlight java %}
try {
  emptyStack.pop();
  fail();
} catch (NoSuchElementException expected) {
}
{% endhighlight %}
####6.3 静态成员：使用类名界定
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当引用一个静态类成员的时候，必须要使用类名字来界定。不应该使用该类的引用或者表达式。
{% highlight java %}
Foo aFoo = ...;
Foo.aStaticMethod(); // good
aFoo.aStaticMethod(); // bad
somethingThatYieldsAFoo().aStaticMethod(); // very bad
{% endhighlight %}
####6.4 终结器（Finalizers）：不要使用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;极少情况下会重写`Object.finalize`。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**贴士：**如果你非要这么做。首先仔细阅读并明白[Effective Java](http://books.google.com/books?isbn=8131726592)第7条，"Avoid Finalizers，"，然后不要这么做。

###Javadoc
####7.1 格式
#####7.1.1 通用格式
Javadoc块的基本格式如下例子所示：
{% highlight java %}
/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String p1) { ... }
{% endhighlight %}
... 或单行的例子
{% highlight java %}
/** An especially short bit of Javadoc. */
{% endhighlight %}
基本格式都是可以接受的，只有在没有（at-分句）或这个Javadoc块可以在一行之内容纳的时候才使用单行格式代替。
####7.1.2 段落
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个空白行，只包含对其的开头的星号——在段落里面出现，如果有"at-clauses"组的话在此之前。除了第一个的所有段落在第一个单词后面使用`<p>`，没有任何空格。
####7.1.3 At-分句
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;任何标准的"at-分句"按照下面的顺序出现`@param`,`@return`,`@throws`,`@deprecated`，这四种类型不要和空白描述一块出现。当at-分句不能用一行容纳下的时候，续行应该从`@`增加4（或更多）的空格的位置开始。
####7.2 summary片段
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一个类或成员的Javadoc都是由一段简短的summary片段开始。这个片段非常重要：它是在特定场景（如类和方法索引）下出现的唯一部分。
片段——一个名词或动词短语，不是一个完整的句子。不是以`A{@code Foo} is a...`或`This method returns...`开头，也不会构成一个完整的动作语句，如`Save the record.`。但是，片段会像一个完整的句子一样大小写和添加标点。
**贴士：** 常犯的错误是使用这样的格式来写简单的Javadoc:`/** @return the customer ID */`，这是不正确的，应该改为`/** Returns the customer ID. */`。
####7.3在哪里使用Javadoc
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最低限度，Javadoc应该包含在每一个`public`类和属于这种类型的每一个`public`或`protected`成员方法，除了下面这些少数例外情况。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其他的类和成员依然根据需要包含Javadoc。当一个实现代码中的评论可以用来定义这个类、方法或成员的大概目的或行为的时候，这个评论就可以代替Javadoc（更加统一和工具友好的）。
#####7.3.1 例外：自我解释的方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于那种“简单，显而易见”的方法（如`getFoo`），Javadoc是可选的，以应对那种真的没什么有价值的内容可以说，除了“Returns the foo”的情况。
**重要：**根据这个例外情况而把普通用户可能需要知道的关联信息省略掉的做法是不合适的。比如，叫`getCanonicalName`的方法，如果一个普通用户可能不清楚单词"canonical name"是什么意思，不要省略其文档（按理说可能只是表达`/** Returns the canonical name. */`）
#####7.3.2 例外：重写的方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于重写父类方法的方法，一般不会总是包含Javadoc。
