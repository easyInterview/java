###   【好好面试】去年面了多个候选人，看看我埋的坑还有他们应该要补的Java面试基础

### 看看我在基础数据类型方面埋了什么坑

> 说说看，Java有多少种基本的数据类型？多大？

Java有8中基本的数据类型，分别是

- byte，占据1个字节8位
- char，占据2个字节16位
- short，占据2个字节16位
- int，占据4个字节32位
- float，占据4个字节32位
- long，占据8个字节64位
- double，占据8个字节64位
- boolean，占据一个字节8位

<u>错，将boolean默认为一个字节基本是所有初学者的通。</u>

**注意：boolean的大小是未知的，虽然我们看boolean只有：true、false两种情况，可以使用 1 bit 来存储，但是实际上没有明确规定是1bit，因为因为对虚拟机来说根本就不存在 boolean 这个类型。在《Java虚拟机规范》中给出了两种定义，分别是4个字节和boolean数组时1个字节的定义，但是具体还要看虚拟机实现是否按照规范来，1个字节、4个字节都是有可能的。这其实是运算效率和存储空间之间的博弈，两者都非常的重要。** 



> 那Integer这些算什么呢？

这些算是包装类型，每个基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。比如：

```java
Integer number1 = 2;     // 装箱 调用了 Integer.valueOf(2)
int number2 = number1;         // 拆箱 调用了 number1.intValue()
```



> 记得挺牢的，那 new Integer(1024) 和Integer.valueOf(1024) 有没有什么区别呢？

首先，new Integer(1024) 每次都会新建一个对象，而Integer.valueOf(1024) 会使用缓冲池中的对象，多次调用会取得同一个对象的引用。

我举个例子：

![image-20210109100030722](https://gitee.com/xi_fan/img/raw/master/image-20210109100030722.png)

<u>错，这是我埋着的坑点，我曾经用这一招坑了多个候选人。</u>

**注意：Integer.valueOf(1024) 和Integer.valueOf(1024) 缺不等于true，而是false。Integer.valueOf从缓冲池取的数值是有大小限制的，并不是任何数**

我们可以看看valueOf() 的源码，其实也比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓冲池的内容。

![image-20210109100401281](https://gitee.com/xi_fan/img/raw/master/image-20210109100401281.png)

目前我的jdk版本是 8 ，在jdk8中Integer 缓冲池的大小默认为 -128\~127。

![image-20210109100503563](https://gitee.com/xi_fan/img/raw/master/image-20210109100503563.png)



> <u>做为一个面试官，我很喜欢挖别人回答问题时暴露的细节点</u>。你刚刚说到了自动装箱和拆箱，说说看你的理解？

编译器会在自动装箱过程中调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象，因此对比的时候会返回true。

![image-20210109101748474](https://gitee.com/xi_fan/img/raw/master/image-20210109101748474.png)



> <u>继续往深挖，看看候选人对知识点的掌握有多深。</u>那说说看你知道的缓冲池有哪些？

目前基本类型对应的缓冲池如下：

- boolean 缓冲池，true and false
- byte缓冲池
- short 缓冲池
- int 缓冲池
- char 缓冲池

因此我们在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，那么就可以直接使用缓冲池中的对象。



> <u>继续挖，看看他有没有看过缓冲池的源码。</u>你说的这些缓冲池的上限下限都是不变的吗？还是说可以设定的？

基本上都是不可变的，不过在 jdk 1.8 中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的。我们可以看源码

![image-20210109102339939](https://gitee.com/xi_fan/img/raw/master/image-20210109102339939.png)

在启动 jvm 的时候，我们可以通过通过 -XX:AutoBoxCacheMax=&lt;size&gt; 来指定这个缓冲池的大小，在JVM初始化的时候，这个设置会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。



*总结：上面的坑分别有boolean的大小、缓冲池的大小、自动拆箱和装箱、缓冲池的大小是否可变，基本上这几个坑点可以坑倒百分之六十的候选人，其次是做为一个面试官，我很喜欢挖候选人回答问题的细节，毕竟深挖可以看得出你是不是真的有料！！！*



### 看看我在String方面埋了什么坑

> 你刚刚说了基本类型了，说说看你对String的了解吧

String 被声明为 final，因此它不可被继承。在 Java 8 中，String 内部使用 char 数组存储数据，并且声明为 final，这意味着 value 数组初始化之后就不能再引用其它数组，String 内部也没有改变 value 数组的方法，因此可以保证 String 不可变。



> <u>继续深挖</u> 说说看不可变的好处？

这个问题的回答比较泛，可以说的点比较多，大致可以分为：

- 首先是不可变自然意味着安全，当String 作为参数引用的时候，不可变性可以保证参数不可变。

- 其次是可以缓存 hash 值，实际上，我们开发的时候经常会用来当做map的key，不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

- 最后自然是String Pool 的需要，如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用，而自然只有 String 是不可变的，才可能使用 String Pool。如果是可变的，那么 String Pool也就无法被设计出来了。



> <u>继续深挖</u> 有没有用过StringBuffer 和 StringBuilder，说说看String, StringBuffer 以及StringBuilder三者的区别？

首先他们都是被final修饰的类，都是不可被继承，不过从可变性上来说，String 我们刚刚说到了，是不可变的，而StringBuffer 和 StringBuilder 可变的，这是内部结构导致的，StringBuffer 和StringBuilder 内部放数据的数组没有被final修饰。

其次从线程安全方面来说

- String 不可变，是线程安全的

- StringBuilder 不是线程安全的，因为内部并没有使用任何的安全处理
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步



> <u>继续挖细节点</u>你刚刚有说到String Pool ，说说看你的理解

String Pool也就是我们经常说的字符串常量池，它保存着所有字符串字面量，而且是在编译时期就确定了。



> String Pool是在编译时期就确定了，那么请问是否不可变的呢？

是的。

<u>错，所有初学者都会犯的一个问题，那就是忽略了String.intern的存在，我经常用这个坑点来区分初学者和中级水平的候选人的区别！！！</u>

我们可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。

我们可以看到

![image-20210109111528457](https://gitee.com/xi_fan/img/raw/master/image-20210109111528457.png)

这是一个本地方法，看不到源码，不过我们可以看到注释

大致意思就是当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

用个demo来解释这个流程

![image-20210109111744460](https://gitee.com/xi_fan/img/raw/master/image-20210109111744460.png)

我上面的s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 和 s2.intern() 方法取得同一个字符串引用。第一个intern() 首先把 "饭谈编程" 放到 String Pool 中，然后返回这个字符串引用，而第二个intern()则直接从String Pool 读取了，因此 s3 和 s4 引用的是同一个字符串。



> <u>继续挖坑，准备埋了候选人</u>刚刚说到 new String("饭谈编程") !=  new String("饭谈编程") ，那么 "饭谈编程" 和 "饭谈编程"相等吗？说下流程？

是相等的，我们可以看到

![image-20210109112317572](https://gitee.com/xi_fan/img/raw/master/image-20210109112317572.png)

流程是因为：采用这种字面量的形式创建字符串，JVM会自动地将字符串放入 String Pool 中，因此它们两个是相等的。



> <u>继续往细节挖，这是一个比较刁钻的问题</u> new String("饭谈编程") JVM做了啥？

首先使用这种方式一共会创建两个字符串对象，当然了，前提是 String Pool 中还没有 "饭谈编程" 这个字符串对象，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "饭谈编程" 字符串字面量。

然后在使用 new 的方式的时候，在堆中创建一个字符串对象，这一步我们可以结合String的构造函数来看看

```java
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```

可以看到，在将一个字符串对象作为另一个字符串对象的构造函数参数时，JVM会从String Pool 中将这个字符串对象取出来，当做参数传进String的构造函数中，将 value 数组和hash值赋予这个新的对象。



*总结：String我们在日常开发中经常用到，不过一个合格的候选人应该要吃透String、StringBuilder 和StringBuffer的区别，并且要对String Pool的原理了解的尽量多一些，不要被我上面挖的坑给埋了。*



### 看看我在运算方面埋了什么坑

> 请问在Java中方法参数的传递方式是引用传递呢？还是值传递呢？

这个要分情况，如果参数是基本类型的话，就是值传递，如果是引用类型的话，则是引用传递。

<u>错，这是很多初学者容易搞错的地方，也是我日常挖坑埋人的地方</u>

Java 的参数全都是是以值传递的形式传入方法中，而不是引用传递。如果参数是基本类型，则传递的是基本类型的字面量值的拷贝。而如果参数是引用类型的话，传递的则值该参数所引用的对象在堆中地址值的拷贝。



> 请看题 float f = 2.2，这么写有没有问题？

看起来是没问题的，其实是有问题的，这个其实一般我不会用来面试，而是用来放在笔试题中。

2.2这个字面量属于 double 类型的，因此不能直接将 2.2 直接赋值给 float 变量，因为这是向下转型，记住Java 不能隐式执行向下转型，因为这会使得精度降低。

正常写法是

```java
float f = 2.2f;
```



> <u>继续挖坑</u>，那么float f = 2.2f; f += 2.2;可以吗

这同样是我会放进笔试题考研候选人基础的一道题，是可以的，因为使用 += 或者 ++ 运算符，JVM会执行隐式类型转换。

上面的语句相当于将 s1 + 1 的计算结果进行了向下转型：

```java
f = (float) (f + 2.2);
```



*总结：在运算方面埋的坑比较基础，一般是放在面试题中，而且其实用idea开发的话实际上可以在开发期就会报错了，但是这并不意味着idea可以检测出来的东西，你就可以不懂，特别是要来我司面试，这意味着你的专业能力是否过关。*



### 看看我在修饰符方面埋了什么坑

> 说说看对修饰符final的理解

首先是在变量上使用了final，意味着声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量，作用可以分为：

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象。

如果是在方法上使用了final，则声明方法不能被子类重写。

如果是在类上使用了final，则声明方法不允许被继承。



> <u>开始挖坑了，等着你跳</u> 挺好的，按照你的说法，final int b = 1; b之后是不可以改的；那如果是这样的例子，A对象的x可以改吗

![image-20210109123855200](https://gitee.com/xi_fan/img/raw/master/image-20210109123855200.png)

是可以改的，这也是引用类型的那一种，fianl是作用在A对象的引用上，而不是作用在A对象的数据成员x上，因此是可以改的。



> <u>继续挖坑</u> 刚刚你说到声明方法不能被子类重写，那么问题来了，为啥这样可以

![image-20210109124438515](https://gitee.com/xi_fan/img/raw/master/image-20210109124438515.png)

<u>一般候选人都会在这里支支吾吾的说不出个所以然来。</u>

其实他回答的理论是对的，只是他没有实际上尝试过我这种写法。实际上在private 方法隐式地被指定为 final的时候，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法并不是重写了基类方法，而是在子类中定义了一个新的方法。



> 聊聊看你对修饰符static的了解

首先是用在变量上的话，这个变量我们一般称之为静态变量，也可以称之为类变，也就是说这个变量属于类的，类所有的实例都共享静态变量，一般我们是直接通过类名来访问它，需要注意的一点事，静态变量在内存中只存在一份。

而如果是用在方法上的话，就被称之为静态方法，这个静态方法在类加载的时候就存在了，它不依赖于任何实例，因此静态方法必须有实现，也就是说它不能是抽象方法。



> <u>开始挖坑</u> 可以在静态方法内使用this或者super关键字吗

不可以的，只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，可以说static和this和super是互相矛盾的存在。



> <u>坑点来了</u> 之前来了个实习生，写代码的时候就犯了这个错，你看看下面的执行结果是啥

![image-20210109125553717](https://gitee.com/xi_fan/img/raw/master/image-20210109125553717.png)

正确答案是

![image-20210109125704464](https://gitee.com/xi_fan/img/raw/master/image-20210109125704464.png)

这里记住一个点就可以了，静态语句块优先于普通语句块，而普通语句块优先于构造函数。



> <u>继续深坑</u> 那么如果是有继承关系在的时候呢？比如这道题，说说他们的执行顺序

![image-20210109130202218](https://gitee.com/xi_fan/img/raw/master/image-20210109130202218.png)

大部分初级的候选人都会在这道题被绊倒，正确答案应该是：

![image-20210109130258183](https://gitee.com/xi_fan/img/raw/master/image-20210109130258183.png)

也就是说，存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

### 看看我在Object 通用方法埋了什么坑

> equals方法用过吧？看看这道题，说说看equals方法在前和在后有什么区别？

![image-20210110172344885](https://gitee.com/xi_fan/img/raw/master/image-20210110172344885.png)

没什么区别。

<u>错，初学者或者代码打得少的人都会犯这个错。</u>

test1会直接报空指针异常，你想想看，null.equals看不起来不就怪怪的吗？空指针怎么可能有方法呢是吧，

**拓展：**我们一般在企业开发中都会将已知的字面量放在equals，未知的参数放在equals后面，这样可以避免空指针，编程新手容易犯这个异常，我review过的代码这么实现的，说实话，挺多次的，不止编程新人，两三年工作经验的都会这么做，实在不应该，



> 说说看equlas 跟 == 的区别

equals方法比较的是字符串的内容是否相等，而 == 比较的则是对象地址。

这么回答也是对的，但是，我们更希望听到更专业的回答，比如：

首先Java中的数据类型可以分为两种，一种是基本数据类型，也称原始数据类型，如byte,short,char,int,long,float,double,boolean
他们之间的比较，应用双等号（==）,比较的是他们的值。

另一种是复合数据类型，包括类，当他们用（==）进行比较的时候，比较的是他们在内存中的存放地址，所以，除非 是同一个new出来的对象，他们的比较后的结果为true，否则比较后结果为false。

 而JAVA当中所有的类都是继承于Object这个基类的，在Object中的基类中定义了一个equals的方法，这个方法的初始行为是比较对象的内存地址，但在一些类库当中这个方法被覆盖掉了，如String,Integer,Date在这些类当中equals有其自身的实现，而不再是比较类在堆内存中的存放地址了。

<u>这么回答显的既专业又牛逼，还大气上档次，你学废了吗？</u>



> 聊聊对hashCode和equals方法的理解

一般问到这种问题，候选人都可以回答：

如果两个对象equals方法相等，则它们的hashCode一定相同；

如果两个对象的hashCode相同，它们的equals()方法则不一定相等。

而两个对象的hashCode()返回值相等不能判断这两个对象是相等的，但两个对象的hashcode()返回值不相等则可以判定两个对象一定不相等。



> <u>考察对hashCode的理解</u> 那请问hashCode有什么作用呢？为什么要这么规范

基本上初学者或者应届生在这一步就被我卡主了， 支支吾吾的回答不了多少，基本上都在这里掉分，因为上面的规范回答百度一查一大把，但是对hashCode作用有深入了解的却很少，这道题也可以看出一个人的专业水平。

正确的回答是：

hashCode的作用实际上是为了提高在散列结构存储中查找的效率，自然只有每个对象的hashCode尽可能的不同才能保证散列存储性能的提高，这也是为什么Object默认提供hash码都是不同的原因。

举个例子：

以HashSet为例，要想保证元素不重复则需要调用对象的equals方法比较一次，但是如果这个结构放了很多元素，比如5000次，如果没有hashCode的话则需要在每次加元素的时候对比5000次，而如果有hashCode则不一样了，当集合要添加新的元素的时候先调用hashCode方法，这样便可以定位到元素的地址，而无需多次判断。



><u>考察对clone方法的理解</u>看看这道题，结果输出什么？

![image-20210110183459155](https://gitee.com/xi_fan/img/raw/master/image-20210110183459155.png)

每次问到这道题，大部分人都是回答2，小部分人是回答1。

<u>都错，正确答案是直接报错</u>

![image-20210110183635113](https://gitee.com/xi_fan/img/raw/master/image-20210110183635113.png)

为什么？因为clone方法是Object的protect方法，需要子类显示的去重写clone方法，并且实现Cloneable 接口，这是规定。



那么问题来了？加上Cloneable 接口后呢？到底是2还是1呢？

<u>这又是我挖的坑，主要为了考察候选人对浅拷贝和深拷贝的理解</u>

最终结果其实是2，因为直接用clone拷贝的时候是一种浅拷贝，最终拷贝对象和原始对象的引用类型引用还是同一个对象。

如果想要实现深拷贝，则需要开发人员自己在clone方法内自己进行重写。

**拓展：**在企业开发中一般我们不允许直接用clone方法来做拷贝，存在抛出异常的风险，还需要进行类型转换。如果是我review到这种代码，都是直接告诉同事：在Effective Java 书上有讲到，最好不要去使用 clone()，我们可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象，比如这样

![image-20210111224630388](https://gitee.com/xi_fan/img/raw/master/image-20210111224630388.png)



*总结：我们都知道Java是面向对象编程的，我的理解实际上就是面对Object编程，能够合理的使用和了解原理equals和hashCode是一个初级程序员必须具备的素养，而对浅拷贝和深拷贝的理解也基本是基础中的基础，希望看完这系列，大家可以了然于胸，下次遇见这种面试题，直接吹。*



### 看看我在抽象类和接口埋了什么坑

> <u>考察候选人编码水平</u> 说说看你对抽象类的理解

抽象类和抽象方法都使用 abstract 关键字进行声明，如果一个类中包含抽象方法，那么这个类必须声明为抽象类，并且抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承。

<u>bingo,答案是对的，但是还不够好</u>

上面的是标准答案，随便一本Java书籍都可以看到，但是可以补充下你们对应用的理解，这可以体现你的编码水平到了哪一步，我这边补充我理想的答案：

抽象类可以实现代码的重用，模板方法设计模式是抽象类的一个典型应用，例如我们游戏的所有活动都要进行一样的初始化，但是初始化的时候需要根据不同活动进行不同功能开启判断，那么就可以定义一个活动的基类，将功能开启判断做成一个抽象方法，让所有活动都继承这个基类，然后在子类自行重写功能开启判断。

补充上自己对应用的理解，可以充分体现你的专业，面试官听完，肯定是内心一顿尼玛的牛逼。



> <u>继续考察候选人编码水平</u>说说看你对接口的理解

接口可以说成是抽象类的一种延伸，接口中的所有方法都必须是抽象的。实际上，接口中的方法定义默认为public abstract类型，接口中的成员变量类型默认为public static final。

<u>你们懂的，这又是一个标准回答，实际上，不够好，可以补充以下几句话</u>

从Java8开始，接口也可以拥有默认的方法实现了，为啥Java团队要做这个支持呢？

实际上，在Java上深耕多年的老司机其实都被接口没有默认实现搞过，维护成本太高了，你想想，如果你现在有100个类实现了一个接口，而在这个时候，需要给接口新增一个方法，特么Java8之前不允许给接口加默认实现，所以你就得改100个类，这特么不得搞出人命吗？

<u>真心话：能够说出对抽象类和接口应用的理解的候选人不多，基本上说出来的我内心都是往牛逼去吹的，大写的服，因为这充分体现一个开发人员对编码方面的思考，你不知道一个写代码有思考和整洁，而不是只顾实现功能的程序员，主管会多喜欢！！！</u>



> <u>真正的直接背的答案</u>说说看抽象类和接口的区别

- 抽象类可以有构造方法，接口中不能有构造方法。

- 抽象类中可以有普通成员变量，接口中没有普通成员变量

- 抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。

- 抽象类中可以包含静态方法，接口中不能包含静态方法

- 抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以任意，但接口中定义的变量只能是public static final类型，并且默认即为public static final类型。

- 一个类可以实现多个接口，但只能继承一个抽象类。

这个就是标准答案了，不过我们面试官比较希望可以在回答区别题的时候自己做些拓展回答，将上面接口和抽象类的理解都说一遍，而不是背书一样的回答，<u>切记，回答问题，遇见自己懂的知识点，尽量做拓展，加上自己的理解，一是可以向面试官展示你的专业，二是可以拖时间，尽量避免让面试官想更多难题坑你</u>



*总结：看到这里，估计很多人对抽象类和接口都嗤之以鼻，都是随口可以说出的答案，但是，你以为我们只是想要听你背书式的回答吗，错了，我们想知道的是后面你的拓展和理解，这才可以体现你的编码能力和专业。*



### 看看我在重写和重载里埋了什么坑

> <u>给大家来一个笔试中超级常见的题</u> 看看以下这道题，说说看结果是啥？

![image-20210111235033082](https://gitee.com/xi_fan/img/raw/master/image-20210111235033082.png)

这道题简直就是了，很多年前我来我司应聘的时候也遇见过这道题，说实话，应届生和初级绕不过这道题，现在直接给大家亮牌：

![image-20210111235450101](https://gitee.com/xi_fan/img/raw/master/image-20210111235450101.png)

<u>给大家说说如何理解这种题，理解以下的原则就可以了：</u>

JVM在调用一个方法时，会优先从本类中查找看是否有对应的方法，如果没有再到父类中查看是否从父类继承来。

如果没有怎么办？那JVM就会对参数进行转型，转成父类之后看是否有对应的方法。

总的来说，方法调用的优先级如下：

- this.func(this)
- super.func(this)
- this.func(super)
- super.func(super)

记住这个过程就可以了。



> <u>开始挖坑</u> 说说看以下的是不是方法重载？

```Java
class TestMain {

    public int show(int x) {
        return x;
    }

    public void show(int x) {
        System.out.println(x);
    }
}
```

我面试过的大部分初级或者应届都会回答是，不得不说这道题实在太坑了，不开玩笑，你看那堆被这道题埋过的人里边就有我。

<u>千万记住了，重载的定义是：在同一个类中，一个方法与已经存在的方法名称上相同，并且参数类型、个数、顺序至少有一个不同。这句话里边并没有包含着返回值，如果只是返回值不同，其它都相同根本不算是重载。</u>

当然，idea是会直接提示报错的，这也是为什么我没有截idea的源码出来给大家看的原因，一看有爆红你们就知道有问题了。不过还是那句话，这并不意味着idea可以检测出来的东西，你就可以不懂，这是基础。



*总结：说实话，在笔试题中出现重载和重写那两道题实在是太常见了，百分之九十的应届生或者初级开发都被坑过，所以看完这篇文章记得收藏啊，并不是你看了就记住了，而是记得在面试前重新看一遍，这可是关系到你offer上那个金光闪闪的数字的上限的啊*



### 看看我在反射方面埋了什么坑

<u>终于到了反射了，我们面试官其实很喜欢这个知识点，可以用它来筛掉一大批的api工程师</u>

> 了解过反射吗？说说看什么是反射？

在运行时，Java 反射，可以获取任意类的名称、package 信息、所有属性、方法、注解、类型、类加载器、现实接口等，并且可以调用任意方法和实例化任意一个类的独享，通过反射我们可以实现动态装配，降低代码的耦合度、实现动态代理等，不过反射的过度使用会严重消耗系统资源。



> <u>上面的回答很标准，不过开始深挖了</u> 你刚刚的回答有提到了反射的作用，说说哪里用到了反射机制？

例子很多，比如：

- JDBC中，利用反射动态加载了数据库驱动程序。
- Web服务器中利用反射调用了Sevlet的服务方法。
- 还有多框架都用到反射机制，注入属性，调用方法，如Spring等。

基本上回答几个就可以了，这里因为还没到框架模块，所以先不对Spring、JDBC等源码进行深挖，一般都是基础题面完了，才开始挖对框架的使用和源码的理解。



> <u>继续挖，我们面试官很喜欢抓细节，所以最好保证你说的东西你都有一定的了解，否则嘿嘿嘿</u>你刚刚还说到了动态代理，说说你对动态代理的理解，以及有什么用

动态代理其实就是是运行时动态生成代理类。
动态代理的应用有 Spring AOP、测试框架的后端 mock、rpc，Java注解对象获取等应用。



> 怎么实现动态代理呢？

目前动态代理可以提供两种，分为JDK 原生动态代理和 CGLIB动态代理；

JDK 原生动态代理是基于接口实现的，而 CGLIB是基于继承当前类的子类实现的，当目标类有接口的时候才会使用JDK动态代理，其实是因为JDK动态代理无法代理一个没有接口的类，因为JDK动态代理是利用反射机制生成一个实现代理接口的匿名类；

而CGLIB是针对类实现代理，主要是对指定的类生成一个子类，并且覆盖其中的方法。



> <u>挖一个大坑</u> 你上面有说到 Spring AOP有用到了动态代理，那你说说看AOP用到了哪种方式？

大部分都会回答JDK 原生动态代理，小部分人会回答CGLIB。

<u>但其实两者都是错的，这是我挖好的坑，太多候选人都只知道AOP用了动态代理了，但是能够完整回答出来用了哪种的却寥寥无几，毕竟只有看过源码的人才能回答这个问题，一般能够回答出来的，我这边都会额外加分</u>

实际答案是：在Spring中默认使用的是JDK动态代理，除非目标类没有实现接口，才会转为CGLIB代理，如果想要强行使用CGLIB代理免责需要在Spring配置文件中加入`<aop:aspectj-autoproxy proxy-target-class="true" />`，

然而在SpringBoot中，从2.0开始就默认使用CGLIB代理。**



> 说说看反射机制的优缺点

优点：可以动态执行，在运行期间根据业务功能动态执行方法、访问属性，最大限度发挥了java的灵活性。
缺点：对性能有影响，这类操作总是慢于直接执行java代码。



> 对性能有影响，那请问怎么解决这个问题？

到了这里基本上很多候选人都无法回答了，我见过太多一开始就说性能性能的，可是一说到如何解决，就基本支支吾吾的回答不出来。

<u>记住了，如果聊到性能，记得想好怎么回答优化方面的问题，否则就是自己搬起石头砸自己的脚。</u>

实际上可以从以下几个方面回答：

- 在系统启动时，将反射得到元数据保存起来，使用时，只需从内存中调用即可。

- 尽量用高点的JDK，因为高版本的虚拟机会对执行次数较多的方法进行优化，例如使用jit技术，而低版本的那个时候还没实现，
- 可以考虑使用高性能的反射库，Spring内部也有提供一些。



*总结：这里其实建议大家多去玩玩反射，不要只会api调用，对于我们面试官而言，熟悉和理解反射是一个中级程序员必备的条件。*



### 看看我在异常和注解方面埋的坑

> 做为一个Javer，应该对Error 和 Exception很熟悉吧，说说看它们的区别

目前来说，可以作为异常抛出的类，分为两种： Error 和 Exception。

而其中 Error 用来表示 JVM 无法处理的错误，然后Exception 也分为两种，分别是：

- 受检异常 ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
- 非受检异常 ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。



> 说说看什么是注解？以及有什么用

注解提供了一种类似注释的机制，用来将任何的信息或数据与类、方法、或者成员变量等进行关联。

Annontation像一种修饰符一样，应用于包、类型、构造方法、方法、成员变量、参数及本地变量的声明语句中。

注解的用处很多，举两个最常见的例子：

- 生成文档。这是最常见的，也是java 最早提供的注解，系统会在扫描到使用了注解的类后，根据注解信息生成文档。
- 在编译时进行格式检查。如@override 放在方法前，如果你这个方法并不是覆盖了超类方法，则编译时就能检查出。




> Java.lang.annotation 提供了四种元注解，是哪四种？有什么用呢？

JAVA 中有以下几个『元注解』：

- @Target：注解的作用目标
- @Retention：注解的生命周期
- @Documented：注解是否应当被包含在 JavaDoc 文档中
- @Inherited：是否允许子类继承该注解

没错，这几个元注解太常见了，但是却很少人能够真的用的好。



> 说说看@Target作用目标有哪些？

关于@Target的作用目标有多个目标类型，直接截图如下：

![image-20210112230657929](https://gitee.com/xi_fan/img/raw/master/image-20210112230657929.png)

基本上回答几个就可以了，不过记住了，上面的TYPE就是指的是类，也就是说这个注解是作用在类上的。



> @Retention注解的生命周期有哪几种？有什么区别？

@Retention注解的生命周期可以分为以下几种：

- etentionPolicy.SOURCE：当前注解编译期可见，不会写入 class 文件
- RetentionPolicy.CLASS：类加载阶段丢弃，会写入 class 文件
- RetentionPolicy.RUNTIME：永久保存，可以反射获取

区别在于：

第一种是只能在编译期可见，编译后会被丢弃；

第二种会被编译器编译进 class 文件中，无论是类或是方法，乃至字段，他们都是有属性表的，而 JAVA 虚拟机也定义了几种注解属性表用于存储注解信息，但是这种可见性不能带到方法区，类加载时会予以丢弃；

第三种则是永久存在的可见性，可以反射获取，基本上用这种居多。

<u>对于注解生命周器的理解可以避免一些坑，还是那句话，基本上注解这块能回答的比较流畅的，说明都是玩过组件或者看过源码的，因为对于开发一个组件来说，注解太常用了。</u>



> <u>送命题，百分之九十的候选人答不上来</u> 父类使用了注解，请问子类继承了这个父类后，是否也携带了这个注解呢？如果没有，要怎么实现这个过程？

这道题很少人会回答的上来，毕竟大家对@Inherited这个元注解太陌生了。

<u>在注解上使用元注解@Inherited，表示该注解会被子类所继承，注意注意，仅针对类哟，成员、方法并不受该元注解的影响</u>

因此，如果想要让子类也继承父类的注解，则需要给注解加上元注解@Inherited。

**拓展：一般初级开发都基本不会用上注解，因为注解更多的是用来写组件用的，我们项目组这边就很喜欢自定义一些注解来实现组件，因为用起来实在是太舒服了。**



*总结：关于异常这块，Error 和 Exception是经常会被问到的基础考点，而注解这块是一个能够加分的点，希望大家别错过这个知识点，学透它，不过记住上面几个坑点，其实也差不多了。*