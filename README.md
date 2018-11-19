[toc]

## 克隆
### 支持泛型的克隆接口和克隆类
### 解决什么问题

我们知道，JDK中的Cloneable接口只是一个空接口，并没有定义成员，它存在的意义仅仅是指明一个类的实例化对象支持位复制（就是对象克隆），如果不实现这个类，调用对象的clone()方法就会抛出CloneNotSupportedException异常。而且，因为clone()方法在Object对象中，返回值也是Object对象，因此克隆后我们需要自己强转下类型。

### 泛型克隆接口

因此，**com.lenovo.brain.common.clone.Cloneable<T>**接口应运而生。此接口定义了一个返回泛型的成员方法，这样，实现此接口后会提示必须实现一个public的clone方法，调用父类clone方法即可：

```java
/**
 * 猫猫类，使用实现Cloneable方式
 *
 */
private static class Cat implements Cloneable&lt;Cat&gt;{
	private String name = "miaomiao";
	private int age = 2;

	@Override
	public Cat clone() {
		try {
			return (Cat) super.clone();
		} catch (CloneNotSupportedException e) {
			throw new CloneRuntimeException(e);
		}
	}
}
```

### 泛型克隆类

但是实现此接口依旧有不方便之处，就是必须自己实现一个public类型的clone()方法，还要调用父类（Object）的clone方法并处理异常。于是**com.lenovo.brain.common.clone.CloneSupport<T>**类产生，这个类帮我们实现了上面的clone方法，因此只要继承此类，不用写任何代码即可使用clone()方法：

```java
/**
 * 狗狗类，用于继承CloneSupport类
 */
private static class Dog extends CloneSupport&lt;Dog&gt;{
	private String name = "wangwang";
	private int age = 3;
}
```
当然，使用**CloneSupport**的前提是你没有继承任何的类，谁让Java不支持多重继承呢（你依旧可以让父类实继承这个类，如果可以的话）。如果没办法继承类，那实现**com.lenovo.brain.common.clone.Cloneable<T>**也是不错的主意，因此提供了这两种方式，任选其一，在便捷和灵活上都提供了支持。

### 深克隆

我们知道实现Cloneable接口后克隆的对象是浅克隆，要想实现深克隆，请使用：

```java
ObjectUtil.cloneByStream(obj)
```
前提是对象必须实现Serializable接口。

**ObjectUtil**同样提供一些静态方法：**clone(obj)**、**cloneIfPossible(obj)**用于简化克隆调用，详细的说明请查看核心类的相关文档。

## 类型转换
### 类型转换工具类-Convert

在Java开发中我们要面对各种各样的类型转换问题，尤其是从命令行获取的用户参数、从HttpRequest获取的Parameter等等，这些参数类型多种多样，我们怎么去转换他们呢？常用的办法是先整成String，然后调用XXX.parseXXX方法，还要承受转换失败的风险，不得不加一层try catch，这个小小的过程混迹在业务代码中会显得非常难看和臃肿。

### Convert类

**Convert**类可以说是一个工具方法类，里面封装了针对Java常见类型的转换，用于简化类型转换。**Convert**类中大部分方法为toXXX，参数为Object，可以实现将任意可能的类型转换为指定类型。同时支持第二个参数**defaultValue**用于在转换失败时返回一个默认值。

#### Java常见类型转换

1、转换为字符串：

```java
int a = 1;
//aStr为"1"
String aStr = Convert.toStr(a);

long[] b = {1,2,3,4,5};
//bStr为："[1, 2, 3, 4, 5]"
String bStr = Convert.toStr(b);
```

2、转换为指定类型数组：

```java
String[] b = { "1", "2", "3", "4" };
//结果为Integer数组
Integer[] intArray = Convert.toIntArray(b);

long[] c = {1,2,3,4,5};
//结果为Integer数组
Integer[] intArray2 = Convert.toIntArray(c);
```

3、转换为日期对象：

```
String a = "2017-05-06";
Date value = Convert.toDate(a);
```

4、转换为集合
```
Object[] a = {"a", "你", "好", "", 1};
List&lt;?&gt; list = Convert.convert(List.class, a);
//从4.1.11开始可以这么用
List&lt;?&gt; list = Convert.toList(a);
```

#### 其它类型转换

通过`Convert.convert(Class&lt;T&gt;, Object)`方法可以将任意类型转换为指定类型，预定义了许多类型转换，例如转换为URI、URL、Calendar等等，这些类型的转换都依托于`ConverterRegistry`类。通过这个类和`Converter`接口，我们可以自定义一些类型转换。详细的使用请参阅“自定义类型转换”一节。

#### 半角和全角转换
在很多文本的统一化中这两个方法非常有用，主要对标点符号的全角半角转换。

半角转全角：
```java
String a = "123456789";

//结果为："１２３４５６７８９"
String sbc = Convert.toSBC(a);
```

全角转半角：
```java
String a = "１２３４５６７８９";

//结果为"123456789"
String dbc = Convert.toDBC(a);
```

#### 16进制（Hex）
在很多加密解密，以及中文字符串传输（比如表单提交）的时候，会用到16进制转换，就是Hex转换，为此专门封装了**HexUtil**工具类，考虑到16进制转换也是转换的一部分，因此将其方法也放在Convert类中，便于理解和查找，使用同样非常简单：

转为16进制（Hex）字符串
```java
String a = "我是一个小小的可爱的字符串";

//结果："e68891e698afe4b880e4b8aae5b08fe5b08fe79a84e58fafe788b1e79a84e5ad97e7aca6e4b8b2"
String hex = Convert.toHex(a, CharsetUtil.CHARSET_UTF_8);
```

将16进制（Hex）字符串转为普通字符串:
```java
String hex = "e68891e698afe4b880e4b8aae5b08fe5b08fe79a84e58fafe788b1e79a84e5ad97e7aca6e4b8b2";

//结果为："我是一个小小的可爱的字符串"
String raw = Convert.hexStrToStr(hex, CharsetUtil.CHARSET_UTF_8);

//注意：在4.1.11之后hexStrToStr将改名为hexToStr
String raw = Convert.hexToStr(hex, CharsetUtil.CHARSET_UTF_8);
```

&gt; 因为字符串牵涉到编码问题，因此必须传入编码对象，此处使用UTF-8编码。
&gt; **toHex**方法同样支持传入byte[]，同样也可以使用**hexToBytes**方法将16进制转为byte[]

#### Unicode和字符串转换

与16进制类似，Convert类同样可以在字符串和Unicode之间轻松转换：

```java
String a = "我是一个小小的可爱的字符串";

//结果为："\\u6211\\u662f\\u4e00\\u4e2a\\u5c0f\\u5c0f\\u7684\\u53ef\\u7231\\u7684\\u5b57\\u7b26\\u4e32"
String unicode = Convert.strToUnicode(a);

//结果为："我是一个小小的可爱的字符串"
String raw = Convert.unicodeToStr(unicode);
```
很熟悉吧？如果你在properties文件中写过中文，你会明白这个方法的重要性。

#### 编码转换

在接收表单的时候，我们常常被中文乱码所困扰，其实大多数原因是使用了不正确的编码方式解码了数据。于是`Convert.convertCharset`方法便派上用场了，它可以把乱码转为正确的编码方式：

```java
String a = "我不是乱码";
//转换后result为乱码
String result = Convert.convertCharset(a, CharsetUtil.UTF_8, CharsetUtil.ISO_8859_1);
String raw = Convert.convertCharset(result, CharsetUtil.ISO_8859_1, "UTF-8");
Assert.assertEquals(raw, a);
```

&gt; 注意
&gt; 经过测试，UTF-8编码后用GBK解码再用GBK编码后用UTF-8解码会存在某些中文转换失败的问题。

#### 时间单位转换
`Convert.convertTime`方法主要用于转换时长单位，比如一个很大的毫秒，我想获得这个毫秒数对应多少分：
```java
long a = 4535345;

//结果为：75
long minutes = Convert.convertTime(a, TimeUnit.MILLISECONDS, TimeUnit.MINUTES);
```

#### 金额大小写转换
面对财务类需求，`Convert.digitToChinese`将金钱数转换为大写形式：
```java
double a = 67556.32;

//结果为："陆万柒仟伍佰伍拾陆元叁角贰分"
String digitUppercase = Convert.digitToChinese(a);
```
&gt; 注意
&gt; 转换为大写只能精确到分（小数点儿后两位），之后的数字会被忽略。

#### 原始类和包装类转换
有的时候，我们需要将包装类和原始类相互转换（比如Integer.classs 和 int.class），这时候我们可以：
```java
//去包装
Class&lt;?&gt; wrapClass = Integer.class;

//结果为：int.class
Class&lt;?&gt; unWraped = Convert.unWrap(wrapClass);

//包装
Class&lt;?&gt; primitiveClass = long.class;

//结果为：Long.class
Class&lt;?&gt; wraped = Convert.wrap(primitiveClass);
```
### 自定义类型转换-ConverterRegistry
#### 由来
类型转换最早只是一个工具类，叫做“Conver”，对于每一种类型转换都是用一个静态方法表示，但是这种方式有一个潜在问题，那就是扩展性不足，这导致只能满足部分类型转换的需求。

#### 解决
为了解决这些问题，对这个类做了扩展。思想如下：

- `Converter` 类型转换接口，通过实现这个接口，重写convert方法，以实现不同类型的对象转换
- `ConverterRegistry` 类型转换登记中心。将各种类型Convert对象放入登记中心，通过`convert`方法查找目标类型对应的转换器，将被转换对象转换之。在此类中，存放着**默认转换器**和**自定义转换器**，默认转换器是预定义的一些转换器，自定义转换器存放用户自定的转换器。

通过这种方式，实现类灵活的类型转换。使用方式如下：
```java
int a = 3423;
ConverterRegistry converterRegistry = ConverterRegistry.getInstance();
String result = converterRegistry.convert(String.class, a);
Assert.assertEquals("3423", result);
```
#### 自定义转换
默认转换有时候并不能满足我们自定义对象的一些需求，这时我们可以使用`ConverterRegistry.getInstance().putCustom()`方法自定义类型转换。

1. 自定义转换器

```java
public static class CustomConverter implements Converter&lt;String&gt;{
	@Override
	public String convert(Object value, String defaultValue) throws IllegalArgumentException {
		return "Custom: " + value.toString();
	}
}
```
2. 注册转换器

```java
ConverterRegistry converterRegistry = ConverterRegistry.getInstance();
//此处做为示例自定义String转换，因为已经提供String转换，请尽量不要替换
//替换可能引发关联转换异常（例如覆盖String转换会影响全局）
converterRegistry.putCustom(String.class, CustomConverter.class);
```

3. 执行转换

```java
int a = 454553;
String result = converterRegistry.convert(String.class, a);
Assert.assertEquals("Custom: 454553", result);
```

&gt; 注意：
&gt; convert(Class&lt;T&gt; type, Object value, T defaultValue, boolean isCustomFirst)方法的最后一个参数可以选择转换时优先使用自定义转换器还是默认转换器。convert(Class&lt;T&gt; type, Object value, T defaultValue)和convert(Class&lt;T&gt; type, Object value)两个重载方法都是使用自定义转换器优先的模式。

#### `ConverterRegistry`单例和对象模式
ConverterRegistry提供一个静态方法getInstance()返回全局单例对象，这也是推荐的使用方式，当然如果想在某个限定范围内自定义转换，可以实例化ConverterRegistry对象。

## 日期时间

日期时间包是核心包之一，提供针对JDK中Date和Calendar对象的封装，封装对象如下：

### 日期时间工具

- `DateUtil` 针对日期时间操作提供一系列静态方法
- `DateTime` 提供类似于Joda-Time中日期时间对象的封装，继承自Date类，并提供更加丰富的对象方法。
- `FastDateFormat` 提供线程安全的针对Date对象的格式化和日期字符串解析支持。此对象在实际使用中并不需要感知，相关操作已经封装在`DateUtil`和`DateTime`的相关方法中。
- `DateBetween` 计算两个时间间隔的类，除了通过构造新对象使用外，相关操作也已封装在`DateUtil`和`DateTime`的相关方法中。
- `TimeInterval` 一个简单的计时器类，常用于计算某段代码的执行时间，提供包括毫秒、秒、分、时、天、周等各种单位的花费时长计算，对象的静态构造已封装在`DateUtil`中。
- `DatePattern` 提供常用的日期格式化模式，包括`String`类型和`FastDateFormat`两种类型。

### 日期枚举

考虑到`Calendar`类中表示时间的字段（field）都是使用`int`表示，在使用中非常不便，因此针对这些`int`字段，封装了与之对应的Enum枚举类，这些枚举类在`DateUtil`和`DateTime`相关方法中做为参数使用，可以更大限度的缩小参数限定范围。

这些定义的枚举值可以通过`getValue()`方法获得其与`Calendar`类对应的int值，通过`of(int)`方法从`Calendar`中int值转为枚举对象。

与`Calendar`对应的这些枚举包括：

- `Month` 表示月份，与Calendar中的int值一一对应。
- `Week` 表示周，与Calendar中的int值一一对应

另外，还定义了**季度**枚举。`Season.SPRING`为第一季度，表示1~3月。季度的概念并不等同于季节，因为季节与月份并不对应，季度常用于统计概念。

### 时间枚举

时间枚举`DateUnit`主要表示某个时间单位对应的毫秒数，常用于计算时间差。

例如：`DateUnit.MINUTE`表示分，也表示一分钟的毫米数，可以通过调用其`getMillis()`方法获得其毫秒数。

### 日期时间工具-DateUtil
#### 由来
考虑到Java本身对日期时间的支持有限，并且Date和Calendar对象的并存导致各种方法使用混乱和复杂，故使用此工具类做了封装。这其中的封装主要是日期和字符串之间的转换，以及提供对日期的定位（一个月前等等）。

对于Date对象，为了便捷，使用了一个DateTime类来代替之，继承自Date对象，主要的便利在于，覆盖了toString()方法，返回yyyy-MM-dd HH:mm:ss形式的字符串，方便在输出时的调用（例如日志记录等），提供了众多便捷的方法对日期对象操作，关于DateTime会在相关章节介绍。

#### Date、long、Calendar之间的相互转换

```java
//当前时间
Date date = DateUtil.date();
//当前时间
Date date2 = DateUtil.date(Calendar.getInstance());
//当前时间
Date date3 = DateUtil.date(System.currentTimeMillis());
//当前时间字符串，格式：yyyy-MM-dd HH:mm:ss
String now = DateUtil.now();
//当前日期字符串，格式：yyyy-MM-dd
String today= DateUtil.today();
```

#### 字符串转日期

`DateUtil.parse`方法会自动识别一些常用格式，包括：
1. yyyy-MM-dd HH:mm:ss
2. yyyy-MM-dd
3. HH:mm:ss
4. yyyy-MM-dd HH:mm
5. yyyy-MM-dd HH:mm:ss.SSS

```java
String dateStr = "2017-03-01";
Date date = DateUtil.parse(dateStr);
```

我们也可以使用自定义日期格式转化：
```java
String dateStr = "2017-03-01";
Date date = DateUtil.parse(dateStr, "yyyy-MM-dd");
```

#### 格式化日期输出

```java
String dateStr = "2017-03-01";
Date date = DateUtil.parse(dateStr);

//结果 2017/03/01
String format = DateUtil.format(date, "yyyy/MM/dd");

//常用格式的格式化，结果：2017-03-01
String formatDate = DateUtil.formatDate(date);

//结果：2017-03-01 00:00:00
String formatDateTime = DateUtil.formatDateTime(date);

//结果：00:00:00
String formatTime = DateUtil.formatTime(date);
```

#### 获取Date对象的某个部分
```
Date date = DateUtil.date();
//获得年的部分
DateUtil.year(date);
//获得月份，从0开始计数
DateUtil.month(date);
//获得月份枚举
DateUtil.monthEnum(date);
//.....
```

#### 开始和结束时间
有的时候我们需要获得每天的开始时间、结束时间，每月的开始和结束时间等等，DateUtil也提供了相关方法：

```java
String dateStr = "2017-03-01 22:33:23";
Date date = DateUtil.parse(dateStr);

//一天的开始，结果：2017-03-01 00:00:00
Date beginOfDay = DateUtil.beginOfDay(date);

//一天的结束，结果：2017-03-01 23:59:59
Date endOfDay = DateUtil.endOfDay(date);
```

#### 日期时间偏移
日期或时间的偏移指针对某个日期增加或减少分、小时、天等等，达到日期变更的目的, 针对其做了大量封装

```java
String dateStr = "2017-03-01 22:33:23";
Date date = DateUtil.parse(dateStr);

//结果：2017-03-03 22:33:23
Date newDate = DateUtil.offset(date, DateField.DAY_OF_MONTH, 2);

//常用偏移，结果：2017-03-04 22:33:23
DateTime newDate2 = DateUtil.offsetDay(date, 3);

//常用偏移，结果：2017-03-01 19:33:23
DateTime newDate3 = DateUtil.offsetHour(date, -3);
```

针对当前时间，提供了简化的偏移方法（例如昨天、上周、上个月等）：
```java
//昨天
DateUtil.yesterday()
//明天
DateUtil.tomorrow()
//上周
DateUtil.lastWeek()
//下周
DateUtil.nextWeek()
//上个月
DateUtil.lastMonth()
//下个月
DateUtil.nextMonth()
```

#### 日期时间差
有时候我们需要计算两个日期之间的时间差（相差天数、相差小时数等等），此类方法封装为between方法：

```java
String dateStr1 = "2017-03-01 22:33:23";
Date date1 = DateUtil.parse(dateStr1);

String dateStr2 = "2017-04-01 23:33:23";
Date date2 = DateUtil.parse(dateStr2);

//相差一个月，31天
long betweenDay = DateUtil.between(date1, date2, DateUnit.DAY);
```

#### 格式化时间差
有时候我们希望看到易读的时间差，比如XX天XX小时XX分XX秒，此时使用`DateUtil.formatBetween`方法：

```java
//Level.MINUTE表示精确到分
String formatBetween = DateUtil.formatBetween(between, Level.MINUTE);
//输出：31天1小时
Console.log(formatBetween);
```

#### 计时器
计时器用于计算某段代码或过程花费的时间

```java
TimeInterval timer = DateUtil.timer();

//---------------------------------
//-------这是执行过程
//---------------------------------

timer.interval();//花费毫秒数
timer.intervalRestart();//返回花费时间，并重置开始时间
timer.intervalMinute();//花费分钟数
```

#### 其它
```java
//年龄
DateUtil.ageOfNow("1990-01-30");

//是否闰年
DateUtil.isLeapYear(2017);
```
### 日期时间对象-DateTime
#### 由来

考虑工具类的局限性，在某些情况下使用并不简便，于是`DateTime`类诞生。`DateTime`对象充分吸取Joda-Time库的优点，并提供更多的便捷方法，这样我们在开发时不必再单独导入Joda-Time库便可以享受简单快速的日期时间处理过程。

#### 说明
**DateTime**类继承于java.util.Date类，为Date类扩展了众多简便方法，这些方法多是`DateUtil`静态方法的对象表现形式，使用DateTime对象可以完全替代开发中Date对象的使用。

#### 新建对象
`DateTime`对象包含众多的构造方法，构造方法支持的参数有：
- Date
- Calendar
- String(日期字符串，第二个参数是日期格式)
- long 毫秒数

构建对象有两种方式：`DateTime.of()`和`new DateTime()`：

```java
Date date = new Date();

//new方式创建
DateTime time = new DateTime(date);
Console.log(time);

//of方式创建
DateTime now = DateTime.now();
DateTime dt = DateTime.of(date);
```

#### 使用对象
`DateTime`的成员方法与`DateUtil`中的静态方法所对应，因为是成员方法，因此可以使用更少的参数操作日期时间。

示例：获取日期成员（年、月、日等）

```java
DateTime dateTime = new DateTime("2017-01-05 12:34:23", DatePattern.NORM_DATETIME_FORMAT);

//年，结果：2017
int year = dateTime.year();

//季度（非季节），结果：Season.SPRING
Season season = dateTime.seasonEnum();

//月份，结果：Month.JANUARY
Month month = dateTime.monthEnum();

//日，结果：5
int day = dateTime.dayOfMonth();
```
更多成员方法请参阅API文档。


#### 对象的可变性
DateTime对象默认是可变对象（调用offset、setField、setTime方法默认变更自身），但是这种可变性有时候会引起很多问题（例如多个地方共用DateTime对象）。我们可以调用`setMutable(false)`方法使其变为不可变对象。在不可变模式下，`offset`、`setField`方法返回一个新对象，`setTime`方法抛出异常。

```java
DateTime dateTime = new DateTime("2017-01-05 12:34:23", DatePattern.NORM_DATETIME_FORMAT);

//默认情况下DateTime为可变对象，此时offsite == dateTime
DateTime offsite = dateTime.offsite(DateField.YEAR, 0);

//设置为不可变对象后变动将返回新对象，此时offsite != dateTime
dateTime.setMutable(false);
offsite = dateTime.offsite(DateField.YEAR, 0);
```

#### 格式化为字符串
调用`toString()`方法即可返回格式为`yyyy-MM-dd HH:mm:ss`的字符串，调用`toString(String format)`可以返回指定格式的字符串。

```java
DateTime dateTime = new DateTime("2017-01-05 12:34:23", DatePattern.NORM_DATETIME_FORMAT);
//结果：2017-01-05 12:34:23
String dateStr = dateTime.toString();

//结果：2017/01/05
```

## IO
### 概述
#### 由来
IO的操作包括**读**和**写**，应用场景包括网络操作和文件操作。IO操作在Java中是一个较为复杂的过程，我们在面对不同的场景时，要选择不同的`InputStream`和`OutputStream`实现来完成这些操作。而如果想读写字节流，还需要`Reader`和`Writer`的各种实现类。这些繁杂的实现类，一方面给我我们提供了更多的灵活性，另一方面也增加了复杂性。

#### 封装
io包的封装主要针对流、文件的读写封装，主要以工具类为主，提供常用功能的封装，这包括：

- `IoUtil` 流操作工具类
- `FileUtil` 文件读写和操作的工具类。
- `FileTypeUtil` 文件类型判断工具类
- `WatchMonitor` 目录、文件监听，封装了JDK1.7中的WatchService
- `ClassPathResource`针对ClassPath中资源的访问封装
- `FileReader` 封装文件读取
- `FileWriter` 封装文件写入

#### 流扩展
除了针对JDK的读写封装外，还针对特定环境和文件扩展了流实现。

包括：
- `BOMInputStream`针对含有BOM头的流读取
- `FastByteArrayOutputStream` 基于快速缓冲FastByteBuffer的OutputStream，随着数据的增长自动扩充缓冲区（from blade）
- `FastByteBuffer` 快速缓冲，将数据存放在缓冲集中，取代以往的单一数组（from blade）
### IO工具类-IoUtil
#### 由来

IO工具类的存在主要针对InputStream、OutputStream、Reader、Writer封装简化，并对NIO相关操作做封装简化。总体来说，对IO的封装，主要是工具层面，我们努力做到在便捷、性能和灵活之间找到最好的平衡点。

#### 拷贝
流的读写可以总结为从输入流读取，从输出流写出，这个过程我们定义为**拷贝**。这个是一个基本过程，也是文件、流操作的基础。

以文件流拷贝为例：
```java
BufferedInputStream in = FileUtil.getInputStream("d:/test.txt");
BufferedOutputStream out = FileUtil.getOutputStream("d:/test2.txt");
long copySize = IoUtil.copy(in, out, IoUtil.DEFAULT_BUFFER_SIZE);
```

copy方法同样针对Reader、Writer、Channel等对象有一些重载方法，并提供可选的缓存大小。默认的，缓存大小为`1024`个字节，如果拷贝大文件或流数据较大，可以适当调整这个参数。

针对NIO，提供了`copyByNIO`方法，以便和BIO有所区别。我查阅过一些资料，使用NIO对文件流的操作有一定的提升，我并没有做具体实验。相关测试请参阅博客：[http://www.cnblogs.com/gaopeng527/p/4896783.html](http://www.cnblogs.com/gaopeng527/p/4896783.html)

#### Stream转Reader、Writer
- `IoUtil.getReader`：将`InputStream`转为`BufferedReader`用于读取字符流，它是部分readXXX方法的基础。
- `IoUtil.getWriter`：将`OutputStream`转为`OutputStreamWriter`用于写入字符流，它是部分writeXXX的基础。

本质上这两个方法只是简单new一个新的Reader或者Writer对象，但是封装为工具方法配合IDE的自动提示可以大大减少查阅次数（例如你对BufferedReader、OutputStreamWriter不熟悉，是不需要搜索一下相关类？）

#### 读取流中的内容
读取流中的内容总结下来，可以分为read方法和readXXX方法。

1. `read`方法有诸多的重载方法，根据参数不同，可以读取不同对象中的内容，这包括：
 - `InputStream`
 - `Reader`
 - `FileChannel`

这三个重载大部分返回String字符串，为字符流读取提供极大便利。

2. `readXXX`方法主要针对返回值做一些处理，例如：
 - `readBytes` 返回byte数组（读取图片等）
 - `readHex` 读取16进制字符串
 - `readObj` 读取序列化对象（反序列化）
 - `readLines` 按行读取

3. `toStream`方法则是将某些对象转换为流对象，便于在某些情况下操作：
 - `String` 转换为`ByteArrayInputStream`
 - `File` 转换为`FileInputStream`

#### 写入到流
- `IoUtil.write`方法有两个重载方法，一个直接调用`OutputStream.write`方法，另一个用于将对象转换为字符串（调用toString方法），然后写入到流中。
- `IoUtil.writeObjects` 用于将可序列化对象序列化后写入到流中。

`write`方法并没有提供writeXXX，需要自己转换为String或byte[]。

#### 关闭
对于IO操作来说，使用频率最高（也是最容易被遗忘）的就是`close`操作，好在Java规范使用了优雅的`Closeable`接口，这样我们只需简单封装调用此接口的方法即可。

关闭操作会面临两个问题：
1. 被关闭对象为空
2. 对象关闭失败（或对象已关闭）

`IoUtil.close`方法很好的解决了这两个问题。

在JDK1.7中，提供了`AutoCloseable`接口，在`IoUtil`中同样提供相应的重载方法，在使用中并不能感觉到有哪些不同。

### 文件工具类-FileUtil
#### 简介
在IO操作中，文件的操作相对来说是比较复杂的，但也是使用频率最高的部分，我们几乎所有的项目中几乎都躺着一个叫做FileUtil或者FileUtils的工具类，应该将这个工具类纳入其中，解决用来解决大部分的文件操作问题。

总体来说，FileUtil类包含以下几类操作工具：
1. 文件操作：包括文件目录的新建、删除、复制、移动、改名等
2. 文件判断：判断文件或目录是否非空，是否为目录，是否为文件等等。
3. 绝对路径：针对ClassPath中的文件转换为绝对路径文件。
4. 文件名：主文件名，扩展名的获取
5. 读操作：包括类似IoUtil中的getReader、readXXX操作
6. 写操作：包括getWriter和writeXXX操作

在FileUtil中，我努力将方法名与Linux相一致，例如创建文件的方法并不是createFile，而是`touch`，这种统一对于熟悉Linux的人来说，大大提高了上手速度。当然，如果你不熟悉Linux，那FileUtil工具类的使用则是在帮助你学习Linux命令。这些类Linux命令的方法包括：

- `ls` 列出目录和文件
- `touch` 创建文件，如果父目录不存在也自动创建
- `mkdir` 创建目录，会递归创建每层目录
- `del` 删除文件或目录（递归删除，不判断是否为空），这个方法相当于Linux的delete命令
- `copy` 拷贝文件或目录

这些方法提供了人性化的操作，例如`touch`方法，在创建文件的情况下会自动创建上层目录（我想对于使用者来说这也是大部分情况下的需求），同样`mkdir`也会创建父目录。

&gt; 需要注意的是，`del`方法会删除目录而不判断其是否为空，这一方面方便了使用，另一方面也可能造成一些预想不到的后果（比如拼写错路径而删除不应该删除的目录），所以请谨慎使用此方法。

关于FileUtil中更多工具方法，请参阅API文档。
### 文件类型判断-FileTypeUtil
#### 由来

在文件上传时，有时候我们需要判断文件类型。但是又不能简单的通过扩展名来判断（防止恶意脚本等通过上传到服务器上），于是我们需要在服务端通过读取文件的首部几个二进制位来判断常用的文件类型。

#### 使用
这个工具类使用非常简单，通过调用`FileTypeUtil.getType`即可判断，这个方法同时提供众多的重载方法，用于读取不同的文件和流。

```java
File file = FileUtil.file("d:/test.jpg");
String type = FileTypeUtil.getType(file);
//输出 jpg则说明确实为jpg文件
Console.log(type);
```

#### 原理和局限性
这个类是通过读取文件流中前N个byte值来判断文件类型，在类中我们通过Map形式将常用的文件类型做了映射，这些映射都是网络上搜集而来。也就是说，我们只能识别有限的几种文件类型。但是这些类型已经涵盖了常用的图片、音频、视频、Office文档类型，可以应对大部分的使用场景。

&gt; 对于某些文本格式的文件我们并不能通过首部byte判断其类型，比如`JSON`，这类文件本质上是文本文件，我们应该读取其文本内容，通过其语法判断类型。

#### 自定义类型
为了提高`FileTypeUtil`的扩展性，我们通过`putFileType`方法可以自定义文件类型。

```java
FileTypeUtil.putFileType("ffd8ffe000104a464946", "new_jpg");
```

第一个参数是文件流的前N个byte的16进制表示，我们可以读取自定义文件查看，选取一定长度即可(长度越长越精确)，第二个参数就是文件类型，然后使用`FileTypeUtil.getType`即可。

### 文件监听-WatchMonitor
#### 由来

很多时候我们需要监听一个文件的变化或者目录的变动，包括文件的创建、修改、删除，以及目录下文件的创建、修改和删除，在JDK7前我们只能靠轮询方式遍历目录或者定时检查文件的修改事件，这样效率非常低，性能也很差。因此在JDK7中引入了`WatchService`。不过考虑到其API并不友好，于是针对其做了简化封装，使监听更简单，也提供了更好的功能，这包括：

- 支持多级目录的监听（WatchService只支持一级目录），可自定义监听目录深度
- 延迟合并触发支持（文件变动时可能触发多次modify，支持在某个时间范围内的多次修改事件合并为一个修改事件）
- 简洁易懂的API方法，一个方法即可搞定监听，无需理解复杂的监听注册机制。
- 多观察者实现，可以根据业务实现多个`Watcher`来响应同一个事件（通过WatcherChain）

#### WatchMonitor

`WatchMonitor`主要针对JDK7中`WatchService`做了封装，针对文件和目录的变动（创建、更新、删除）做一个钩子，在`Watcher`中定义相应的逻辑来应对这些文件的变化。

#### 内部应用
在setting模块，使用WatchMonitor监测配置文件变化，然后自动load到内存中。WatchMonitor的使用可以避免轮询，以事件响应的方式应对文件变化。

#### 使用

`WatchMonitor`提供的事件有：

- `ENTRY_MODIFY` 文件修改的事件
- `ENTRY_CREATE` 文件或目录创建的事件
- `ENTRY_DELETE` 文件或目录删除的事件
- `OVERFLOW` 丢失的事件

这些事件对应`StandardWatchEventKinds`中的事件。

下面我们介绍WatchMonitor的使用：

#### 监听指定事件

```java
File file = FileUtil.file("example.properties");
//这里只监听文件或目录的修改事件
WatchMonitor watchMonitor = WatchMonitor.create(file, WatchMonitor.ENTRY_MODIFY);
watchMonitor.setWatcher(new Watcher(){
	@Override
	public void onCreate(WatchEvent&lt;?&gt; event, Path currentPath) {
		Object obj = event.context();
		Console.log("创建：{}-&gt; {}", currentPath, obj);
	}

	@Override
	public void onModify(WatchEvent&lt;?&gt; event, Path currentPath) {
		Object obj = event.context();
		Console.log("修改：{}-&gt; {}", currentPath, obj);
	}

	@Override
	public void onDelete(WatchEvent&lt;?&gt; event, Path currentPath) {
		Object obj = event.context();
		Console.log("删除：{}-&gt; {}", currentPath, obj);
	}

	@Override
	public void onOverflow(WatchEvent&lt;?&gt; event, Path currentPath) {
		Object obj = event.context();
		Console.log("Overflow：{}-&gt; {}", currentPath, obj);
	}
});

//设置监听目录的最大深入，目录层级大于制定层级的变更将不被监听，默认只监听当前层级目录
watchMonitor.setMaxDepth(3);
//启动监听
watchMonitor.start();
```

#### 监听全部事件

其实我们不必实现`Watcher`的所有接口方法，同时提供了`SimpleWatcher`类，只需重写对应方法即可。

同样，如果我们想监听所有事件，可以：
```java
WatchMonitor.createAll(file, new SimpleWatcher(){
	@Override
	public void onModify(WatchEvent&lt;?&gt; event, Path currentPath) {
		Console.log("EVENT modify");
	}
}).start();
```

`createAll`方法会创建一个监听所有事件的WatchMonitor，同时在第二个参数中定义Watcher来负责处理这些变动。

#### 延迟处理监听事件

在监听目录或文件时，如果这个文件有修改操作，JDK会多次触发modify方法，为了解决这个问题，我们定义了`DelayWatcher`，此类通过维护一个Set将短时间内相同文件多次modify的事件合并处理触发，从而避免以上问题。

```java
WatchMonitor monitor = WatchMonitor.createAll("d:/", new DelayWatcher(watcher, 500));
monitor.start();
```
### ClassPath资源访问-ClassPathResource
#### 什么是ClassPath
简单说来ClassPath就是查找class文件的路径，在Tomcat等容器下，ClassPath一般是`WEB-INF/classes`，在普通java程序中，我们可以通过定义`-cp`或者`-classpath`参数来定义查找class文件的路径，这些路径就是ClassPath。

为了项目方便，我们定义的配置文件肯定不能使用绝对路径，所以需要使用相对路径，这时候最好的办法就是把配置文件和class文件放在一起，便于查找。

#### 由来
在Java编码过程中，我们常常希望读取项目内的配置文件，按照Maven的习惯，这些文件一般放在项目的`src/main/resources`下，读取的时候使用：

```java
String path = "config.properties";
InputStream in = this.class.getResource(path).openStream();
```

使用当前类来获得资源其实就是使用当前类的类加载器获取资源，最后openStream()方法获取输入流来读取文件流。

#### 封装
面对这种复杂的读取操作，我们封装了`ClassPathResource`类来简化这种资源的读取：

```java
ClassPathResource resource = new ClassPathResource("test.properties");
Properties properties = new Properties();
properties.load(resource.getStream());

Console.log("Properties: {}", properties);
```

这样就大大简化了ClassPath中资源的读取。提供针对properties的封装类`Props`，同时提供更加强大的配置文件Setting类，这两个类已经针对ClassPath做过相应封装，可以以更加便捷的方式读取配置文件。相关文档请参阅setting章节

### 文件读取-FileReader
#### 由来
在`FileUtil`中本来已经针对文件的读操作做了大量的静态封装，但是根据职责分离原则，我觉得有必要针对文件读取单独封装一个类，这样项目更加清晰。当然，使用FileUtil操作文件是最方便的。

#### 使用
在JDK中，同样有一个FileReader类，但是并不如想象中的那样好用，便提供了更加便捷FileReader类。

```java
//默认UTF-8编码，可以在构造中传入第二个参数做为编码
FileReader fileReader = new FileReader("test.properties");
String result = fileReader.readString();
```

FileReader提供了以下方法来快速读取文件内容：

- `readBytes`
- `readString`
- `readLines`

同时，此类还提供了以下方法用于转换为流或者BufferedReader：
- `getReader`
- `getInputStream`

### 文件写入-FileWriter

相应的，文件读取有了，自然有文件写入类，使用方式与`FileReader`也类似：

```java
FileWriter writer = new FileWriter("test.properties");
writer.write("test");
```

写入文件分为追加模式和覆盖模式两类，追加模式可以用`append`方法，覆盖模式可以用`write`方法，同时也提供了一个write方法，第二个参数是可选覆盖模式。

同样，此类提供了：
- `getOutputStream`
- `getWriter`
- `getPrintWriter`

这些方法用于转换为相应的类提供更加灵活的写入操作。

## 工具类
### 概述
此包中的工具类为未经过分类的一些工具类，提供一些常用的工具方法。

此包中根据用途归类为XXXUtil，提供大量的工具方法。在工具类中，主要以类方法（static方法）为主，且各个类无法实例化为对象，一个方法是一个独立功能，无相互影响。

关于工具类的说明和使用，请参阅下面的章节。

### 数组工具-ArrayUtil
#### 介绍
数组工具中的方法在2.x版本中都在CollectionUtil中存在，3.x版本中拆分出来作为ArrayUtil。数组工具类主要针对原始类型数组和泛型数组相关方案进行封装。

数组工具类主要是解决对象数组（包括包装类型数组）和原始类型数组使用方法不统一的问题。

#### 判空
数组的判空类似于字符串的判空，标准是`null`或者数组长度为0，ArrayUtil中封装了针对原始类型和泛型数组的判空和判非空：

1. 判断空
```java
int[] a = {};
int[] b = null;
ArrayUtil.isEmpty(a);
ArrayUtil.isEmpty(b);
```

2. 判断非空
```java
int[] a = {1,2};
ArrayUtil.isNotEmpty(a);
```

#### 新建泛型数组
`Array.newInstance`并不支持泛型返回值，在此封装此方法使之支持泛型返回值。

```java
String[] newArray = ArrayUtil.newArray(String.class, 3);
```

#### 调整大小
使用 `ArrayUtil.resize`方法生成一个新的重新设置大小的数组。

#### 合并数组
`ArrayUtil.addAll`方法采用可变参数方式，将多个泛型数组合并为一个数组。

#### 克隆
数组本身支持clone方法，因此确定为某种类型数组时调用`ArrayUtil.clone(T[])`,不确定类型的使用`ArrayUtil.clone(T)`，两种重载方法在实现上有所不同，但是在使用中并不能感知出差别。

1. 泛型数组调用原生克隆
```java
Integer[] b = {1,2,3};
Integer[] cloneB = ArrayUtil.clone(b);
Assert.assertArrayEquals(b, cloneB);
```
2. 非泛型数组（原始类型数组）调用第二种重载方法
```java
int[] a = {1,2,3};
int[] clone = ArrayUtil.clone(a);
Assert.assertArrayEquals(a, clone);
```

#### 有序列表生成
`ArrayUtil.range`方法有三个重载，这三个重载配合可以实现支持步进的有序数组或者步进为1的有序数组。这种列表生成器在Python中做为语法糖存在。

#### 拆分数组
`ArrayUtil.split`方法用于拆分一个byte数组，将byte数组平均分成几等份，常用于消息拆分。

#### 过滤
`ArrayUtil.filter`方法用于编辑已有数组元素，只针对泛型数组操作，原始类型数组并未提供。
方法中Editor接口用于返回每个元素编辑后的值，返回null此元素将被抛弃。

一个大栗子：过滤数组，只保留偶数
```java
Integer[] a = {1,2,3,4,5,6};
Integer[] filter = ArrayUtil.filter(a, new Editor&lt;Integer&gt;(){
	@Override
	public Integer edit(Integer t) {
		return (t % 2 == 0) ? t : null;
	}});
Assert.assertArrayEquals(filter, new Integer[]{2,4,6});
```

#### zip
`ArrayUtil.zip`方法传入两个数组，第一个数组为key，第二个数组对应位置为value，此方法在Python中为zip()函数。

```java
String[] keys = {"a", "b", "c"};
Integer[] values = {1,2,3};
Map&lt;String, Integer&gt; map = ArrayUtil.zip(keys, values, true);

//{a=1, b=2, c=3}
```

#### 是否包含元素
`ArrayUtil.contains`方法只针对泛型数组，检测指定元素是否在数组中。

#### 包装和拆包
在原始类型元素和包装类型中，Java实现了自动包装和拆包，但是相应的数组无法实现，于是便是用`ArrayUtil.wrap`和`ArrayUtil.unwrap`对原始类型数组和包装类型数组进行转换。

#### 判断对象是否为数组
`ArrayUtil.isArray`方法封装了`obj.getClass().isArray()`。

#### 转为字符串

1. `ArrayUtil.toString` 通常原始类型的数组输出为字符串时无法正常显示，于是封装此方法可以完美兼容原始类型数组和包装类型数组的转为字符串操作。

2. `ArrayUtil.join` 方法使用间隔符将一个数组转为字符串，比如[1,2,3,4]这个数组转为字符串，间隔符使用“-”的话，结果为 1-2-3-4，join方法同样支持泛型数组和原始类型数组。

#### toArray
`ArrayUtil.toArray`方法针对ByteBuffer转数组提供便利。

### 字符编码工具-CharsetUtil
#### 介绍
CharsetUtil主要针对编码操作做了工具化封装，同时提供了一些常用编码常量。

#### 常量
常量在需要编码的地方直接引用，可以很好的提高便利性。

#### 字符串形式
1. ISO_8859_1
2. UTF_8
3. GBK

#### Charset对象形式
1. CHARSET_ISO_8859_1
2. CHARSET_UTF_8
3. CHARSET_GBK

#### 编码字符串转为Charset对象
`CharsetUtil.charset`方法用于将编码形式字符串转为Charset对象。

#### 转换编码
`CharsetUtil.convert`方法主要是在两种编码中转换。主要针对因为编码识别错误而导致的乱码问题的一种解决方法。

#### 系统默认编码
`CharsetUtil.defaultCharset`方法是`Charset.defaultCharset()`的封装方法。返回系统编码。
`CharsetUtil.defaultCharsetName`方法返回字符串形式的编码类型。

### 类工具-ClassUtil
这个工具主要是封装了一些反射的方法，使调用更加方便。而这个类中最有用的方法是`scanPackage`方法，这个方法会扫描classpath下所有类，这个在Spring中是特性之一，框架中类扫描的一个基础。下面介绍下这个类中的方法。
#### `getShortClassName`
获取完整类名的短格式如：`com.lenovo.brain.common.util.StrUtil` -&gt; `c.h.c.u.StrUtil`

#### `isAllAssignableFrom`
比较判断types1和types2两组类，如果types1中所有的类都与types2对应位置的类相同，或者是其父类或接口，则返回true

#### `isPrimitiveWrapper`
是否为包装类型

#### `isBasicType`
是否为基本类型（包括包装类和原始类）

#### `getPackage`
获得给定类所在包的名称，例如：
`com.lenovo.brain.common.util.ClassUtil` -&gt; `com.lenovo.brain.common.util`

#### `scanPackage`方法
此方法唯一的参数是包的名称，返回结果为此包以及子包下所有的类。方法使用很简单，但是过程复杂一些，包扫面首先会调用 `getClassPaths`方法获得ClassPath，然后扫描ClassPath，如果是目录，扫描目录下的类文件，或者jar文件。如果是jar包，则直接从jar包中获取类名。这个方法的作用显而易见，就是要找出所有的类，在Spring中用于依赖注入，。当然，你也可以传一个`ClassFilter`对象，用于过滤不需要的类。

#### `getClassPaths`方法
此方法是获得当前线程的ClassPath，核心是`Thread.currentThread().getContextClassLoader().getResources`的调用。

#### `getJavaClassPaths`方法
此方法用于获得java的系统变量定义的ClassPath。

#### `getClassLoader`和`getContextClassLoader`方法
后者只是获得当前线程的ClassLoader，前者在获取失败的时候获取`ClassUtil`这个类的ClassLoader。

#### `getDefaultValue`
获取指定类型分的默认值，默认值规则为：
1. 如果为原始类型，返回0
2. 非原始类型返回null

### 类加载工具-ClassLoaderUtil
#### 介绍
提供ClassLoader相关的工具类，例如类加载（Class.forName包装）等

#### 获取ClassLoader

##### `getContextClassLoader`

获取当前线程的ClassLoader，本质上调用`Thread.currentThread().getContextClassLoader()`

##### `getClassLoader`

按照以下顺序规则查找获取ClassLoader：

1. 获取当前线程的ContextClassLoader
2. 获取ClassLoaderUtil类对应的ClassLoader
3. 获取系统ClassLoader（ClassLoader.getSystemClassLoader()）

#### 加载Class

##### `loadClass`

加载类，通过传入类的字符串，返回其对应的类名，使用默认ClassLoader并初始化类（调用static模块内容和可选的初始化static属性）

扩展`Class.forName`方法，支持以下几类类名的加载：

1. 原始类型，例如：int
2. 数组类型，例如：int[]、Long[]、String[]
3. 内部类，例如：java.lang.Thread.State会被转为java.lang.Thread$State加载

同时提供`loadPrimitiveClass`方法用于快速加载原始类型的类。包括原始类型、原始类型数组和void

##### `isPresent`

指定类是否被提供，通过调用`loadClass`方法尝试加载指定类名的类，如果加载失败返回false。加载失败的原因可能是此类不存在或其关联引用类不存在。

### Escape工具-EscapeUtil
#### 介绍

转义和反转义工具类Escape / Unescape。escape采用ISO Latin字符集对指定的字符串进行编码。所有的空格符、标点符号、特殊字符以及其他非ASCII字符都将被转化成%xx格式的字符编码(xx等于该字符在字符集表里面的编码的16进制数字)。

此类中的方法对应Javascript中的`escape()`函数和`unescape()`函数。

#### 方法
1. `EscapeUtil.escape` Escape编码（Unicode），该方法不会对 ASCII 字母和数字进行编码，也不会对下面这些 ASCII 标点符号进行编码： * @ - _ + . / 。其他所有的字符都会被转义序列替换。
2. `EscapeUtil.unescape` Escape解码。
3. `EscapeUtil.safeUnescape` 安全的unescape文本，当文本不是被escape的时候，返回原文。

### 16进制工具-HexUtil

#### 介绍
十六进制（简写为hex或下标16）在数学中是一种逢16进1的进位制，一般用数字0到9和字母A到F表示（其中:A~F即10~15）。例如十进制数57，在二进制写作111001，在16进制写作39。

像java,c这样的语言为了区分十六进制和十进制数值,会在十六进制数的前面加上 0x,比如0x20是十进制的32,而不是十进制的20。`HexUtil`就是将字符串或byte数组与16进制表示转换的工具类。

#### 用于
16进制一般针对无法显示的一些二进制进行显示，常用于：
1、图片的字符串表现形式
2、加密解密
3、编码转换

#### 使用

`HexUtil`主要以`encodeHex`和`decodeHex`两个方法为核心，提供一些针对字符串的重载方法。

```java
String str = "我是一个字符串";

String hex = HexUtil.encodeHexStr(str, CharsetUtil.CHARSET_UTF_8);

//hex是：
//e68891e698afe4b880e4b8aae5ad97e7aca6e4b8b2

String decodedStr = HexUtil.decodeHexStr(hex);

//解码后与str相同
```
### Hash算法-HashUtil

#### 介绍
`HashUtil`其实是一个hash算法的集合，此工具类中融合了各种hash算法。

#### 方法
这些算法包括：

1. `additiveHash` 加法hash
2. `rotatingHash` 旋转hash
3. `oneByOneHash` 一次一个hash
4. `bernstein` Bernstein's hash
5. `universal` Universal Hashing
6. `zobrist` Zobrist Hashing
7. `fnvHash` 改进的32位FNV算法1
8. `intHash` Thomas Wang的算法，整数hash
9. `rsHash` RS算法hash
10. `jsHash` JS算法
11. `pjwHash` PJW算法
12. `elfHash` ELF算法
13. `bkdrHash` BKDR算法
14. `sdbmHash` SDBM算法
15. `djbHash` DJB算法
16. `dekHash` DEK算法
17. `apHash` AP算法
18. `tianlHash` TianL Hash算法
19. `javaDefaultHash` JAVA自己带的算法
20. `mixHash` 混合hash算法，输出64位的值

### 身份证工具-IdcardUtil
#### 由来
在日常开发中，我们对身份证的验证主要是正则方式（位数，数字范围等），但是中国身份证，尤其18位身份证每一位都有严格规定，并且最后一位为校验位。而我们在实际应用中，针对身份证的验证理应严格至此。于是`IdcardUtil`应运而生。

#### 介绍
`IdcardUtil`现在支持大陆15位、18位身份证，港澳台10位身份证。

工具中主要的方法包括：
1. `isValidCard` 验证身份证是否合法
2. `convert15To18` 身份证15位转18位
3. `getBirthByIdCard` 获取生日
4. `getAgeByIdCard` 获取年龄
5. `getYearByIdCard` 获取生日年
6. `getMonthByIdCard` 获取生日月
7. `getDayByIdCard` 获取生日天
8. `getGenderByIdCard` 获取性别
9. `getProvinceByIdCard` 获取省份

#### 使用

```java
String ID_18 = "321083197812162119";
String ID_15 = "150102880730303";

//是否有效
boolean valid = IdcardUtil.isValidCard(ID_18);
boolean valid15 = IdcardUtil.isValidCard(ID_15);

//转换
String convert15To18 = IdcardUtil.convert15To18(ID_15);
Assert.assertEquals(convert15To18, "150102198807303035");

//年龄
DateTime date = DateUtil.parse("2017-04-10");

int age = IdcardUtil.getAgeByIdCard(ID_18, date);
Assert.assertEquals(age, 38);

int age2 = IdcardUtil.getAgeByIdCard(ID_15, date);
Assert.assertEquals(age2, 28);

//生日
String birth = IdcardUtil.getBirthByIdCard(ID_18);
Assert.assertEquals(birth, "19781216");

String birth2 = IdcardUtil.getBirthByIdCard(ID_15);
Assert.assertEquals(birth2, "19880730");

//省份
String province = IdcardUtil.getProvinceByIdCard(ID_18);
Assert.assertEquals(province, "江苏");

String province2 = IdcardUtil.getProvinceByIdCard(ID_15);
Assert.assertEquals(province2, "内蒙古");
```

### 图片工具-ImageUtil

#### 介绍

针对awt中图片处理进行封装，这些封装包括：缩放、裁剪、转为黑白、加水印等操作。

#### 方法介绍

1. `scale` 缩放图片，提供两种重载方法：其中一个是按照长宽缩放，另一种是按照比例缩放。
2.  `cut` 剪裁图片
3. `cutByRowsAndCols` 按照行列剪裁（将图片分为20行和20列）
4. `convert` 图片类型转换，支持GIF-&gt;JPG、GIF-&gt;PNG、PNG-&gt;JPG、PNG-&gt;GIF(X)、BMP-&gt;PNG等
5. `gray` 彩色转为黑白
6. `pressText` 添加文字水印
7. `pressImage` 添加图片水印
8. `rotate` 旋转图片
9. `flip` 水平翻转图片

### 数字工具-NumberUtil
#### 由来

数字工具针对数学运算做工具性封装


#### 加减乘除

- `NumberUtil.add`  针对double类型做加法
- `NumberUtil.sub`  针对double类型做减法
- `NumberUtil.mul`  针对double类型做乘法
- `NumberUtil.div`  针对double类型做除法，并提供重载方法用于规定除不尽的情况下保留小数位数和舍弃方式。

以上四种运算都会将double转为BigDecimal后计算，解决float和double类型无法进行精确计算的问题。这些方法常用于商业计算。

#### 保留小数

保留小数的方法主要有两种：

- `NumberUtil.round` 方法主要封装BigDecimal中的方法来保留小数，返回double，这个方法更加灵活，可以选择四舍五入或者全部舍弃等模式。

```java
double te1=123456.123456;
double te2=123456.128456;
Console.log(round(te1,4));//结果:123456.12
Console.log(round(te2,4));//结果:123456.13
```

- `NumberUtil.roundStr` 方法主要封装`String.format`方法,舍弃方式采用四舍五入。

```java
double te1=123456.123456;
double te2=123456.128456;
Console.log(roundStr(te1,2));//结果:123456.12
Console.log(roundStr(te2,2));//结果:123456.13
```

#### decimalFormat

针对 `DecimalFormat.format`进行简单封装。按照固定格式对double或long类型的数字做格式化操作。

```java
long c=299792458;//光速
String format = NumberUtil.decimalFormat(",###", c);//299,792,458
```

格式中主要以 # 和 0 两种占位符号来指定数字长度。0 表示如果位数不足则以 0 填充，# 表示只要有可能就把数字拉上这个位置。

- 0 -&gt; 取一位整数
- 0.00 -&gt; 取一位整数和两位小数
- 00.000 -&gt; 取两位整数和三位小数
- \# -&gt; 取所有整数部分
- \#.##% -&gt; 以百分比方式计数，并取两位小数
- \#.#####E0 -&gt; 显示为科学计数法，并取五位小数
- ,### -&gt; 每三位以逗号进行分隔，例如：299,792,458
- 光速大小为每秒,###米 -&gt; 将格式嵌入文本

关于格式的更多说明，请参阅：[Java DecimalFormat的主要功能及使用方法](http://blog.csdn.net/evangel_z/article/details/7624503)

#### 是否为数字
- `NumberUtil.isNumber` 是否为数字
- `NumberUtil.isInteger` 是否为整数
- `NumberUtil.isDouble` 是否为浮点数
- `NumberUtil.isPrimes` 是否为质数

#### 随机数
- `NumberUtil.generateRandomNumber` 生成不重复随机数 根据给定的最小数字和最大数字，以及随机数的个数，产生指定的不重复的数组。
- `NumberUtil.generateBySet` 生成不重复随机数 根据给定的最小数字和最大数字，以及随机数的个数，产生指定的不重复的数组。

#### 整数列表

`NumberUtil.range` 方法根据范围和步进，生成一个有序整数列表。
`NumberUtil.appendRange` 将给定范围内的整数添加到已有集合中

#### 其它
- `NumberUtil.factorial` 阶乘
- `NumberUtil.sqrt` 平方根
- `NumberUtil.divisor` 最大公约数
- `NumberUtil.multiple` 最小公倍数
- `NumberUtil.getBinaryStr` 获得数字对应的二进制字符串
- `NumberUtil.binaryToInt` 二进制转int
- `NumberUtil.binaryToLong` 二进制转long
- `NumberUtil.compare` 比较两个值的大小
- `NumberUtil.toStr` 数字转字符串，自动并去除尾小数点儿后多余的0

### 网络工具-NetUtil
#### 由来
在日常开发中，网络连接这块儿必不可少。日常用到的一些功能,隐藏掉部分IP地址、绝对相对路径的转换等等。

#### 介绍

`NetUtil` 工具中主要的方法包括：
1. `longToIpv4` 根据long值获取ip v4地址
2. `ipv4ToLong` 根据ip地址计算出long型的数据
3. `isUsableLocalPort` 检测本地端口可用性
4. `isValidPort` 是否为有效的端口
5. `isInnerIP` 判定是否为内网IP
6. `localIpv4s` 获得本机的IP地址列表
7. `toAbsoluteUrl` 相对URL转换为绝对URL
8. `hideIpPart` 隐藏掉IP地址的最后一部分为 * 代替
9. `buildInetSocketAddress` 构建InetSocketAddress
10. `getIpByHost` 通过域名得到IP
11. `isInner` 指定IP的long是否在指定范围内

#### 使用

```java
String ip= "127.0.0.1";
long iplong = 2130706433L;

//根据long值获取ip v4地址
String ip= NetUtil.longToIpv4(iplong);


//根据ip地址计算出long型的数据
long ip= NetUtil.ipv4ToLong(ip);

//检测本地端口可用性
boolean result= NetUtil.isUsableLocalPort(6379);

//是否为有效的端口
boolean result= NetUtil.isValidPort(6379);

//隐藏掉IP地址
 String result =NetUtil.hideIpPart(ip);
```

### 分页工具-PageUtil

#### 由来
分页工具类并不是数据库分页的封装，而是分页方式的转换。在我们手动分页的时候，常常使用页码+每页个数的方式，但是有些数据库需要使用开始位置和结束位置来表示。很多时候这种转换容易出错（边界问题），于是封装了PageUtil工具类。

#### transToStartEnd
将页数和每页条目数转换为开始位置和结束位置。
此方法用于不包括结束位置的分页方法。

例如：
- 页码：1，每页10 -&gt; [0, 10]
- 页码：2，每页10 -&gt; [10, 20]

```java
int[] startEnd1 = PageUtil.transToStartEnd(1, 10);//[0, 10]
int[] startEnd2 = PageUtil.transToStartEnd(2, 10);//[10, 20]
```

&gt; 方法中，页码从1开始，位置从0开始

#### totalPage

根据总数计算总页数

```java
int totalPage = PageUtil.totalPage(20, 3);//7
```

#### 分页彩虹算法

此方法来自：https://github.com/iceroot/iceroot/blob/master/src/main/java/com/icexxx/util/IceUtil.java

在页面上显示下一页时，常常需要显示前N页和后N页，`PageUtil.rainbow`作用于此。

例如我们当前页为第5页，共有20页，只显示6个页码，显示的分页列表应为：

```
上一页 3 4 [5] 6 7 8 下一页
```

```java
//参数意义分别为：当前页、总页数、每屏展示的页数
int[] rainbow = PageUtil.rainbow(5, 20, 6);
//结果：[3, 4, 5, 6, 7, 8]
```

### 随机工具-RandomUtil
#### 说明
`RandomUtil`主要针对JDK中`Random`对象做封装，严格来说，Java产生的随机数都是伪随机数，因此封装后产生的随机结果也是伪随机结果。不过这种随机结果对于大多数情况已经够用。

#### 使用

- `RandomUtil.randomInt` 获得指定范围内的随机数
- `RandomUtil.randomBytes` 随机bytes
- `RandomUtil.randomEle` 随机获得列表中的元素
- `RandomUtil.randomEleSet` 随机获得列表中的一定量的不重复元素，返回Set
- `RandomUtil.randomString` 获得一个随机的字符串（只包含数字和字符）
- `RandomUtil.randomNumbers` 获得一个只包含数字的字符串
- `RandomUtil.randomUUID` 随机UUID
- `RandomUtil.weightRandom` 权重随机生成器，传入带权重的对象，然后根据权重随机获取对象
-
### 对象工具-ObjectUtil

#### 由来

在我们的日常使用中，有些方法是针对Object通用的，这些方法不区分何种对象，针对这些方法，封装为`ObjectUtil`。

#### `ObjectUtil.equal`
比较两个对象是否相等，相等需满足以下条件之一：
1. obj1 == null &amp;&amp; obj2 == null
2. obj1.equals(obj2)

#### `ObjectUtil.length`
计算对象长度，如果是字符串调用其length方法，集合类调用其size方法，数组调用其length属性，其他可遍历对象遍历计算长度。

支持的类型包括：
- CharSequence
- Collection
- Map
- Iterator
- Enumeration
- Array

#### `ObjectUtil.contains`
对象中是否包含元素。

支持的对象类型包括：
- String
- Collection
- Map
- Iterator
- Enumeration
- Array

#### 判断是否为null
- `ObjectUtil.isNull`
- `ObjectUtil.isNotNull`

#### 克隆

- `ObjectUtil.clone` 克隆对象，如果对象实现Cloneable接口，调用其clone方法，如果实现Serializable接口，执行深度克隆，否则返回`null`。

- `ObjectUtil.cloneIfPossible` 返回克隆后的对象，如果克隆失败，返回原对象

- `ObjectUtil.cloneByStream` 序列化后拷贝流的方式克隆，对象必须实现Serializable接口

#### 序列化和反序列化

- `serialize` 序列化，调用JDK序列化
- `unserialize`  反序列化，调用JDK

#### 判断基本类型

`ObjectUtil.isBasicType` 判断是否为基本类型，包括包装类型和非包装类型。

### 字符串工具-StrUtil
#### 由来

这个工具的用处类似于[Apache Commons Lang](http://commons.apache.org/)中的`StringUtil`，之所以使用`StrUtil`而不是使用`StringUtil`是因为前者更短，而且`Str`这个简写我想已经深入人心了，大家都知道是字符串的意思。常用的方法例如`isBlank`、`isNotBlank`、`isEmpty`、`isNotEmpty`这些我就不做介绍了，判断字符串是否为空，下面我说几个比较好用的功能。

#### 1. `hasBlank`、`hasEmpty`方法
就是给定一些字符串，如果一旦有空的就返回true，常用于判断好多字段是否有空的（例如web表单数据）。

**这两个方法的区别是`hasEmpty`只判断是否为null或者空字符串（""），`hasBlank`则会把不可见字符也算做空，`isEmpty`和`isBlank`同理。**

#### 2. `removePrefix`、`removeSuffix`方法
这两个是去掉字符串的前缀后缀的，例如去个文件名的扩展名啥。

```Java
String fileName = StrUtil.removeSuffix("pretty_girl.jpg", ".jpg")  //fileName -&gt; pretty_girl
```
还有忽略大小写的`removePrefixIgnoreCase`和`removeSuffixIgnoreCase`都比较实用。

#### 3. `sub`方法
不得不提一下这个方法，有人说String有了subString你还写它干啥，我想说subString方法越界啥的都会报异常，你还得自己判断，难受死了，我把各种情况判断都加进来了，而且index的位置还支付负数哦，-1表示最后一个字符（这个思想来自于[Python](https://www.python.org/)，如果学过[Python](https://www.python.org/)的应该会很喜欢的），还有就是如果不小心把第一个位置和第二个位置搞反了，也会自动修正（例如想截取第4个和第2个字符之间的部分也是可以的哦~）
举个栗子

```Java
String str = "abcdefgh";
String strSub1 = StrUtil.sub(str, 2, 3); //strSub1 -&gt; c
String strSub2 = StrUtil.sub(str, 2, -3); //strSub2 -&gt; cde
String strSub3 = StrUtil.sub(str, 3, 2); //strSub2 -&gt; c
```

#### 4. `str`、`bytes`方法
好吧，我承认把`String.getByte(String charsetName)`方法封装在这里了，原生的`String.getByte()`这个方法太坑了，使用系统编码，经常会有人跳进来导致乱码问题，所以我就加了这两个方法强制指定字符集了，包了个try抛出一个运行时异常，省的我得在我业务代码里处理那个恶心的`UnsupportedEncodingException`。

#### 5. format方法
我会告诉你这是我最引以为豪的方法吗？灵感来自slf4j，可以使用字符串模板代替字符串拼接，我也自己实现了一个，而且变量的标识符都一样，神马叫无缝兼容~~来，上栗子（吃多了上火吧……）
````Java
String template = "{}爱{}，就像老鼠爱大米";
String str = StrUtil.format(template, "我", "你"); //str -&gt; 我爱你，就像老鼠爱大米
````
参数我定义成了Object类型，如果传别的类型的也可以，会自动调用toString()方法的。

#### 6. 定义的一些常量
为了方便，我定义了一些比较常见的字符串常量在里面，像点、空串、换行符等等，还有HTML中的一些转移字符。

### 正则工具-ReUtil
#### 由来
在文本处理中，正则表达式几乎是全能的，但是Java的正则表达式有时候处理一些事情还是有些繁琐，所以我封装了部分常用功能。就比如说我要匹配一段文本中的某些部分，我们需要这样做：

```Java
Pattern pattern = Pattern.compile(regex, Pattern.DOTALL);
Matcher matcher = pattern.matcher(content);
if (matcher.find()) {
    String result= matcher.group();
}
```

其中牵涉到多个对象，想用的时候真心记不住。好吧，既然功能如此常用，我就封装一下：

```Java
/**
* 获得匹配的字符串
*
* @param pattern 编译后的正则模式
* @param content 被匹配的内容
* @param groupIndex 匹配正则的分组序号
* @return 匹配后得到的字符串，未匹配返回null
*/
public static String get(Pattern pattern, String content, int groupIndex) {
    Matcher matcher = pattern.matcher(content);
    if (matcher.find()) {
        return matcher.group(groupIndex);
    }
    return null;
}

/**
* 获得匹配的字符串
*
* @param regex 匹配的正则
* @param content 被匹配的内容
* @param groupIndex 匹配正则的分组序号
* @return 匹配后得到的字符串，未匹配返回null
*/
public static String get(String regex, String content, int groupIndex) {
    Pattern pattern = Pattern.compile(regex, Pattern.DOTALL);
    return get(pattern, content, groupIndex);
}
```

#### ReUtil.extractMulti
抽取多个分组然后把它们拼接起来

```java
String resultExtractMulti = ReUtil.extractMulti("(\\w)aa(\\w)", content, "$1-$2");
Assert.assertEquals("Z-a", resultExtractMulti);
```

#### ReUtil.delFirst
删除第一个匹配到的内容

```java
String resultDelFirst = ReUtil.delFirst("(\\w)aa(\\w)", content);
Assert.assertEquals("ZZbbbccc中文1234", resultDelFirst);
```

#### ReUtil.findAll
查找所有匹配文本

```java
List&lt;String&gt; resultFindAll = ReUtil.findAll("\\w{2}", content, 0, new ArrayList&lt;String&gt;());
ArrayList&lt;String&gt; expected = CollectionUtil.newArrayList("ZZ", "Za", "aa", "bb", "bc", "cc", "12", "34");
Assert.assertEquals(expected, resultFindAll);
```

#### ReUtil.getFirstNumber
找到匹配的第一个数字

```java
Integer resultGetFirstNumber = ReUtil.getFirstNumber(content);
Assert.assertEquals(Integer.valueOf(1234), resultGetFirstNumber);
```

#### ReUtil.isMatch
给定字符串是否匹配给定正则

```java
boolean isMatch = ReUtil.isMatch("\\w+[\u4E00-\u9FFF]+\\d+", content);
Assert.assertTrue(isMatch);
```

#### ReUtil.replaceAll
通过正则查找到字符串，然后把匹配到的字符串加入到replacementTemplate中，$1表示分组1的字符串

```java
//此处把1234替换为 -&gt;1234&lt;-
String replaceAll = ReUtil.replaceAll(content, "(\\d+)", "-&gt;$1&lt;-");
Assert.assertEquals("ZZZaaabbbccc中文-&gt;1234&lt;-", replaceAll);
```

#### ReUtil.escape
转义给定字符串，为正则相关的特殊符号转义

```java
String escape = ReUtil.escape("我有个$符号{}");
Assert.assertEquals("我有个\\$符号\\{\\}", escape);
```
### URL工具-URLUtil

#### 介绍

URL（Uniform Resource Locator）中文名为统一资源定位符，有时也被俗称为网页地址。表示为互联网上的资源，如网页或者FTP地址。在Java中，也可以使用URL表示Classpath中的资源（Resource）地址。

#### 获取URL对象

- `URLUtil.url` 通过一个字符串形式的URL地址创建对象
- `URLUtil.getURL` 主要获得ClassPath下资源的URL，方便读取Classpath下的配置文件等信息。

#### 其它

- `URLUtil.formatUrl` 格式化URL链接。对于不带http://头的地址做简单补全。
- `URLUtil.encode` 封装`URLEncoder.encode`，将需要转换的内容（ASCII码形式之外的内容），用十六进制表示法转换出来，并在之前加上%开头。
- `URLUtil.decode` 封装`URLDecoder.decode`，将%开头的16进制表示的内容解码。
- `URLUtil.getPath` 获得path部分 URI -&gt; http://www.aaa.bbb/search?scope=ccc&amp;q=ddd PATH -&gt; /search
- `URLUtil.toURI` 转URL或URL字符串为URI。

### XML工具-XmlUtil
#### 由来

在日常编码中，我们接触最多的除了JSON外，就是XML格式了，一般而言，我们首先想到的是引入Dom4j包，却不知JDK已经封装有XML解析和构建工具：w3c dom。但是由于这个API操作比较繁琐，因此提供了XmlUtil简化XML的创建、读和写的过程。

#### 读取XML

读取XML分为两个方法：

- `XmlUtil.readXML` 读取XML文件
- `XmlUtil.parseXml` 解析XML字符串为Document对象

#### 写XML

- `XmlUtil.toStr` 将XML文档转换为String
- `XmlUtil.toFile` 将XML文档写入到文件

#### 创建XML

- `XmlUtil.createXml` 创建XML文档, 创建的XML默认是utf8编码，修改编码的过程是在toStr和toFile方法里，既XML在转为文本的时候才定义编码。

### XML操作

通过以下工具方法，可以完成基本的节点读取操作。

- `XmlUtil.cleanInvalid` 除XML文本中的无效字符
- `XmlUtil.getElements` 根据节点名获得子节点列表
- `XmlUtil.getElement` 根据节点名获得第一个子节点
- `XmlUtil.elementText` 根据节点名获得第一个子节点
- `XmlUtil.transElements` 将NodeList转换为Element列表

#### XML与对象转换

- `writeObjectAsXml` 将可序列化的对象转换为XML写入文件，已经存在的文件将被覆盖。
- `readObjectFromXml` 从XML中读取对象。

&gt; 注意
&gt; 这两个方法严重依赖JDK的`XMLEncoder`和`XMLDecoder`，生成和解析必须成对存在（遵循固定格式），普通的XML转Bean会报错。

#### Xpath操作
Xpath的更多介绍请看文章：[https://www.ibm.com/developerworks/cn/xml/x-javaxpathapi.html](https://www.ibm.com/developerworks/cn/xml/x-javaxpathapi.html)

- `createXPath` 创建XPath
- `getByXPath` 通过XPath方式读取XML节点等信息

栗子：

```xml
&lt;?xml version="1.0" encoding="utf-8"?&gt;

&lt;returnsms&gt;
  &lt;returnstatus&gt;Success（成功）&lt;/returnstatus&gt;
  &lt;message&gt;ok&lt;/message&gt;
  &lt;remainpoint&gt;1490&lt;/remainpoint&gt;
  &lt;taskID&gt;885&lt;/taskID&gt;
  &lt;successCounts&gt;1&lt;/successCounts&gt;
&lt;/returnsms&gt;
```

```java
Document docResult=XmlUtil.readXML(xmlFile);
//结果为“ok”
Object value = XmlUtil.getByXPath("//returnsms/message", docResult, XPathConstants.STRING);
```

#### 总结

XmlUtil只是w3c dom的简单工具化封装，减少操作dom的难度，如果项目对XML依赖较大，依旧推荐Dom4j框架。

### 压缩工具-ZipUtil

#### 由来

在Java中，对文件、文件夹打包，压缩是一件比较繁琐的事情，我们常常引入Zip4j进行此类操作。但是很多时候，JDK中的zip包就可满足我们大部分需求。ZipUtil就是针对java.util.zip做工具化封装，使压缩解压操作可以一个方法搞定，并且自动处理文件和目录的问题，不再需要用户判断，压缩后的文件也会自动创建文件，自动创建父目录，大大简化的压缩解压的复杂度。

#### Zip

1. 压缩

`ZipUtil.zip` 方法提供一系列的重载方法，满足不同需求的压缩需求，这包括：

- 打包到当前目录（可以打包文件，也可以打包文件夹，根据路径自动判断）

```java
//将aaa目录下的所有文件目录打包到d:/aaa.zip
ZipUtil.zip("d:/aaa");
```

- 指定打包后保存的目的地，自动判断目标是文件还是文件夹

```java
//将aaa目录下的所有文件目录打包到d:/bbb/目录下的aaa.zip文件中
ZipUtil.zip("d:/aaa", "d:/bbb/");

//将aaa目录下的所有文件目录打包到d:/bbb/目录下的ccc.zip文件中
ZipUtil.zip("d:/aaa", "d:/bbb/ccc.zip");
```

- 可选是否包含被打包的目录。比如我们打包一个照片的目录，打开这个压缩包有可能是带目录的，也有可能是打开压缩包直接看到的是文件。zip方法增加一个boolean参数可选这两种模式，以应对众多需求。

```
//将aaa目录以及其目录下的所有文件目录打包到d:/bbb/目录下的ccc.zip文件中
ZipUtil.zip("d:/aaa", "d:/bbb/ccc.zip", true);
```

- 多文件或目录压缩。可以选择多个文件或目录一起打成zip包。

```java
ZipUtil.zip(FileUtil.file("d:/bbb/ccc.zip"), false,
    FileUtil.file("d:/test1/file1.txt"),
    FileUtil.file("d:/test1/file2.txt"),
    FileUtil.file("d:/test2/file1.txt"),
    FileUtil.file("d:/test2/file2.txt"),
);
```

2. 解压

`ZipUtil.unzip` 解压。同样提供几个重载，满足不同需求。

```
//将test.zip解压到e:\\aaa目录下，返回解压到的目录
File unzip = ZipUtil.unzip("E:\\aaa\\test.zip", "e:\\aaa");
```

#### Gzip

Gzip是网页传输中广泛使用的压缩方式，同样提供其工具方法简化其过程。

`ZipUtil.gzip` 压缩，可压缩字符串，也可压缩文件
`ZipUtil.unGzip` 解压Gzip文件

### Zlib

`ZipUtil.zlib` 压缩，可压缩字符串，也可压缩文件
`ZipUtil.unZlib` 解压zlib文件

&gt; 注意
&gt; ZipUtil默认情况下使用系统编码，也就是说：
&gt; 1. 如果你在命令行下运行，则调用系统编码（一般Windows下为GBK、Linux下为UTF-8）
&gt; 2. 如果你在IDE（如Eclipse）下运行代码，则读取的是当前项目的编码（详细请查阅IDE设置，我的项目默认都是UTF-8编码，因此解压和压缩都是用这个编码）

### 反射工具-ReflectUtil
#### 介绍
Java的反射机制，可以让语言变得更加灵活，对对象的操作也更加“动态”，因此在某些情况下，反射可以做到事半功倍的效果。针对Java的反射机制做了工具化封装，封装包括：

1. 获取构造方法
2. 获取字段
3. 获取字段值
4. 获取方法
5. 执行方法（对象方法和静态方法）

#### 获取某个类的所有方法

```java
Method[] methods = ReflectUtil.getMethods(ExamInfoDict.class);
```

#### 获取某个类的指定方法
```java
Method method = ReflectUtil.getMethod(ExamInfoDict.class, "getId");
```

#### 构造对象

```java
ReflectUtil.newInstance(ExamInfoDict.class);
```

#### 执行方法

```java
class TestClass {
	private int a;

	public int getA() {
		return a;
	}

	public void setA(int a) {
		this.a = a;
	}
}
```

```java
TestClass testClass = new TestClass();
ReflectUtil.invoke(testClass, "setA", 10);
```
### 命令行工具-RuntimeUtil
#### 介绍
在Java世界中，如果想与其它语言打交道，处理调用接口，或者JNI，就是通过本地命令方式调用了。封装了JDK的Process类，用于执行命令行命令（在Windows下是cmd，在Linux下是shell命令）。

#### 基础方法
1. `exec` 执行命令行命令，返回Process对象，Process可以读取执行命令后的返回内容的流

### 快捷方法
1. `execForStr` 执行系统命令，返回字符串
2. `execForLines` 执行系统命令，返回行列表

#### 使用

```java
String str = RuntimeUtil.execForStr("ipconfig");
```

执行这个命令后，在Windows下可以获取网卡信息。

### 剪贴板工具-ClipboardUtil
#### 介绍
`ClipboardUtil`这个类用于简化操作剪贴板（当然使用场景被局限）。`ClipboardUtil` 封装了几个常用的静态方法:

#### 通用方法
1. `getClipboard` 获取系统剪贴板
2. `set` 设置内容到剪贴板
3. `get` 获取剪贴板内容

#### 针对文本
1. `setStr` 设置文本到剪贴板
2. `getStr` 从剪贴板获取文本

#### 针对Image对象（图片）
1. `setImage` 设置图片到剪贴板
2. `getImage` 从剪贴板获取图片

### 枚举工具-EnumUtil
#### 介绍
枚举（enum）算一种“语法糖”，是指一个经过排序的、被打包成一个单一实体的项列表。一个枚举的实例可以使用枚举项列表中任意单一项的值。枚举在各个语言当中都有着广泛的应用，通常用来表示诸如颜色、方式、类别、状态等等数目有限、形式离散、表达又极为明确的量。Java从JDK5开始，引入了对枚举的支持。

`EnumUtil` 用于对未知枚举类型进行操作。

#### 方法

首先我们定义一个枚举对象：

```java
//定义枚举
public enum TestEnum{
	TEST1("type1"), TEST2("type2"), TEST3("type3");

	private TestEnum(String type) {
		this.type = type;
	}

	private String type;

	public String getType() {
		return this.type;
	}
}
```

#### `getNames`

获取枚举类中所有枚举对象的name列表。栗子：

```java
//定义枚举
public enum TestEnum {
	TEST1, TEST2, TEST3;
}
```

```java
List&lt;String&gt; names = EnumUtil.getNames(TestEnum.class);
//结果：[TEST1, TEST2, TEST3]
```

#### `getFieldValues`

获得枚举类中各枚举对象下指定字段的值。栗子：

```java
List&lt;Object&gt; types = EnumUtil.getFieldValues(TestEnum.class, "type");
//结果：[type1, type2, type3]
```

#### `getEnumMap`

获取枚举字符串值和枚举对象的Map对应，使用LinkedHashMap保证有序，结果中键为枚举名，值为枚举对象。栗子：

```java
Map&lt;String,TestEnum&gt; enumMap = EnumUtil.getEnumMap(TestEnum.class);
enumMap.get("TEST1") // 结果为：TestEnum.TEST1
```

#### `getNameFieldMap`

获得枚举名对应指定字段值的Map，键为枚举名，值为字段值。栗子：

```java
Map&lt;String, Object&gt; enumMap = EnumUtil.getNameFieldMap(TestEnum.class, "type");
enumMap.get("TEST1") // 结果为：type1
```

### 引用工具-ReferenceUtil
#### 介绍

引用工具类，主要针对Reference 工具化封装

主要封装包括：

1. SoftReference 软引用，在GC报告内存不足时会被GC回收
2. WeakReference 弱引用，在GC时发现弱引用会回收其对象
3. PhantomReference 虚引用，在GC时发现虚引用对象，会将PhantomReference插入ReferenceQueue。此时对象未被真正回收，要等到ReferenceQueue被真正处理后才会被回收。

#### `create`

根据类型枚举创建引用。

### 泛型类型工具-TypeUtil
#### 介绍

针对 `java.lang.reflect.Type` 的工具类封装，最主要功能包括：

1. 获取方法的参数和返回值类型（包括Type和Class）
2. 获取泛型参数类型（包括对象的泛型参数或集合元素的泛型类型）

#### 方法

首先我们定义一个类：

```java
public class TestClass {
	public List&lt;String&gt; getList(){
		return new ArrayList&lt;&gt;();
	}

	public Integer intTest(Integer integer) {
		return 1;
	}
}
```

##### `getClass`

获得Type对应的原始类

##### `getParamType`

```java
Method method = ReflectUtil.getMethod(TestClass.class, "intTest", Integer.class);
Type type = TypeUtil.getParamType(method, 0);
// 结果：Integer.class
```

获取方法参数的泛型类型

##### `getReturnType`

获取方法的返回值类型

```java
Method method = ReflectUtil.getMethod(TestClass.class, "getList");
Type type = TypeUtil.getReturnType(method);
// 结果：java.util.List&lt;java.lang.String&gt;
```

##### `getTypeArgument`

获取泛型类子类中泛型的填充类型。

```java
Method method = ReflectUtil.getMethod(TestClass.class, "getList");
Type type = TypeUtil.getReturnType(method);

Type type2 = TypeUtil.getTypeArgument(type);
// 结果：String.class
```

### 唯一ID工具-IdUtil
#### 介绍
在分布式环境中，唯一ID生成应用十分广泛，生成方法也多种多样，针对一些常用生成策略做了简单封装。唯一ID生成器的工具类，涵盖了：

- UUID
- ObjectId（MongoDB）
- Snowflake（Twitter）


#### UUID

UUID全称通用唯一识别码（universally unique identifier），JDK通过`java.util.UUID`提供了 Leach-Salz 变体的封装。生成一个UUID字符串方法如下：

```java
//生成的UUID是带-的字符串，类似于：a5c8a5e8-df2b-4706-bea4-08d0939410e3
String uuid = IdUtil.randomUUID();

//生成的是不带-的字符串，类似于：b17f24ff026d40949c85a24f4f375d42
String simpleUUID = IdUtil.simpleUUID();
```

重写`java.util.UUID`的逻辑，对应类为`com.lenovo.brain.common.lang.UUID`，使生成不带-的UUID字符串不再需要做字符替换，性能提升一倍左右。

#### ObjectId

ObjectId是MongoDB数据库的一种唯一ID生成策略，是UUID version1的变种，详细介绍可见：[服务化框架－分布式Unique ID的生成方法一览](http://calvin1978.blogcn.com/articles/uuid.html)。

针对此封装了`com.lenovo.brain.common.lang.ObjectId`，快捷创建方法为：

```java
//生成类似：5b9e306a4df4f8c54a39fb0c
String id = ObjectId.next();
String id2 = IdUtil.objectId();
```

#### Snowflake

分布式系统中，有一些需要使用全局唯一ID的场景，有些时候我们希望能使用一种简单一些的ID，并且希望ID能够按照时间有序生成。Twitter的Snowflake 算法就是这种生成器。

使用方法如下：

```java
//参数1为终端ID
//参数2为数据中心ID
Snowflake snowflake = IdUtil.createSnowflake(1, 1);
long id = snowflake.nextId();
```
