# 简介
　java.io.File 类是文件和目录路径名的抽象表示，主要用于文件和目录的创建、查找和删除等操作。

## File 类的几点认识
- File类是一个与系统无关的类,任何的操作系统都可以使用这个类中的方法。
- File 是文件和目录路径名的抽象表示形式，即File类的对象代表一个文件或一个目录（文件夹）的路径，而不是文件本身。
- File 类不能直接访问文件内容本身，如果需要访问文件内容本身，则需要使用输入/输出流。

## 绝对路径和相对路径
- 绝对路径：从盘符开始的路径，这是一个完整的路径。如：E:\\a.txt

- 相对路径：相当于项目目录的路径，这个一个便捷的路径。如：../a.txt


## 注意点
1. 路径不区分大小写
2. 路径中的文件名称分隔符 window 使用反斜杠（\）也支持斜杠（/），反斜杠是转义字符,两个反斜杠代表一个普通的反斜杠
3. 其他的平台都使用斜杠作为分隔符（/），网络地址也是(http://www.baidu.com)。
4. window 的路径分隔符使用“\”，而Java程序中的“\”表示转义字符，所以在Windows中表示路径，需要用“\\”。或者直接使用“/”也可以，Java程序支持将“/”当成平台无关的路径分隔符。或者直接使用File.separator常量值表示。
5. 路径中如果出现 “..” 表示上一级目录，路径名如果以“/”开头，表示从“根目录”下开始导航。

# 构造方法

#### 构造方法1
public File(String pathname) ：通过将给定的路径名字符串转换为抽象路径名来创建新的 File实例。

##### 说明
1. String pathname： 代表字符串的路径名称
2. 路径可以是文件结尾，也可以是文件夹结尾
3. 路径可以是相对路径，也可以绝对路径
4. 路径可以是存在,也可以是不存在
5.  创建File对象,只是把字符串路径封装为File对象,不考虑路径的真假情况

##### 例子
```
//绝对路径构建
File file = new File("D:\\test\\test.txt");
//在在相对路径下构建  工程的话 在工程更目录 需要调用创建才会有
File file1 = new File("test.txt");
```

#### 构建方法2
public File(String parent, String child) ：从父路径名字符串和子路径名字符串创建新的 File实例。
##### 说明
参数：把路径分为了两部分
- String parent：父路径
- String child：子路径
 优点：父路径和子路径,可以单独书写,使用起来非常灵活;父路径和子路径都可以变化
##### 例子
```
File file2 = new File("D:\\test", "uset.txt");
```

#### 构造方法3
public File(File parent, String child) ：从父抽象路径名和子路径名字符串创建新的 File实例。
##### 说明
把路径分成两部分
- File parent：父路径
- String child：子路径
优点：
1. 父路径和子路径,可以单独书写,使用起来非常灵活;父路径和子路径都可以变化
2. 父路径是File类型,可以使用File的方法对路径进行一些操作,再使用路径创建对象

##### 例子
```
File file2 = new File("D:\\test");
File file3 = new File("D:\\test", "user.txt");
```
注意：
1. 一个 File 对象代表硬盘实际存在一个文件或者目录。
2. 无论该路径下是否存在文件或者目录，都不影响 File 对象的创建。
## 构造方法4
public File(URI uri):通过将给定的 file: URI 转换为一个抽象路径名来创建一个新的 File 实例。





## File类中的字段信息
File 类中提供了四个类常量：
1. `separatorChar`
public static final char separatorChar：与系统有关的默认名称分隔符。此字段被初始化为包含系统属性 file.separator 值的第一个字符。 在 UNIX 系统上，此字段的值为 '/'；在 Microsoft Windows 系统上，它为 '\\'。
2. `separator`
public static final String separator：与系统有关的默认名称分隔符，为了方便，它被表示为一个字符串。此字符串只包含一个字符，即 separatorChar。
3. `pathSeparatorChar`
public static final char pathSeparatorChar：与系统有关的路径分隔符。此字段被初始为包含系统属性 path.separator 值的第一个字符。　　　　　　　　　　　　　　　　　　　　　　　　　　 此字符用于分隔以路径列表 形式给定的文件序列中的文件名。在 UNIX 系统上，此字段为 ':'；在 Microsoft Windows 系统上，它为 ';'。
4. `pathSeparator`
public static final String pathSeparator：与系统有关的路径分隔符，为了方便，它被表示为一个字符串。此字符串只包含一个字符，即 pathSeparatorChar
```
String pathSeparator = File.pathSeparator;
//System.out.println(pathSeparator);//路径分隔符 windows:分号;  linux:冒号:

String separator = File.separator;
//System.out.println(separator);// 文件名称分隔符 windows:反斜杠\  linux:正斜杠/

System.out.println(File.separator);          //  \
System.out.println(File.separatorChar);      //  \
System.out.println(File.pathSeparator);      //  ;
System.out.println(File.pathSeparatorChar);  //  ;
```



# 常用方法

## （1）获取文件或目录的详细信息

| 函数签名                         | 含义                                                         |
| :------------------------------- | :------------------------------ |
|String getName()|   返回由此抽象路径名表示的文件或目录的名称。``long` `length() ：返回由此抽象路径名表示的文件的长度。（只能返回文件大小，不能直接返回目录大小）|
|`boolean` exists() |  测试此抽象路径名表示的文件或目录是否存在。|
|`boolean` isHidden() | 测试此抽象路径名指定的文件是否是一个隐藏文件。|
|`boolean` canRead() |  测试应用程序是否可以读取此抽象路径名表示的文件。|
|`boolean` canWrite()| 测试应用程序是否可以修改此抽象路径名表示的文件 |
|`long` `lastModified()` | 返回此抽象路径名表示的文件最后一次被修改的时间。（毫秒值）|
|String toString() |  返回此抽象路径名的路径名字符串。|
|`boolean` isAbsolute()| 测试File对象对应的文件或目录是否是绝对路径。|
|String getParent() | 返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回 null。（返回父目录名称） File getParentFile() ：返回此抽象路径名父目录的抽象路径名；如果此路径名没有指定父目录，则返回 ``null``。（返回父目录的File对象）|
|public long length()  |返回由此File表示的文件的长度。获取的是构造方法指定的文件的大小，以字节为单位。1文件夹是没有大小概念的,不能获取文件夹的大小 2 如果构造方法中给出的路径不存在,那么length方法返回0|

**注意：**以上都是对应于 File 对象的属性信息，如果File对象是一个不存在的文件或目录，返回的都是对应属性的默认值，例：获取的length()为0，isFile()为False等， name,path这些都是有值的，因为这些属性是通过构造方法创建 File 时指定的，而其他的属性都是默认值。

## 　　（2）获取文件或目录的路径
| 函数签名                         | 含义                                                         |
| :------------------------------- | :------------------------------ |
|String getPath() | 将此抽象路径名转换为一个路径名字符串。|
|File getAbsoluteFile() | 返回此抽象路径名的绝对路径名形式。|
| String getAbsolutePath() |返回此抽象路径名的绝对路径名字符串。 返回绝对路径，从根目录开始导航`|
|File getCanonicalFile()| 返回此抽象路径名的规范形式。|
|String getCanonicalPath() |返回此抽象路径名的规范路径名字符串。规范路径就是路径里没有./或../或/，会把这些自动解析|


## （3）操作文件或目录
| 函数签名                         | 含义                                                         |
| :------------------------------- | :------------------------------ |
|`boolean` `createNewFile()`|当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件。（只创建文件）,创建文件的路径必须存在，否则会抛出异常|
|`boolean` `delete()` | 删除此抽象路径名表示的文件或目录。（可以删除文件或删除空目录）删除成功，返回``true``,失败``false``。|
|boolean` `equals(Object obj)|测试此抽象路径名与给定对象是否相等。|
|``boolean` `mkdir() | 创建此抽象路径名指定的目录。（只能创建一级目录）必须确保父目录存在，否则创建失败。|
|`boolean` `mkdirs()| 创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。 （可以创建多级目录）如果父目录不存在，会一同创建父目录|
|`boolean` renameTo(File dest) |重新命名此抽象路径名表示的文件。（可以给文件或目录重命名）|

## （4）判断是文件还是目录
| 函数签名                         | 含义                                                         |
| :------------------------------- | :------------------------------ |
|boolean` `isFile() |测试此抽象路径名表示的文件是否是一个标准文件。|
|`boolean` isDirectory() | 测试此抽象路径名表示的文件是否是一个目录。|


## （5）获取目录的下一级
| 函数签名                         | 含义                                                         |
| :------------------------------- | :------------------------------ |
|String[] list() |返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中的文件和目录。|
|File[] listFiles() |返回一个抽象路径名数组，这些路径名表示此抽象路径名表示的目录中的文件。（获取下一级的文件/目录的File对象）|
|static File[] listRoots()|列出可用的文件系统根。|

注意：　

1. list 方法和 listFiles 方法遍历的是构造方法中给出的目录

　2.  如果构造方法中给出的目录的路径不存在，会抛出空指针异常

　3.  如果构造方法中给出的路径不是一个目录，也会抛出异常。

　4. 调用listFiles方法的File对象，表示的必须是实际存在的目录，否则返回null，无法进行遍历