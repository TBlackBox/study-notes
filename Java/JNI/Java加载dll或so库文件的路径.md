# Java加载dll或so库文件的路径 java.library.path

 

1. Java的System.load 和 System.loadLibrary都可以用来加载库文件

2. `例如你可以这样载入一个windows平台下JNI库文件：`

```
System.load("C://Documents and Settings//TestJNI.dll"); //绝对路径
```

3. System.loadLibrary参数为库文件名

```
例如你可以这样载入一个windows平台下JNI库文件
System.loadLibrary (``"TestJNI"``);
```

这里TestJNI必须在 java.library.path这一jvm变量所指向的路径中，可以通过如下方法获得该变量的值：

```
System.getProperty("java.library.path");
```

默认情况下，Windows平台下包含下面的路径：

 1）和jre相关的目录

 2）程序当前目录

 3）Windows目录

 4）系统目录(system32)

 5）系统环境变量path指定的目录

 

4.在linux下添加一个java.library.path的方法如下：

```
 在/etc/profile 后面加上一行 export LB_LIBRARY_PATH=路径
```

5.在执行程序的时候可以显示指定， -Djava.library.path=路径，这种会清除掉预设置的java.library.path的值 。实例如下：

```
java -jar -Djava.library.path=/home/fly/Desktop/sound_dream sound.war
```

 