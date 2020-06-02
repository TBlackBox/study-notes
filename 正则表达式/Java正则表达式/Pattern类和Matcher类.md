# 简介
在`java.util.regex`包是一个用正则表达式所订制的模式来对字符串进行匹配工作的类库包。该包下的主要类：
```
MatchResult（接口）
	Matcher（实现类）
Pattern
```

# 捕获组的概念

捕获组可以通过从左到右计算其开括号来编号，编号是从1 开始的。
例如，在表达式 ((A)(B(C)))中，存在四个这样的组：
```
1 ((A)(B(C)))
2 (A)
3 (B(C))
4 (C)
```
组零始终代表整个表达式。 以 (?) 开头的组是纯的非捕获 组，它不捕获文本，也不针对组合计进行计数。与组关联的捕获输入始终是与组最近匹配的子序列。如果由于量化的缘故再次计算了组，则在第二次计算失败时将保留其以前捕获的值（如果有的话）
```
例如，将字符串"aba" 与表达式(a(b)?)+ 相匹配，会将第二组设置为 "b"。在每个匹配的开头，所有捕获的输入都会被丢弃。
```

# Pattern
Pattern 包含的主要方法
| 返回值|方法|
| ---------------- | --------------------------------------- |
| Predicate      | asPredicate()创建可用于匹配字符串的pattern。        |
| `static Pattern` | `compile(String regex)`将给定的正则表达式编译为Pattern。 |
| `static Pattern` | `compile(String regex, int flags)`将给定的正则表达式编译为带有给定标志的Pattern。 |
| `int`            | `flags()`返回此模式的匹配标志.       |
| `Matcher`        | `matcher(CharSequence input)`创建一个匹配器，该匹配器将使给定输入与此模式匹配。 |
| `static boolean` | `matches(String regex, CharSequence input)编译给定的正则表达式，并尝试将给定的输入与其匹配。 |
| `String`         | `pattern()`返回从中编译此模式的正则表达式。 |
| `static String`  | `quote(String s)`返回指定字符串的文字模式字符串。 |
| `String[]`       | `split(CharSequence input)`在此模式的匹配周围拆分给定的输入序列。 |
| `String[]`       | `split(CharSequence input, int limit)`在此模式的匹配周围拆分给定的输入序列。 |
| `Stream`         | `splitAsStream(CharSequence input)`根据此模式的匹配从给定的输入序列创建一个流。 |
| `String`         | `toString()`返回此模式的字符串表示形式。 |

一个Pattern是一个正则表达式经编译后的表现模式。Pattern类用于创建一个正则表达式,也可以说创建一个匹配模式,它的构造方法是私有的,不可以直接创建,但可以通过Pattern.complie(String regex)简单工厂方法创建一个正则表达式。

# Pattern常用的方法与Matcher

## 创建pattern

```
Pattern pattern = Pattern.compile("\\w+");
//方式之一  忽略大小写 Pattern.CASE_INSENSITIVE 当然还有很对其他模式  可以看api
Pattern pattern1 = Pattern.compile("\\w+", Pattern.CASE_INSENSITIVE);
```

## split方法
```
String a = "abcd4dds5fa";

Pattern pattern2 = Pattern.compile("\\d");
//limit 结果阀值
strArr = pattern2.split(a); //["abcd","dds",fa]

//String 的split都可以理解是他的分隔
String[] strArr =  a.split("\\d"); //["abcd","dds",fa]
```

## matcher方法
一个静态方法,用于快速匹配字符串,该方法适合用于只匹配一次,且匹配全部字符串.
```
//返回true 
Pattern.matches("\\d+","1241");
//返回false,需要匹配到所有字符串才能返回true,这里ss不能匹配到 
Pattern.matches("\\d+","1212ss");
```


## Pattern.matcher(CharSequence input)

返回一个Matcher对象,例如
```
Pattern p =Pattern.compile("\\d+"); 
Matcher m = p.matcher("251ddd21f"); 
//返回p 也就是返回该Matcher对象是由哪个Pattern对象的创建的 
m.pattern();
```
# Matcher

一个Matcher对象是一个状态机器，它依据Pattern对象做为匹配模式对字符串展开匹配检查。Matcher类的构造方法也是私有的,不能随意创建,只能通过Pattern.matcher(CharSequence input)方法得到该类的实例.Pattern类只能做一些简单的匹配操作,要想得到更强更便捷的正则匹配操作,那就需要将Pattern与Matcher一起合作.Matcher类提供了对正则表达式的分组支持,以及对正则表达式的多次匹配支持，创建一个Matcher 对象。
```
Pattern p =Pattern.compile("\\d+"); 
Matcher m = p.matcher("251ddd21f"); 
//返回p 也就是返回该Matcher对象是由哪个Pattern对象的创建的 
m.pattern();
```

## Matcher 提供了3个匹配操作方法
三个方法均返回boolean类型,当匹配到时返回true,没匹配到则返回false 。
### matches()
对整个字符串进行匹配,只有整个字符串都匹配了才返回true 。
```
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("22bb23"); 
m.matches();//返回false,因为bb不能被\d+匹配,导致整个字符串匹配未成功. 
Matcher m2=p.matcher("2223"); 
m2.matches();//返回true,因为\d+匹配到了整个字符串
```
** 注意：** 这种匹配模式与上面的`Pattern.matcher(String regex,CharSequence input)`等价的。
### lookingAt()
对前面的字符串进行匹配,只有匹配到的字符串在最前面才返回true 
```
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("22bb23"); 
m.lookingAt();//返回true,因为\d+匹配到了前面的22 
Matcher m2=p.matcher("aa2223"); 
m2.lookingAt();//返回false,因为\d+不能匹配前面的aa 
```

### find()
对字符串进行匹配,匹配到的字符串可以在任何位置. 
```
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("22bb23"); 
m.find();//返回true 
Matcher m2=p.matcher("aa2223"); 
m2.find();//返回true 
Matcher m3=p.matcher("aa2223bb"); 
m3.find();//返回true 
Matcher m4=p.matcher("aabb"); 
m4.find();//返回false 
```

## Mathcer.start()/ Matcher.end()/ Matcher.group()

当使用matches(),lookingAt(),find()执行匹配操作后(注意这3个方法为true的时候，才能使用后面的3个方法，否则会抛出异常),就可以利用以上三个方法得到更详细的信息. 
- start() 返回匹配到的子字符串在字符串中的索引位置. 
- end() 返回匹配到的子字符串的最后一个字符在字符串中的索引位置. 
- group() 返回匹配到的子字符串 
```
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("aaa2223bb"); 
m.find();//匹配2223 
m.start();//返回3 
m.end();//返回7,返回的是2223后的索引号 
m.group();//返回2223 

Mathcer m2=m.matcher("2223bb"); 
m.lookingAt();   //匹配2223 
m.start();   //返回0,由于lookingAt()只能匹配前面的字符串,所以当使用lookingAt()匹配时,start()方法总是返回0 
m.end();   //返回4 
m.group();   //返回2223 

Matcher m3=m.matcher("2223bb"); 
m.matches();   //匹配整个字符串 
m.start();   //返回0,原因相信大家也清楚了 
m.end();   //返回6,原因相信大家也清楚了,因为matches()需要匹配所有字符串 
m.group();   //返回2223bb 
```

## 分组
start(),end(),group()均有一个重载方法它们是start(int i),end(int i),group(int i)专用于分组操作,Mathcer类还有一个groupCount()用于返回有多少组. 
```
Pattern p=Pattern.compile("([a-z]+)(\\d+)"); 
Matcher m=p.matcher("aaa2223bb"); 
m.find();   //匹配aaa2223 
m.groupCount();   //返回2,因为有2组 
m.start(1);   //返回0 返回第一组匹配到的子字符串在字符串中的索引号 
m.start(2);   //返回3 
m.end(1);   //返回3 返回第一组匹配到的子字符串的最后一个字符在字符串中的索引位置. 
m.end(2);   //返回7 
m.group(1);   //返回aaa,返回第一组匹配到的子字符串 
m.group(2);   //返回2223,返回第二组匹配到的子字符串 
```
现在我们使用一下稍微高级点的正则匹配操作,例如有一段文本,里面有很多数字,而且这些数字是分开的,我们现在要将文本中所有数字都取出来,利用java的正则操作是那么的简单. 
```
Pattern p=Pattern.compile("\\d+"); 
Matcher m=p.matcher("我的QQ是:456456 我的电话是:0532214 我的邮箱是:aaa123@aaa.com"); 
while(m.find()) { 
     System.out.println(m.group()); 
} 
```

** 注意：**
1. 每次执行匹配操作后`start()`,`end()`,`group()`三个方法的值都会改变,改变成匹配到的子字符串的信息,以及它们的重载方法,也会改变成相应的信息. 
2. 只有当匹配操作成功,才可以使用start(),end(),group()三个方法,否则会抛出java.lang.IllegalStateException,也就是当matches(),lookingAt(),find()其中任意一个方法返回true时,才可以使用.