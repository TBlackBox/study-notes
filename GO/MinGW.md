# 简介

1、从目标上说

- MinGW 是让Windows 用户可以用上GNU 工具，比如GCC。
- Cygwin 提供完整的类Unix 环境，Windows 用户不仅可以使用GNU 工具，理论上Linux 上的程序只要**用****Cygwin 重新编译**，就可以在Windows 上运行。

2、从能力上说

- 如果程序只用到C/C++ 标准库，可以用MinGW 或Cygwin 编译。
- 如果程序还用到了**POSIX API**，则只能用Cygwin 编译。

3、从依赖上说

- 程序经MinGW 编译后可以直接在Windows 上面运行。
- 程序经Cygwin 编译后运行，需要依赖安装时附带的cygwin1.dll。



MinGW 就是GCC 的Windows版本

MinGW-w64 与MinGW的区别在于MinGW只能编译生成32位可执行程序，MinGW-w64 即可编译生成32 位也可以编译生成64位

MinGW 已被MinGW-w64取代，MinGW已停止更新，内置的GCC 停留在4.8.1版本



[下载地址](https://sourceforge.net/projects/mingw/files/ )

[官网](http://mingw-w64.org/)





# 安装

- 下载地址

  ```
  https://sourceforge.net/projects/mingw/files/
  ```

- 运行在线安装（国内需要翻墙，最好全局代理）

  version 8.1.0

  architecture x68_64

  threads posix

  exection seh

  build revision 0

- 添加环境变量

  将安装目录下的bin目录添加到环境变量中（即path中）

  ```
  d:\mingw64\bin
  ```

  